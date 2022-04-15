---
title:  "assembly"
mathjax: true
layout: post
categories: [Assembly]
date: 2017-05-10 11:04 +0200
---


<!-- Output copied to clipboard! -->

<!-----

Yay, no errors, warnings, or alerts!

Conversion time: 1.103 seconds.


Using this Markdown file:

1. Paste this output into your source file.
2. See the notes and action items below regarding this conversion run.
3. Check the rendered output (headings, lists, code blocks, tables) for proper
   formatting and use a linkchecker before you publish this page.

Conversion notes:

* Docs to Markdown version 1.0β33
* Thu Apr 14 2022 10:08:49 GMT-0700 (PDT)
* Source doc: Assembly Notes
* This is a partial selection. Check to make sure intra-doc links work.
----->


**<span style="text-decoration:underline;">Data Types</span>**

Char -> Byte

Short -> Word

int/long -> DoubleWord

double/long long -> Quad Word

Long Double -> Double Quad Word

**<span style="text-decoration:underline;">Negative numbers</span>** 

1’s complement - flip bits 0->1 , 1->0

2’s complement - 1’s complement + 1

Positive Number is between 0x01 - 0x7F

Negative Number is between 0x80 - 0xFF

**<span style="text-decoration:underline;">Architecture CISC and RISC</span>**

CISC:-

Eg; Intel

RISC:

More registers , less and fixed instructions

Eg: ARM, PowerPC

Little Endian -> 0x12345678 -> 78,56,34,12 ->  Ram , Eg:Intel

Big Endian    -> 0x12345678   -> 12,34,56,78  -> Registers

**<span style="text-decoration:underline;">Registers</span>**



* Small memory built into the processor(volatile memory)
* In Intel we have 8 general purpose registers + IP

**<span style="text-decoration:underline;">Register Conventions -1</span>**



* EAX - Stores function return values eg: int sub(){return 0xbeef;}
* EBX - Base pointer to the data section
* ECX - Counter for string  and loop operations
* EDX - I/O pointer

**<span style="text-decoration:underline;">Register Conventions -2</span>**



* ESI - Source pointer for string operations
* EDI - Destination pointer for string operations
* ESP - stack pointer, always points towards the top of the stack.
* EBP - Stack frame base pointer
* EIP - Pointer to next instruction to execute

**<span style="text-decoration:underline;">Register Conventions -3</span>**



* Caller-save registers - eax,edx,ecx , values get deleted when calling another function
* Callee -save registers - ebp,ebx,esi,edi values would not get deleted 

**<span style="text-decoration:underline;">EFlags </span>**



* Registers
* Zero Flag- set if the result of some instruction is zero
* Sign Flag - 0 indicates positive value and 1 indicates negative

**<span style="text-decoration:underline;">Instructions</span>**



1. **NOP**  
    1. No operation . Just to pad/align byte or to delay time. Used in buffer overflow exploit. ( intel's example 16 byte )
2. **PUSH**  
    2. Can  be a numeric constant. For instance, push 4 will push 00000004 to the stack.
    3. Can be the value in a register. Eg: push eax; will push the value inside eax
    4. Push instruction automatically decrements the ESP by 4 bytes
3. **POP**
    5. Take Dword off the stack
    6. Put in a register
    7. Increment by 4 bytes
4. **CALL**
    8. Implicitly modifying EIP
    9. Transfer control to a different function, in a way that it can be resumed where it left off
    10. What’s happening behind when CALL instruction is done
        1. Push the address of the next instruction onto the stack
        2.  Then it changes EIP to the address given in the instruction
    11. Destination address can be specified in multiple ways 
        3. Absolute address eg : go to 0x00239082
        4. Relative address eg: go to hex 50 bytes passed to the end of the next instruction
5. **RET**
    12. Two types
        5. CDECL -  POP the top of the stack into EIP
        6. STDL - POP the top of the stack into EIP and add a constant number of bytes to esp. Eg ret 0x20 
6. **MOV **
    13. Can move
        7. Register to register
        8. Memory to register and vice versa
        9. Immediate(constants hard coded to the instruction stream ) to register and vice versa
    14. Never memory to memory
7.  **LEA**
    15. Load Effective Address
    16. Calculates whatever address inside the ‘[ ]’ and store it back into EAX
8. **ADD**
    17. ADD ESP, 8 -> ESP = ESP + 8
9. **SUB**
    18. SUB eax, [ebx*2] = takes the value inside ebx, mul 2, treat it as address then eax - the memory content and store it into eax
10. **JMP**
    19. Changes the eip to the given address
    20. Main forms of address
        10. Short relative - 1 byte from end of the instruction eg jmp 0x0E forward
        11. Near relative - 4 bytes displacement from the current EIP
        12. Absolute - hardcoded address in the instruction
        13. Absolute indirect - address calculated with r/m32
11.  **JCC**
    21. Jump if condition is met
    22. JNE = JNZ because both instructions check for zero flag
    23. JZ/JE -> if true go to jmp . If false, falls into the next instruction
    24. JLE
    25. JGE
12. **CMP**
    26. Is performed by subtracting the second operand from the first operand and then setting the status flags
    27. Would not store after subtracting
13. **TEST**
    28. Does logical and on operators
    29. Does not store the result
    30. Set the flag 
14.  **AND **
    31. Does and operations
    32. Eg: And al,bl
    33. Destination can r/m32 and source can be r/m32 or register or immidiate 
    34. Sets flag in the background(all logical operators)
15. **OR**
    35. Does or operation
16. **XOR**
    36. Does XOR operation
17. **NOT**
    37. Does NOT operation
18. **SHL**
    38. Shift logical left &lt;<
    39. Can be used to multiply( power of 2)
19. **SHR**
    40. Shift logical right >>
    41. Can be used to divide number(power of 2)
20. I**MUL**
    42. 3 forms
        14. IMUL r/m32 -> edx:eax=eax*r/m32
        15. IMUL reg, r/m32 -> reg = reg*r/m32
        16. IMUL reg. r/m32, immediate -> reg=r/m32 * immediate
21. **DIV**
    43. Two forms:
        17. Unsigned divide ax by r/m8,al=quotient, ah =remainder
        18. Unsigned divide  EDX:EAX by r/m8 ,eax= quotient ,edx =remainder
22. **REP STOS**
    44. Repeat a single instruction multiple times
    45. Specifies the number of time (loop) in ECX
    46. Size should be specified
    47. Then data keeps on writing into eax

**<span style="text-decoration:underline;">Stack</span>**



* Stack is a conceptual area of main memory(RAM) which is designed by the OS when a program is started.
* Has certain type of Data structure
* It can be anywhere in the ram, decided by the OS
* When a program is started, OS reserves some chunk of ram and put the stack over there
* LIFO/FILO data structure
* Stack grows towards the lower memory addresses
* Top of the stack is always the lowest numeric addresses
* ESP points towards the top of the stack , lower address -> 00000
* Keeps tracks of which functions were called before the current one(eg: main and subroutine 1)
* Holds local variable and frequently used to pass arguments to the next function to be called
* 

**<span style="text-decoration:underline;">Calling Conventions</span>**



* How to pass parameters to a function and how you get parameters back 
* Conventions
    * CDECL
    * STDL
* **CDECL**
    * C Declaration - most calling convention
    * Functions parameters are pushed onto stack right to left. Eg print(“%d”, myvar). myvar->%d->printf
    * When a function is called, it saves the old stack frame and set up new frame(eg if new new function is called it saves the copy of  main( ) frame and set up the stack frame of the newly called function. So after it's done it can go back to main() stack frame )
    * **Caller is responsible for cleaning up the stack(responsible for poping up the parameters of the stack )**
* **STDL**
    * Used by Microsoft C++ code
    * Functions parameters are pushed onto stack right to left
    * When a function is called, it saves the old stack frame and set up new frame
    * **Callee is responsible for cleaning up any stack parameters it takes**

**<span style="text-decoration:underline;">General stack frame operation</span>**

When a program is executed, first main() reserves space on the stack for it’s local variable. This is done by subtracting ESP in order to make space for local variables. If main() wants to call a subroutine(), main() becomes “the caller”. And performs caller-save register to store those registers if its present. In the next step the parameters are pushed into the stack. Ie: arguments are passed to the callee. When we actually execute the CALL instruction(subroutine()), the saved return address of the next instruction after the CALL instruction will be pushed onto the stack. So that when Callee is done executing the instructions, it can look up the this value and know where to go back.The entire main() stack frame is till this step. EBP always points to the start of the stack frame.

Now the first thing the subroutine() going to do is that it takes the EBP, the pointer which points to the top of the local variables and save it under the stack. So when it's done and ready to destroy its own stack frame, it takes the stack pointer and put backs to EBP so that it again points to mains frame instead of its own. Then the subroutine() takes any callee-save registers, it will go ahead and save them.Stack frames are a linked list 

If subroutine() wants subroutine2(), subroutine has to go through the same steps.

After subroutine() is done, it will clean all parameter, caller-save register, local variables,callee-save registers in sequential order and it takes the saved frame pointer and replace EBP with this value. So that it will point to main() frame. Then all the parameters and local variables in main() will get cleaned. 

**CPUID**



    48. CPU feature identification
    49. Tells what features your pc currently supports eg: virtualization
    50. Doesn’t have operands
    51. Before issue cpuid eax should be set
    52. After issuing cpuid eax,ebx,ecx,edx are going to be overridden

**PUSHFD**



    53. Push EFlags onto the stack

**POPFD**



    54. POP stack into EFlags

**<span style="text-decoration:underline;">Control Flow</span>**



* Conditional  - if, switch,loops
* Unconditional - calls, goto,exceptions,interrupts

**Inline Assembly**

Syntax intel

__asm{

    Mov eax, [esp + 0x4]

}

Syntax AT&T

asm(“push EBP \n”

“Mov ESP , EBP\n”

);

**Compliling**

Gcc -o &lt;output> &lt;input>

Gcc -o file file.c

-ggdb ->to add debug symbols

**Buffer Overflow**


