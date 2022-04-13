---
title:  "assembly"
mathjax: true
layout: post
categories: assembly
---

Data Types

Char -> Byte
Short -> Word
int/long -> DoubleWord
double/long long -> Quad Word
Long Double -> Double Quad Word

Negative numbers 

1’s complement - flip bits 0->1 , 1->0
2’s complement - 1’s complement + 1

Positive Number is between 0x01 - 0x7F
Negative Number is between 0x80 - 0xFF

Architecture CISC and RISC

CISC:-
Eg; Intel

RISC:
More registers , less and fixed instructions
Eg: ARM, PowerPC

Little Endian -> 0x12345678 -> 78,56,34,12 ->  Ram , Eg:Intel
Big Endian    -> 0x12345678   -> 12,34,56,78  -> Registers

Registers

Small memory built into the processor(volatile memory)
In Intel we have 8 general purpose registers + IP

Register Conventions -1
EAX - Stores function return values eg: int sub(){return 0xbeef;}
EBX - Base pointer to the data section
ECX - Counter for string  and loop operations
EDX - I/O pointer

Register Conventions -2
ESI - Source pointer for string operations
EDI - Destination pointer for string operations
ESP - stack pointer, always points towards the top of the stack.
EBP - Stack frame base pointer
EIP - Pointer to next instruction to execute

Register Conventions -3

Caller-save registers - eax,edx,ecx , values get deleted when calling another function
Callee -save registers - ebp,ebx,esi,edi values would not get deleted 

EFlags 

Registers
Zero Flag- set if the result of some instruction is zero
Sign Flag - 0 indicates positive value and 1 indicates negative

Instructions

NOP  
No operation . Just to pad/align byte or to delay time. Used in buffer overflow exploit. ( intel's example 16 byte )
PUSH  
Can  be a numeric constant. For instance, push 4 will push 00000004 to the stack.
Can be the value in a register. Eg: push eax; will push the value inside eax
Push instruction automatically decrements the ESP by 4 bytes
POP
Take Dword off the stack
Put in a register
Increment by 4 bytes
CALL
Implicitly modifying EIP
Transfer control to a different function, in a way that it can be resumed where it left off
What’s happening behind when CALL instruction is done
Push the address of the next instruction onto the stack
 Then it changes EIP to the address given in the instruction
Destination address can be specified in multiple ways 
Absolute address eg : go to 0x00239082
Relative address eg: go to hex 50 bytes passed to the end of the next instruction
RET
Two types
CDECL -  POP the top of the stack into EIP
STDL - POP the top of the stack into EIP and add a constant number of bytes to esp. Eg ret 0x20 
MOV 
Can move
Register to register
Memory to register and vice versa
Immediate(constants hard coded to the instruction stream ) to register and vice versa
Never memory to memory
 LEA
Load Effective Address
Calculates whatever address inside the ‘[ ]’ and store it back into EAX
ADD
ADD ESP, 8 -> ESP = ESP + 8
SUB
SUB eax, [ebx*2] = takes the value inside ebx, mul 2, treat it as address then eax - the memory content and store it into eax
JMP
Changes the eip to the given address
Main forms of address
Short relative - 1 byte from end of the instruction eg jmp 0x0E forward
Near relative - 4 bytes displacement from the current EIP
Absolute - hardcoded address in the instruction
Absolute indirect - address calculated with r/m32
 JCC
Jump if condition is met
JNE = JNZ because both instructions check for zero flag
JZ/JE -> if true go to jmp . If false, falls into the next instruction
JLE
JGE
CMP
Is performed by subtracting the second operand from the first operand and then setting the status flags
Would not store after subtracting
TEST
Does logical and on operators
Does not store the result
Set the flag 
 AND 
Does and operations
Eg: And al,bl
Destination can r/m32 and source can be r/m32 or register or immidiate 
Sets flag in the background(all logical operators)
OR
Does or operation
XOR
Does XOR operation
NOT
Does NOT operation
SHL
Shift logical left <<
Can be used to multiply( power of 2)
SHR
Shift logical right >>
Can be used to divide number(power of 2)
IMUL
3 forms
IMUL r/m32 -> edx:eax=eax*r/m32
IMUL reg, r/m32 -> reg = reg*r/m32
IMUL reg. r/m32, immediate -> reg=r/m32 * immediate
DIV
Two forms:
Unsigned divide ax by r/m8,al=quotient, ah =remainder
Unsigned divide  EDX:EAX by r/m8 ,eax= quotient ,edx =remainder
REP STOS
Repeat a single instruction multiple times
Specifies the number of time (loop) in ECX
Size should be specified
Then data keeps on writing into eax

Stack

Stack is a conceptual area of main memory(RAM) which is designed by the OS when a program is started.
Has certain type of Data structure
It can be anywhere in the ram, decided by the OS
When a program is started, OS reserves some chunk of ram and put the stack over there
LIFO/FILO data structure
Stack grows towards the lower memory addresses
Top of the stack is always the lowest numeric addresses
ESP points towards the top of the stack , lower address -> 00000
Keeps tracks of which functions were called before the current one(eg: main and subroutine 1)
Holds local variable and frequently used to pass arguments to the next function to be called


Calling Conventions

How to pass parameters to a function and how you get parameters back 
Conventions
CDECL
STDL

CDECL
C Declaration - most calling convention
Functions parameters are pushed onto stack right to left. Eg print(“%d”, myvar). myvar->%d->printf
When a function is called, it saves the old stack frame and set up new frame(eg if new new function is called it saves the copy of  main( ) frame and set up the stack frame of the newly called function. So after it's done it can go back to main() stack frame )
Caller is responsible for cleaning up the stack(responsible for poping up the parameters of the stack )
STDL
Used by Microsoft C++ code
Functions parameters are pushed onto stack right to left
When a function is called, it saves the old stack frame and set up new frame
Callee is responsible for cleaning up any stack parameters it takes

General stack frame operation

When a program is executed, first main() reserves space on the stack for it’s local variable. This is done by subtracting ESP in order to make space for local variables. If main() wants to call a subroutine(), main() becomes “the caller”. And performs caller-save register to store those registers if its present. In the next step the parameters are pushed into the stack. Ie: arguments are passed to the callee. When we actually execute the CALL instruction(subroutine()), the saved return address of the next instruction after the CALL instruction will be pushed onto the stack. So that when Callee is done executing the instructions, it can look up the this value and know where to go back.The entire main() stack frame is till this step. EBP always points to the start of the stack frame.


Now the first thing the subroutine() going to do is that it takes the EBP, the pointer which points to the top of the local variables and save it under the stack. So when it's done and ready to destroy its own stack frame, it takes the stack pointer and put backs to EBP so that it again points to mains frame instead of its own. Then the subroutine() takes any callee-save registers, it will go ahead and save them.Stack frames are a linked list 

If subroutine() wants subroutine2(), subroutine has to go through the same steps.

After subroutine() is done, it will clean all parameter, caller-save register, local variables,callee-save registers in sequential order and it takes the saved frame pointer and replace EBP with this value. So that it will point to main() frame. Then all the parameters and local variables in main() will get cleaned. 




CPUID
CPU feature identification
Tells what features your pc currently supports eg: virtualization
Doesn’t have operands
Before issue cpuid eax should be set
After issuing cpuid eax,ebx,ecx,edx are going to be overridden
PUSHFD
Push EFlags onto the stack
POPFD
POP stack into EFlags


Control Flow

Conditional  - if, switch,loops
Unconditional - calls, goto,exceptions,interrupts


Inline Assembly

Syntax intel

__asm{
    Mov eax, [esp + 0x4]
}

Syntax AT&T

asm(“push EBP \n”
“Mov ESP , EBP\n”
);



Compliling
Gcc -o <output> <input>
Gcc -o file file.c
-ggdb ->to add debug symbols

Buffer Overflow
