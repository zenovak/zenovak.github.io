---
title: CS - 4 Control flow statements
date: 2023-10-01 #TTTT Means timezone
categories: [C#, C#I-Fundamentals]
tags: [c#]     # TAG names should always be lowercase
---


## Enhanced if statements / conditional operators
There is a shorter way to write a single if else block:

```cs
string number_confirm;
number_confirm = (a == 10) ? "Yes a is 10" : "no a is not 10";
Console.WriteLine(number_confirm);
```

<br>

> **Conditional operators**\
> `variable = (condition) ? "Assign this value when true": "assign this value when false";`
{: .prompt-tip}

<br>

*What about else if blocks?*

```cs
int temp = 100;
var matter = (temp > 100) ? "Gas" : (temp > 0) ? "Liquid" : "Solid"
```

> **Note**\
>``a ? b : c ? d : e `` is evaluated as ``a ? b : (c ? d : e)`` and not 
>``(a ? b : c ) ? d : e``
{: .prompt-info}


<br><br>

---
## Switch Cases:
Switch cases are similar to if else statement. Except it can only validate ``==`` operators. We can't use it to validate complex logics.
Often times we use switch cases with ``enum``. We declare enums in the namespace container. 

```cs
namespace ANameSpace
{
	public enum Season
    {
        Spring,
        Summer,
        Autumn,
        Winter,
    }
}
```

<br>

To use a switch case, in the main method:
```cs
var current_season = Season.Autumn;
switch (current_season)
{
	case Season.Spring:
		Console.WriteLine("Yes its spring flowers are here");
		break;
	case Season.Autumn:
		Console.WriteLine("Its autumn, my leaves has fallen");
		break;
	case Season.Winter:
		Console.WriteLine("Its Winter, let's have a snow flight");
		break;
	default:
		Console.WriteLine("Yuck its summer I/'m melting");
		break;
}
```

The ``default`` here works like the else block. This code will run when non of the previous conditions are true. 

<br>

We also have to option to use the switch case as it is. without the use of an ``enum``.
```cs
var age = 25;
switch (age)
{
	case 15;
		Console.WriteLine("You can drive");
		break;
	case 21;
		Console.WriteLine("You can drink")
		break;
}
```

<br>

> **Note!**\
> Switch cases use a jump table to make flows. It works differently than if else blocks and can be said to be more efficient. A rule of thumb to follow is to use switch case when there are a lot of variables to account for, and to use if else if there are only a few. 
{: .prompt-info}

<br>

---
## Appendix

```cs
namespace ControlFlowStatements
{
    // to use switch case lets create an enum
    public enum Season
    {
        Spring,
        Summer,
        Autumn,
        Winter,
    }
    internal class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World! Let's test out What is a (say 10, 20, or anything)");
            int a = 10;
            int b = 10;
            // This is an if else block
            if (a == 10 && b ==10)
            {
                Console.WriteLine("a is equal to 10 just like b");
            }
            else if (a== 20)
            {
                Console.WriteLine("a is actually 20");
            }
            else
            {
                Console.WriteLine("I don't know what is a");
            }
			
            // This is a conditional operator
            // variable = (condition) ? "condition is true": "condition is false";
            string number_confirm;
            number_confirm = (a == 10) ? "Yes a is 10" : "no a is not 10";
            Console.WriteLine(number_confirm);
            
            // This is a switch case this format creates an if else block
            var current_season = Season.Autumn;
            switch (current_season)
            {
                case Season.Spring:
                    Console.WriteLine("Yes its spring flowers are here");
                    break;
                case Season.Autumn:
                    Console.WriteLine("Its autumn, my leaves has fallen");
                    break;
                case Season.Winter:
                    Console.WriteLine("Its Winter, let's have a snow flight");
                    break;
                default:
                    Console.WriteLine("Yuck its summer I/'m melting");
                    break;
            }
			
            // This is a switch case. This is how we do an OR condition.
            // if (current_season == Season.Spring || current_season == Season.Autumn);
            switch (current_season)
            {
                case Season.Spring:
                case Season.Autumn:
                    Console.WriteLine("Ah we got promotion for both autumn and spring.");
                    break;
                case Season.Winter:
                    Console.WriteLine("Its Winter, let's have a snow flight");
                    break;
                default:
                    Console.WriteLine("Yuck its summer I/'m melting");
                    break;
            }
        }
    }
}
```
