---
title: CS - 1 Primitive Types and Expressions
date: 2023-06-18 #TTTT Means timezone
categories: [C#, C#Fundamentals]
tags: [c#]     # TAG names should always be lowercase
---

>**Primitive Types and Expressions**
>By Mosh

## Variables and Constants

A variable is a name that we give to a storage location in memory. We use variables to store temporary values in memory.

A constant is a value that cannot be changed. We use constants in situations where we need to ensure that a value does not change. For example, if you're working on a program where you need to calculate the area of a circle, you need to use the Pi number (3.14). You can define Pi as a constant to ensure you don't accidentally change the value of Pi in computations.


To define a variable, we specify a type and an identifier:
```cs
int number;
```

Here, int represents the integer type, which takes 4 bytes of memory. You can find the most frequently used data types in the next section.
Number is the identifier for our variable. We can optionally set the value of a variable upon declaration. This is called “initializing a variable”:
```cs
int number = 5;
```

Remember: in C#, you cannot read the value of a variable unless you have set it before.

To define a constant. we add the *const* keyword:
```cs
 const int banana = 10;
```

<br>

> **NOTE**\
> You may declare a variable and not assign it a value, but constants must be initialised
{: .prompt-warning}

<br>

### Primitive Types

| Type    | Bytes |
|---------|-------|
| byte    | 1     |
| short   | 2     |
| int     | 4     |
| long    | 8     |
| float   | 4     |
| double  | 8     |
| decimal | 16    |
| bool    | 1     |
| char    | 2     |

These types have an equivalent type in .NET Framework. So when you compile your application, the compiler maps your types to the underlying type in .NET Framework.

| C# Type | .NET Type |
|---------|-----------|
| byte    | Byte      |
| short   | Int16     |
| int     | Int32     |
| long    | Int64     |
| float   | Single    |
| double  | Double    |
| decimal | Decimal   |
| bool    | Boolean   |
| char    | Char      |


The easy way to memorize Int* types is by remembering the number of bytes each type uses. For example, a "short" takes 2 bytes. We have 8 bits in each byte. So a "short" is 16 bits, hence the underlying .NET type is Int16.

We also have a few non-primitive types (string, array, class, struct) that I’ll discuss in the following sections. [](obsidian://open?vault=AbyssArchive&file=CSharp%20Notes%2FCS%20-%20Fundamentals%2FCS%20-%203%20Non%20premitive%20types)


> **Real Numbers**\
> Notice that float and decimal has to be declared with an f and m at the end of the number
>```cs
>float pie = 3.142f;
>double bigPie = 3.142;
>decimal extraBigPie = 3.142m;
>```
{: .prompt-info}

<br>

---
## Scope

Scope determines where a value has meaning and is accessible. A variable has a scope in the block it is defined and in any child blocks. But it is not accessible outside that block. A block is indicated by curly braces ({ }).

<br>

---
## Overflowing

Each type, depending on the number of types allocated to it, can store a range of values. If we store a value in a variable, but that value exceeds the boundary of values for the underlying type, overflow happens. For example, we can store any values between 0 and 255 in a byte. If the value of a byte exceeds this boundary during computations, overflow happens. Here is an example:
```cs
byte b = 255; 
b = b + 1;
```

As a result of the second line, the value of b will be 0.


At compile time, if we assigns a value greater than the type. It will throw an exeption:
```cs
byte x = 256     //Error
```

<br>

---
## Type Conversion

There are times that you need to temporarily convert the value of a variable to a different type. Note that this conversion does not impact the original variable since C# is a _statically-typed language_, which in simple term means: once you declare the type of a variable, you cannot change it. But you may need to convert the "value" of a variable as part of assigning that value to a variable of a different type.

There are a few conversion scenarios:

If types are compatible (e.g. integral numbers and real numbers) and the target type is bigger, you don't need to do anything. The value will be automatically converted by the runtime and stored in the target type.
```cs
byte b = 1; 
int i = b;
```

Here because b is a byte and takes only 1 byte of memory, we can easily convert it to an int, which takes 4 bytes. So we don't need to do anything. This is known as an implicit type conversion.

If the target type, however, is smaller than the source type, the compiler will generate an error. The reason for that is that overflow may happen as part of the conversion. For example, if you have an int with the value 1000, you cannot store it in a byte because the max value a byte can store is 255. In this case, some of the bits will be lost in memory. And that's the reason compiler warns you about these scenarios. If you're sure that no bits will be lost as part of the conversion, you can tell the compiler that you're aware of the overflow and would still like the conversion to happen. In this case, you use a *cast*:

```cs
// Casting
int i = 1;
byte b = (byte)i;
```


In this example, our int holds the value 1, which can perfectly be stored in a byte. So, we use a cast to tell the compiler to ignore the overflow. A cast means prefixing the variable with the target type. So here we are casting the variable i to a byte in the second line.

Finally, if the source and target type are not compatible (eg a string and a number), you need to use the Convert class.
```cs
string s = "1234";
int i = Convert.ToInt32(s);
```

Convert class has a number of methods for converting values to various types.

> **SUMMARY**
>
>```cs
>// Implicit type conversion. 
>byte b = 1;
>int i = b;
>
>// Explicit type conversion.
>int num = 1;
>byte byt = num; // error
>
>// This is called a cast. its how we convert explicitly
>int num = 1;
>byte byt = (byte)num;
>
>// incompatible type conversion.
>string example = "1";
>int myNumber = (int)example; // error
>
>// This is how we do it in c#
>string example2 = "1";
>int myNumber2 = Convert.ToInt32(example);  // Using System namespace Convert class
>int myNumber3 = int.Parse(example2);       // Using Parse method. 
>```
{: .prompt-tip}

<br>

---
## Operators

In C# we have 4 types of operators:

•    **Arithmetic**: used for computations

•    **Comparison**: used for comparing values in boolean expressions

•    **Logical**: represent logical AND, OR and NOT

•    **Bitwise**: represent bitwise AND, OR and NOT


### Arithmetic Operators

| Operator |      Description      |
|:--------:|:---------------------:|
|    +     |          Add          |
|    -     |       Subtract        |
|    *     |       Multiply        |
|    /     |        Divide         |
|    %     | Remainder of Division |
|    ++    |    Increment by 1     |
|    —     |    Decrement by 1     |


### Comparison Operators

| Operator |       Description        |
|:--------:|:------------------------:|
|    >     |       Greater than       |
|    >=    | Greater than or equal to |
|    <     |        Less than         |
|    <=    |  Less than or equal to   |
|   .==    |          Equal           |
|    !=    |        Not equal         |



### Logical operators

| Operator | Description |
|:--------:|:-----------:|
|    &&    | Logical AND |
|   \|\|   | Logical OR  |
|    !     | Logical NOT |


### Bitwise Operators

| Operator | Description |
|:--------:| ----------- |
|    &     | Bitwise AND |
|    \|    | Bitwise OR  |

<br>

---

### Doing Math in C\#
In C# divisions of 2 type int numbers do not return a float. Instead, it returns an int by truncating the decimals places.

So you must do this instead:
```cs
int a = 3;

int b = 10;

int c =5;

(float)b / (float)a // Casting the int into type float
```


Mathematical operators in c# follows regular math rules. PEMDAS.
```cs
a + b * c; // 53
```
