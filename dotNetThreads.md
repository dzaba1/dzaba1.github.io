## Lock and Monitor

The `lock` uses the Monitor class in a try/finally block.

It has 2 major methods: Enter and Exit. It has also Wait and Pulse.

It's a static class which uses an object for locking.

It's like a semaphore with only one slot.

## Mutex

Mutex is like a Monitor but it's not static and it is OS managed.

It means it can be used for locking things between OS processes.

Mutex derives from WaitHandle.

2 major methods: WaitOne and ReleaseMutex

Mutex can be named.

## Semaphore(Slim)

It controls number of threads which can access some resource or just something.

Main methods: Wait and Release.

It derives from WaitHandle so it is OS managed and can be used in multiple processes.

There's Slim version which is not WaitHandle so it works only in the one process so it's faster.

## Interlocked

Static class which provides methods for simple, atomic operations. 

Methods like: Add, Compare, Decrement, Increment, Exchange

## SpinLock

Value type Monitor wannabe which uses a loop for checking of something is accessible.

Methods: Enter and Exit

## ReadWriterLock(Slim)

Class similar to Monitor but can have N readers and M writers.

Methods: Enter and Exit

There's a Slim version which faster and safer so it should be used now instead.

## ManualResetEvent(Slim) vs. AutoResetEvent

AutoResetEvent while invoking WaitOne() invokes also Reset()

Without Reset() doors are opened and WaitOne() always pass.

They derived from WaitHandle.

There's also ManualResetEventSlim which doesn't derive from WaitHandle.

## Volatile

`volatile` turns off optimization for a field. Sometimes someone says that it makes a field thread safe but this is not true.

`volatile` is only in C#.

## IAsyncResult - Asynchronous Programming Model (APM)

Firing delegates in async way

BeginInvoke uses the ThreadPool

Examples:
```
public override void RunOneValue(int i)
{
    Action<int, string, int?> del = WriteValue;
    del.BeginInvoke(i, nameof(RunOneValue), 10, del.EndInvoke, null);
}
```

```
public override void RunTwoValues(int i1, int i2)
{
    Action<int, string, int?> del = WriteValue;
    var r1 = del.BeginInvoke(i1, nameof(RunTwoValues), 10, del.EndInvoke, null);
    var r2 = del.BeginInvoke(i2, nameof(RunTwoValues), 10, del.EndInvoke, null);
    r1.AsyncWaitHandle.WaitOne();
    r2.AsyncWaitHandle.WaitOne();
}
```

## Event based asynchronous pattern (EAP)

Uses events to manage multithreading.

Example: BackgroundWorker

## Task based asynchronous pattern (TAP)

Separates business logic from thread handling. A little Command pattern.

There's a major Task type which allows you to control the multithreading but you pass a delegate with business logic.

Tasks works with ThreadPool by default. But you can make your own TaskScheduler.

Usually we add Async to method name.

## How async/await works

Function coloring based on Task usually. But you can await anything by duck typing.
Example:
```
class Program
{
    private class MyTask : INotifyCompletion
    {
        public MyTask GetAwaiter()
        {
            return this;
        }

        public bool IsCompleted => true;

        public void OnCompleted(Action continuation)
        {
            
        }

        public void GetResult()
        {

        }
    }

    public async Task Test()
    {
        var myTask = new MyTask();

        await myTask;
    }
}
```

TODO

## What are task create options?

- AttachedToParent - Makes the child task to be in the same hierarchy as parent. Usually they are not in the same hierarchy and they are invoked independently. So then it's like synchronously. A child waits for the parent and gets threads from him
- DenyChildAttach - Invalidates AttachedToParent options for all child tasks even if set.
- HideScheduler - Usually children uses the TaskScheduler passed to parent as a default one. But having this it will be hidden so then the Default will be taken as default.
- LongRunning - May happen that additional Thread will be used instead of default ThreadPool.
- PreferFairness - Makes some queue of tasks based on schedule time.

## Exception handling in TPL

If Wait() or Result then AggregateException.

Having async/await we can try/catch exceptions normally.

## Differences between background thread and foreground (main) thread.

If all main threads end then the process ends. If there's an unhandled exception then process immediately ends.

If the process ends then all background threads are killed.

Having the Thread class you can check it by IsBackground property.

## PLINQ

PLINQ starts from .AsParallel() extension. It produces a ParallelQuery.

Uses the Partitioner as default but you can pass your own partitioner.

Main methods:
- AsSequental() - Invokes everything now sequentially.
- AsOrdered() - Specifies that the order should be preserved.
- AsUnordered() - Specifies that order is not important.
- WithCancelation() - You can pass your CancellationToken and control enumeration.
- WithDegreeOfParallelism() - Like max number of threads.
- WithExecutionMode() - You can mark the query as be parallel always even if behavior of elements doesn't indicate it.
- WithMergeOption() - You can setup if some buffer should be used while merging results.
- ForAll() - The same as Parallel.ForEach

Example:
```
class Program
{
    static void Main(string[] args)
    {
        var enumerable = Enumerable.Range(0, 24);
        var plinq = from e in enumerable.AsParallel() where Process(e) select e;
        Console.WriteLine(plinq.Count());
        Console.ReadKey();
    }

    private static bool Process(int e)
    {
        Console.WriteLine("Processing {0}. Thread ID {1}", e, Thread.CurrentThread.ManagedThreadId);
        Thread.Sleep(500);
        return e >= 12;
    }
}
```

## ThreadPool

As the name says it's a pool of threads which are initiated and waits for some work. Every thread is a background one.

Making threads is expensive.

Number of threads is dynamic and depending on equipment and load.

It's better to use the Thread object when:
- The thread has to be a main one.
- Some other apartment state.
- To dynamically change priorities of threads.
- The operation will be really really long.

## What’s the apartment state?

It controls access to COM objects. Some of them are not thread safe. COM objects sit in that apartment. Also threads sit in that apartment. A process can have multiple apartments.

Apartment can be:
- STA - Single-threaded apartment
The apartment has only one thread which proxies to COM objects. Message pump is synchronous.

- MTA - Multi-threaded apartment
Multiple threads proxies to COM objects. Every object then should be thread safe.

# Concurrent collections

From System.Collections.Concurrent

## Partitioner

Chunks a collection.
Example:
```
class Program
{
    private static List<T> ToList<T>(IEnumerator<T> enumerator)
    {
        List<T> list = new List<T>();
        while (enumerator.MoveNext())
        {
            list.Add(enumerator.Current);
        }

        return list;
    }

    static void Main(string[] args)
    {
        int[] array = new int[26];
        OrderablePartitioner<int> partitioner = Partitioner.Create(array, false);
        var partitions = partitioner.GetPartitions(Environment.ProcessorCount);
        Console.WriteLine("There is {0} partitions.", partitions.Count);
        foreach (IEnumerator<int> item in partitions)
        {
            List<int> temp = ToList(item);
            Console.WriteLine(temp.Count);
        }

        Console.ReadKey();
    }
}
```

## ConcurrentBag

A collection where you can add things safely.

Uses a linked list.

## ConcurrentDictionary

Thread safe hashtable.

Has initially 1kB!

## ConcurrentQueue

Thread safe queue.

Implements `IProducerConsumerCollection<T>`.

Uses a linked list behind.

## ConcurrentStack

Thread safe stack.

Uses a linked list behind.

## BlockingCollection

It wraps a `IProducerConsumerCollection<T>` as a buffer and implements the producer/consumer pattern.

If buffer is full then producers wait. If buffer is empty then consumers wait.