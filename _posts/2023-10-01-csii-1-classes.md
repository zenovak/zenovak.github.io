---
title: CS - 1 Classes
date: 2023-10-01 #TTTT Means timezone
categories: [C#, C#II-Intermediate]
tags: [c#]     # TAG names should always be lowercase
---


## A class consists of 2 parts
**Methods and fields**
Method is basically functions that are contained within a class
Fields are variables that are contained in a class

Lets take this analogy
We have a class called 'car'
- it has fields such as num of wheels, seats, colors, brandname etc.
- it has methods, such as Drive(), Turn(), etc. 

Now this car class can be said to be just a blueprint. It tells you how to make a car. An object is an instance of a class. This can be said as a materialization of a class (blueprint) into a "physical" object.

Here is how we declare a class:
```cs
public class Car
{
    // attr and methods
    public string Name;
    public int NumOfWheels;
    public int NumOfDoors;
    public string EngineType;
    // You can also create an attr list that takes in item of obj type Passangers
    public List<Passangers> passangers; 
    
    public void drive()
    {
        Console.WriteLine("Vroom Vroom");
    }
}
```

> **NOTE**\
> Classes are declared in the namespace region. 
> The naming convention for C# classes are in PascalCase.
{: .prompt-info}

<br>

## Constructors
Constructors allows automatic initiation of fields and call methods uppon the initialization of the class instance/object. It is essentially a method which is called when the object is initialised.

> **Here is how you would declare a constructor**
> 
> ```cs
> public Car()
> {
> 	// Run somecode here
> }
>```
{: .prompt-info}


As per conventions, it is often recommended to declare the constructors after attributes.
```cs
public class Car
{
    public string Name;
    public int NumOfWheels;
    public int NumOfDoors;
    public string EngineType;
    public List<Passangers> passangers; 
    
    public Car()
    {
        this.passangers = new List<Passangers>();
    }
    public void drive()
    {
        Console.WriteLine("Vroom Vroom");
    }
}
```

<br>

**Overloaded constructors**
Overloaded constructors means you declare multiple constructors, by giving args. However, in the main method. If we were to use this constructor instead, the default args-less constructor would be skipped.
```cs
public Car(string name)
{
    this.Name = name;
}
```

There for we must specify :this() at the end of the overloaded constructor so that it will call the default constructor.

```cs
public Car(string name) : this() 
{
    this.Name = name;
}
public Car(string engineType, int numofwheels) : this()
{
    this.EngineType = engineType;
    this.NumOfWheels = numofwheels;
}
```

<br>

## Methods
Methods are functions within classes. A constructor is an example of a method. 

```cs
public class Calculator
{
    // This is a default constructor. It takes no params
    public Calculator()
    {
        
    }
    public static int Add(int[] numbers)
    {
        var sum = 0;
        foreach (int number in numbers)
        {
            sum += number;
        }
        return sum;
    }
}
```

In the main(), here's how you would use the Add() method. We use ``new int[]`` because our ``Add()`` method takes an array as arguement, and we do not have an Array.
```cs
internal class Program
{
    static void Main(string[] args)
    {
        Calculator.Add(new int[] {1, 2, 3})
    }
}
```


<br>

### Methods modifiers
However there is a better way using the *params* modifier. 
```cs
// Calculator.Multiply(10, 10)
public static int Multiply(params int[] numbers)
{
    var product = numbers[0];
    for (var i = 1; i < numbers.Length; i++)
    {
        product *= numbers[i];
    }
    return product;
}
```

With this implementation, you no longer need ``new[] {x, y, z}`` in your main().
```cs
internal class Program
{
    static void Main(string[] args)
    {
        Calculator.Multiply(1, 2, 3)
    }
}
```

<br>

> **Code Smell**\
> The ref and out modifier, in Hadi's, is a smell in the design of the C# language. Please don’t use it when defining your methods.
{: .prompt-warning}


**ref modifier**
This modifier allows your method to use var parameters as a reference type instead of value.

By default, when we pass a value type (eg int, char, bool) to a method, a copy of that variable is sent to the method. So changes applied to that variable in the method will not be visible upon return from the method. This can be modified using the ref modifier. When we use the ref modifier, a reference to the original variable will be sent to the target method. 

```cs
public static void Minus(ref int a)
{
    a -= 2;
}
```

```cs
// This method can modify value type variables. 
Calculator.Minus(ref a); 
Console.WriteLine(a);
```

<br>

**out modifier**
The out modifier can be used to return multiple values from a method. Any parameter declared with the out modifier is expected to receive a value at the end of the method.
```cs
public void Weirdo(out int a)
{
    a = 1;
}

// in the main():
int a;
Weirdo(out a);
```

<br><br>

---
## Fields
A field can be initialized in two ways: In a constructor, or directly upon declaration. The benefit of initialising a field during declaration is that if your class has one or more constructors, you’ll make sure that the field will always be initialised irrespective of which constructor is going to be called.

```cs
public class Customer
{
    public List<Order> Orders = new List<Order>();
}

```

<br>

We use the **readonly** modifier to improve the robustness of our code. When a field is declared with readonly, it needs to be initialized either during declaration or in a constructor. The value cannot be changed. This prevents you from accidentally overwriting the value of a field, which can result in an unexpected state. As an example, think of the Orders in the above example. If we accidentally re-initialize this field somewhere else in the class, all the Order objects stored in the list will be lost. So we should declare it as readonly:

```cs
public class Customer
{
    public readonly List<Order> Orders = new List<Order>();
}
```

<br><br>

---
## Access Modifiers
In C# we have 5 access modifiers: **public**, **private**, **protected**, **internal** and **protected internal**.

-     A class member declared with **public** is accessible everywhere.

-     A class member declared with **private** is accessible only from inside the class.

-     We’ll learn about the other access modifiers when we get to the [[CS - 2 Inheritance|inheritance]].

We use access modifiers to hide the implementation details of a class. So anything that is about “how” a class does its job should be declared as private. This way, we make sure other parts of the code will not touch the implementation detail of a class. And as a result we improve the robustness of our code. If change the implementation of a class, we only need to make changes inside the class. No other parts of the code will need to be changed.


> **Note**\
> private fields are named in camelCase, with an underscore infront:
> ```cs
> public class Dog
> {
> 	private string _age;
> }
> ```
{: .prompt-info}

<br><br>

---
## Properties
A property is a kind of class member that is used for providing access to fields of a class.

> **Notes**\
> As a best practice, we must declare fields as private and create public properties to provide access to them.
{: .prompt-tip}

<br>

A property encapsulates a get and a set method:

```cs
public class Customer
{
    private string _name;
    public string Name
    {
        get { return _name; } 
        set { _name = value; }
    }
}
```

Inside the get/set methods we can have some logic.  If you don’t need to write any specific logic in the get or set method, it’s more efficient to create an auto-implemented property. An auto-implemented property encapsulates a private field behind the scene. So you don’t need to manually create one. The compiler creates one for you:

```cs
public class Customer
{
    public string Name { get; set; }
}
```

> **NOTE**\
> You can use a code snippet in VS code, just type `prop`
{: .prompt-info}

<br>

### **Example**
Explore the example code below for implementation details:
```cs
public class Person
{
    // Constructor
    public Person(DateTime birthday)
    {
        this.Birthday = birthday;
    }
    
    // Using custom methods to access private fields
    private string _name;
    public void SetName(string name)
    {
        if (!string.IsNullOrEmpty(name))
            this._name = name;
    }
    public string GetName()
    {
        return _name;
    }
    
    
    // Using properties
    // set is private. Use constructor to set bdate
    private DateTime _birthday;
    public DateTime Birthday
    {
        private set { _birthday = value; }  
        get { return _birthday; }
    }
    
    
    // Autoimplementation of properties. 
    public string FamilyName { get; set; }
    
    // Implementing logics in get methods requires manual properties declaration
    // Note this does not have a set method.
    public int Age
    {
        get 
        { 
            var timespan = DateTime.Today - Birthday;
            var years = timespan.Days / 360;
            return years;
        }
    }
}


internal class Program
{
    static void Main(string[] args)
    {
        // Here we see the way to use properties
        var alex = new Person(new DateTime(year: 2000, month: 10, day: 10));
        
        // Properties can be accessed without calling get or set.
        Console.WriteLine(alex.Age);
    }
}
```

<br><br>

---
## Indexers
Indexer is a special kind of property that allows accessing elements of a list by an index.

If a class has the semantics of a list, or collection, we can define an indexer property for it. This way it’s easier to get or set items in the collection.

```cs
public class HttpCookie
{
    public string this[string key]
    {
        get {}
        set {}
    }
}

//Example Implementation
// Indexers are properties inplementation for dict type fields
public class Biskuts 
    {
        private Dictionary<string, string> _biskutDetails= new Dictionary<string, string>();
        public string this[string brand]
        {
            get { return _biskutDetails[brand]; }
            set { _biskutDetails[brand] = value; }    
        }   
    }
```
