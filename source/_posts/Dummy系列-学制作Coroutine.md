---
title: 'Dummy系列:学制作Coroutine'
date: 2018-12-10 00:34:22
tags:
    - C++
    - Coroutine
---
这是第一个Dummy系列, Dummy系列意味着弱智学习系列, 因为我本身不聪明, 所以想做些简单易懂的教程, 让我这种程度的人能理解.

听到协程这个词, 第一个想到的是线程, 但是线程是由**系统控制**的, 而协程是在**用户态**控制的, 意味着用户会控制和保存栈, 寄存器以及其他相关状态(大雾). 为了搞清楚C++是如何实现的协程的, 我做了一些资料收集, 制作了这个玩具级协程Demo.

<!--more-->

## 协程的实现方式

* 调用系统库<ucontext\>保存上下文, 性能会较差
* 使用汇编自己保存上下文, 兼容性可能会有点问题
* 以及其他方法

因为是Dummy系列, 就直接调用系统API实现协程

## 核心实现

核心类有两个, 分别为CoroutineScheduler和SchedulerImpl组成, 其中SchedulerImpl是内部类, 封装所有核心操作和数据, CoroutineScheduler只是间接调用SchedulerImpl的接口, 防止用户篡改数据.

现在看下CoroutineScheduler的接口

```C++
class CoroutineScheduler
{
public:
    enum Status
    {
        READY,
        SUSPEND,
        RUNNING,
        FINISHED
    };
    
    typedef uintptr_t(*func)(void* arg);

public:
    CoroutineScheduler(int size = 1024 * 1000);

    ~CoroutineScheduler();

    // 创建一个协程
    int CreateCoroutine(func f, void* args);

    // 销毁一个协程
    int DestoryCoroutine(int id);

    // 暂停执行
    uintptr_t Yield(uintptr_t p = 0);

    // 继续执行
    uintptr_t Resume(int id, uintptr_t p = 0);

    // 协程是否还有效
    bool IsAlive(int id) const;

private:
    // Can't impl
    CoroutineScheduler(const CoroutineScheduler&);

    CoroutineScheduler& operator=(const CoroutineScheduler&);

private:
    class SchedulerImpl;
    SchedulerImpl* m_impl;
};
```

### 协程状态

协程的状态和线程一样, 共有四种状态
1. 准备运行Ready
2. 暂停Suspend
3. 正在运行Running
4. 结束

### 协程结构

```
struct coroutine
{
    void* arg; // 函数参数
    int status; // 协程状态
    ucontext_t cxt; // 上下文
    uintptr_t yield; // yield返回的参数
    CoroutineScheduler::func f; // 函数
    char stack[0]; // 协程栈
};
```

ctx是ucontext要求的结构,其中保存着栈的位置、返回函数等, stack为自己分配的栈空间，该空间应该足够大, 否则会core

### 创建协程

分配coroutine空间, 设置状态, 加入coroutine的map中就完成了
```
int CoroutineScheduler::SchedulerImpl::CreateCoroutine(CoroutineScheduler::func f, void* arg)
{
    coroutine* cor = (coroutine*)malloc(sizeof(coroutine) + m_stacksize);

    if (!cor)
    {
        return -1;
    }

    cor->arg = arg;
    cor->f = f;
    cor->yield = 0;
    cor->status = READY;

    int index = m_index++;
    m_id2coroutine[index] = cor;

    return index;
}
```

### 销毁协程

销毁协程也是一个很简单的操作, 只需要free掉空间, 以及消除map中记录即可
```
int CoroutineScheduler::SchedulerImpl::DestoryCoroutine(int id)
{
    auto iter = m_id2coroutine.find(id);
    if (iter == m_id2coroutine.end())
    {
        return -1;
    }
    coroutine* cor = iter->second;
    free(cor);
    m_id2coroutine.erase(id);
    return id;
}
```

### 暂停协程

暂停协程需要做的事情会比以前的两个函数多, 而且会调用到系统API**swapcontext**切换上下文,
以下函数有注释
```
uintptr_t CoroutineScheduler::SchedulerImpl::Yield(uintptr_t p)
{
    if (m_running < 0)
    {
        return 0;
    }

    int cur = m_running;  // 获取当前运行协程
    m_running = -1;       // 暂停协程标志
    
    coroutine* cor = m_id2coroutine[cur]; // 获取协程结构
    cor->yield = p; // 保存返回值给主线程使用
    cor->status = SUSPEND; // 标志为暂停

    swapcontext(&cor->cxt, &m_mainContext); // 保存当前协程, 并且切换到主线程

    uintptr_t ret = cor->yield; // 走到这一步的时候, 已经从主线程切换到协程了
    cor->yield = 0;
    return ret; // 继续执行
}
```

### 恢复协程

恢复协程是最难的地方, 分为两种状态, 一种为已经执行, 另外一种为暂停的协程
```C++
uintptr_t CoroutineScheduler::SchedulerImpl::Resume(int id, uintptr_t p)
{
    coroutine* cor = m_id2coroutine[id];

    if (cor == NULL || cor->status == RUNNING)
    {
        return 0;
    }

    cor->yield = p; // 保存调用值给协程恢复使用
    switch (cor->status)
    {
    // 未开始, 可以运行的协程
    case READY:
        {
            getcontext(&cor->cxt); // 初始化协程结构, 系统API, 有兴趣可以看Man Page
            
            cor->status = RUNNING; // 标志为执行
            cor->cxt.uc_stack.ss_sp = cor->stack; // 指向栈空间
            cor->cxt.uc_stack.ss_size = m_stacksize; // 保存栈大小
            cor->cxt.uc_link = &m_mainContext; // 执行完成之后走向的流

            m_running = id; // 标志现在执行的ID
            makecontext(&cor->cxt, (void(*)())Schedule, 1, this); // 初始化要开始执行的函数
            swapcontext(&m_mainContext, &cor->cxt); // 开始执行协程
        }
        break;
    case SUSPEND:
        {
            m_running = id; // 标志现在执行的ID
            cor->status = RUNNING;  // 标志为执行
            swapcontext(&m_mainContext, &cor->cxt); // 切换到协程
        }
        break;
    default:
        assert(0);
    }

    uintptr_t ret = cor->yield; // 保存协程返回值, 可以给主线程使用
    cor->yield = 0; // 清空

    if (m_running == -1 && cor->status == FINISHED)
    {
        DestoryCoroutine(id); // 如果现在协程已经完成了, 则进行销毁
    }

    return ret;
}
```

## 总结
这是一个玩具级的Demo, 所以一些代码并没有很严谨, 但并不影响我们理解协程, 简单来说就是保存上下文以及恢复上下文的一个过程, 也就是用户态的"线程".所有代码已经上传到[GitHub](https://github.com/bbdLe/DummyCoroutine), 希望能帮到大家理解, 因为现在已经快要1点半了, 有些细节没有完全解释清楚, 等有空再来填坑. 有时间的话, 再写一个汇编调用的协程玩具