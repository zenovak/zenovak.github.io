---
title: CS - 4 Polymorphism
date: 2023-10-01 #TTTT Means timezone
categories: [C#, C# II - Intermediate]
tags: [c#]     # TAG names should always be lowercase
---


> Pulled from: Mosh Hamedan

## Method Overriding

^84042a

Method overriding means changing the implementation of an inherited method. 
If we declare a method as virtual in the base class, we can override it in a derived class.
this is one of the polymorphic features.


In c# we override inherited methods using the virtual and override keyword:
```cs
public class Shape
{
    public virtual void Draw()
    {
        // Default implementation
        Console.WriteLine("I draw a box");
    }
    public void outline()
    {
        Console.WriteLine("I draw outline");
    }
}

public class Circle : Shape
{
    public override void Draw()
    {
        // new implamentation
        Console.WriteLine("I draw a circle");
    }
}
```


> **NOTE**\
> This technique allows us to achieve polymorphism. Polymorphism is a great object-oriented technique that allows use get rid of long procedural switch statements (or conditionals)
{: .prompt-info}

<br><br>

---
## Abstract Classes and Members
Remember this example of using method overrides?

We can declare a class and its members as abstract instead meaning it has no implementation and let its child classes do the implementation. It is designed specificaly for polymorphic behavior


> **Note**\
> - Abstract modifier states that a class or a member misses implementation. We use abstract members when it doesn’t make sense to implement them in a base class. 
> - For example, the concept of drawing a shape is too abstract. We don’t know how to draw a shape. This needs to be implemented in the derived classes. 
> - When a class member is declared as abstract, that class needs to be declared as abstract as well. That means that class is not complete. In derived classes, we need to override the abstract members in the base class
{: .prompt-info}

<br>

Here is how we declare an abstract class:
```cs
public abstract class Shape
{
    // This method doesnt have a body
    public abstract void Draw();
}

public class Circle : Shape
{
    public override void Draw()
    {
        // The implimentations...
        Console.WriteLine("Drawing a circle");
    }
}
```

<br>

> **WARNING**\
> All members declared as abstract in the classes must be override in the child classes, otherwise that derived class is going to be abstract too.\
> Abstract classes **cannot** be instancelized. It will not compile.
{: .prompt-warning}

<br><br>

---
## Sealed Classes and Members

> when declared on the class - Prevents derivation of _child class_
> when declared on the methods - Prevents overriding of the _methods_.


> **Note**\
> The string class is declared as sealed, and that’s why we cannot inherit from it
{: .prompt-tip}

<br><br>

---
## String class method overrides?
In the string class we can override one useful method `ToString()`. 

Whenever we use this method on class objects, or event built in non-primitive types like Lists It always returns Nonsense. However for our custom classes, we can override it:

Now this object's ToString method would return its fields, intead of nonsense.
```cs
public class Coords
{
    float x;
    float y;
    float z;
    
    public Coords(float _X, float _Y)
    {
        x = _X;
        y = _Y;
        z = 0;
    }
    
    public override string ToString()
    {
        return x + ", " + y ", " + z;
    }
}
```
