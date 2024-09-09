---
layout:     page
title:      Turing Machine Programming Language in Python, TuringLang
summary:    A programming language which compiles to turing machine states, compiler written in Python.
categories: jekyll pixyll
---

![Turing Tape Image](/images/turing-code.png)
_![Turing Tape Image](/images/turing-tape.png){:width="2000px"}_

### Why build this? Well, great question!

This language would be the first step in building a whole computer with a Turing Machine (TM). And such a computer
(although it may be slow) would be __platform independent__, and would __run identically on any Turing Complete device__.
So in this post, I will walk you through the development and justification behind this project. Hopefully
the language will make sense by the end of this post, and you might even be interested to write some code in the language yourself!

## A Turing Computer

With the idea of creating a Turing Machine computer, the first step I considered is how will such a computer work? From now on we will refer to
the current position of the turing machine on the tape as PTR.
The states of the turing machine will perform the logical operations that are indicated by what is
on the tape. But then an issue arose! 

I wanted all instructions to be represented by preset binary numbers of some length on
the tape. 
This seems sensible, working conceivably like Assembly language, with lots of instructions for different low level
interactions with the computer. One immediate issue that presents itself is that if you write something like:
```
STORE( ADD( 5, MULTIPLY( 15, 2) ) )
```
Consider, PTR has just identified that a STORE instruction should start
but then it must somehow divert it's path to perform and addition and then go back to storing.
This was the __core issue__ that I found working on this project, the idea that the Turing machine
states have no internal memory about what they were doing before some interruption. 

## Solution is a Tree

![Turing Tree](/images/tree-turing.png)

Imagine that every time the PTR looks for some sequence of digits it is also necessarily 
looking for the root of an instruction tree. If this root is found then it starts the tree process instead. And the tree process involves searching for the next instruction name to perform. And then once an instruction is performed the PTR is sent out to the left, to find an instruction it has not yet completed.

Here is a worked example to help clarify!
#### Calculation:
```
   \/
 TREE_ROOT ADD_INSTRUCTION( 6, TREE_ROOT MULTIPLY_INSTRUCTION(2, 5))
```
#### Step One:
 1. First PTR is moving right
 2. Then PTR reads the first __TREE_ROOT__, indicating that an instruction is about to be named
 3. PTR starts an __ADD_INSTRUCTION__
 4. PTR reads a 6 and then continues on to find the next number to add
State:
```
                            \/
 TREE_ROOT ADD_INSTRUCTION( 6, TREE_ROOT MULTIPLY_INSTRUCTION(2, 5))
```

#### Step Two:

 5. PTR finds the second __TREE_ROOT__, indicating that an instruction is about to be named
 6. PTR starts an __MULTIPLY_INSTRUCTION__
 7. PTR reads a 2 and then continues on to find the next number to add
 8. PTR reads a 5 and performs the multiplication, getting the result 10
 9. PTR writes the result over the instruction to multiply

State:
```
                               \/
 TREE_ROOT ADD_INSTRUCTION( 6, 10)
```
#### Step Three:

 10. PTR is moving left
 11. PTR reads the first __TREE_ROOT__, indicating that an instruction is about to be named, and moves right
 12. PTR starts an __ADD_INSTRUCTION__
 13. PTR reads a 6 and then continues on to find the next number to add
 14. PTR reads a 10 and performs the addition, getting the result 16
 15. PTR writes the result over the instruction to add
 16. Calculation done.

State:
``` 
 \/ 
 16
```

__Universal computation complete!!!__

## Why was a new language necessary for this task?

The moment that I realised a language would be necessary for this task was
when I noticed quite how many times you would have to __detect__ some number on the right or left of the PTR. When you are adding you do it very often indeed. 

Instructions in this language have to be greater than 16 bits as well, so that they do not get confused for regular binary numbers. This means that every detect commands would have to be at least 16 states long. And detect commands not be reused, as if you were to reuse it then all reuses would send PTR to one particular state, but what you want is it to send it to a custom state as a result, again, this is caused by the issue of no state memory.

So I wrote a new language, TuringLang. In this language,
all valid turing machine instructions are valid programs, but there are commands that make it easier to build complex turing machines.

### Note on representing numbers and instructions on the tape:
Numbers are below 16 bits and represented in binary. Before a binary number there is the binary start indicator, and after there is a binary end indicator.
```
BINARY_START 233 BINARY_END
```
Instructions are represented on the tape like this
```
TREE_ROOT ARITHMETIC_UNIT ADDITION 4 5
```

## What are the commands?
### detect 
#### When is it used?
The detect statement is used when you want to detect some sequence of bits. For example if you are adding and you want to find the next number to add to. You would detect RIGHT BINARY_START and PTR would move right until it finds a binary start. A special note on the detect statement: If PTR encounters a TREE_ROOT then it will follow that new instruction.

#### How do I write it?
```
FORMAT - <start_state_name> detect <thing_detected> <direction=R/L> <state_to_go_after>
EXAMPLE - AddFifteenthDigit.5 detect be L AddFifteenthDigit.6
```

### sdetect 
#### When is it used?
The sdetect statement is almost identical to the detect statement except it does not stop when it encounters a TREE_ROOT.

#### How do I write it?
```
FORMAT - <start_state_name> sdetect <thing_detected> <direction=R/L> <state_to_go_after>
EXAMPLE - endFunction sdetect i L endFunction.2
```

### define 
#### When is it used?
The define statement is used to set up constant variables for values that would be difficult to remember in binary version but can be easily remembered with a name.

#### How do I write it?
```
FORMAT - define <constant_name> <constant_value>
EXAMPLE - define bs 0111111111111111111110
NOTE - This is saying that if we ever write "bs" meaning BINARY_START we actually mean "0111111111111111111110", which is the actual value of BINARY_START.
```

### skip 
#### When is it used?
Move PTR a certain number of spaces in one direction without detecting a TREE_ROOT. Helpful for moving around binary numbers of a known size.

#### How do I write it?
```
FORMAT - <start_state_name> skip <number_of_spaces> <direction=R/L> <state_to_go_after>
EXAMPLE - StartAdding.2 skip 15 R AddFirstDigit.1
```

### out 
#### When is it used?
Write some binary value to the tape, writes this value in a specified direction. Generally used to remove the instruction that made a calculation once it has been complete.

#### How do I write it?
```
FORMAT - <start_state_name> out <what_to_write> <direction=R/L> <state_to_go_after>
EXAMPLE - AfterAddedDigit.5 out 000 R AfterAddedDigit.6
```
### comment 
#### When is it used?
Allows the programmer to write a comment and have it ignored by the compiler. Helps for explaining what is currently going on in the code.

#### How do I write it?
```
FORMAT - comment <comment info>
EXAMPLE - comment This section adds digit 5 to digit 2
```

### include 
#### When is it used?
Indicates that the compiler should also read the code in an extra file as well. This is helpful as it means all the code does not have to be in one file but can instead be split across many files with helpful names.

#### How do I write it?
```
FORMAT - include <file_name_to_include>
EXAMPLE - include TuringArithmetic.txt
```

### Turing Machine State
#### When is it used?
This is not a command but just represents a normal turing machine state.

#### How do I write it?
```
FORMAT - <start_state_name> 1 <what_to_write_if_one> <which_direction_to_go_if_one=R/L> <which_state_to_go_if_one>       0 <what_to_write_if_zero> <which_direction_to_go_if_zero=R/L> <which_state_to_go_if_zero>
EXAMPLE - Hub0_ 1 1 R Hub01_      0 0 R InPlaceHub
```

## What can you create?

In the file TuringArithmeticAdd.txt I use TuringLang to add any two 16 bit numbers together. This addition, because of the tree, can also be stacked. Here is a problem that can be solved universally with this program:
```
5 + ((2+2)+234) + (13 + (23 + 12))
```
In the future I hope to write similar programs for subtraction, multiplication and division. If these were implemented they would automatically be stackable. I also hope to implement variable storage and loops so I can generate prime numbers.

If you would like to write your own program in TuringLang, then run
```
python PythonTuringClasses.py <file_to_compile_and_run> <tape_to_use>

EXAMPLE:
python PythonTuringClasses.py TuringText.txt 00000011111111111111111001111111111111111110101001111111111111111111100000010011000110011111111111111111111100111111111111111111010100111111111111111111110000000010001110001111111111111111111110011111111111111111111000000010011110010111111111111111111111001111111111111111111001111111111111111111001111111111111111111111000000  
```
If you want to change the state of the tape then just scroll to the bottom of that python file and modify it.

Find [the link to the github here.](https://github.com/jc10101010/TuringLang)

