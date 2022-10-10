## Time complexity for hash search / binary search

For hashtables it's like O(1).

For binary search it's O(log n).

## `List<>` and ArrayList

They uses dynamically doubled arrays.

- Count - Number of elements.
- Capacity - Length of underlying array.

For performance reasons is better to use the ctor with capacity if you know it before adding things.
Because copying and making a new array is slow.

Example:
```
class Program
{
    static void Main(string[] args)
    {
        List<int> list = new List<int>();
        int capacity = list.Capacity;
        Console.WriteLine("Count: {0}; Capacity: {1}", list.Count, list.Capacity);
        for (int i = 0; i < 10000; i++)
        {
            list.Add(1);
            if (list.Capacity != capacity)
            {
                capacity = list.Capacity;
                Console.WriteLine("Count: {0}; Capacity: {1}", list.Count, list.Capacity);
            }
        }

        Console.ReadKey();
    }
}
```

## Stack and `Stack<>`

Non and generic stack. They use dynamically doubled arrays.

## Queue and `Queue<>`

Non and generic queue. They use dynamically doubled arrays.

## `LinkedList<>`

A doubled linked list like you make it on first semester. So head and then pointer to a next element.

Access is slower than 'List<>' but adding/removing things in the middle are faster.

## BitArray

Compacts bools as bits. API is like Add, Or, Xor, etc. Capacity equals count.

BitVector32 is similar but always has 4 bytes and it is a struct.

## StringCollection

ArrayList for strings. It's better now to use `List<string>`

## `Dictionary<,>` and Hashtable

Generic and not generic hash table. They uses dynamically doubled arrays.

Again, for performance reasons is better to use the ctor with capacity if you know it before adding things.

Keys have to be unique.

## `HashSet<>`

Set of data. It is also a hash table.

Elements are distinct and not sorted. 

It's like a dictionary but without Value.

It has method for sets based operations.

Example:
```
class Program
{
    static void Main(string[] args)
    {
        var col = new HashSet<int>();
        col.Add(1);
        col.Add(1);
        Console.WriteLine(col.Count);
        Console.ReadKey();
    }
}
```

## `SortedSet<>`

Sorted `HashSet<>`. It uses a red-black tree.

## `SortedDictionary<,>`

Sorted dictionary based on the key. It uses a `SortedSet<>`.

## SortedList and `SortedList<,>`

Sorted list based on the key. They have 2 arrays - one for values and the other one for keys.

They make binary search available all the time.

It uses less memory than `SortedDictionary<,>` but adding things is slower.

## StringDictionary

Hashtable for strings and strings. It's better to use `Dictionary<string, string>` now.

## ListDictionary

Implements IDictionary with linked list. Better performance when there's less than 10 elements. But srsly...

## HybridDictionary

Uses ListDictionary and Hashtable based on number of elements.

## NameValueCollection

Uses 2 arrays with strings. But keys can be redundant. So maybe it's better to use like `Dictionary<string, string[]>`.

## OrderedDictionary

Elements can be accessed by key and index. Uses ArrayList and Hashtable.
