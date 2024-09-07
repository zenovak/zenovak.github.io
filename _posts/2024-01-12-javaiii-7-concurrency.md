---
title: Java - 7 Concurrency and Multithreadding
date: 2024-01-12 #TTTT Means timezone
categories: [Java, JavaIII-Advanced]
tags: [java]     # TAG names should always be lowercase
---



## About
Most computers these days have multiple cores, which allows multiple instructions to be carried out simultaneously. Nowadays it is an essential skill to master, and untilize this advantage in parallel computing for performance. 

<br>

### Process and Thread
![](/assets/img/attachments/java/concurrency%20process%20and%20thread.png)

A **process** is an instance of an application or program. When you run a program, a copy of the program is loaded into a process, which contains some memory, instructions. Our operating system can run multiple processes that the same time. 

For example your antivirus software can run in the background, while you stream music on the browser. This is concurrency at the *process level*. 

![](/assets/img/attachments/java/concurrency%20process.png)

<br>

### Threads
We can also have concurrency within a process, which is called a thread. Each Process has at least 1 thread called the ***main*** thread. Utilizing multiple threads to run instructions simultaneously is called multithreading.

Applications that utilizes multipole threads are called multithreaded applications. 

We can see threading in action by these following:
```java
public static void main(String[] args) {
    // The number of active threads
    System.out.println(Thread.activeCount());
    
    // The number of maximum available threads
    System.out.println(Runtime.getRuntime().availableProcessors());
}
```

We usually get the following response. 
```
2
8

Process finished with exit copde 0
```

Any barebones stock java program will have 2 threads: The *main* thread and a *background* thread which handles garbage collection. The threads available are hardware dependant. A 4 core 8 thread CPU will have 8 threads. 

Nowadays CPUs usually have 6 cores, which makes 12 threads. To exploit parallel hardware we need to learn how to use threads:
- How to create and start new threads
- How to safely share data between threads

<br><br>

---
## Starting new threads

To create a new thread in java we need to create a thread object:
```java
public void main(String[] args) {
    Thread thread = new Thread(...);
}
```

A Thread object constructor is overloaded with additional arguements. 
```java
Thread(Runnable target);

Thread(@Nullable ThreadGroup group, Runnable target);

Thread(@NotNull String name);

Thread(@Nullable ThreadGroup group, @NotNull String name);

Thread(Runnable target, @NonNIs @NotNull String name);

Thread(@Nullable ThreadGroup group, Runnable target, @NotNull String name);

Thread(@Nullable ThreadGroup group, Runnable target, @NotNull String name, long stackSize);

Thread(@Nullable ThreadGroup group, Runnable target, @NotNull String name, long stackSize, boolean inheritThreadLocals);
```

<br>

## Runnable
The most commonly used constructor is `Runnable target` that implements the `Runnable` interface. It is a very simple interface with 1 method:
```java
public interface Runnable {
    void run();
}
```

Let's say we want to implement a concurrent method for downloading multiple files. We can declare a class that implements the `Runnable` interface.

```java
public class DownloadFileTask implements Runnable {
    @Override
    public void run() {
        // some implementation mechanics
        System.out.println("Downloading");
    }
}
```

<br>

## Using the new Runnable class:
Once we declare our runnable class, we can use them inside our main method:
```java
public void main(String[] args) {
    Thread thread = new Thread(new DownloadFileTask());
    thread.start();
}
```

We can say that `Runnable` objects are essentially single methods that can run on a separate thread. In some more complex design pattern use-cases, `Runnable` objects can be set to run on the main thread.

An equivalent to a `Runnable` is to think of them as a task object. A task can be done by any thread. It does not have a return value, so a task have to work on an object via reference.

On the lowest level, a thread is what's used to run a `Runnable` task. 

<br>

> **Thread lifecycle**\
> Threads closes or dies automatically when they finish a task. We do not need to explicitly close them.
{: .prompt-info}

<br><br>

---
## Useful methods for threads

### Common Debug methods:
The thread class provides useful static methods that allows us to debug and diagnose threads: 
```java
// Receives the current thread reference
var thread = Thread.currentThread();

// name of current thread.
thread.getName();
```

Together we can see the different threads being activated:
```java
public class DownloadFileTask implements Runnable {
    @Override
    public void run() {
        // some implementation mechanics
        System.out.println("Downloading " + Thread.currentThread().getName());
    }
}

...
// Inside a main method:
public void main(String[] args) {
    System.out.println("Main " + Thread.currentThread().getName());
    Thread thread = new Thread(new DownloadFileTask);
    thread.start();
}
```

We get the following output:
```
Main main
Downloading Thread-0

Process finished with exit code 0
```

<br>

## Pausing a thread
Threads allow us to pause an execution to stimulate certain situations that requires wait time such as downloading a file or querying a database.

```java
public class DownloadFileTask implements Runnable {
    @Override
    public void run() {
        // some implementation mechanics
        System.out.println("Downloading");
        
        // Pausing a thread (5000 mili-seconds)
        Thread.sleep(5000);
        
        System.out.println("Download complete");
    }
}
```

However, having a raw `Thread.sleep()` makes the compiler unhappy as this is an `Unhandled exception: java.lang.InterruptedException`. So we must always surround a thread sleep with a try catch block:

```java
...
try {
    Thread.sleep(5000);
} catch (InterruptedException e){
    e.printStackTrace();
}
```

<br><br>

---
## Thread scheduler

> What happens if I download 100 files concurrently? My machine only has 4 cores, 8 threads how's that gonna work?


The java virtual machine has something called the **Thread Scheduler**. The thread scheduler decides what threads to run and how long. If we have more tasks than the available hardware threads, the scheduler switches between tasks giving them each a slice of the CPU time.

This switching happens very fast giving the illusion of multiple tasks being handled simultaneously. This is parallelism at the software level.

<br><br>

---
## Joining a thread
Joining a thread can be said as running a task after a task, back to back. This is required in situations where 1 task needs to be finished before the next can proceed.

We do this by calling the `join()` method on a thread instance. However, doing this inside the main thread will cause it to wait for our download thread to finish. This makes the application unresponsive.
```java
public void main(String[] args) {
    Thread thread = new Thread(new DownloadFileTask());
    thread.start();
    
    try {
        thread.join();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    
    // Now the next thread can start
}
```


> **Note**\
> Its important to note that the join method is blocking method. Once our main method hits the join, execution will pause.
> 
> If you have multiple threads launched (from a loop), consider adding each thread into a list to keep their references for joining later on.
{: .prompt-tip}

<br><br>

---
## Interrupting a thread
Interrupting a thread means stopping a thread while it is running. The way thread interacting works is via event/call-backs. 


We make a thread stops by calling its instance `.interrupt()` method. 
```java
public class DownloadFileTask implements Runnable {
    @Override
    public void run() {
        // some implementation mechanics
        for (int i = 0; i < Integer.MAX_VALUE; i++)
            System.out.println("Downloading "+ (String) i );
        
        System.out.println("Download complete");
    }
}


public void main(String[] args) {
    Thread thread = new Thread(new DownloadFileTask());
    thread.start();
    
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    
    thread.interrupt();
}
```

However, this does not work. Thread interruptions works on requests. We must handle by performing interruption checks. See below.

<br>

Furthermore, if a thread is asleep. Sending an `interrupt` request will throw an exception. Which is why we must always handle such occasions with the try catch block:
```java
public class DownloadFileTask implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < Integer.MAX_VALUE; i++)
            // Check if its being interrupted
            if (Thread.currentThread().isInterrupted())
                return;
            
            System.out.println("Downloading "+ (String) i );
        
        System.out.println("Download complete");
    }
}


public void main(String[] args) {
    Thread thread = new Thread(new DownloadFileTask());
    thread.start();
    
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    
    thread.interrupt();
}
```
