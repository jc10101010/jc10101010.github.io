---
layout:     page
title:      Turing Machine Programming Language Compiler in Python, TuringLang
summary:    A programming language which compiles to turing machine states, compiler written in Python.
categories: jekyll pixyll
---

![Turing Tape Image](/images/turing-code.png)  
_![Turing Tape Image](/images/turing-tape.png){:width="2000px"}_  

### Why Build This? Well, Great Question!

This language is the first step in building an entire computer with a Turing Machine (TM). Such a computer, although it may be slow, would be **platform-independent** and would run identically on any Turing-complete device. In this post, I'll walk you through the development and reasoning behind this project. Hopefully, by the end, the language will make sense to you, and you might even be inspired to write some code in TuringLang yourself!

## A Turing Computer

To create a Turing Machine computer, the first challenge I faced was figuring out how such a computer would function. From now on, we’ll refer to the current position of the Turing machine on the tape as PTR. The states of the Turing machine perform the logical operations indicated by what is on the tape. But then, an issue arose!

I wanted all instructions to be represented by preset binary numbers of some length on the tape. This idea works similarly to Assembly language, with many instructions for different low-level interactions with the computer. One immediate issue, however, is if you write something like:

```
STORE( ADD( 5, MULTIPLY( 15, 2) ) )
```

Now consider, PTR has just identified that a STORE instruction should start, but it must somehow divert its path to perform the addition and then return to storing. This was the **core issue** I encountered during this project—the fact that the Turing machine states have no internal memory of what they were doing before the interruption.

## Solution: A Tree Structure

![Turing Tree](/images/tree-turing.png)

Imagine that every time the PTR looks for a sequence of digits, it also searches for the root of an instruction tree. If this root is found, the tree process begins. This process involves searching for the next instruction name to perform, and once an instruction is performed, the PTR moves to the left to find the next incomplete instruction.

Here’s an example to help clarify:

#### Calculation:
```
   \/
 TREE_ROOT ADD_INSTRUCTION( 6, TREE_ROOT MULTIPLY_INSTRUCTION(2, 5))
```

#### Step One:
1. PTR moves right.
2. PTR reads the first **TREE_ROOT**, indicating that an instruction is about to be named.
3. PTR begins the **ADD_INSTRUCTION**.
4. PTR reads a 6 and continues to find the next number to add.

State:
```
                            \/
 TREE_ROOT ADD_INSTRUCTION( 6, TREE_ROOT MULTIPLY_INSTRUCTION(2, 5))
```

#### Step Two:
5. PTR finds the second **TREE_ROOT**, indicating a new instruction is about to be named.
6. PTR starts a **MULTIPLY_INSTRUCTION**.
7. PTR reads a 2 and continues to find the next number to multiply.
8. PTR reads a 5, performs the multiplication, and gets the result 10.
9. PTR writes the result over the multiplication instruction.

State:
```
                               \/
 TREE_ROOT ADD_INSTRUCTION( 6, 10)
```

#### Step Three:
10. PTR moves left.
11. PTR reads the first **TREE_ROOT**, indicating that an instruction is about to be named, and moves right.
12. PTR resumes the **ADD_INSTRUCTION**.
13. PTR reads a 6 and continues to find the next number to add.
14. PTR reads a 10 and performs the addition, getting the result 16.
15. PTR writes the result over the addition instruction.
16. Calculation complete.

State:
```
 \/ 
 16
```

**Universal computation complete!**

## Why Was a New Language Necessary for This Task?

I realized a new language was necessary when I noticed how frequently you need to **detect** a number on the right or left of the PTR while adding. This happens quite often. Additionally, instructions in this language need to be longer than 16 bits to avoid being mistaken for regular binary numbers. This means every detect command must be at least 16 states long. Reusing detect commands was also problematic because reuses would always send the PTR to a specific state, but we want it to send the PTR to a custom state based on context. This issue stems from the lack of memory between states.

Thus, I wrote a new language, TuringLang. In this language, all valid Turing machine instructions are valid programs, but it includes commands that make it easier to build complex Turing machines.

### Representing Numbers and Instructions on the Tape:

Numbers are represented in binary (less than 16 bits), and each binary number is bracketed by start and end indicators:

```
BINARY_START 233 BINARY_END
```

Instructions are represented as follows:

```
TREE_ROOT ARITHMETIC_UNIT ADDITION 4 5
```

## What Are the Commands?

### detect 
#### When is it used?
The `detect` command is used to find a specific sequence of bits. For example, if you are adding and need to find the next number to add, you would `detect` RIGHT `BINARY_START`, and the PTR would move right until it finds a binary start. A note: If PTR encounters a **TREE_ROOT**, it will follow that new instruction.

#### Syntax:
```
<start_state_name> detect <thing_detected> <direction=R/L> <state_to_go_after>
Example: AddFifteenthDigit.5 detect be L AddFifteenthDigit.6
```

### sdetect 
#### When is it used?
The `sdetect` command is almost identical to `detect`, except that it does not stop when it encounters a **TREE_ROOT**.

#### Syntax:
```
<start_state_name> sdetect <thing_detected> <direction=R/L> <state_to_go_after>
Example: endFunction sdetect i L endFunction.2
```

### define 
#### When is it used?
The `define` command is used to create constant variables for values that would be hard to remember in binary form but can be easily recalled by name.

#### Syntax:
```
define <constant_name> <constant_value>
Example: define bs 0111111111111111111110
Note: This means that whenever "bs" is written, it is equivalent to "0111111111111111111110" (the binary for BINARY_START).
```

### skip 
#### When is it used?
The `skip` command moves the PTR a specified number of spaces in one direction without detecting a **TREE_ROOT**. This is helpful for moving past binary numbers of known size.

#### Syntax:
```
<start_state_name> skip <number_of_spaces> <direction=R/L> <state_to_go_after>
Example: StartAdding.2 skip 15 R AddFirstDigit.1
```

### out 
#### When is it used?
The `out` command writes a binary value to the tape in the specified direction. This is generally used to remove the instruction that performed a calculation once it's complete.

#### Syntax:
```
<start_state_name> out <what_to_write> <direction=R/L> <state_to_go_after>
Example: AfterAddedDigit.5 out 000 R AfterAddedDigit.6
```

### comment 
#### When is it used?
The `comment` command allows the programmer to write notes that the compiler will ignore. This is useful for explaining sections of the code.

#### Syntax:
```
comment <comment_info>
Example: comment This section adds digit 5 to digit 2
```

### include 
#### When is it used?
The `include` command tells the compiler to read the code in another file. This is useful for breaking code into separate, logically named files.

#### Syntax:
```
include <file_name_to_include>
Example: include TuringArithmetic.txt
```

### Turing Machine State
#### When is it used?
This represents a standard Turing machine state and is not a specific command.

#### Syntax:
```
<start_state_name> 1 <what_to_write_if_one> <which_direction_to_go_if_one=R/L> <which_state_to_go_if_one>
                     0 <what_to_write_if_zero> <which_direction_to_go_if_zero=R/L> <which_state_to_go_if_zero>
Example: Hub0_ 1 1 R Hub01_      0 0 R InPlaceHub
```

## What Can You Create?

In the file `TuringArithmeticAdd.txt`, I use TuringLang to add any two 16-bit numbers. This addition, thanks to the tree structure, can also be nested. Here’s an example problem that can be universally solved with this program:

```
5 + ((2+2)+234) + (13 + (23 + 12))
```

In the future, I plan to write similar programs for subtraction, multiplication, and division. If implemented, these operations would also be stackable. I also hope to add variable storage and loops to enable generating prime numbers.

If you’d like to write your own program in TuringLang, run:

```
python PythonTuringClasses.py <file_to_compile_and_run> <tape_to_use>

Example:
python PythonTuringClasses.py TuringText.txt 000000111111111111111110011111111111111111101010011111111111111111111000000100110001100

11111111111111111111100111111111111111111010100111111111111111111110000000010001110001111111111111111111110011111111111111111111000000010011110010111111111111111111111001111111111111111111001111111111111111111001111111111111111111111000000
```

To modify the state of the tape, scroll to the bottom of the Python file and adjust it as needed.

[Find the link to the GitHub here.](https://github.com/jc10101010/TuringLang)
