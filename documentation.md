#PLA documentation.md 
MIT license 2015 (C) Helmut Guenther. 
This is the main documentation file for developers and programmers and will be continued...
##Introduction
###Design considerations
I do not like segments and selectors in developing x86 software. I love *flat binary files* like the old CP/M or the DOS COM files. But you have only 64 KB for your text segment. For big data I have found a solution (see below). Even the PLA compiler needs only 26 KB for the code and works. There is no need for a linker. You load the program as it is into memory and starts at location 100h. Thats all.

Today the computers are so fast, that you need no precompiled libraries for small programs. On my MacBook Pro it takes only two seconds to compile, including the source libraries, writing a huge listing file with a cross reference listing and statistics. I put all library stuff in an *archive source file* **AR.C** and the compiler takes only the needed funcions to keep the binary small. The following netwide assembler needs more time to produce the binary. So I started to write an x86 assembler, it will be an COM file, too.

###Program Structure
As every C program, so PLA starts with the function **main**() (line 618 after the eye catcher). The main function is kept short and calls only some other functions. The last function called, except on errors,  is  **epilog**() (line 1011), which finishes with calling the function **end1**() in line 947. This function cloes all handles and calls **exitR**(). This function is in the archive file (line 157) and return to DOS. The letter R means that it is a real mode function. So I have a namepsace left for the protected mode function.
The main function start by calling the following routines:

1. **getarg**() line 628. This function gets the parameter from the command line and parses them. It write the help screen, opens the input and list file. The function checks protected mode and writes the header to the listing file.
2. **memresize**() line 1044. When a COM program starts, it reserves all the memory. As the maximum size of a COM program is only 64 Kbyte, we reserve only 4096 paragraphs, which are 64 kbyte. So we have place for another 64 kbyte, which will be addressed by the es register by the function **memalloc**() in line 1046. This is done to store the label and variable names. 
3. **getfirstchar**() line 625 is called to initialize the input buffer.
4. After sending a notice only to the screen "Compiling", **parse**() line 672 is called. this function has the endless loop *while(1)* in line 680. it returns to the main program, when all token are processed in line 673. The primary function is to look for a**#** and then calls the two preprocessors *dodefine* or *doinclude*. Otherwise it gets the next name. A PLA program consists either of a declaration of global variables or a function. If the name follows a **(**, then *dofunc*() is called, otherwise **doglob**() is called. The token **!** is a hack and calls the function **doLdata**() in line 110, which stores uninitialized data at the address *ldata*. This address starts at 2 MByte and stores big data in unreal mode.

###Variables
All names are case sensitive.
###Functions
###Statements
###Expressions
####pexpr()
####constantexpr()
####simplexpr()

###Debugging

##BA.BAT batch file
The DOS batch file calls A.COM with its own source code A.C. If there is no errror, it calls the assembler NASM wich outputs the binary fiele A.COM again. If you call the batch file again, you see, if the new generated file ist working. Otherwise you start thinking and a debug session. Take attention for the parameter -t for Turbo Assembler compatibility of the NASM file. My first assembler was TASM and now it is difficult to get rid of the TASM compatibility.
##AR.C archive source file
This file collects all handy, old and well tested (and not so well tested) routines in PLA language. I started collecting in the 1990 years and therefore it is sorted by date, starting with the oldest routines. There are mostly BIOS, DOS, direct I/O routines. Also parts of the C library with string routines, print and dump routines. It is divided in several blocks:

1. (Line 2) Commented out of key mapping variables for a German keyboard. The small German keyboar driver KeybGer() in line 57 needs these variables. If you use a German keyboard, copy these variables to your program, because PLA looks only for functions and not for variables in the archive file.
2. BSCREEN (line 13) The function writetty() Int 10h with some functions to output text and numbers without using DOS operating functions. I need the functions for writing error messages to the screen, befor I opened a file at start time. You need these files also, if you have no operating system.
3. KEYBOARD (line30) Int 16h BIOS routines. In function waitkey() means the expression *emit(0x74, 0xFA)* that jump zero to begin of the function, until a key is pressed. In function kbhit (line 34) the last expression **0;**  compiles to *mov eax, 0* and is the retunr value.
4. TURBOIO (line 35) Fast BIOS screen routines.
5. PROMPT (line 57) BIOS keyboard input routines with mapping a limited set of correcting the input string befor pressing the *return* key. 
6. BOX (line 65) signing a box on the screen with BIOS routines.
7. RTC (line 82) get the BIOS time and wait routines.
8. DIRECTIO (line 89) Direct write to the screen with the help of the es register, which is saved and *not* changed.
9. STRING (line 96) Some of the C library string routines. Some are untested and waiting to be tested.
10. DUMP (line 132) Two routines for register and memory dump. There are not a real debugger, but helped me in many situations.
11. DOSINT (line 147) This is the main function for the following DOS fucntions and is called by them. The PLA statement *ifcarry* increment the vraiable *DOS_ERR*. This **must** be the first statement after a DOS call, because otherwise the *carry flag* gets lost.
12. ATOI (line173) A family of functions to convert from ascii to integer. The function ultoa2() is a test bed for 32 bit processing.

