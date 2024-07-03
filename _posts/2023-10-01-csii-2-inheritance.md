---
title: CS - 2 Inheritance
date: 2023-10-01 #TTTT Means timezone
categories: [C#, C#II-Intermediate]
tags: [c#]     # TAG names should always be lowercase
---



## Access Modifiers
- Your classes should be like a black box. They should have limited visibility from the outside. The implementation, the detail, should be hidden. We use access modifiers (mostly private) to achieve this. This is referred to as Information Hiding (and sometimes Encapsulation) in object-oriented programming.

- Public: is accessible everywhere.

- Private: is accessible only from the class.

- Protected: is accessibly only from the class and its derived classes. Protected breaks encapsulation (because the implementation details of a class will leak into its derived classes) and is better to be avoided.

- Internal:  is accessibly only from the same assembly.

- Protected Internal: is accessible only from the same assembly or any derived classes. (Sounds weird? Forget it! It’s not really used.)



> **Constructors and Inheritance**\
> Constructors are not inherited and need to explicitly defined in derived class. When creating an object of a type that is part of an inheritance hierarchy, base class constructors are always executed first. 
> 
> We can use the base keyword to pass control to a base class constructor:
> ```cs
> public class Car : Vehicle 
> { 
> 	public Car(string registration) : base(registration) 
> 	{ 
> 		//code
> 	}
> }
{: .prompt-tip}

<br><br>

---
## Upcasting and Downcasting

> *Upcasting*: conversion from a derived class to a base class 
> *Downcasting*: conversion from a base class to a derived class 

See also: [[Boxing and Unboxing]]

> **Note**\
> All objects can be implicitly converted to a base class reference.
{: .prompt-info}

<br>

Lets say we have the following few classes:
```cs
public class Shape
{
    public int size { get; set; }
}
public class Circle: Shape 
{ 
}
public class Car
{
} 
```

<br>

In the main() we can see how upcasting and downcasting works:
```cs
// Upcasting. Implicit conversion can happen.
var circle = new Circle();
Shape shape = circle;

// Downcasting. Explicit cast is required.
Circle anotherCircle = (Circle) shape;
```


Below is an example of casting in action:
```cs
// We assign a class into its parent class
var circle = new Circle();
Shape shape = circle;

circle.size = 200;
shape.size = 100;
// what would happen?
Console.WriteLine(circle.size);

//Because shape object is a reference to the same circle object in memory
// circle.size would become 100.
```

<br>

### **Exceptions**
Casting can throw an exception if the conversion is not successful. We can use the ``as`` keyword to prevent this.

```cs
// If conversion is not successful, null is returned. 
Circle circle = shape as Circle; 
if (circle != null)
{
     //Run code
}
```

We can also use the ``is`` keyword:
```cs
if (shape is Circle)
{
    Console.WriteLine("Yes shape is a Circle");
    // Cast Circle to shape
}
```


> **NOTE**\
> If the cast does not have a child relations. InvalidCastException is thrown.
> ```cs
> Car car = (Car) anotherCircle;
> ```
{: .prompt-warning}

<br>

---
## Boxing and Unboxing
C# types are divided into two categories: **_value types_** and **_reference types_**. See [[Boxing and Unboxing]] for full explaination by Microsoft.

- **_Value types_** (e.g. `int`, `char`, `bool`, all primitive types and struct) are stored in the stack. 
They have a short life time and as soon as they go out of scope are removed from memory. 

- **_Reference types_** (e.g. all classes) are stored in the heap. Every object can be implicitly cast to a base class reference. 


> **Note**\
> The object class is the parent of all classes in .NET Framework. So a value type instance (eg int) can be implicitly cast to an object reference. 
{: .prompt-info}

Boxing happens when a value type instance is converted to an object reference.
Unboxing is the opposite: when an object reference is converted to a value type.
```cs
// Boxing 
object obj = 1;

// Unboxing 
int i = (int)obj;
```

> **PERFORMANCE**\
> Both boxing and unboxing come with a performance penalty. This is not noticeable when dealing with small number of objects. But if you’re dealing with several thousands or tens of thousands of objects, it’s better to avoid it.
{: .prompt-warning}

