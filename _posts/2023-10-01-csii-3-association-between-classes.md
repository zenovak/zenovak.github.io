---
title: CS - 3 Association Between Classes
date: 2023-10-01 #TTTT Means timezone
categories: [C#, C#II-Intermediate]
tags: [c#]     # TAG names should always be lowercase
---


> Pulled from: Mosh Hamedani\
> [Programming with Mosh - Learn the Skills to Land Your Dream Job](https://programmingwithmosh.com/)

<br>

---

## Class Coupling 
A measure of how interconnected classes and subsystems are. 
The more coupled classes, the harder it is to change them. A change in one class may affect many other classes. 
Loosely coupled software, as opposed to tightly coupled software, is easier to change. 

Two types of relationships between classes: Inheritance and Composition.

<br>

### Inheritance
A kind of relationship between two classes that allows one to inherit code from the other.
- Referred to as Is-A relationship: A Car is a Vehicle 
- Benefits: code re-use and polymorphic behaviour.

```cs
public class Car : Vehicle 
{ 
    // Members
}
```

<br>

### Composition 
A kind of relationship that allows one class to contain another. 
- Referred to as Has-A relationship: A Car has an Engine 1 
- Benefits: Code re-use, flexibility and a means to loose-coupling

```cs
// Base
public class Animal
{
    public int Age { get; set; }

    public void Eat() 
    {
        Console.WriteLine("Nom Nom");
    }
    public void Breath() 
    {
        Console.WriteLine("Snore snore");
    }
}

// derived with composition
public class Dog
{
    private readonly Animal _Animal;
    
    public Dog()
    {
    
    }
    public Dog(Animal animal)
    {
        _Animal = animal;
    }
    
    public void Eat() 
    {
        _Animal.Eat();
    }
    public void Breath() 
    { 
        _Animal.Breath();
    }
}
```

<br><br>

---
## Favour Composition over Inheritance 
- Problems with inheritance: 
    - Easily abused by amateur designers / developers 
    - Leads to large complex hierarchies 
    - Such hierarchies are very fragile and a change may affect many classes 
    - Results in tight coupling 
<br>

- Benefits of composition: 
    - Flexible
    - Leads to loose coupling
<br>
- Having said all that, it doesn’t mean inheritance should be avoided at all times. In fact, it’s great to use inheritance when dealing with very stable classes on top of small hierarchies. As the hierarchy grows (or variations of classes increase), the hierarchy, however, becomes fragile. And that’s where composition can give you a better design.

<br><br>

---
## Composition variants:

Lets look at 2 slightly different means of composition:
```cs
// Default variant
public class Dog
{
    private readonly Animal _Animal;
    
    public Dog()
    {
    
    }
    public Dog(Animal animal)
    {
        _Animal = animal;
    }
    
    public void Eat() 
    {
        _Animal.Eat();
    }
    public void Breath() 
    { 
        _Animal.Breath();
    }
}
```

<br>

```cs
// Variant 2
public class Dog 
{ 
    private readonly Animal _Animal; 
    
    public Dog() 
    { 
        _Animal = new Animal(); 
    } 
    
    public void Eat() 
    { 
        _Animal.Eat(); 
    } 
    
    public void Breath() 
    { 
        _Animal.Breath(); 
    } 
}
```


In the first example, the `Dog` class has a "has-a" relationship with the `Animal` class. This means that a `Dog` object has an instance of the `Animal` class as a member variable. The `Dog` class relies on the `Animal` class to provide some of its functionality, specifically the `Eat()` and `Breath()` methods, which are called through the `_Animal` member variable.

In the second example, the `Dog` class creates a new instance of the `Animal` class within its constructor. This means that a `Dog` object has its own `Animal` object as a member variable, rather than relying on an existing `Animal` object that was passed to the constructor.

The main difference between these two approaches is in how the `Animal` object is created and managed. In the first example, the `Animal` object is created externally and passed to the `Dog` constructor, which allows for more flexibility in how the `Dog` object is created and used. For example, you could create multiple `Dog` objects that all share the same `Animal` object. This can be useful in some situations, but it can also make the code more complex.

In the second example, the `Animal` object is created internally by the `Dog` constructor, which simplifies the code somewhat. However, this approach is less flexible, since each `Dog` object has its own `Animal` object and there is no way to share an `Animal` object between multiple `Dog` objects.

So, which approach to use depends on the specific needs of your program. If you need more flexibility and want to be able to share an `Animal` object between multiple `Dog` objects, then the first approach is probably better. If you just need a simple `Dog` object that always has its own `Animal` object, then the second approach may be sufficient.

