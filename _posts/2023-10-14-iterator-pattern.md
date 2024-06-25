---
title: DP - 3 Iterator Pattern
date: 2023-10-14
categories: [DesignPatterns, DesignPatternsFundamentals]
tags: [dp, java]     # TAG names should always be lowercase
---

## About
Let's say we want to build a web browser. Every brower has the concept of history where we can visit previous websites using the back button. The problem is very similar to the momento pattern.

```java
public class BrowseHistory {
    private List<String> urls = new ArrayList<>();
    
    public void push(String url) {
        urls.add(url);
    }
    
    public String pop() {
        var lastIndex = url.size() - 1;
        var lastURL = urls.get(lastIndex);
        url.remove(lasIndex);
        
        return lastURL;
    }
}
```

Then in the main:
```java
public void main(Sting[] args) {
    
    var history = new BrowseHistory();
    history.push("website1");
    history.push("website2");    
    history.push("website3");
}
```

<br>

### The problem
Let's say we want to iterate over the entire history, and print the url we have visited. Currently the `List<String> url` is set to private.

If we were to expose this as a method for acess by creating a getter:
```java
    ...
    private List<String> urls = new ArrayList<>();
    
    public List<String> getUrls() {
        return this.urls;
    }
    ...
```

Then in main:
```java
public void main(Sting[] args) {
    
    var history = new BrowseHistory();
    history.push("website1");
    history.push("website2");    
    history.push("website3");
    
    for (var i = 0; i < history.getUrls().size(); i++) {
        System.out.println(history.getUrls()[i]);
    }
}
```

<br>

However, we have a problem. If we were to change the implementation of our `urls` to some other data structure like a stack or linked list, our main method will break.

```java
    ...
    private String[] urls = new String[10];
    
    // Broken change
    public List<String> getUrls() {
        return this.urls;
    }
    ...

// inside main, broken change, String[] does not have size() and get()
for (var i = 0; i < history.getUrls().size(); i++) {
        System.out.println(history.getUrls().get(i));
    }
```

This is a bad practice for larger applications. Everytime we make a change to the implementation of our class, we introduce breaking changes in other places.

<br><br>

---
## Design Pattern - Solution

> We should always encapsulate implementation details inside our class, and dont allow implementation changes to affect public interfaces. 

<br>

### The naive
We need to introducing iteration friendly methods: `current()`, `hasNext()`, and `next()`

```
 _______________
| BrowseHistory |
|---------------|
| push(url)     |
| pop()         |
| next()        |
| current()     |
| hasNext()     |
|_______________|
```

So we can write iteration code such as the following:
```java
while (history.hasNext()) {
    var current = history.current();
    // Do soemthing with current
    
    history.next()
}
```

<br>

### The Expert
However, this implementation breaks the single responsibility principle. The BrowserHistory class is doing too much, it is managing the history and also iterating. We need to split it into 2 classes:

```
 _______________          ___________
| BrowseHistory |        | Iterator  |
|---------------|        |-----------| 
| push(url)     |        | next()    |
| pop()         |        | current() |
|_______________|        | hasNext() |
                         |___________|
```

If the `BrowseHistory` class decides to use a new datastructrure internally, then different types of iterator will be required and will need to be derrived. So an iterator should be an interface.

```
                                            INTERFACE
 __________________                        ___________
| BrowseHistory    |                      | Iterator  |
|------------------|                      |-----------| 
| push(url)        |      Dependency      | next()    |
| pop()            |--------------------->| current() |
| createIterator() |                      | hasNext() |
|__________________|                      |___________|
                                                ^
                                                |
                                        ____________________
                                       |                    |
                                _______|______        ______|________
                               | ListIterator |      | ArrayIterator |
                               |--------------|      |---------------|
                               |______________|      |_______________|
```

We introduce a `createInterator()` method which returns an `Iterator` object, so the **clients** of the `BrowseHistory` class will have no connections to the concrete implementations of the Iterator interface. 

If we change the data structure of `BrowseHistory`, we can simply change the implementation that the `createIterator()` method returns.

```java
// Option1: Returns an ArrayIterator
public Iterator createIterator(){
    
    ...
    return arrayIterator;
}
```

```java
// Option2: Returns a ListIterator
public Iterator createIterator() {
    
    ...
    return listIterator;
}
```

This does not break the main method.

<br><br>

---
# Implemetation
We can create a generic interface, and an implementation  of this class: 


```java
public interface Iterator<T> {
    public void next();
    public T current();
    public boolean hasNext();
}
```

Then we implement a generic implementation of a ListIterator
```java
public class ListIterator<T> implements Iterator {
    
    private List<T> data;
    
    private int currentIndex;
    
    public ListIterator<T>(List<T> list) {
        this.data = list;
        this.currentIndex = 0;
    }
    
    public void next() {
        currentIndex += 1;
    }
    public T current() {
        return this.data.get(this.currentIndex);
    }
    public boolean hasNext() {
        return (currentIndex < data.size());
    }
}
```

Then we can use them in the BrowserHistory class. 
```java
public class BrowserHistory {
    private List<String> urls = new List<String>();
    
    public void push(String url) {
        urls.add(url);
    }
    
    public String pop() {
        var item = urls.get(urls.size() - 1);
        urls.remove(urls.size() -1);
        
        return item;
    }
    
    public Iterator createIterator() {
        return new ListIterator<String>(this.urls);
    }
}
```

In the main method, we are simply programming towards the iterator interface. 
```java
public void main(Sting[] args) {
    
    var history = new BrowseHistory();
    history.push("website1");
    history.push("website2");    
    history.push("website3");
    
    Iterator iterator = history.createIterator();
    while (iterator.hasNext()) {
        var url = iterator.current();
        System.out.println(url);
        iterator.next();
    }
}
```

<br>

> **Exercise**\
> Try changing the implementation detail of the `BrowseHistory` class by changing the ulrs list into a fixed size array.
> You will see this does not introduce a breaking change inside the main method.
{: .prompt-tip}

<br><br>

---
## Recap
What does the Iterator Pattern try to solve?

The Iterator patterns allows a data structure class to better encapsulate its internal iteration mechanism without introducing breaking changes to its clients. This is done by introducing an `Iterator` interface
