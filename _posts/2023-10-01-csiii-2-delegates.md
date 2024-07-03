---
title: CS - 2 Delegates
date: 2023-10-01 #TTTT Means timezone
categories: [C#, C#III-Advanced]
tags: [c#]     # TAG names should always be lowercase
---


DELEGATES
- an object which calls other methods
- a reference to a function/METHOD

> WHY DO WE NEED THEM?
> For designing extensible and flexible applicaitons (eg. frameworks). Its usage is similar to interfaces

> **Interface or Delegates?**\
> Use a delegate when:
> - an eventing design pattern is used
> - the caller doesnt need to access other properties or methods on the object implementing the method
{: .prompt-tip}

We can think of a delegate as a reference to a function or like an interface but for methods, and use them as such. Let's say we want a function that applies an effect on photos like a photo filter, but we leave the implementation details out of the class, and only in the main() where we will specify which method we should use as our implementation. Therefore methods that implements the delegates should have the same arguements types and return types. 


<br>

---
## The Problem
Everything here in `PhotoProcessorV1` works. This can be considered a type of stored procedure class where the filter processes are executed. But if this is a framework. We may want to add our own methods to the filter, this class would need redeployment and thus not extensible.

````cs
public class PhotoProcessorV1
{	
    public void Process(string path)
    {
        var photo = Photo.Load(path);
        var filter = new Filters();
        filter.Brightness(photo);
        filter.Contrast(photo);
        filter.Resize(photo);
    }
}
````

<br>

### Now we use a delegate. 
The `PhotoProcessorV2` takes an additional arguement "delegate". This removes the need of redeployment of the class, when adding additional filters to the Process method.

``````cs
public delegate void PhotoFilterHandler(Photo photo);
public class PhotoProcessorV2
{
    public void Process(string path, PhotoFilterHandler filterHandler)
    {
        var photo = Photo.Load(path);
        filterHandler(photo);
    }
}
``````

<br>

### The Main method
`PhotoProcessorV1` is used as it is in an instance, and all the stored procedure is hard coded into the class. 

`PhotoProcessorV2` implements the delegate. `PhotoFilterHandler` is declared and an instance of the filter method is added to the filterHandler. You can add as many filters to this handler. With the use of method extensions on the filters class, only the Main would need redeployment.

````cs
internal class Program
{
    static void Main(string[] args)
    {
        // Version 1
        var photprocess1 = new PhotoProcessorV1();
        photprocess1.Process("path");
        
        // Version 2 with delegates
        var photprocess2 = new PhotoProcessorV2();
        var filters = new Filters();
        PhotoFilterHandler filterHandler = filters.Brightness;
        photprocess2.Process("path", filterHandler);
        
        // Adding extra items to the delegate to run more filters
        filterHandler += filters.Resize;                                    
        photprocess2.Process("path", filterHandler);
        
        // Version 3 with .NET Generic delegates
        var photprocess3 = new PhotoProcessorV3();
        var filters2 = new Filters();
        Action<Photo> filterHandler2 = filters2.Brightness;
        photprocess3.Process("path", filterHandler2);
        
        // Adding extra items to the delegate to run more filters
        filterHandler2 += filters.Resize;                                    
        photprocess3.Process("path", filterHandler2);
    }
}
````
<br>

---
## Extras

> **Generic Delegates**\
> In C# we have generic delegates
> ```
> // This is used for void methods
> System.Action<T>
>
> // This is used for methods with returns
> System.Func<Targ, Treturn>
> System.Func<Targ1, Targ2, Treturn>
> ....
> ```
{: .prompt-tip}

^fa837d

<br>

Example of generic delegate implementation
````cs
public class PhotoProcessorV3
{
    public void Process(string path, Action<Photo> filterhander)	
    {
        var photo = Photo.Load(path);
        filterhander(photo);
    }
}


internal class Program
{
    static void Main(string[] args)
    {
        // Version 3 with .NET Generic delegates
        var photprocess3 = new PhotoProcessorV3();
        var filters2 = new Filters();
        Action<Photo> filterHandler2 = filters2.Brightness;
        photprocess3.Process("path", filterHandler2);
        
        // Adding extra items to the delegate to run more filters
        filterHandler2 += filters.Resize;                                    
        photprocess3.Process("path", filterHandler2);
    }
}
````

<br.<br>

---
## Anonymous Methods

Below is a simple demonstration of using Anonymous methods:
```cs
// Delegate must be declared in class scope.
public delegate bool lessThanDelegate(int num);

//in main()
static void DisplayResults(List<int> listofNum, lessThanDelegate filter)
{
    foreach (var i in listofNum)
    {
        if(filter(i))
        {
            Console.WriteLine(i);
        }
    }
}


// We declare our anon method:
lessThanDelegate filter5 = delegate(int num)
{
    return num < 5;
}

// Using it...
var myList = new List<int>() {1, 2, 6, 7, 8};
DisplayResults(myList, filter5);
```

<br><br>

---
## Appendix
Classes used in this example

This here is the Filters Class. Leaving out implementation details. If we were to add new filters. We would recompile this class or use extensions.
````cs
public class Filters
    {
        public void Brightness(Photo photo)
        {
            Console.WriteLine("Applying brigthness");
        }
        public void Contrast(Photo photo)
        {
            Console.WriteLine("Applying contrast");
        }
        public void Resize(Photo photo)
        {
            Console.WriteLine("Resizing");
        }
    }
````

```cs
public class Photo
    {
        public static Photo Load(string path)
        {
            return new Photo();
        }
    }
```


<br>

Here's another example of using delegtes
```cs
namespace Exercise_2___Delegates
{
    internal class Program
    {
        //Delegate declaration
        public delegate float OperationDelegate(float x, float y);
        
        
        static void Main(string[] args)
        {
            
            // Opeation methods
            static float ApplyOperation(float x, float y, OperationDelegate operation)
            {
                return operation(x, y);
            }
            
            
            
            // Implementation methods This uses the delegates
            static float Add(float x, float y)
            {
                return x + y;
            }
            static float Subtract(float x, float y)
            {
                return x - y;
            }
            static float Multiply(float x, float y)
            {
                return x * y;
            }
            static float Divide(float x, float y)
            {
                return x / y;
            }
        }
    }
}

```