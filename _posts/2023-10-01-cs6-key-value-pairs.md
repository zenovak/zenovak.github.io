---
title: CS - 6 Key Value Pairs
date: 2023-10-01 #TTTT Means timezone
categories: [C#, C# I - Fundamentals]
tags: [c#]     # TAG names should always be lowercase
---


## Hashtable
Hash tables are a non-primitive data type. They store data in key value pairs. They are from the ``System.Collections`` namespace.

Here is how we define a hash table:
```cs
Hashtable myTable = new Hashtable();
```

To use a Hashtable to store data:
```cs
myTable.Add(key, val);
```

To retrive an item, we must cast the base object back to the original object (Unboxing):
```cs
var item = (val)myTable[key];
```


> **Boxing & Unboxing, Downcasting & Upcasting**\
> Read more [here](/posts/csii-2-inheritance/#boxing-and-unboxing)
{: .prompt-tip}

<br>

### **Example**:
Say we have this student object that we want to store inside a hash table:
```cs
public class Student
{
	public int Id { get; set; }
	public string Name { get; set; }
	public int Score { get; set; }
	public Student(int id, string name, int score)
	{
		this.Id = id;
		this.Name = name;
		Score = score;
	}
}
```


We will first create our students and our Hashtable, then add the students to our table:
```cs
var studentTable = new Hashtable();

var stud1 = new Student(1, "Alvin", 50);
var stud2 = new Student(2, "Ben", 90);
var stud3 = new Student(3, "Cole", 80);
var stud4 = new Student(4, "Danial", 75);

// Adding Entries into the Hashtable.
studentTable.Add(stud1.Id, stud1);
studentTable.Add(stud2.Id, stud2);
studentTable.Add(stud3.Id, stud3);
studentTable.Add(stud4.Id, stud4);
```

To retrieve the student object, we must use a cast, as all items in Hashtables are boxed.
```cs
var student = (Student)studentTable[1];
Console.WriteLine(student.Name);
```

Entries inside Hashtable's are stored as ``DictionaryEntry`` object. To iterate thru, we can use `foreach` loop:
```cs
foreach (DictionaryEntry entry in studentTable)
{
	var student = (Student)entry.Value
	Console.WriteLine(student.Name);
}

// Alternatively:
foreach (Student item in student.Table.Value)
{
	Console.WriteLine(item.Name)
}
```

<br><br>

---
## Dictionary
Like Hashtables. Dictionary store entries in key value pairs. But they implement a type safety therfore avoids boxing and unboxing issues. It is a generic from `System.Collections.Generic` just like lists. 

To declare a dictionary:
```cs
var myDictionary = new Dictionary<int, string>();

// Using object initializer synthax:
var myDictionary2 = new Dictionary<int, string>() 
{
	{1, "Sun"},
	{2, "Moon"},
};
```


> [!INFO] Note
> Fundamentally Dictionary follows the following:
> ```cs
> var myDict = new Dictionary<Tkey, Tvalue>();
> ```
> Where Tkey and Tvalue specifies the object type. 

<br><br>

### Iterating dictionaries

```cs
foreach (KeyValuePair<string, string> entry in myDictionary)
{
	// do something with entry.Value or entry.Key
	doSomething(item.Key);
	doSomething(item.Value);
}
```


<br>

### Removing entires in iterations
Remember, when using foreach emunerable. **You cannot remove entries**
```cs
foreach (var item in myDic)
{
  if (item.key == 42)
		myDic.remove(item.key);
		// Crashes at runtime!!
}
```

One conventional way of doing this is to iterate the entire dictionary, add the entries you wish to remove to an array. Then remove them from the dictionary:
```cs
var itemsToRemove = new List<Tkey>();

foreach(var item in Dictionary){
	if (item.value == 42)
		itemsToRemove.Add(item.key);
}

foreach (var item in itemsToRemove)
    myDic.Remove(item.Key);
```


