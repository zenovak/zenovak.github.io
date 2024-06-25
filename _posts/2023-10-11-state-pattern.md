---
title: DP - 2 State Pattern
date: 2023-10-11
categories: [DesignPatterns, DesignPatternsFundamentals]
tags: [dp, java]     # TAG names should always be lowercase
---


> Reference, Code with mosh

## About - The problem
Let's say we want to build a drawing application like photoshop. So we can have a canvas and a dashboard of tools. We can interact with the canvas and the behavior of the canvas changes depending on the tools that we have selected. 

This canvas object is listening on mouse events like mouse up and mouse down, and what it does is dependant on the selected tools.

We might see pattern like this in our code:
```java
public class State {
    
}

public class Canvas {
    private ToolType currentTool;
    public void mouseDown() {
        if (currentTool == ToolType.Selection)
            Console.WriteLine("Selection icon");
        else if (currentTool == ToolType.Brush)
            Console.WriteLine("Brush icon");
        else if (currentTool == ToolType.Eraser)
            Console.WriteLine("Eraser icon")
    }
    
    public void mouseUp() {
        if (currentTool == ToolType.Selection)
            Console.WriteLine("Draw dashed rectangle");
        else if (currentTool == ToolType.Brush)
            Console.WriteLine("Draw lines");
        else if (currentTool == ToolType.Eraser)
            Console.WriteLine("Erase something")
    }
    
    public ToolType getCurrentTool() {
        return this.currentTool;
    }
    
    public void setCurrentTool(ToolType tool) {
        this.currentTool = tool;
    }
}

public enum ToolType {
    Selection,
    Brush,
    Eraser
}
```

At first it might seem like an easy fix to just use a switch case statement. However the problem is deeper than that. This solution is not extensible, if you decided to add new tools in the future you'll have to change `mouseUp()` and `mouseDown()`. And in real world applications, we have to also listen to keyboard inputs as well. Which means we must also implement these behavior inside: `keyA_pressed()`, `keyX_pressed()` -- very unextensible.

<br><br>

---
## Solution

![](/assets/img/attachments/dp/dp%20state%20pattern%20uml.png)

Now, let's think a bit deeper. The canvas's methods implementation changes depending on the selected tool. There are 4 pillars of object oriented programming:
- Encapsulation
- Abstraction
- Inheritance
- Polymorphism

In our case it is obvious that we need a solution that uses polymorphism!

```java
public abstract class UIControl {
    public void enable() {
        Console.WriteLine("Enabled");
    }
    
    public abstract void draw();
}
```

So an implementation of the UIControl class:
```java
public class TextBox extends UIControl {
    public override void draw() {
        Console.WriteLine("Drawing a textbox");
    }
}
```

and in the main method:
```java
public class Main {
    
    public static void main(String args[]) {
        drawUIControl(new TextBox());
    }
    private static void drawUIControl(UIControl control) {
        control.draw();
    }
}
```

<br>

### UML
Essentially we would have this pattern in UML:
```

 ______                            __________________
| Main |   Dependency Injection   |    UIControl     |
|------|------------------------->|------------------|
|      |                          |     draw()       |
|______|                          |__________________|
                                          ^
                                          |
                            Inheritance   |
                               ___________________________
                         _____|_______            ________|______
                        |   TextBox   |          |    CheckBox   |
                        |-------------|          |---------------|
                        |             |          |               |
                        |-------------|          |---------------|
                        | draw()      |          | draw()        |
                        |_____________|          |_______________|
```

<br><br>

---
## Implementation

We need an abstract class to represent the different type of tools
```java
public abstract class Tool {
    public abstract void mouseDown();
    public abstract void mouseUp();
}
```

And then our tools that implement this abstract class:
```java
public class BrushTool extends Tool {
    @Override
    public void mouseDown() {
        System.out.println("Brush icon");
    }
    
    @Override
    public void mouseUp() {
        System.out.println("Draw a line");
    }
}


public class SelectionTool extends Tool {
    @Override
    public void mouseDown() {
        System.out.println("Selection icon");
    }
    
    @Override
    public void mouseUp() {
        System.out.println("Draw a dashed rectangle");
    }
}


public class EraserTool extends Tool {
    @Override
    public void mouseDown() {
        System.out.println("Eraser icon");
    }
    
    @Override
    public void mouseUp() {
        System.out.println("Erase something");
    }
}
```

<br>

So we can use this inside the Canvas:
```java
public class Canvas {
    private Tool currentTool;
    
    public void mouseDown() {
        currentTool.mouseDown();
    }
    
    public void mouseUp() {
        currentTool.mouseUp();
    }
    
    public Tool getCurrentTool() {
        return this.currentTool;
    }
    
    public void setCurrentTool(Tool tool) {
        this.currentTool = tool;
    }
}
```

and in the main class where it all fits together:
```java
public class Main {
    public static void main(String[] args) {
        var canvas = new Canvas();
        canvas.setCurrentTool(new BrushTool()); 
        canvas.mouseDown();
        canvas.mouseUp();
    }
}
```

<br><br>

---
## Design smells -- Abusing patterns
Design patterns are great, they help us build extensible and reusable objects. Here's an example:

> John refactors some of the existing codebase to apply new patterns. Every pattern has a context it is designed to solve specific patterns. Blindly applying patterns to a code base introduces design smells.
> 
> It creates an overly complicated design. Keep things simplky and pragmatic. Dont overly use patterns. 

<br><br>

## State pattern abuse
Let's take this simple stopwatch example here. The stopwatch behavior changes depending on its current state. If it is running then it stops, if it is stopped, then it runs.

```java
public class Stopwatch {
    private boolean isRunning;
    
    public void click() {
        if (isRunning) {
            isRunning = false;
            System.out.println("Stopped");
        }
        else {
            isRunning = true;
            System.out.println("Running");
        }
    }
}
```

<br>

Here's an example of implementing this stopwatch with a state pattern abuse. We declare the interface / abstract class that represents the state of the stopwatch. 

```java
public interface State {
    void click();
}

public class RunningState implements State{
    public void click() {
        System.out.println("Stopped");
    }
}

public class StoppedState implements State {
    public void click() {
        System.out.println("Runnning");
    }
}
```

But we are unable to change the state of the stopwatch in this design. To be able to access the stop watch state, we must add a constructor that hold a reference to the stopwatch, and pass the reference around:
```java
public class RunningState implements State{
    private Stopwatch stopwatch;
    
    public RunningState(Stopwatch stopwatch) {
        this.stopwatch = stopwach
    }
    @Override
    public void click() {
        stopwatch.setCurrentState(new StoppedState(this.stopwatch));
        System.out.println("Stopped");
    }
}

public class StoppedState implements State{
    private Stopwatch stopwatch;
    
    public RunningState(Stopwatch stopwatch) {
        this.stopwatch = stopwach
    }
    @Override
    public void click() {
        stopwatch.setCurrentState(new RunningState(this.stopwatch));
        System.out.println("Running");
    }
}
```


Then in the stopwatch class, it has become less bloated. But at the great expense of increased moving parts with interfaces and 2 new implementation classes. 
```java
public class Stopwatch {
    private State state = new StoppedState(this);
    
    public void click() {
        state.click();
    }
    
    public State getState() {
        return this.state;
    }
    
    public void setState(State state) {
        this.state = state;
    }
}
```

<br><br>

---
## Dont abuse design patterns

You can see how much complexity has been added to the stopwatch, compared to a simple if else that we did prior. This is counter productive. 

The state pattern is used in cases when extensibility is required where a class may have many states, and multiple method's implementation that changes depending on the state. The stopwatch only ever have 2 states, start and stop. There is no need for such complexity.

Keep in mind when using state pattern. Does the class requires multiple states, are there a lot of methods that the class implements that differs due to its states?
