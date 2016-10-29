---
layout: post
title:  "Writing a c++ crackme and use ollydbg to reverse engineer and crack it"
date:   2016-10-29 10:52:07 +0200
categories: main
icons: 
- icon-cplusplus
- icon-shell
- fa-windows
---
In this post I'll write a simple crackme in c++ (a very very simple one) and try to crack into it with ollydbg.
The provided crackme is really simple and it's basically just an "insert correct password" one, code should be pretty self-explanatory:

{% highlight c++ %}
#include <iostream>

using namespace std;

int main(void)
{
    string password;
    cout << "Insert password: ";
    cin >> password;
    if (password == "abcdef") {
        cout << "Welcome to the private area" << endl;
    }
    else {
        cout << "Wrong password" << endl;
    }
    return 0;
}
{% endhighlight %}

We'll use [ollydbg][olly-dbg] to open the generated exe file and check out its assembly code.
Once we've attached the executable to ollydbg we can step-over / step-into the code and try to figure out what's happening. If we skip ahead enough we'll find the following piece of code: 

![AsmCodeCrackme]({{ site.url }}/assets/images/asmCrackme.PNG){: .center-image}

Since this is such a simple crackme the "ASCII" comments created automatically by ollydbg are very helpful. Let's try and check out what the code does.

The following part makes a system call and asks the user input for the password. Notice the CALL instruction. if you try and run the program it will be stuck into this call until the user actually inputs the code.
{% highlight NASM %}
00401461   MOV DWORD PTR SS:[ESP+4],untitled.004A2065      ;  ASCII "Insert password: "
00401469   MOV DWORD PTR SS:[ESP],untitled.004A1780
00401470   CALL untitled.00496C50
00401475   LEA EAX,DWORD PTR SS:[EBP-20]
00401478   MOV DWORD PTR SS:[ESP+4],EAX
0040147C   MOV DWORD PTR SS:[ESP],untitled.004A15A0
00401483   CALL untitled.00497F80

{% endhighlight %}	

This piece of code is actually the one that prepares the correct password ("abcdef") and makes a CALL to 00496BD0 which is going to make the actual comparision. 
__The result of the call is going to set the status register__.

{% highlight NASM %}

00401488   MOV DWORD PTR SS:[ESP+4],untitled.004A2077      ;  ASCII "abcdef"
00401490   LEA EAX,DWORD PTR SS:[EBP-20]
00401493   MOV DWORD PTR SS:[ESP],EAX
00401496   CALL untitled.00496BD0

{% endhighlight %}

Now that we have compared the two strings, we have the result of such comparision. __It all boils down to the TEST and JUMP EQUALS instruction__ . If the strings are different, the software will jump to 004014C6 which is the "WRONG PASSWORD" section. 
If the strings are actually the same, it doesn't jump and goes to the next instruction which is the "private area": 
{% highlight NASM %}

0040149B   TEST AL,AL                                      ;  THE TEST
0040149D   JE SHORT untitled.004014C6                      ;  THE JUMP IF THE STRINGS ARE DIFFERENT
0040149F   MOV DWORD PTR SS:[ESP+4],untitled.004A207E      ;  ASCII "Welcome to the private area"
004014A7   MOV DWORD PTR SS:[ESP],untitled.004A1780
004014AE   CALL untitled.00496C50
004014B3   MOV DWORD PTR SS:[ESP],untitled.00493F40                 
004014BA   MOV ECX,EAX                                              
004014BC   CALL untitled.00460EB0                          ; \untitled.00460EB0
004014C1   SUB ESP,4
004014C4   JMP SHORT untitled.004014EB
004014C6   MOV DWORD PTR SS:[ESP+4],untitled.004A209A      ;  ASCII "Wrong password"
004014CE   MOV DWORD PTR SS:[ESP],untitled.004A1780
004014D5   CALL untitled.00496C50
004014DA   MOV DWORD PTR SS:[ESP],untitled.00493F40                 
004014E1   MOV ECX,EAX                                              
004014E3   CALL untitled.00460EB0                          ; \untitled.00460EB0

{% endhighlight %}	


So, here we are, we understood how the assembly works. How our C++ code is compiled and now we are asking ourselves:

how do we crack this?

Here are a few ways:

## We insert a JMP instruction to go straight to the private area: 

the TEST and JE part are going to be skipped and you can create a "cure" for your program. We can even put the JMP before it asks the user for the password! 
Let's try and understand how we can do this:

We basically want to JMP from one instruction to another. Suppose we want to make a jump from the "Insert password" instruction to the "Welcome to private area" one.

Basically we wanna jump from:

**00401461**   . C74424 04 6520>MOV DWORD PTR SS:[ESP+4],untitled.004A2065               ;  ASCII "Insert password: "

to:

**0040149F**   . C74424 04 7E20>MOV DWORD PTR SS:[ESP+4],untitled.004A207E               ;  ASCII "Welcome to the private area"

Grabbing a programmer calculator and subtracting the two addresses gives us: 

**0040149F**-**00401461** = **3E**

Right click on the first instruction, and select binary->edit. You'll see some hex code, **we need to change that hex code so that translated into assembly it's gonna be "JMP SHORT untitled.0040149F"**. 
You can use a hex to assembly disassembler such as [this one][disassembler] and you can verify yourself that the hex string "EB 3C" reads to jmp 0x3e:

{% highlight nasm %}

Raw Hex (zero bytes in bold):

EB3C   

String Literal:

"\xEB\x3C"

Array Literal:

{ 0xEB, 0x3C }

Disassembly:
0:  eb 3c                   jmp    0x3e

{% endhighlight %}

Once we have our string, all we want to do is to change the previous hex code to the new one and fill with NOPs the remaining instructions. Here's the outcome:

![outcome1]({{ site.url }}/assets/images/outcome1crackme.PNG){: .center-image}

if you execute the code it will go straight to the private area.

## Fill the JE instruction with NOP

We can just fill the JE instruction right after the test with a NOP. This way, even tho the test is indeed showing that the strings are different, it will still go to the private area and not go for the jump.

To do this, just go to the instruction: 

0040149D   . 74 27          JE SHORT untitled.004014C6

**right click it and select "fill with NOP".**

Even if you input a wrong password, it will still get you in the private area: 

![outcome2]({{ site.url }}/assets/images/outcome2crackme.PNG){: .center-image}

# Conclusions

The simplest solution, after all, would be just to input the right password. Once you've disassembled it, you can read it clearly. But the crackme can be made in such a way that the ascii string showing the password is not going to be showed and such method won't hold up at all, while the others... well, in principle they will still work. Even tho in harder crackmes it wouldn't be so linear and easy as showed in this overly-simplistic exercise.

Also, all of this is done for accademic purposes and in not to show how to crack things. Also it's just fun to mess around with stuff sometimes. :) 

[olly-dbg]: http://ollydbg.de/download.htm
[disassembler]: https://defuse.ca/online-x86-assembler.htm#disassembly2
