---
title: CS - 3 Logical Operators
date: 2023-07-09 #TTTT Means timezone
categories: [C#, C#Fundamentals]
tags: [c#]     # TAG names should always be lowercase
---

>**Non-primitive Types**\
>By Mosh Hamedani\
>Extras by Zenovak


## Classes
Classes are building blocks of our applications. A class combines related variables (also called fields, attributes or properties) and functions (methods) together.

> **Fields & properties**\
> Fields and properties are technically different in C# but conceptually they mean the same thing. They represent attributes about a class. I’ll explain the difference between fields and properties in detail in my C# Intermediate course.
{: .prompt-tip}

An _object_ is an instance of a class. At runtime, many objects collaborate with each other to provide some functionality. As a metaphor, think of a supermarket. At a supermarket, there are multiple people working together to provide services to customers. Each person has a role and is focused only on one area of functionality. Software is exactly the same. A role in a supermarket is like a class in a C# application. A person filling that role during work hours, is like an object in an application at runtime.

> **NOTE**\
> Even though there is a slight different between the Class and Object, these words are often used interchangeably.
{: .prompt-info}

To create a class:
```cs
public class Person
{
}
```

Here, public is what we call an _access modifier_. It determines whether a class is visible to other classes or not. Access modifiers are beyond the scope of this course and I've covered them in the second part of this course: C# Intermediate.


Here is a class with a field and a method:
```cs
public class Person
{
    public string Name;
    //Here void means this method does not return a value.
    public void Introduce()
    {
        Console.WriteLine("My name is " + Name);
    }
}
```


To create an object, we use the new operator:
```cs
Person person = new Person();
```


A cleaner way of writing the same code is:
```cs
var person = new Person();
```

We use the new operator to allocate memory to an object. In C# you don’t have to worry about de-allocating the memory. CLR has a component called Garbage Collector, which automatically removes unused objects from memory.

Once we have an object, we can access its fields and methods with the dot notation:
```cs
Person person = new Person(); 
erson.Name = "Mosh"; 
person.Introduce();
```

<br>

### Static modifier
When applied to a class member (field or method), makes that member accessible only via the class, not any objects. So in the earlier example, if the Introduce method was static, we could access it via the Person class:
```cs
Person.Introduce();
```


We use static members in situations where we want only one instance of that member to exist in memory. As an example, the Main method in every program is declared as static, because we need only one entry point to the application.

In the real-world, it’s best to stay away from static as much as you can because that makes writing automated tests for applications hard. Automated testing is the topic for another course, but for now, just remember how to use members that are already declared as static, and prefer not to declare your own class members as static.

<br>

---
## Structs

A struct (structure) is a type similar to a class. It combines related fields and methods together. Structs are all value types. They do not support inheritance, but you may use Interfaces [[CS - 5 Interfaces|Read more]].
```cs
public struct RgbColor
{
    public int Red; 
    public int Green;
    public int Blue;
}
```

Structs also do not support parameterless constructors, only support constructors with parameters.. Struct members do not support abstract, virtual, or protected. 

Use structs only when creating small lightweight objects. That is for a subtle performance optimization. In the real-world, 99% of the time, you create new types using classes, not structures.

> **NOTE**\
> In .NET, all primitive types are declared as a structure. They are small and lightweight. The biggest primitive type doesn’t take more than 16 bytes.
{: .prompt-info}

<br>

---
## Arrays
An array is a data structure that is used to store a collection of variables of the same type.

For example, instead of declaring three int variables (that are related), we can create an int array like this:
```cs
int[] numbers = new int[3];
```

An array in C# is actually an instance of the Array class. So, that's why here we have to use the new operator to allocate memory to this object. Here, the number 3 specifies the size of the array. Once an array is created, its size cannot be changed. If you need a list with dynamic size, you need to use the List class (explained later in the course).

To access elements in an array, we use the square bracket notation:
```cs
numbers[0] = 1;
```

> **Important!**\
> C# arrays are zero-indexed. So the first element has index 0.\
> Arrays are also **Reference Types**, any changes you make to the array, will directly affects the original array! Remember this when you write functions that modifies arrays.
{: .prompt-tip}

<br>

### Reference type effects
Consider the 2 following scenario when we want to make changes to a copy of an array:

```cs
var arrayA = new byte[3] { 1, 2, 3 };
```

<br>

#### Senario1
```cs
var arrayB = arrayA;
arrayB[0] = 0;
```
In this scenario, because Arrays are reference types, both `arrayB[0]` and `arrayA[0]` has become 0! When you assign an array like this, you are only assigning its memory address to another variable. Under the hood, both variables point to the same array in memory! 

You have made changes to the original array .

<br>

#### Senario2
```cs
var arrayB = (byte[]) arrayA.Clone();
arrayB[0] = 0;
```
In this scenario we are creating a carbon coby of the arrayA and assign it as arrayB. Changes made to arrayB will not affect arrayA, as we now effectively created 2 arrays in memory!

<br>

This affect is refected even when you are using functions!

```cs
// Arrays are ref class. This changes the original array.
public static byte[] UpdateArray(byte[] array, byte update) {
    array[0] = update;
    return array;
}

// this performs the changes on a modified copy of the original array.
public static byte[] ShadowChangeArray(byte[] array, byte update) {
    byte[] shadow = new byte[3];
    array.CopyTo(shadow, 0);
    shadow[0] = update;
    return shadow;
}
```

<br>

### Extra Tips
There are multiple ways to declare an array. We could also use the object initializer to immediately assign some values to the array.
```cs
var numbers = new[] { 1, 2, 3, 7, 5, 9, 8};
var numbers2 = new int[3];
int[] numbers3 = new int[] { 1, 2, };
```

Below are some of the useful methods in dealing with Arrays.
```cs
// Returns the size of an array
numbers.Length;

// Below are static methods from the array class:
// Returns the index of an element in an array.
Array.IndexOf(numbers, 2);

// Clear(). Clears the element of an array.
// Array.Clear(array, startAtindex, stopAfter)
Array.Clear(numbers, 0, 3);

// Copy(). Copy elements from one array to another from first element to the specific index
Array.Copy(numbers3, numbers2, 2);

// Sort(). This sorts the array. Smallest to largest
Array.Sort(numbers);

// Reverse(). This reverse the sorting of the array. Largest to smallest. 
Array.Reverse(numbers);
```

<br>

### Multidimensional Arrays
![[Pasted image 20230405181927.png]]
We have 2 types of multidimensional array in C# - Rectangular and Jagged Arrays.

- Jagged Array can be said to be a single Dimension Array, that contains multiple elements and each of those elements would represent an array. 
- Rectangular array.


> **NOTE!**\
> CLR is optimised for single Dimension Arrays. Implementing a matrix with jagged arrays is better than with rectangular array.
{: .prompt-tip}

<br>

### Rectangular Arrays
To declare a rectangular 2D array (alternatively use ``var``): 
```cs
int[,] board = new int[3, 3];
```

We can use the same object initilization synthax:
```cs
var checker = new int[3, 5]
{
    {1, 2, 3, 4, 5},
    {6, 7, 8, 9, 10},
    {11, 12, 13, 14, 15}
}
```


To declare a rectangular 3D array:

```cs
var colors = new int[3, 5, 7]
```

> **Iterations**\
> We can loop thru all values in the rectangular array with a nested for loop:
> ```cs
> for (int i = 0; i < 3; i++) {
>     for (int j = 0; j < 5; j++)
>     {
>         Console.WriteLine(arrays[i, j]);
>     }
> }
> ```
{: .prompt-tip}


**3D array indexes and declarations**

```cs
var cube = new int[1,2,3] {
    {
        {1, 2, 3}, {3, 4, 5}
    }
}
```

Its depiction is as follows:
```
Specifies rows (x axis):
{}
{}
{}

Specifies collumns(y axis)
{ , }

Specifies depth (z axis)
{ {}, {} }
```


<br>

### Jagged Arrays
To declare a jagged array, we must first specify the top level arrays. In this case would be `3`. Then we must initialize the top level arrays:
```cs
// Jagged array declaration
var cartons = new int[3][];

// Initializing the top level arrays
cartons[0] = new int[3];
cartons[1] = new int[5];
cartons[2] = new int[2];

// Alternatively
var arrays = new int[3][]
{
    new int[2] {1, 2},
    new int[4] {1, 2, 3, 4},
    new int[3] {1, 2, 3},
};
```

In jagged arrays. The arrays within them are not fixed sizes. To access a Jagged Array:
```cs
cartons[0][2] = 10;
Console.WriteLine(cartons[0][2]);
```

> **Iterations**\
> We iterate thru jagged arrays using .Length in the ``for`` loops as we dont know the length of the array.
> ```cs
> for (int i = 0; i < arrays.Length; i++) {
>     for (int j = 0; j < arrays[i].Length; j++) {
>         Console.WriteLine(arrays[i][j]);
>     }
> }
> ```
{: .prompt-tip}


```csharp
// Two-dimensional GetLength example.
int[,] two = new int[5, 10];
Console.WriteLine(two.GetLength(0)); // Writes 5
Console.WriteLine(two.GetLength(1)); // Writes 10
```

<br>

---

## Strings

A string is a sequence of characters. In C# a string is surrounded by double quotes, whereas a character is surrounded by a single quote.
```cs
string name = "Mosh"; 
char ch = 'A';
```


There are a few different ways to create a string:

**Using a string literal:**
```cs
string firstName = "Mosh";
```


**Using concatenation:** useful if you wanna combine two or more strings.
```cs
string name = firstName + " " + lastName;
```


**Using string.Format:** cleaner than concatenating multiple strings since you can see the output.
```cs
string name = string.Format("{0} {1}", firstName, lastName);
```


**Using string.Join:** useful when you have an array and would like to join all elements of that array with a character:
```cs
var numbers = new int[3] { 1, 2, 3 }; 
string list = string.Join(",", numbers);
```


> **NOTE**\
> C# strings are immutable, which means once you create them, you cannot change their value or any of their characters. The String class has a few methods for modifying strings, but all these methods return a new string and do not modify the original string.
{: .prompt-tip}


<br>

**String vs string**
Remember, all types in C# map to a type in .NET Framework. So, the “string” type in C# (all lowercase), maps to the String class in .NET, which means we can declare a string in either of the following ways:
```cs
string name;
String name;
```


The only difference is that if you use the String type, you need to import the System namespace on top of the file, because that’s where the String class is defined.
```cs
using System;
```

<br>

### Escape Characters

There are few special characters in C# called escape characters:

| Escape Character | Description                    |
|:----------------:|:------------------------------ |
|        \n        | New line                       |
|        \t        | Tab                            |
|        \\        | The \ character itself         |
|        \'        | The ' (single quote) character |
|        \"        | The " (double quote) character |


So if you want to have a new line in your string, you use ``\n.

Since the backslash character is used to prefix escape characters, if you want to use the backslash character itself in your string (eg path to a folder), you need to prefix it with another backslash:
```cs
string path = "c:\\folder\\file.txt";
```

<br>

### Verbatim Strings

Sometimes if there are many escape characters in a string, that string becomes hard to read and understand.
```cs
var message = "Hi John\nLook at the following path:c:\\folder1\\folder2";
```

Note the ``\n`` and double backslashes (``\\``) here. We can re-write this string using a _verbatim string._ We simply prefix our string with an ``@`` sign, and get rid of escape characters:
```cs
var message = @"Hi John Look at the following path: c:\folder1\folder2";
```

<br>


---

## Reference Types and Value Types

In C#, we have two main types from which we can create new types: classes and structures (structs).

Classes are _Reference_ _Types_ while structures are _Value_ _Types._
<br>

### Value Types

When you copy a value type to another variable, a copy of the value stored in the source variable is taken and stored in the target variable. Hence, these two variables will be independent.
```cs
var i = 10; 
var j = i; j++;
```

Here, incrementing j does not impact i.

In practical terms, it means: if you pass an argument to a method and that argument is a value type, its value will be copied. So any modifications made to that argument in the method will be lost upon returning from that method.

Remember: Primitive types are structures so they are value types. Any custom structure you define will also be a value type.

<br>

### Reference Types

With a reference type, however, the reference (or memory address) of the object is copied to the target variable. This means: if you copy a reference type to another variable, any changes you make to the object referenced by either of these variables, will be visible through the other variable.
```cs
var array1 = new int[3] { 1, 2, 3 }; 
var array2 = array1;

array2[0] = 0;
```

Here, both array1 and array2 reference (or point) the same array object in memory. So, after the third line, the first element of both array1 and array2 will be 0.


> **Remember**\
> Arrays and strings are classes, so they are reference types. Any custom classes you define will also be a reference type.
{: .prompt-tip}

<br>

---
## Enums

An enum is a data type that represents a set of name/value pairs. Use enums when you need to define multiple related constants. 


> **NOTE**\
> Enums are also often used to for switch statements
{: .prompt-tip}

```cs
// Enums are declared in namespace region
public enum ShippingMethod
{
    Regular = 1,
    Express = 2
}
```

<br>

Now we can declare a variable of type ShippingMethod enum and use the dot notation to initialize it:
```cs
var method = ShippingMethod.Express;
```

<br>

Enums are internally integers. So you can easily cast them to and from an int:

```cs
var methodId = 1;
var method = (ShippingMethod)methodId;
```


```cs
var method = ShippingMethod.Express; 
var methodId = (int)method;
```

<br>

To convert an enum to a string use the ToString method. Every object in C# has this method and can be converted to a string:
```cs
var method = ShippingMethod.Express;
var methodName = method.ToString();
```

<br>

To convert a string to an enum (called parsing), use Enum.Parse:
```cs
var method = (ShippingMethod)Enum.Parse(typeof(ShippingMethod), methodName);
```

<br>

---
## ArrayList
ArrayList are a type of Non-generic from the System.Collections namespace. They can store any type of Object, without type specification, and is fully dynamic in size. [[CS - 1 Collections and Generics|More on Collections]]

Here is how we declare an ArrayList:
```cs
var myArrayList = new ArrayList();
```

We can add different data types to ArrayList:
```cs
int number = 10;
string name = "Eddie";
float digits = 2.5f;

myArrayList.Add(number);
myArrayList.Add(name);
myArrayList.Add(digits);
```

To iterate thru ArrayList. We must use ``object`` synthax:
```cs
foreach (object i in myArrayList)
{
    Console.WriteLine(i);
}
```

<br>

> **Performance Penalty**\
> When using ArrayLists, boxing and unboxing will occur!. It is better to avoid it and use List<> [[QNA Boxing and Unboxing with Generics|See Why]]
{: .prompt-warning}



<br>

---
## Lists
Lists are like arrays, but they are not restricted in their sizes. We can add more objects to the list without specifiying how many objects we would store., they are dynamic in size. They are constructed from the .NET [[CS - 1 Collections and Generics|Generics]]. We use lists when we dont know ahead of time how many objects will be there in the list.

Here is how we create a list:
```cs
var myList = new list<int>();

// Using object initialize synthax
var myList2 = new list<int>() {1, 2, 3};
```


> **Generics List**\
> Fundamentally List is a .Net Generic, it works like this:
> ```cs
> var myList = new list<T>();
> ```
> Where T represents the specific template / data type that the list would store.
{: .prompt-info}


<br>

### List Class Methods
```cs
// Adds an item to the list
myList.Add();

// Adds an another list to my list or an array
myList.AddRange();

// Remove an item from the list
myList.Remove();

// Removes all items from the list
myList.RemoveAll();

// Returns the Index of the item in the list
myList.IndexOf();

// Returns bool of whether an item exist in a list
myList.Contains();

// Property. Gets the number of items in the list
Console.WriteLine(myList.Count);
```
