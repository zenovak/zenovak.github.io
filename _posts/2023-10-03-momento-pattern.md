---
title: DP - 1 Momento Pattern
date: 2023-10-03
categories: [DesignPatterns, DesignPatternsFundamentals]
tags: [dp, java]     # TAG names should always be lowercase
---



## About the problem
The momento pattern is one of the most fundamental patterns in design. It is used to implement the undo mechanism. 

Imagine this scenario, we have an editor class
```java
public class Editor {
    private String content;
    
    public String getContent() {
        return content;
    }
    
    public void setContent(String content) {
        this.content = content;
    }
}
```

and we want to implement an undo method which will reset the content:
```java
public static void main(String[] args) {
    var editor = new Editor();
    editor.setContent("a");
    editor.setContent("b");
    editor.setContent("c");
    editor.undo(); // *
}
```

<br><br>

---
## Design Pattern - Solution
A straight forward solution would be something like this:
```java
public class Editor {
    private String content;
    private String prevContent;
    
    public String getContent() {
        return content;
    }
    
    public void setContent(String content) {
        this.prevContent = this.content;
        this.content = content;
    }
    
    public void undo() {
        this.content = this.prevContent
    }
}
```

<br>

### The naive
However, this have a weakness, we cant undo more than once. To do this, we would have to implement a list instead:

```
 __________________
|    Editor        |
|------------------|
|content: string   |
|prevContent: List |
|__________________|
```

However if we were to add an additional fields that we need to track, this would quickly become unscalable:

```
 __________________
|    Editor        |
|------------------|
|content: string   |
|prevContent: List |
|                  |
|title: string     |
|prevTtitle: List  |
|__________________|
```

<br>

### The expert
A better way to implement this is to create a separate `EditorState` class that can be used to track these many fields of changes, which makes it more scalable:

```
 __________________                       ____________________
|      Editor      |                     |     EditorState    |
|------------------|    Composition      |--------------------|
|content: string   |#===================>|content: string     |
|prevStates: List  |                     |title: string       |
|                  |                     |____________________|
|__________________|
```

This is a better solution as we can undo multiple times, and we are not polluting the editor class with a bunch of fields and methods. However this implementation **breaks** a fundamental principle of OOP ***Single Responsibility Principle***.

<br><br>

---
## Single Responsibility
To build maintainable software we should design our classes to have single responsibility. Our expert design isnt good enough as the Editor class is doing too much work: 
1. State management
2. Providing features for the editor

```uml
 __________________                           ____________________
|      Editor      |                         |     EditorState    |
|------------------|        Composition      |--------------------|
|content: string   |#=======================>|content: string     |
|prevStates: List  |                         |title: string       |
|                  |                         |____________________|
|__________________|
```



<br>

We need to take all these state management and put it outside. Our editor should not have to keep a list of previous states, so it does not have coupled relationship with `EditorStates`. We need to introduce a new class called `HJistory` which will keep track of all these changes in the state for us.

```
 __________________                           ____________________
|      Editor      |                         |     EditorState    |
|------------------|                         |--------------------|
| content: string  |                         | content: string    |
|------------------|                         | title: string      |
|                  |                         |____________________|
|                  |                                  ^
|__________________|                                  | Composition
                                                      |
                                              ________#_________
                                             |     History      |
                                             |------------------|
                                             | EditorSate: list |
                                             |------------------|
                                             | push(state)      |
                                             | pop()            |
                                             |__________________|
```

<br>

With this implementation we now see that our `editor` class would still have a relationship on the `EditorState` class. It is a dependency pattern now. We do not store the states but we would use the state to initialize our editor. 

```uml
 __________________                           ____________________
|      Editor      |                         |     EditorState    |
|------------------|       Dependency        |--------------------|
|content: string   |------------------------>|content: string     |
|------------------|                         |title: string       |
|createState()     |                         |____________________|
|restore(state)    |                                  ^
|__________________|                                  | Composition
                                                      |
                                              ________#_________
                                             |     History      |
                                             |------------------|
                                             | EditorState: list|
                                             |------------------|
                                             | push(state)      |
                                             | pop()            |
                                             |__________________|
```

<br><br>

---
## Implementation

The editor state would be a purely data class which has no logical methods. We use the `final` keyword to make the class more robust so this class is immutable, and should only be destroyed after creation.

```java
public class EditorState {
    private final String content;
    
    public EditorState(String content) {
        this.content = content;
    }
    public String getContent() {
        return this.content;
    }
}
```

<br>

The editor class should only implement methods that it uses to restore and return a state object. It should not worry about keeping track of the states. It has an external dependency on the state objects. 

```java
public class Editor {
    private String content;
    
    public EditorState createState() {
        return new EditorState(this.content);
    }
    
    public void retore(EditorState state) {
        this.content = state.getContent();
    }
    
    public String getContent() {
        return content;
    }
    
    public void setContent(String content) {
        this.content = content;
    }
}
```

<br>

The hisotry class would internally use a stacks data structure which is built ideally for this scenario. [[DSA - 3 Stacks|About stacks.]]
```java
public class History {
    private Stack<EditorState> states;
    
    public History() {
        this.states = new Stack<EditorState>();
    }
    
    public void pushState(EditorState state) {
        this.states.push(state);
    }
    public EditorState() {
        return this.states.pop();
    }
}
```

<br>

### Usage
All the pieces would come together in the main method where the history is an external object compare to the state and editor classes:

```java
public static void main(String[] args) {
    var editor = new Editor();
    var history = new History();
    
    editor.setContent("a");
    history.push(editor.createState());
    
    editor.setContent("b");
    history.push(editor.createState());
    
    editor.setContent("c");
    editor.restore(history.pop());
}
```

This may come as lengthy and boilerplate-ish. "*Why not just have the `setContent()` method automatically store a history*". Well there is one important lesson and consideration here. Having a bloated Editor class would be a nightmare to maintain, extend, and improve.

The trade off may be having more boilerplate. But the history class can be improved without affecting the editor class or the editor state class.

<br><br>

---
## Example 2 - Document

```java
public class Document {
    private String content;
    private String fontName;
    private int fontSize;
    
    public String getContent() {
        return content;
    }
    
    public void setContent(String content) {
        this.content = content;
    }
    
    public String getFontName() {
        return fontName;
    }
    
    public void setFontName(String fontName) {\
        this.fontName = fontName;
    }
    
    public int getFontSize() {
        return fontSize;
    }
    
    public void setFontSize(int fontSize) {
        this.fontSize = fontSize;
    }
    
    @Override
    public String toString() {
        return "Document{" +
                "content='" + content + '\'' +
                ", fontName='" + fontName + '\'' +
                ", fontSize=" + fontSize +
                '}';
    }
}
```

The class represents a document in a word processor like MS Word or Apply Pages, we have 3 attributes:

- content
- fontName
- fontSize

We should allow the user to undo the changes to any of these attributes. In the future, we may add additional attributes in this class and these attributes should also be undoable. 

Implement the undo feature using the momento pattern.

<br><br>

---
## Recap
What does the momento pattern try to solve?

The momento pattern tries to decouple 2 things. It decouples an object's state, and its history of states from the object itself.

This allows you to easily decouple your applications. Its current configurations and data can be packaged as a state object, and restore from it. A History class is introduced to manage the list of states.

