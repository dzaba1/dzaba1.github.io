## Consts vs. Readonly

Const is not a variable. Const is initialized during a compilation time. Always one value. Consts are static.
Readonly is a variable. Initialized during runtime during initialization or in ctor. Can have multiple value depending on ctor.

Readonly can be hack-changed via reflection.
Example:
```
class TestClass
{
    public const int TestConsts = 1;
    public readonly int TestReadonly1 = 1;
    public readonly int TestReadonly2;

    public TestClass(int value)
    {
        TestReadonly2 = value;
        TestReadonly1 = 2;
    }
}

class Program
{
    private static void ChangeConstsByReflection(Type type, string constName, object value)
    {
        var constInfo = type.GetField(constName,
            BindingFlags.Static | BindingFlags.Public | BindingFlags.NonPublic);
        constInfo.SetValue(null, value);
    }
    private static void ChangeReadonlyByReflection(object obj, string field, object value)
    {
        var fieldInfo = obj.GetType().GetField(field,
            BindingFlags.Instance | BindingFlags.Static | BindingFlags.Public | BindingFlags.NonPublic);
        fieldInfo.SetValue(obj, value);
    }
    static void Main(string[] args)
    {
        try
        {
            TestClass test = new TestClass(3);
            Console.WriteLine(test.TestReadonly1);
            Console.WriteLine(test.TestReadonly2);
            //test.TestReadonly1 = 4; ChangeReadonlyByReflection(test, "TestReadonly1", 4); Console.WriteLine(test.TestReadonly1);
            try
            {
                ChangeConstsByReflection(typeof(TestClass), "TestConsts", 2);
            }
            catch (Exception ex)
            {
                Console.WriteLine("{0}: {1}", ex.GetType().Name, ex.Message);
            }
            Console.WriteLine("Done.");
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
        finally
        {
            Console.ReadKey();
        }
    }
}
```

## What are 4 methods every type implements in .net?

- ToString() - By default full type name.
- Finalize() - Destructor. Executed by GC. By default it does nothing.
- GetHashCode() - Returns unique int value for every class/object. Used mostly by hash tables. By default it doesn't guarantee uniquenes.
- Equals() - Checks if the passed object is 'the same'. By default for reference types it checks the pointer (object address). By default for value types it firsts checks the type and then byte per byte.

## IEquatable<T> and IEqualityComparer<T>

`IEquatable<T>` is like generic Equals(). Made for improving performance. Used by generic collections.
`IEqualityComparer<T>` is like `IEquatable<T>` but separated from the exact class.

## ICloneable

One method for cloning objects. But this interface is not used in common because it's not generic and you don't know if it's shallow or deep copy.

## IConvertable

Contains methods for changing types to some other simple types. Used by conversion.
Methods like ToByte(), ToChar(), etc.

It uses IFormatProvider. This interface has method GetFormat which return some object which formats something depending of input type.

## IComparable, IComparable<T> and IComparer<T>

`IComparable` has one method which returns signum value when comparing two object/values. Used mostly by sorting.
`IComparer<T>` is like `IComparable<T>` but separated from the exact class.

## What's immutability?

State of object can't be changed after initialization. By this, the object is thread safe.
Moreover, we have a copy after each operation.
Classes and structs are not immutable by default.
Structs should be immutable because it's easy to make a mistake while executing some method with this struct as a param. Then we have to add `ref`.

## What is string interning?

Interning pool is a string pull in the whole app. A little bit flighweitght patter.
By default every literal string goes to the interning pool. Every literal string is the same and unique.

Example:
```
class Program
{
    static void Main(string[] args)
    {
        string s1 = "MyTest";
        string s2 = new StringBuilder().Append("My").Append("Test").ToString();
        string s3 = String.Intern(s2);
        Console.WriteLine((Object)s2 ==
                          (Object)s1); // Different references. Console.WriteLine((Object)s3 == (Object)s1); // The same reference.
        Console.ReadKey();
    }
}
```

## What is boxing and unboxing?

It's a change from value type to reference type so object or interface. And vice versa.
Used to fulfil the polymorphism and in non generic collections.
It's not efficient because it makes new references on heaps.

Example:
```
static void Main(string[] args)
{
    int i = 1;
    object box = i;
    int j = (int)box;
    Console.WriteLine("i == j: {0}", i == j);
    Console.WriteLine("i.Equals(box): {0}", i.Equals(box));
    Console.WriteLine("box.Equals(i): {0}", box.Equals(i));
    Console.WriteLine("Object.ReferenceEquals(i, box): {0}", Object.ReferenceEquals(i, box));
    Console.WriteLine("Object.ReferenceEquals(j, box): {0}", Object.ReferenceEquals(j, box));
    Console.WriteLine("Object.ReferenceEquals(i, j): {0}", Object.ReferenceEquals(i, j));
    Console.ReadKey();
}
```

## What is an anonymous method?

Anonymous method is a method with body but without name.
Assigned to delegete types by the `delegate` keyword or lambda.
Having the `delegate` we can ommit parameters but we can't do that having lambdas.

Example:
```
class Program
{
    static void Main(string[] args)
    {
        Action<int> del = delegate(int i) { Console.WriteLine(i); };
        del(2);
        del = delegate { Console.WriteLine("Omitting params"); };
        del(3);
        Console.ReadKey();
    }
}
```

## What is a lambda expression?

Basic lambda is an anonomous method written in a different form. Used by delegates or for making Expression trees.

Example:
```
static void Main(string[] args)
{
    Action<int> del = i => Console.WriteLine(i);
    del(2);
    Console.ReadKey();
}
```

## What is an expression tree?

It's a structure which represents some logic in a tree form. Every expression is a tree node.
Made by `Expression` class from LINQ.
Example:
```
class Program
{
    private static void WriteExpression(Expression expr, int indent)
    {
        if (expr is BinaryExpression)
        {
            BinaryExpression b = (BinaryExpression)expr;
            Console.WriteLine(new string('\t', indent) + "Binary expression: " + b.NodeType);
            WriteExpression(b.Left, indent + 1);
            WriteExpression(b.Right, indent + 1);
        }
        else if (expr is ParameterExpression)
        {
            ParameterExpression p = (ParameterExpression)expr;
            Console.WriteLine(new string('\t', indent) + "Parameter expression: " + p.Type.Name);
        }
        else if (expr is ConstantExpression)
        {
            ConstantExpression c = (ConstantExpression)expr;
            Console.WriteLine(new string('\t', indent) + "Constant expression: " + c.Value);
        }
        else
        {
            Console.WriteLine(new string('\t', indent) + "Expression: " + expr.NodeType);
        }
    }

    static void Main(string[] args)
    {
        Expression<Func<int, bool>> expr = i => (i > 5 && i < 15) || i == -1;
        WriteExpression(expr.Body, 0);
        Console.ReadKey();
    }
}
```

## What is dynamic keyword?

It makes a variable where the type check is omitted during compilation. Type is checked when the variable is used. So late binding.
Helpful when working with COM objects.

## What's late binding and early binding?

Early - Type and signature are checked during compilation time.
Late - Type and signatures are not checked during compilation time. Those are checked during runtime when used. Could end with some error that something is missing in the variable.

## What's reflection?

Reflection is an ability to read own metadata. By reflection we can have data about types, assemblies, modules, attributes, etc.
Example: Execute some private method.

## What's the Activator class?

A static class which creates new local or remote objects or gets references to remote objects.
Remote here means another AppDomain or COM.

Example:
```
class TestClass
{
    public void Test()
    {
        Console.WriteLine("Test");
    }
}

class Program
{
    static void Main(string[] args)
    {
        TestClass test = (TestClass)Activator.CreateInstance(typeof(TestClass));
        test.Test();
        Console.ReadKey();
    }
}
```

## What's the var key word?

Non formal notation of some type. Compiler is 'guessing' a type when compiling first assignment.
`var` is not `dynamic` - all the time a variable is strongly typed.

`var` has to be used when working with anonymous types. Anonymous types are also strongly typed.

Example:
```
class TestClass
{
    public string A { get; set; }
    public string B { get; set; }
    public string C { get; set; }
}

class Program
{
    static void Main(string[] args)
    {
        var test1 = new TestClass() { A = "Test1" };
        Console.WriteLine(test1.A);
        Console.WriteLine();
        var test2 = 2;
        Console.WriteLine(test2);
        Console.WriteLine(test2.GetType().Name);
        Console.WriteLine();
        TestClass[] testArray = new TestClass[3];
        for (int i = 0; i < testArray.Length; i++)
        {
            TestClass item = new TestClass();
            item.A = String.Format("Test{0}_A", i);
            item.B = String.Format("Test{0}_B", i);
            item.C = String.Format("Test{0}_C", i);
            testArray[i] = item;
        }
        var test3 = from t in testArray select new { t.A, t.B };
        foreach (var item in test3)
        {
            Console.WriteLine(item.A);
            Console.WriteLine(item.B);
        }
        Console.WriteLine(test3.ElementAt(0).GetType().Name);
        Console.ReadKey();
    }
}
```

## Anonymous types, what are they?

Anonumous types is a mechanism of making types and class inderectly with read only properties.
Can be assigned to `object`, `var` and `dynamic`.
Anonymous type name is made by the compiler.

If there are 2 anonymous types with the same propertes in the same order then it will be one type generated.

They have GetHashCode and Equals implemented. They compare all properties.

Every property in the `{}` list has to be initialized.

You can pass anonymous type objects to different methods as `object` or `dynamic`.

Example:
```
static void Main(string[] args)
{
    var bigType = new { A = "A", B = "B", C = "C" };
    var smallType = new { A = "A", B = "B" };
    var array = new { bigType, smallType };
    Console.WriteLine(bigType.GetType().Name);
    Console.WriteLine(smallType.GetType().Name);
    Console.WriteLine(array.GetType().Name);
    Console.WriteLine();
    Console.WriteLine(new { bigType.A, bigType.B }.GetType().Name);
    Console.WriteLine();
    Console.WriteLine(smallType.Equals(new { bigType.A, bigType.B }));
    Console.ReadKey();
}
```

## What's the delegate?

A method pointer. Delegate is also a reference type.
Can be binded together by the Delegate.Combine method. When one execution fails then everything fails.
Executed on the same caller thread.

Example:
```
static void Main(string[] args)
{
    Action<string> del1 = i => Console.WriteLine(i + "_Del1");
    Action<string> del2 = i => Console.WriteLine(i + "_Del2");
    Action<string> delCombined = (Action<string>)Delegate.Combine(del1, del2);
    delCombined("Test");
    Console.WriteLine();
    del1 = delegate(string i) { throw new Exception("Test error."); };
    delCombined = (Action<string>)Delegate.Combine(del1, del2);
    try
    {
        delCombined("Test");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
    Console.WriteLine();
    Action asyncDel = () => Console.WriteLine("Thread: " + Thread.CurrentThread.ManagedThreadId);
    asyncDel();
    Task async = new Task(asyncDel);
    async.Start();
    async.Wait();
    Console.WriteLine();
    Console.ReadKey();
}
```

## What are events?

Events are just abstraction made on delegates. A client can add or remove delegates to an event. It's similar to .NET property.

Events can have `add` and `remove` accessors. Then you can change a logic of adding and removing delegates.
Having `add` and `remove` you can't directly invoke the event.

You can have events in interfaces but not delegates.

From an even you can get an invocation list by the GetInvocationList method and manually trigger every delegate on some separated thread or with some custom error handling.

Example:
```
class Program
{
    private static event Action<string> Event1;
    private static event Action<string> EventError;
    private static event Action EventAsync;
    private static Action<string> _del;

    public static event Action<string> ExplicitEvent
    {
        add
        {
            Console.WriteLine("Adding a delegate to ExplicitEvent");
            _del += value;
        }
        remove
        {
            Console.WriteLine("Removing a delegate to ExplicitEvent");
            _del -= value;
        }
    }

    private static event Action<string> EventErrorSafe;

    private static void RaiseEventErrorSafe(string obj)
    {
        if (EventErrorSafe != null)
        {
            foreach (Action<string> handler in EventErrorSafe.GetInvocationList())
            {
                try
                {
                    handler(obj);
                }
                catch (Exception ex)
                {
                    Console.WriteLine("Error in handler occured: " + ex.Message);
                }
            }
        }
    }

    static void Main(string[] args)
    {
        Event1 += i => Console.WriteLine(i + "_Del1");
        Event1 += i => Console.WriteLine(i + "_Del2");
        Event1("Test");
        Console.WriteLine();
        ExplicitEvent += i => Console.WriteLine(i + "_Del1");
        ExplicitEvent += i => Console.WriteLine(i + "_Del2");
        _del("Explicit Test");
        Console.WriteLine();
        EventError += delegate(string i) { throw new Exception("Test error."); };
        EventError += i => Console.WriteLine(i + "_Del2");
        try
        {
            EventError("Test");
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
        Console.WriteLine();
        EventErrorSafe += delegate(string i) { throw new Exception("Test error."); };
        EventErrorSafe += i => Console.WriteLine(i + "_Del2");
        RaiseEventErrorSafe("Test");
        Console.WriteLine();
        EventAsync += () => Console.WriteLine("Thread: " + Thread.CurrentThread.ManagedThreadId);
        EventAsync();
        Task async = new Task(() => EventAsync());
        async.Start();
        async.Wait();
        Console.WriteLine();
        Console.ReadKey();
    }
}
```

## What is a closure?

If some anonymous method uses some variable which is outside its body then a closure is made.
It's a type generated by compiler and the outside variable will be assigned as field before making the execution.
Having this, if you change the outside variable before triggering the anonymous method then during the execution the value will be correct.

Example:
```
static void Main(string[] args)
{
    int value = 1;
    Action writeLine = () => Console.WriteLine(value);
    writeLine();
    value = 2;
    writeLine();
    Console.ReadKey();
}
```

## How to perform a deep copy of any object automatically?

The easiest way is to use the `BinaryFormatter` but the type has to be serailzable.

## What are generic constraints?

Generics looks like passing types as definition parameters.
Having that you can make a collection without boxing/unboxing and with fulfiling abstraction.
Example: `List<T>`

Constraints:
- `T:struct` - The `T` type has to be a value type
- `T:class` - The `T` type has to be a reference type
- `T:new()` - The `T` type has to have a parameterless ctor.
- `T:new()` - The `T` type has to have a parameterless ctor.
- `T:<base class or interface name>` - The `T` type has to implement provided interface or dervie from provided class.
- `T:U` - The `T` type has to implement generic `U` or derive from generic `U`.

## What's the difference between explicit and implicit interface implementation?

Implicit makes the method is public and it covers the interface method.
Explicit is private for the implementing type.
If there are 2 different interfaces with methods with the same name then during implementation some of them has to be explicit.

Example:
```
class Program
{
    interface IInterface1
    {
        void Do();
    }

    interface IInterface2
    {
        void Do();
    }

    class SomeClass : IInterface1, IInterface2
    {
        public void Do()
        {
            Console.WriteLine("Invoking Do.");
        }
        void IInterface2.Do()
        {
            Console.WriteLine("Invoking Do from IInterface2.");
        }
    }

    static void Main(string[] args)
    {
        SomeClass obj = new SomeClass();
        obj.Do();
        ((IInterface1)obj).Do();
        ((IInterface2)obj).Do();
        Console.ReadKey();
    }
}
```

## Differences between new and virtual

Having `virtual` you can execute derived methods having base class reference so polymorphism.

Example:
```
class Program
{
    class BaseClass
    {
        public void Test1()
        {
            Console.WriteLine("Test1 from BaseClass");
        }
        public virtual void Test2()
        {
            Console.WriteLine("Test2 from BaseClass");
        }
    }

    class NextClass : BaseClass
    {
        public new void Test1()
        {
            Console.WriteLine("Test1 from NextClass");
        }
        public override void Test2()
        {
            Console.WriteLine("Test2 from NextClass");
        }
    }

    static void Main(string[] args)
    {
        try
        {
            BaseClass baseCls = new BaseClass();
            baseCls.Test1();
            baseCls.Test2();
            Console.WriteLine();
            NextClass nextCls = new NextClass();
            nextCls.Test1();
            nextCls.Test2();
            Console.WriteLine();
            baseCls = nextCls;
            baseCls.Test1();
            baseCls.Test2();
            Console.WriteLine();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
        finally
        {
            Console.ReadKey();
        }
    }
}
```

## Differences between reference types and value types

Reference typed objects are put on heaps. So reference is a pointer.
Value types don't have finalizers. Value types can implement interfaces but no inheritance.
If value types are passed to methods then copies are made. You can use `ref` or `out` so then address will be passed instead.

Strings are immutable reference types. Nullables are value types.

Example:
```
class Program
{
    struct Struct
    {
        public int x;
    }

    private static void Change1(Struct s, int value)
    {
        s.x = value;
    }

    private static void Change2(ref Struct s, int value)
    {
        s.x = value;
    }

    static void Main(string[] args)
    {
        Struct s1 = new Struct();
        s1.x = 1;
        Change1(s1, 2);
        Console.WriteLine(s1.x);
        Change2(ref s1, 3);
        Console.WriteLine(s1.x);
        Console.ReadKey();
    }
}
```

## Differences between union and concat, first and single, select and select many, cast and oftype?

Union joins distinct elements (SQL union) but concat every element (SQL union all)

First returns first element. If empty collection then exception.
Single return first element. If empty collection or found multiple elements having the same predicate then exception.

Select works on single elements in a collections but SelectMany wants collection of collections and then it flattens results.

Cast casts all elements to provided type. If some type doesn't match then exception.
OfType filters elements based on provided type.

Example:
```
class Program
{
    class Class1
    {
        public Class1(int a, int b, int c)
        {
            A = a;
            B = b;
            C = c;
            Array = new int[] { A, B, C };
        }
        public int A { get; set; }
        public int B { get; set; }
        public int C { get; set; }
        public int[] Array { get; private set; }
    }

    static void Main(string[] args)
    {
        int[] t1 = new int[] { 1, 2, 3 };
        int[] t2 = new int[] { 3, 4, 5 };
        foreach (var item in t1.Union(t2))
        {
            Console.WriteLine(item);
        }
        Console.WriteLine();
        foreach (var item in t1.Concat(t2))
        {
            Console.WriteLine(item);
        }
        Console.WriteLine();
        Class1[] t3 = new Class1[] { new Class1(1, 1, 1), new Class1(2, 2, 2), new Class1(3, 3, 3) };
        foreach (var item in t3.Select(x => x.A))
        {
            Console.WriteLine(item);
        }
        Console.WriteLine();
        foreach (var item in t3.SelectMany(x => x.Array))
        {
            Console.WriteLine(item);
        }
        Console.WriteLine();
        object[] t4 = new object[] { "1", 2, 3.0d };
        try
        {
            foreach (var item in t4.Cast<int>())
            {
                Console.WriteLine(item);
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("{0}: {1}", ex.GetType().Name, ex.Message);
        }
        Console.WriteLine();
        foreach (var item in t4.OfType<int>())
        {
            Console.WriteLine(item);
        }
        Console.WriteLine();
        Console.ReadKey();
    }
}
```

## Time complexity for hash search / binary search

For hashtables it's like O(1).
For binary search it's O(log n).

## What is LINQ?

Language INtegrated Query - LINQ extends the language by the addition of query expressions, which are akin to SQL statements, and can be used to conveniently extract and process data from arrays, enumerable classes, XML documents, relational databases, and third-party data sources.

Having one representation it is similar to SQL.

## Differences between IEnumerable and IQueryable?

`IEnumerable` - Iterator pattern. 2 methods: Current() and MoveNext(). Executes on client side.
`IQueryable` - Has `IQueryProvider` which uses Expression tree. Executes on server side.

## What's the covariance and contravariance?

They describes casting of objects having generic interfaces and delegates.
Two key words:
- `in` - contravariance
- `out` - covariance

Can't use in generic classes.

Contravariance is a casting to more generic type.
Covariance is a casting to more specific type.

Example:
```
interface IVariant<out A, in B>
{
    A Get(B b);
}

class Variant<A, B> : IVariant<A, B>
{
    public A Get(B b)
    {
        Console.WriteLine("Get: " + b.GetType().Name);
        return default(A);
    }
}

class Program
{
    private static string GetString()
    {
        return String.Empty;
    }

    private static object GetObject()
    {
        return null;
    }

    private static void SetString(string str)
    {
        
    }

    private static void SetObject(object obj)
    {
    }

    static void Main(string[] args)
    {
        //References string str = "test"; object obj = str;
        //Array covariance object[] array = new string[10]; try { array[0] = 1; } catch (Exception ex) { Console.WriteLine(ex.GetType().Name + ": " + ex.Message); }
        //Delegates covariance and contravariance //Func covariance // TResult Func<out TResult>() Func<object> objFunc = GetString;
        //Action contravariance // Action<in T>(T obj) Action<string> strAction = SetObject;
        //Enumerable covariance // IEnumerable<out T> IEnumerable<object> objCol = new List<string>();
        IVariant<object, string> var1 = new Variant<string, object>();
        Console.ReadKey();
    }
}
```

## How generics works in run-time

If the generic type is a value type then there will be made separated, JITed typed for every generic type passed.
Similar to tempaltes in C++ so `List<int>` and `List<byte>` will be 2 separated compiled types.

If generic type is a reference type then there will be `object` like references casted in runtime.

## What does yield keyword do?

Lazy loading. `yield` makes an interator implementation automatically.

`yield return value` - Sets the value as a current one.
`yield break` - Stops the iteration.

The function has to return `IEnumerable<T>` or `IEnumerator<T>`.

Having `yield` the compiler makes a new class which represents a machine state.

Example:
```
class Program
{
    private static IEnumerable<int> GetEnumerable(int count)
    {
        Console.WriteLine("Start");
        for (int i = 0; i < count; i++)
        {
            Console.WriteLine("i: " + i);
            yield return i;
        }
        Console.WriteLine("End");
    }

    private static IEnumerator<int> GetEnumerator(int count)
    {
        Console.WriteLine("Start");
        for (int i = 0; i < count; i++)
        {
            Console.WriteLine("i: " + i);
            yield return i;
        }
        Console.WriteLine("End");
    }
    
    static void Main(string[] args)
    {
        var list = GetEnumerable(4);
        Console.WriteLine("Printing enumerable...");
        foreach (int item in list)
        {
            Console.WriteLine(item);
        }
        Console.WriteLine();
        Console.WriteLine("Printing enumerator...");
        var iterarotor = GetEnumerator(4);
        while (iterarotor.MoveNext())
        {
            Console.WriteLine(iterarotor.Current);
        }
        Console.ReadKey();
    }
}
```