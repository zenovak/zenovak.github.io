---
title: CS - 1 Collections and Generics
date: 2023-10-01 #TTTT Means timezone
categories: [C#, C#III-Advanced]
tags: [c#]     # TAG names should always be lowercase
---


## Collections
Collections are classes that we can use to a collection of objects. They are not limited to one type of objects, and they have no size constrains.

We use collections to create dynamic arrays. Remember Lists? It is a type of Generic collection. 

There are 2 types of Collections:

### Non Generic
Can store any type of objects.

Located in:
```
System.Collections
```

### Generic
Limited to one type of object
Located in:
```
System.Collections.Generic
```

<br><br>

---
## Generics

With generic lists, adding an item of type X will no longer have performance penakty due to boxing and unboxing as we specify the type of list or dict object.\
[[QNA Boxing and Unboxing with Generics#ANSWER|Why?]]

The following are examples of how we can implement our own generic lists and dictonaries.

### Generic Lists
```cs
public class GenericList<T>
{
    public void Add(T value)
    {
    }
    public T this[int Index]
    {
        get { throw new NotImplementedException(); }
    }
}
```

### Generic Dictionary (Key Value)
```cs
public class GenericDictionary<Tkey, Tvalue>
{
    public void Add(Tkey key, Tvalue val)
    {
    }
}
```


> **.NET Built In Generics**\
> We do not need to create our own implementation of generic lists and dictionary as it is already available on .NET : See [[CS - 6 Key Value Pairs#Dictionary|Dictionary]] and [[CS - 3 Non premitive types#Lists|List]]
> ```cs
> > System.Collections.Generic
> ```
{: .prompt-tip}

<br><br>

---
## Generics and Comparables.

The following represents a function that returns the largest number.
```cs
public class Utitlies {
    // if a is bigger than b return a, else return b
    public int Max(int a, int b) {
        return a > b ? a : b;
    }
}
```
<br>

However, if we were to declare a generic method as below, it wont compile because T is not comparable:
```cs
//This code will not compile
public T Max<T>(T a, T b) {
    return a > b ? a : b;        
}
```
<br>

We must apply some type of constrains:
- `where T : Icomparable`
- `where T : Product`
- `where T : struct`
- `where T : class`
- `where T : new()`

```cs
public T Max<T>(T a, T b) where T : IComparable
{
    return a.CompareTo(b) > 0 ? a : b;
}
```

<br><br>

---
## Generic Dictionary and Lambda Expression

Coding Examples:
```cs
using System;
using System.Collections.Generic;

namespace Coding.Exercise
{
    public class Run
    {
        // TODO
        // Expressions
        static Func<float, float, float> Plus = (a, b) => a + b;
        static Func<float, float, float> Minus = (a, b) => a - b;
        static Func<float, float, float> Divide = (a, b) => a / b;
        static Func<float, float, float> Multiply = (a, b) => a * b;
        
        
        static Dictionary<string, Func<float, float, float>> Operators = new Dictionary<string, Func<float, float, float>>()
        {
            {"+", Plus},
            {"-", Minus},
            {"/", Divide},
            {"*", Multiply}
        };
        
        public static Func<float, float, float> OperationGet(string key)
        {
            if (Operators.ContainsKey(key))
            {
                return Operators[key];
            }
            else
            {
                return null;
            }
        }
    }
}
```
