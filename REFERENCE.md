---

# pForth Reference Manual

---

### pForth - a Portable ANSI style Forth written in ANSI 'C'.

### **Last updated: July 21, 2016 V23**

by Phil Burk with Larry Polansky,
David Rosenboom. Special thanks to contributors Darren Gibbs, Herb Maeder,
Gary Arakaki, Mike Haas.

Back to [pForth Home Page](../pforth)

## LEGAL NOTICE

The pForth software code is dedicated to the public domain, and any third
party may reproduce, distribute and modify the pForth software code or
any derivative works thereof without any compensation or license. The pForth
software code is provided on an "as is" basis without any warranty of any
kind, including, without limitation, the implied warranties of merchantability
and fitness for a particular purpose and their equivalents under the laws
of any jurisdiction.

---

## Table of Contents

* [What is pForth?](#what-is)
* [Compiling pForth for your System](#Compiling-pForth-System)

+ [Description of Source Files](#Description-Files)

* [Running pForth](#Running-pForth)
* [ANSI Compliance](#ANSI-Compliance)
* [pForth Special Features](#pForth-Features)

+ [Compiling from a File - INCLUDE](#Compiling-File)
+ [Saving Precompiled Dictionaries](#Saving-Dictionaries)
+ [Creating Turnkey Applications](#Turnkey-Apps)
+ [Recompiling Code - ANEW INCLUDE?](#Recompiling-Code)
+ [Customising Forget with [FORGET]](#Customising-FORGET)
+ [Smart Conditionals](#Smart-Conditionals)
+ [Development Tools](#Development-Tools)

- [WORDS.LIKE](#WORDS.LIKE)
- [FILE?](#FILEQ)
- [SEE](#SEE)
- [Single Step Trace and Debug](#single-step-trace)

+ [Conditional Compilation
  - [IF] [ELSE] [THEN]](#Conditional-Compilation)
+ [Miscellaneous Handy Words](#Miscellaneous-Words)
+ [Local Variables { foo -- }](#Local-Variables)
+ ['C' like Structures. :STRUCT](#C-Structures)
+ [Vectorred execution - DEFER](#Vectorred-Execution)
+ [Floating Point](#Floating-Point)

* [pForth Design](#pForth-Design)

+ ['C' kernel](#C-kernel)
+ [Dictionary Structures](#Dictionary-Structures)

* [Compiling pForth](#Compiling-pForth)

+ [Compiler Options](#Compiler-Options)
+ [Building pForth on Supported
  Hosts](#Building-pForth-Hosts)
+ [Compiling for Embedded Systems](#Compiling-Embedded)
+ [Linking with Custom 'C' Functions](#Link-Custom-C)
+ [Testing your Compiled pForth](#Testing-pForth)

---

## What is pForth?

PForth is an ANSI style Forth designed to be portable across many platforms.
The 'P' in pForth stands for "Portable". PForth is based on a Forth kernel
written in ANSI standard 'C'.

### What is Forth?

Forth is a stack based language invented by astronomer Charles Moore for
controlling telescopes. Forth is an interactive language. You can enter
commands at the keyboard and have them be immediately executed, similar
to BASIC or LISP. Forth has a dictionary of words that can be executed
or used to construct new words that are then added to the dictionary. Forth
words operate on a data stack that contains numbers and addresses.

To learn more about Forth, see the [Forth Tutorial](pf_tut.php).

### The Origins of pForth

PForth began as a JSR threaded 68000 Forth called HForth that was used
to support [HMSL](/hmsl/), the Hierarchical Music Specification Language. HMSL was
a music experimentation language developed by Phil Burk, Larry Polansky
and David Rosenboom while working at the Mills College Center for Contemporary
Music. Phil moved from Mills to the 3DO Company where he ported the Forth
kernel to 'C'. It was used extensively at 3DO as a tool for verifying ASIC design and
for bringing up new hardware platforms. At 3DO, the Forth had to run on
many systems including SUN, SGI, Macintosh, PC, Amiga, the 3DO ARM based
Opera system, and the 3DO PowerPC based M2 system.

### pForth Design Goals

PForth has been designed with portability as the primary design goal. As
a result, pForth avoids any fancy UNIX calls. pForth also avoids using
any clever and original ways of constructing the Forth dictionary. It just
compiles its kernel from ANSI compatible 'C' code then loads ANS compatible
Forth code to build the dictionary. Very boring but very likely to work
on almost any platform.

The dictionary files that can be saved from pForth are almost host independent.
They can be compiled on one processor, and then run on another processor.
as long as the endian-ness is the same. In other words, dictionaries built
on a PC will only work on a PC. Dictionaries built on almost any other
computer will work on almost any other computer.

PForth can be used to bring up minimal hardware systems that have very
few system services implemented. It is possible to compile pForth for systems
that only support routines to send and receive a single character. If malloc()
and free() are not available, equivalent functions are available in standard
'C' code. If file I/O is not available, the dictionary can be saved as
a static data array in 'C' source format on a host system. The dictionary
in 'C' source form is then compiled with a custom pForth kernel to avoid
having to read the dictionary from disk.

---

## Compiling pForth for your System

Up-to-date instructions on compiling, possibly with comments from the community, may be found at:

> <https://github.com/philburk/pforth/wiki/Compiling-on-Unix>

The process of building pForth involves several steps. This process is
typically handled automatically by the Makefile or IDE Project.

1. Compile the 'C' based pForth kernel called "pforth" or "pforth.exe".
2. Execute "pforth" with the -i option to build the dictionary from scratch.
   Compile the "system.fth" file which will add all the top level Forth words.
   This can be done in one command by entering "pforth -i system.fth".
3. Save the compiled dictionary as "pforth.dic".
4. The next time you run pforth, the precompiled pforth.dic file will be loaded
   automatically.

### Unix and Max OS X

A Makefile has been provided that should work on most Unix based platforms.

1. cd to "platforms/unix" folder.
2. Enter: make all
3. Enter: ./pforth

Note that the platforms folder used to be called build.

### Description of Source Files

#### Forth Source in /fth/

```
ansilocs.fth    = support for ANSI (LOCAL) word
c_struct.fth    = 'C' like data structures
case.fth        = CASE OF ENDOF ENDCASE
catch.fth       = CATCH and THROW
condcomp.fth    = [IF] [ELSE] [THEN] conditional compiler
filefind.fth    = FILE?
floats.fth      = floating point support
forget.fth      = FORGET [FORGET] IF.FORGOTTEN
loadp4th.fth    = loads basic dictionary
locals.fth      = { } style locals using (LOCAL)
math.fth        = misc math words
member.fth      = additional 'C' like data structure support
misc1.fth       = miscellaneous words
misc2.fth       = miscellaneous words
numberio.fth    = formatted numeric input/output
private.fth     = hide low level words
quit.fth        = QUIT EVALUATE INTERPRET in high level
smart_if.fth    = allows conditionals outside colon definition
see.fth         = Forth "disassembler".  Eg.  SEE SPACES
strings.fth     = string support
system.fth      = bootstraps pForth dictionary
trace.fth       = single step trace for debugging
```

#### 'C' Source in /csrc/

```

pfcompil.c  = pForth compiler support
pfcustom.c  = example of 'C' functions callable from pForth
pfinnrfp.h  = float extensions to interpreter
pforth.h    = include this in app that embeds pForth
pf_cglue.c  = glue for pForth calling 'C'
pf_clib.c   = replacement routines for 'C' stdlib
pf_core.c   = primary words called from 'C' app that embeds pForth
pf_float.h  = defines PF_FLOAT, and the floating point math functions such as fp_sin
pf_inner.c  = inner interpreter
pf_guts.h   = primary include file, define structures
pf_io.c     = input/output
pf_main.c   = basic application for standalone pForth
pf_mem.c    = optional malloc() implementation
pf_save.c   = save and load dictionaries
pf_text.c   = string tools, error message text
pf_words.c  = miscellaneous pForth words implemented

```

---

## Running pForth

PForth can be run from a shell or by double clicking on its icon, depending
on the system you are using. The execution options for pForth are described
assuming that you are running it from a shell.

Usage:

```
pforth [-i] [-dDictionaryFilename] [SourceFilename]
```

-i
:   Initialize pForth by building dictionary from scratch. Used when building
    pForth or when debugging pForth on new systems.

-dDictionaryFilename
:   Specify a custom dictionary to be loaded in place of the default "pforth.dic".
    For example:

```
pforth -dgame.dic
```

SourceFilename
:   A Forth source file can be automatically compiled by passing its name to
    pForth. This is useful when using Forth as an assembler or for automated
    hardware testing. Remember that the source file can compile code and execute
    it all in the same file.

#### Quick Verification of pForth

To verify that PForth is working, enter:

```
3 4 + .
```

It should print "7 ok". Now enter:

WORDS

You should see a long list of all the words in the pForth dictionary. Don't
worry. You won't need to learn all of these.  More tests are described
in the README.txt file.

If you want to learn how to program in Forth, try our [tutorial](pf_tut.php).

---

## ANSI Compliance

This Forth is intended to be ANS compatible. I will not claim that it is
compatible until more people bang on it. If you find areas where it deviates
from the standard, please let me know.

Word sets supported include:

* FLOAT
* LOCAL with support for { lv1 lv2 | lv3 -- } style locals
* EXCEPTION but standard throw codes not implemented
* FILE ACCESS
* MEMORY ALLOCATION

Here are the areas that I know are not compatible:

The ENVIRONMENT queries are not implemented.

Word sets NOT supported include:

* BLOCK - a matter of religion
* SEARCH ORDER
* PROGRAMMING TOOLS - only has .S ? DUMP WORDS BYE
* STRING - only has CMOVE CMOVE> COMPARE
* DOUBLE NUMBER - but cell is 32 bits

---

## pForth Special Features

These features are not part of the ANS standard for Forth.  They have
been added to assist developers.

### Compiling from a File

Use INCLUDE to compile source code from a file:

```
INCLUDE filename
```

You can nest calls to INCLUDE. INCLUDE simply redirects Forth to takes
its input from the file instead of the keyboard so you can place any legal
Forth code in the source code file.

### Saving Precompiled Dictionaries

Use SAVE-FORTH save your precompiled code to a file. To save the current
dictionary to a file called "custom.dic", enter:

```
c" custom.dic" SAVE-FORTH
```

You can then leave pForth and use your custom dictionary by entering:

```
pforth -dcustom.dic
```

On icon based systems, you may wish to name your custom dictionary "pforth.dic"
so that it will be loaded automatically.

Be careful that you do not leave absolute addresses stored in the dictionary
because they will not work when you reload pForth at a different address.
Use A! to store an address in a variable in a relocatable form and A@ to
get it back if you need to.

```
VARIABLE DATA-PTR
CREATE DATA 100 ALLOT
DATA DATA-PTR !    \ storing absolute address!  BAD
DATA DATA-PTR A!   \ storing relocatable address!  GOOD
DATA-PTR A@        \ fetch relocatable address
```

### Creating Turnkey Applications

Use TURNKEY to save a dictionary with a word that will run automatically.
The headers (names) will be discarded to save space in the dictionary.
Suppose you have defined a word called MYAPP to prints the ASCII code when you press a
key on the keyboard.

```
: MYAPP ( -- , print key codes )
    BEGIN ." #" key dup ascii q = not
    WHILE . cr REPEAT ;
```

Save a dictionary named "turnkey.dic" that will run MYAPP. Other names are OK.

```
c" turnkey.dic"  ' MYAPP  TURNKEY
```

Run the app. Press some letters to see the code. Then press 'q' to exit.

```
./pforth -dturnkey.dic
```

### Recompiling Code - ANEW INCLUDE?

When you are testing a file full of code, you will probably recompile many
times. You will probably want to FORGET the old code before loading the
new code. You could put a line at the beginning of your file like this:

```
FORGET XXXX-MINE     : XXXX-MINE ;
```

This would automatically FORGET for you every time you load. Unfortunately,
you must define XXXX-MINE before you can ever load this file. We have a
word that will automatically define a word for you the first time, then
FORGET and redefine it each time after that. It is called ANEW and can
be found at the beginning of most Forth source files. We use a prefix of
TASK- followed by the filename just to be consistent. This TASK-name word
is handy when working with INCLUDE? as well. Here is an example:

```
\ Start of file
INCLUDE? TASK-MYTHING.FTH MYTHING.FTH
ANEW TASK-THISFILE.FTH
\ the rest of the file follows...
```

Notice that the INCLUDE? comes before the call to ANEW so that we don't
FORGET MYTHING.FTH every time we recompile.

FORGET allows you to get rid of code that you have already compiled.
This is an unusual feature in a programming language. It is very convenient
in Forth but can cause problems. Most problems with FORGET involve leaving
addresses that point to the forgotten code that are not themselves forgotten.
This can occur if you set a deferred system word to your word then FORGET
your word. The system word which is below your word in the dictionary is
pointing up to code that no longer exists. It will probably crash if called.
(See discussion of DEFER below.) Another problem is if your code allocates
memory, opens files, or opens windows. If your code is forgotten you may
have no way to free or close these thing. You could also have a problems
if you add addresses from your code to a table that is below your code.
This might be a jump table or data table.

Since this is a common problem we have provided a tool for handling
it. If you have some code that you know could potentially cause a problem
if forgotten, then write a cleanup word that will eliminate the problem.
This word could UNdefer words, free memory, etc. Then tell the system to
call this word if the code is forgotten. Here is how:

```
: MY.CLEANUP  ( -- , do whatever )
    MY-MEM @ FREE DROP
    0 MY-MEM !
;
IF.FORGOTTEN  MY.CLEANUP
```

IF.FORGOTTEN creates a linked list node containing your CFA that is checked
by FORGET. Any nodes that end up above HERE (the Forth pointer to the top
of the dictionary) after FORGET is done are executed.

### Customising FORGET with [FORGET]

Sometimes, you may need to extend the way that FORGET works. FORGET is
not deferred, however, because that could cause some real problems. Instead,
you can define a new version of [FORGET] which is searched for and executed
by FORGET. You MUST call [FORGET] from your program or FORGET will not
actually FORGET. Here is an example.

```
: [FORGET]  ( -- , my version )
    ." Change things around!" CR
    [FORGET]  ( must be called )
    ." Now put them back!" CR
;
: FOO ." Hello!" ;
FORGET FOO  ( Will print "Change things around!", etc.)
```

This is recommended over redefining FORGET because words like ANEW that
call FORGET will now pick up your changes.

### Smart Conditionals

In pForth, you can use IF THEN DO LOOP and other conditionals outside of
colon definitions. PForth will switch temporarily into the compile state,
then automatically execute the conditional code. (Thank you Mitch Bradley)
For example, just enter this at the keyboard.

```
10 0 DO I . LOOP
```

### Development Tools

#### WORDS.LIKE

If you cannot remember the exact name of a word, you can use WORDS.LIKE
to search the dictionary for all words that contain a substring. For an
example, enter:

```
WORDS.LIKE   FOR
WORDS.LIKE   EMIT
```

#### FILE?

You can use FILE? to find out what file a word was compiled from. If a
word was defined in multiple files then it will list each file. The execution
token of each definition of the word is listed on the same line.

```
FILE? IF
FILE? AUTO.INIT
```

#### SEE

You can use SEE to "disassemble" a word in the pForth dictionary. SEE will
attempt to print out Forth source in a form that is similar to the source
code. SEE will give you some idea of how the word was defined but is not
perfect. Certain compiler words, like BEGIN and LITERAL, are difficult
to disassemble and may not print properly. For an example, enter:

```
SEE SPACES
SEE WORDS
```

#### Single Step Trace and Debug

It is often useful to proceed step by step through your code when debugging.
PForth provides a simple single step trace facility for this purpose.
Here is an example of using TRACE to debug a simple program.  Enter
the following program:

```
: SQUARE ( n -- n**2 )
    DUP  *
;
: TSQ  ( n -- , test square )
    ." Square of "   DUP   .
    ." is "   SQUARE   .   CR
;
```

Even though this program should work, let's pretend it doesn't and try
to debug it.  Enter:

7  TRACE  TSQ

You should see:

```
7 trace tsq
<<  TSQ +0           <10:1> 7             ||  (.")  Square of "          >>    ok
```

The "TSQ +0" means that you are about to execute code at an offset of "+0"
from the beginning of TSQ.  The <10:1> means that we are in base
10, and that there is 1 item on the stack, which is shown to be "7". The
(.") is the word that is about to be executed.  (.") is the word that
is compiled when use use .".  Now to single step, enter:

```
s
```

You should see:

```
Square of
<<  TSQ +16          <10:1> 7             ||  DUP                         >>    ok
```

The "Square os" was printed by (."). We can step multiple times using the "sm" command. Enter:

```
3 sm
```

You should see:

```
<<  TSQ +20          <10:2> 7 7           ||  .                         >> 7
<<  TSQ +24          <10:1> 7             ||  (.")  is "                >> is
<<  TSQ +32          <10:1> 7             ||  SQUARE                    >>    ok
```

The "7" after the ">>" was printed by the . word. If we entered "s", we
would
step over the SQUARE word. If we want to dive down into SQUARE, we can
enter:

```
sd
```

You should see:

```
<<  SQUARE +0        <10:1> 7             ||    DUP                     >>    ok
```

To step once in SQUARE, enter:

```
s
```

You should see:

```
<<  SQUARE +4        <10:2> 7 7           ||    *                        >>    ok
```

To go to the end of the current word, enter:

```
g
```

You should see:

```
<<  SQUARE +8        <10:1> 49            ||    EXIT                      >>
<<  TSQ +36          <10:1> 49            ||  .                           >>    ok
```

EXIT is compiled at the end of every Forth word. For more information on
TRACE, enter TRACE.HELP:

```
TRACE  ( i*x <name> -- , setup trace for Forth word )
S      ( -- , step over )
SM     ( many -- , step over many times )
SD     ( -- , step down )
G      ( -- , go to end of word )
GD     ( n -- , go down N levels from current level,
                stop at end of this level )
```

### Conditional Compilation [IF] [ELSE] [THEN]

PForth supports conditional compilation words similar to 'C''s #if, #else,
and #endif.

[IF] ( flag -- , if true, skip to [ELSE] or [THEN] )

[ELSE] ( -- , skip to [THEN] )

[THEN] ( -- , noop, used to terminate [IF] and [ELSE] section )

For example:

```
TRUE constant USE_FRENCH

USE_FRENCH  [IF]
  : WELCOME  ." Bienvenue!" cr ;
[ELSE]
  : WELCOME  ." Welcome!" cr ;
[THEN]
```

Here is how to conditionally compile within a colon definition by using
[ and ].

```
: DOIT  ( -- )
    START.REACTOR
    IF
        [ USE_FRENCH [IF] ] ." Zut alors!"
        [ [ELSE] ] ." Uh oh!"
        [THEN]
    THEN cr
;
```

### Miscellaneous Handy Words

.HEX ( n -- , print N as hex number )

CHOOSE ( n -- rand , select random number between 0 and N-1 )

MAP ( -- , print dictionary information )

### Local Variables { foo --}

In a complicated Forth word it is sometimes hard to keep track of where
things are on the stack. If you find you are doing a lot of stack operations
like DUP SWAP ROT PICK etc. then you may want to use local variables. They
can greatly simplify your code. You can declare local variables for a word
using a syntax similar to the stack diagram. These variables will only
be accessible within that word. Thus they are "local" as opposed to "global"
like regular variables. Local variables are self-fetching. They automatically
put their values on the stack when you give their name. You don't need
to @ the contents. Local variables do not take up space in the dictionary.
They reside on the return stack where space is made for them as needed.
Words written with them can be reentrant and recursive.

Consider a word that calculates the difference of two squares, Here
are two ways of writing the same word.

```
: DIFF.SQUARES ( A B -- A*A-B*B )
    DUP *
    SWAP DUP *
    SWAP -
;
  ( or )
: DIFF.SQUARES { A B -- A*A-B*B }
    A A *
    B B * -
;
3 2 DIFF.SQUARES  ( would return 5 )
```

In the second definition of DIFF.SQUARES the curly bracket '{' told the
compiler to start declaring local variables. Two locals were defined, A
and B. The names could be as long as regular Forth words if desired. The
"--" marked the end of the local variable list. When the word is executed,
the values will automatically be pulled from the stack and placed in the
local variables. When a local variable is executed it places its value
on the stack instead of its address. This is called self-fetching. Since
there is no address, you may wonder how you can store into a local variable.
There is a special operator for local variables that does a store. It looks
like -> and is pronounced "to".

Local variables need not be passed on the stack. You can declare a local
variable by placing it after a "vertical bar" ( | )character. These are
automatically set to zero when created. Here is a simple example that uses
-> and | in a word:

```
: SHOW2*
        { loc1 | unvar --  , 1 regular, 1 uninitialized }
        LOC1  2*  ->  UNVAR
                (set unver to 2*LOC1 )
        UNVAR   .   ( print UNVAR )
;
3 SHOW2*   ( pass only 1 parameter, prints 6 )
```

Since local variable often used as counters or accumulators, we have a
special operator for adding to a local variable It is +-> which is pronounced
"plus to". These next two lines are functionally equivalent but the second
line is faster and smaller:

```
ACCUM   10 +   -> ACCUM
10 +-> ACCUM
```

If you name a local variable the same as a Forth word in the dictionary,
eg. INDEX or COUNT, you will be given a warning message. The local variable
will still work but one could easily get confused so we warn you about
this. Other errors that can occur include, missing a closing '}', missing
'--', or having too many local variables.

### 'C' like Structures. :STRUCT

You can define 'C' like data structures in pForth using :STRUCT. For example:

```
:STRUCT  SONG
    LONG     SONG_NUMNOTES  \ define 32 bit structure member named SONG_NUMNOTES
    SHORT    SONG_SECONDS   \ define 16 bit structure member
    BYTE     SONG_QUALITY   \ define 8 bit member
    LONG     SONG_NUMBYTES  \ auto aligns after SHORT or BYTE
    RPTR     SONG_DATA      \ relocatable pointer to data
;STRUCT
```

```
SONG  HAPPY   \ define a song structure called happy
```

```
400  HAPPY  S!  SONG_NUMNOTES  \ set number of notes to 400
17   HAPPY  S!  SONG_SECONDS   \ S! works with all size members
```

```
CREATE  SONG-DATA  23 , 17 , 19 , 27 ,
SONG-DATA  HAPPY S! SONG_DATA  \ store pointer in relocatable form
```

```
HAPPY  DST  SONG    \ dump HAPPY as a SONG structure
```

```
HAPPY   S@  SONG_NUMNOTES .  \ fetch numnotes and print
```

See the file "c\_struct.fth" for more information.

### Vectorred Execution - DEFER

Using DEFER for vectored words. In Forth and other languages you can save
the address of a function in a variable. You can later fetch from that
variable and execute the function it points to.This is called vectored
execution. PForth provides a tool that simplifies this process. You can
define a word using DEFER. This word will contain the execution token of
another Forth function. When you execute the deferred word, it will execute
the function it points to. By changing the contents of this deferred word,
you can change what it will do. There are several words that support this
process.

DEFER ( <name> -- , define a deferred word )

IS ( CFA <name> -- , set the function for a deferred word )

WHAT'S ( <name> -- CFA , return the CFA set by IS )

Simple way to see the name of what's in a deferred word:

```
WHAT'S EMIT >NAME ID.
```

should print name of current word that's in EMIT.

Here is an example that uses a deferred word.

```
DEFER PRINTIT
' . IS PRINTIT   ( make PRINTIT use . )
8 3 + PRINTIT

: COUNTUP  ( -- , call deferred word )
        ." Hit RETURN to stop!" CR
        0 ( first value )
        BEGIN 1+ DUP PRINTIT CR
                ?TERMINAL
        UNTIL
;
COUNTUP  ( uses simple . )

: FANCY.PRINT  ( N -- , print in DECIMAL and HEX)
        DUP ." DECIMAL = " .
        ." , HEX = " .HEX
;
' FANCY.PRINT  IS PRINTIT  ( change printit )
WHAT'S PRINTIT >NAME ID. ( shows use of WHAT'S )
8 3 + PRINTIT
COUNTUP  ( notice that it now uses FANCY.PRINT )
```

Many words in the system have been defined using DEFER which means that
we can change how they work without recompiling the entire system. Here
is a partial list of those words

```
ABORT EMIT NUMBER?
```

#### Potential Problems with Defer

Deferred words are very handy to use, however, you must be careful with
them. One problem that can occur is if you initialize a deferred system
more than once. In the below example, suppose we called STUTTER twice.
The first time we would save the original EMIT vector in OLD-EMIT and put
in a new one. The second time we called it we would take our new function
from EMIT and save it in OLD-EMIT overwriting what we had saved previously.
Thus we would lose the original vector for EMIT . You can avoid this if
you check to see whether you have already done the defer. Here's an example
of this technique.

```
DEFER OLD-EMIT
' QUIT  IS OLD-EMIT  ( set to known value )
: EEMMIITT  ( char --- , our fun EMIT )
    DUP OLD-EMIT OLD-EMIT
;
: STUTTER   ( --- )
    WHAT'S OLD-EMIT  'C QUIT =  ( still the same? )
    IF  ( this must be the first time )
        WHAT'S EMIT  ( get the current value of EMIT )
        IS OLD-EMIT  ( save this value in OLD-EMIT )
        'C EEMMIITT IS EMIT
    ELSE ."  Attempt to STUTTER twice!" CR
    THEN
;
: STOP-IT!  ( --- )
    WHAT'S OLD-EMIT ' QUIT =
    IF  ." STUTTER not installed!" CR

    ELSE  WHAT'S OLD-EMIT IS EMIT
        'C QUIT IS OLD-EMIT
                ( reset to show termination )
    THEN
;
```

In the above example, we could call STUTTER or STOP-IT! as many times as
we want and still be safe.

Suppose you forget your word that EMIT now calls. As you compile new
code you will overwrite the code that EMIT calls and it will crash miserably.
You must reset any deferred words that call your code before you FORGET
your code. The easiest way to do this is to use the word IF.FORGOTTEN to
specify a cleanup word to be called if you ever FORGET the code in question.
In the above example using EMIT , we could have said:

```
IF.FORGOTTEN STOP-IT!
```

### Floating Point

PForth supports the FLOAT word set and much of the FLOATEXT word set as
a compile time option.  You can select single or double precision
as the default by changing the typedef of PF\_FLOAT.

PForth has several options for floating point output.

FS. ( r -f- , prints in scientific/exponential format )

FE. ( r -f- , prints in engineering format, exponent if multiple of 3  )

FG. ( r -f- , prints in normal or exponential format depending on size )

F. ( r -f- , as defined by the standard )

Here is an example of output from each word for a number ranging from large to very small.

```
     FS.             FE.            FG.           F.
1.234000e+12     1.234000e+12     1.234e+12     1234000000000.
1.234000e+11     123.4000e+09     1.234e+11     123400000000.
1.234000e+10     12.34000e+09     1.234e+10     12340000000.
1.234000e+09     1.234000e+09     1.234e+09     1234000000.
1.234000e+08     123.4000e+06     1.234e+08     123400000.
1.234000e+07     12.34000e+06     1.234e+07     12340000.
1.234000e+06     1.234000e+06     1234000.     1234000.
1.234000e+05     123.4000e+03     123400.     123400.0
1.234000e+04     12.34000e+03     12340.     12340.00
1.234000e+03     1.234000e+03     1234.     1234.000
1.234000e+02     123.4000e+00     123.4     123.4000
1.234000e+01     12.34000e+00     12.34     12.34000
1.234000e+00     1.234000e+00     1.234     1.234000
1.234000e-01     123.4000e-03     0.1234     0.1234000
1.234000e-02     12.34000e-03     0.01234     0.0123400
1.234000e-03     1.234000e-03     0.001234     0.0012340
1.234000e-04     123.4000e-06     0.0001234     0.0001234
1.234000e-05     12.34000e-06     1.234e-05     0.0000123
1.234000e-06     1.234000e-06     1.234e-06     0.0000012
1.234000e-07     123.4000e-09     1.234e-07     0.0000001
1.234000e-08     12.34000e-09     1.234e-08     0.0000000
1.234000e-09     1.234000e-09     1.234e-09     0.0000000
1.234000e-10     123.4000e-12     1.234e-10     0.0000000
1.234000e-11     12.34000e-12     1.234e-11     0.0000000

1.234568e+12     1.234568e+12     1.234568e+12     1234567890000.
1.234568e+11     123.4568e+09     1.234568e+11     123456789000.
1.234568e+10     12.34568e+09     1.234568e+10     12345678900.
1.234568e+09     1.234568e+09     1.234568e+09     1234567890.
1.234568e+08     123.4568e+06     1.234568e+08     123456789.
1.234568e+07     12.34568e+06     1.234568e+07     12345679.
1.234568e+06     1.234568e+06     1234568.     1234568.
1.234568e+05     123.4568e+03     123456.8     123456.8
1.234568e+04     12.34568e+03     12345.68     12345.68
1.234568e+03     1.234568e+03     1234.568     1234.568
1.234568e+02     123.4568e+00     123.4568     123.4568
1.234568e+01     12.34568e+00     12.34568     12.34568
1.234568e+00     1.234568e+00     1.234568     1.234568
1.234568e-01     123.4568e-03     0.1234568     0.1234568
1.234568e-02     12.34568e-03     0.01234568     0.0123456
1.234568e-03     1.234568e-03     0.001234568     0.0012345
1.234568e-04     123.4568e-06     0.0001234568     0.0001234
1.234568e-05     12.34568e-06     1.234568e-05     0.0000123
1.234568e-06     1.234568e-06     1.234568e-06     0.0000012
1.234568e-07     123.4568e-09     1.234568e-07     0.0000001
1.234568e-08     12.34568e-09     1.234568e-08     0.0000000
1.234568e-09     1.234568e-09     1.234568e-09     0.0000000
1.234568e-10     123.4568e-12     1.234568e-10     0.0000000
1.234568e-11     12.34568e-12     1.234568e-11     0.0000000

```

```

---

```

## pForth Design

### 'C' kernel

The pForth kernel is written in 'C' for portability. The inner interpreter
is implemented in the function ExecuteToken() which is in pf\_inner.c.

```
void pfExecuteToken( ExecToken XT );
```

It is passed an execution token the same as EXECUTE would accept. It handles
threading of secondaries and also has a large switch() case statement to
interpret primitives. It is in one huge routine to take advantage of register
variables, and to reduce calling overhead. Hopefully, your compiler will
optimise the switch() statement into a jump table so it will run fast.

### Dictionary Structures

This Forth supports multiple dictionaries. Each dictionary consists of
a header segment and a seperate code segment. The header segment contains
link fields and names. The code segment contains tokens and data. The headers,
as well as some entire dictionaries such as the compiler support words,
can be discarded when creating a stand-alone app.

[NOT IMPLEMENTED] Dictionaries can be split so that the compile time
words can be placed above the main dictionary. Thus they can use the same
relative addressing but be discarded when turnkeying.

Execution tokens are either an index of a primitive ( n < NUM\_PRIMITIVES),
or the offset of a secondary in the code segment. ( n >= NUM\_PRIMITIVES
)

The NAME HEADER portion of the dictionary contains a structure for each
named word in the dictionary. It contains the following fields:

```
bytes
4 Link Field = relative address of previous name header
4 Code Pointer = relative address of corresponding code
n Name Field = name as counted string Headers are quad byte aligned.
```

The CODE portion of the dictionary consists of the following structures:

#### Primitive

No Forth code. 'C' code in "pf\_inner.c".

#### Secondary

```
4*n Parameter Field containing execution tokens
4 ID_NEXT = 0 terminates secondary
```

#### CREATE DOES>

```
4 ID_CREATE_P token
4 Token for optional DOES> code, OR ID_NEXT = 0
4 ID_NEXT = 0
n Body = arbitrary data
```

#### Deferred Word

```
4 ID_DEFER_P same action as ID_NOOP, identifies deferred words
4 Execution Token of word to execute.
4 ID_NEXT = 0
```

#### Call to custom 'C' function.

```
4 ID_CALL_C
4 Pack C Call Info Bits
```

```
0-15 = Function Index Bits
16-23 = FunctionTable Index (Unused) Bits
24-30 = NumParams Bit
31 = 1 if function returns value
```

```
4 ID_NEXT = 0
```

---

## Compiling pForth

A makefile is supplied that will help you compile pForth for your environment.
You can customize the build by setting various compiler options.

### Compiler Options

There are several versions of PForth that can be built. By default, the
full kernel will be built. For custom builds, define the following options
in the Makefile before compiling the 'C' code:

PF\_DEFAULT\_DICTIONARY="filename"
> Specify a dictionary to use in place of the default "pforth.dic",
> for example "/usr/lib/pforth/pforth.dic".

PF\_NO\_INIT

Don't compile the code used to initially build the dictionary. This
can be used to save space if you already have a prebuilt dictionary.
PF\_NO\_SHELL

Don't compile the outer interpreter and Forth compiler. This can be
used with Cloned dictionaries.
PF\_NO\_MALLOC

Replace malloc() and free() function with pForth's own version. See
pf\_mem.c for more details.
PF\_USER\_MALLOC='"filename.h"'

Replace malloc() and free() function with users custom version. See
pf\_mem.h for details.
PF\_MEM\_POOL\_SIZE=numbytes

Size of array in bytes used by pForth custom allocator.
PF\_NO\_GLOBAL\_INIT

Define this if you want pForth to not rely on initialization of global
variables by the loader. This may be required for some embedded systems
that may not have a fully functioning loader.  Take a look in "pfcustom.c"
for an example of its use.
PF\_USER\_INC1='"filename.h"'

File to include BEFORE other include files. Generally set to host dependent
files such as "pf\_mac.h".
PF\_USER\_INC2='"filename.h"'

File to include AFTER other include files. Generally used to #undef
and re#define symbols. See "pf\_win32.h" for an example.
PF\_NO\_CLIB

Replace 'C' lib calls like toupper and memcpy with pForth's own version.
This is useful for embedded systems.
PF\_USER\_CLIB='"filename.h"'

Rreplace 'C' lib calls like toupper and memcpy with users custom version.
See pf\_clib.h for details.
PF\_NO\_FILEIO

System does not support standard file I/O so stub it out. Setting this
flag will automatically set PF\_STATIC\_DIC.
PF\_USER\_CHARIO='"filename.h"'

Replace stdio terminal calls like getchar() and putchar() with users
custom version. See pf\_io.h for details.
PF\_USER\_FILEIO='"filename.h"'

Replace stdio file calls like fopen and fread with users custom version.
See pf\_io.h for details.
PF\_USER\_FLOAT='"filename.h"'

Replace floating point math calls like sin and pow with users custom
version. Also defines PF\_FLOAT.
PF\_USER\_INIT=MyInit()

Call a user defined initialization function that returns a negative
error code if it fails.
PF\_USER\_TERM=MyTerm()

Call a user defined void termination function.
PF\_STATIC\_DIC

Compile in static dictionary instead of loading dictionary. from file.
Use "utils/savedicd.fth" to save a dictionary as 'C' source code in a file
called "pfdicdat.h".
PF\_SUPPORT\_FP

Compile ANSI floating point support.

### Building pForth on Supported Hosts

To build on UNIX, do nothing, system will default to "pf\_unix.h".

To build on Macintosh:

```
-DPF_USER_INC1='"pf_mac.h"'
```

To build on PCs:

```
-DPF_USER_INC2='"pf_win32.h"'
```

To build a system that only runs turnkey or cloned binaries:

```
-DPF_NO_INIT -DPF_NO_SHELL
```

### Compiling for Embedded Systems

You may want to create a version of pForth that can be run on a small system
that does not support file I/O. This is useful when bringing up new computer
systems. On UNIX systems, you can use the supplied gmake target. Simply
enter:

```
gmake pfemb
```

For other systems, here are the steps to create an embedded pForth.

1. Determine whether your target system has a different endian-ness than your
   host system.  If the address of a long word is the address of the
   most significant byte, then it is "big endian". Examples of big endian
   processors are Sparc, Motorola 680x0 and PowerPC60x.  If the address
   of a long word is the address of the least significant byte, then it is
   "Little Endian". Examples of little endian processors are Intel 8088 and
   derivatives such as the Intel Pentium, X86. ARM processors can be configured as either big or little endian.
2. If your target system has a different endian-ness than your host system,
   then you must compile a version of pForth for your host that matches the
   target.  Rebuild pForth with either PF\_BIG\_ENDIAN\_DIC or PF\_LITTLE\_ENDIAN\_DIC
   defined.  You will need to rebuild pforth.dic as well as the executable
   Forth.  If you do not specify one of these variables, then the dictionary
   will match the native endian-ness of the host processor.
3. Execute pForth. Notice the message regarding the endian-ness of the dictionary.
4. Compile your custom Forth words on the host development system.
5. Compile the pForth utulity "utils/savedicd.fth".
6. Enter in pForth: SDAD
7. SDAD will generate a file called "pfdicdat.h" that contains your dictionary
   in source code form.
8. Rewrite the character primitives sdTerminalOut(), sdTerminalIn() and sdTerminalFlush()
   defined in pf\_io.h to use your new computers communications port.
9. Write a "user\_chario.h" file based on the API defined in "pf\_io.h".
10. Compile a new version of pForth for your target machine with the following
    options:

```
-DPF_NO_INIT -DPF_NO_MALLOC -DPF_NO_FILEIO \
-DPF_USER_CHARIO="user_chario.h" \
-DPF_NO_CLIB -DPF_STATIC_DIC
```

11. The file "pfdicdat.h" will be compiled into this executable and your dictionary
    will thus be included in the pForth executable as a static array.
12. Burn a ROM with your new pForth and run it on your target machine.
13. If you compiled a version of pForth with different endian-ness than your
    host system, do not use it for daily operation because it will be much
    slower than a native version.

### Linking with Custom 'C' Functions

You can call the pForth interpreter as an embedded tool in a 'C' application.
For an example of this, see the file pf\_main.c. This application does nothing
but load the dictionary and call the pForth interpreter.

You can call 'C' from pForth by adding your own custom 'C' functions
to a dispatch table, and then adding Forth words to the dictionary that
call those functions. See the file "pfcustom.c" for more information.

### Testing your Compiled pForth

Once you have compiled pForth, you can test it using the small verification
suite we provide.  The first test you should run was written by John
Hayes at John Hopkins University.  Enter:

```
pforth
include tester.fth
include coretest.fth
bye
```

The output will be self explanatory.  There are also a number of tests
that I have added that print the number of successes and failures. Enter:

```
pforth t_corex.fth
pforth t_locals.fth
pforth t_strings.fth
pforth t_floats.ft
```

Note that t\_corex.fth reveals an expected error because SAVE-INPUT is not
fully implemented. (FIXME)

---

PForth source code is freely available and is in the public domain.

Back to [pForth Home Page](../pforth)

(C) 1997-2015 Mobileer Inc - All Rights Reserved - [Contact Us](/contacts.php
