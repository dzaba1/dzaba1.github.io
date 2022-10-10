## What happens if exception is thrown inside of a using block?

Nothing bad. It is compiled as try/finally. So Dispose is all the time fired.

Example:
```
class Program
{
    class Disposable : IDisposable
    {
        public void Dispose()
        {
            Console.WriteLine("Disposable Dispose");
        }
    }

    static void Main(string[] args)
    {
        try
        {
            using (Disposable disp = new Disposable())
            {
                throw new Exception("Test");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }

        Console.ReadKey();
    }
}
```

## Does .net have destructors? Purpose of Finalizer? Finalize vs Dispose vs Destructor? Dispose pattern.

Example:
```
class Program
{
    class Disposable : IDisposable
    {
        public Disposable(string name)
        {
            Name = name;
        }

        public string Name { get; }

        ~Disposable()
        {
            Console.WriteLine(Name + " ~Disposable");
            Dispose(false);
        }

        public void Dispose()
        {
            Console.WriteLine(Name + " Dispose");
            Dispose(true);
            GC.SuppressFinalize(this);
        }

        private void Dispose(bool dispose)
        {
            if (!dispose)
            {
                Console.WriteLine("Dispose called from finalizer. Thread id: " +
                                  Thread.CurrentThread.ManagedThreadId);
            }
            else
            {
                Console.WriteLine("Dispose called from dispose. Thread id: " +
                                  Thread.CurrentThread.ManagedThreadId);
            }
        }
    }

    static void Create()
    {
        new Disposable("Disp2");
    }

    static void Main(string[] args)
    {
        Console.WriteLine("Main thread id: " + Thread.CurrentThread.ManagedThreadId);
        using (new Disposable("Disp1"))
        {
        }

        Console.WriteLine();
        Create();
        GC.Collect();
        Console.ReadKey();
    }
}
```

## Exceptions in finalizers

Process is killed.

Example:
```
class Program
{
    class Test
    {
        ~Test()
        {
            throw new Exception("Test");
        }
    }

    static void Create()
    {
        new Test();
    }

    static void Main(string[] args)
    {
        try
        {
            Create();
            GC.Collect();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }

        Console.ReadKey();
    }
}
```

## What is a garbage collector?

It searches for not used objects and then deletes them. First, starting from roots it marks objects in a graph as used. Then it deletes not marked objects.

## What are GC roots?

- Stack
- Static object references
- CPU registers
- Finalization queues
- Interops (COM)

## What is a heap compaction?

Marked objects are placed to spaces left by deleted objects. So defragmentation. It happens in SOH.

## What is a generation based garbage collection?

There are 3 generations: 0, 1, 2. When an object is created then it goes to Gen 0. When it survives first GC then it goes to Gen 1. When it survives second GC then it goes to Gen 2.

Made for performance. Objects in general are not long living.

## What is LOH, what is SOH, LOG vs SOH?

Small Object Heap contains objects less than 58kB. In SOH objects are compacted.

In Large Object Heap addresses of objects are tracked by a free space table. From .NET 4.5.1 you can enable compaction in LOH.

Objects in SOH are grouped to 3 generations.

## When does garbage collection run?

SOH:
- Gen 0
	- reaches ~256kB
	- Gen 1 triggered
	- Gen 2 triggered
- Gen 1
	- reaches ~2MB
	- Gen 2 triggered
- Gen 2
	- reaches ~10MB
	- GC.Collect()
	- System calls for low available memory
	- Unloaded AppDomain
	
LOH:
- GC.Collect()
- System calls for low available memory
- Unloaded AppDomain
- Reaches ???? (some) level
- Gen 2 in SOH triggered

## Can programmer force garbage collection and is it good practice?

GC.Collect()

No, because if you have a memory hog then it won't solve it.

## What thread do finalizers run on?

Finalizers are fired on some other thread (I don't know if main on a background one).

## What is a finialization queue?

When an object has a finalizer then it is added to the finialization queue to not be destroyed while invoking the finalizer.

When there's a garbage collection and an object is not used by a business logic then it goes to a next gen because of the finialization queue.

Having that collection the reference from the main finialization queue is moved to fReachable queue which contains a list of objects which have to be finalized.

When finalization finished then the reference is removed from fReachable queue and now it can be physically deleted.

## Weak references

A weak reference makes that some object, which is used, can be deleted.

It is helpful when making a cache of big objects which can be easily recreated.

Example:
```
class Program
{
    private static int[] GetData()
    {
        return Enumerable.Range(0, 1000).ToArray();
    }

    private static WeakReference GetRef()
    {
        return new WeakReference(GetData(), false);
    }

    private static int[] GetDataFromRef(WeakReference refr)
    {
        if (refr.IsAlive)
        {
            return (int[])refr.Target;
        }

        return GetData();
    }

    static void Main(string[] args)
    {
        var collection = GetRef();
        GC.Collect();
        GC.WaitForPendingFinalizers();
        var data = GetDataFromRef(collection);
        Console.ReadKey();
    }
}
```

## What thread does GC run on?

It depends. There are multiple modes.

Workstation GC makes UI as a priority.

Server GC makes requests handling as a priority.

You can setup this by `<gcServer>` in the app.config. Or your CPU has only one core so then it is always Workstation GC.

### Workstation non-concurrent GC mode

First mode ever made when CPU's didn't have cores. GC runs on a main thread. Then, everything else is stopped. Now this is disabled by default and not even an option.

### Workstation concurrent GC mode

From .NET 3.5 fired on a separated thread. (I don't know if a main or background one).

Having GC new objects can be created on the ephemeral segment. When the ephemeral segment is full then everything else is stopped to the end of GC.

Concurrent fired only on full GC. GC gen 0 and 1 are still non-concurrent because they are fast.

When full GC starts then there's a GC domain created. Only object in that domain are marked. New objects are created outside of that domain.
It's a little bit bad because some other not used objects can survive current GC and wait to a next one.

### Server non-concurrent GC mode

There are multiple GC thread with the highest priority. There's like SOH and LOH for every CPU core.

When GC threads are running then everything else is stopped.

### Server background GC mode

From .NET 4.5 a default mode. Every CPU core has a background GC thread and normal GC thread for foreground GC.

These threads always exist and the wait for GC.
