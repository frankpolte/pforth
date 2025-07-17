---

# Forth Tutorial

---

Translations: [Chinese](http://vision.twbbs.org/~letoh/forth/pf_tuttw.html) by [Letoh](http://vision.twbbs.org/~letoh/blog/?page_id=169)

by [Phil Burk](http://www.softsynth.com/philburk.html)
of [SoftSynth.com](http://www.softsynth.com)

## Table of Contents

* [Forth Syntax](#Forth Syntax)
* [Stack Manipulation](#The Stack)
* [Arithmetic](#Arithmetic)
* [Defining a New Word](#Defining a New Word)
* [More Arithmetic](#More Arithmetic)

+ [Arithmetic Overflow](#Arithmetic Overflow)
+ [Convert Algebraic Expressions
  to Forth](#Convert Algebraic Expressions to Forth)

* [Character Input and Output](#Character Input and Output)
* [Compiling from Files](#Compiling from Files)
* [Variables](#Variables)
* [Constants](#Constants)
* [Logical Operators](#Logical Operators)
* [Conditionals - IF ELSE THEN
  CASE](#Conditionals - IF ELSE THEN CASE)
* [Loops](#Loops)
* [Text Input and Output](#Text Input and Output)
* [Changing Numeric Base](#Changing Numeric Base)
* [Answers to Problems](#Answers to Problems)

The intent of this tutorial is to provide a series of experiments that
will introduce you to the major concepts of Forth. It is only a starting
point. Feel free to deviate from the sequences I provide. A free form investigation
that is based on your curiosity is probably the best way to learn any language.
Forth is especially well adapted to this type of learning.

This tutorial is written for the PForth implementation of the ANS Forth
standard. I have tried to restrict this tutorial to words that are part
of the ANS standard but some PForth specific words may have crept in.

In the tutorials, I will print the things you need to type in upper
case, and indent them. You can enter them in upper or lower case. At the
end of each line, press the RETURN (or ENTER) key; this causes Forth to
interpret what you've entered.

## Forth Syntax

Forth has one of the simplest syntaxes of any computer language. The syntax
can be stated as follows, "**Forth code is a bunch of words with spaces
between them.**" This is even simpler than English! Each *word*
is equivalent to a function or subroutine in a language like 'C'. They
are executed in the order they appear in the code. The following statement,
for example, could appear in a Forth program:

```
 WAKE.UP EAT.BREAKFAST WORK EAT.DINNER PLAY SLEEP
```

Notice that WAKE.UP has a dot between the WAKE and UP. The dot has no particular
meaning to the Forth compiler. I simply used a dot to connect the two words
together to make one word. Forth word names can have any combination of
letters, numbers, or punctuation. We will encounter words with names like:

```
 ." #S SWAP ! @ ACCEPT . *
```

They are all called *words*. The word **$%%-GL7OP** is a legal
Forth name, although not a very good one. It is up to the programmer to
name words in a sensible manner.

Now it is time to run your Forth and begin experimenting. Please consult
the manual for your Forth for instructions on how to run it.

## Stack Manipulation

The Forth language is based on the concept of a *stack*. Imagine a
stack of blocks with numbers on them. You can add or remove numbers from
the top of the stack. You can also rearrange the order of the numbers.
Forth uses several stacks. The *DataStack* is the one used for passing
data between Forth words so we will concentrate our attention there. The
*Return Stack* is another Forth stack that is primarily for internal
system use. In this tutorial, when we refer to the "stack," we will be
referring to the Data Stack.

The stack is initially empty. To put some numbers on the stack, enter:

```
23 7 9182
```

Let's now print the number on top of the stack using the Forth word ' **.** ', which is pronounced " dot ". This is a hard word to write about in a
manual because it is a single period.

Enter: **.**

You should see the last number you entered, 9182 , printed. Forth has
a very handy word for showing you what's on the stack. It is **.S**
, which is pronounced "dot S". The name was constructed from "dot" for
print, and "S" for stack. (PForth will automatically print the stack after
every line if the TRACE-STACK variable is set to TRUE.) If you enter:

```
.S
```

you will see your numbers in a list. The number at the far right is the
one on top of the stack.

You will notice that the 9182 is not on the stack. The word ' . ' removes
the number on top of the stack before printing it. In contrast, ' .S '
leaves the stack untouched.

We have a way of documenting the effect of words on the stack with a
*stack diagram*. A stack diagram is contained in parentheses. In Forth,
the parentheses indicate a comment. In the examples that follow, you do
not need to type in the comments. When you are programming, of course,
we encourage the use of comments and stack diagrams to make your code more
readable. In this manual, we often indicate stack diagrams in **bold text**
like the one that follows. Do not type these in. The stack diagram for
a word like ' . ' would be:

**. ( N -- , print number on top of stack )**

The symbols to the left of -- describe the parameters that a word expects
to process. In this example, N stands for any integer number. To the right
of --, up to the comma, is a description of the stack parameters when the
word is finished, in this case there are none because 'dot' "eats" the
N that was passed in. (Note that the stack descriptions are not necessary,
but they are a great help when learning other peoples programs.)

The text following the comma is an English description of the word.
You will note that after the -- , N is gone. You may be concerned about
the fact that there were other numbers on the stack, namely 23 and 7 .
The stack diagram, however, only describes the portion of the stack that
is affected by the word. For a more detailed description of the stack diagrams,
there is a special section on them in this manual right before the main
glossary section.

Between examples, you will probably want to clear the stack. If you
enter **0SP**, pronounced "zero S P", then the stack will be cleared.

Since the stack is central to Forth, it is important to be able to alter
the stack easily. Let's look at some more words that manipulate the stack.
Enter:

```
0SP .S \ That's a 'zero' 0, not an 'oh' O.
777 DUP .S
```

You will notice that there are two copies of 777 on the stack. The word
**DUP** duplicates the top item on the stack. This is useful when you
want to use the number on top of the stack and still have a copy. The stack
diagram for DUP would be:

**DUP ( n -- n n , DUPlicate top of stack )**

Another useful word, is **SWAP**. Enter:

```
0SP
23 7 .S
SWAP .S
SWAP .S
```

The stack diagram for SWAP would be:

**SWAP ( a b -- b a , swap top two items on stack )**

Now enter:

```
OVER .S
OVER .S
```

The word **OVER** causes a copy of the second item on the stack to leapfrog
over the first. It's stack diagram would be:

**OVER ( a b -- a b a , copy second item on stack )**

Here is another commonly used Forth word:

**DROP ( a -- , remove item from the stack )**

Can you guess what we will see if we enter:

```
0SP 11 22 .S
DROP .S
```

Another handy word for manipulating the stack is **ROT**. Enter:

```
0SP
11 22 33 44 .S
ROT .S
```

The stack diagram for ROT is, therefore:

**ROT ( a b c -- b c a , ROTate third item to top )**

You have now learned the more important stack manipulation words. You
will see these in almost every Forth program. I should caution you that
if you see too many stack manipulation words being used in your code then
you may want to reexamine and perhaps reorganize your code. You will often
find that you can avoid excessive stack manipulations by using *local
or global VARIABLES* which will be discussed later.

If you want to grab any arbitrary item on the stack, use **PICK**
. Try entering:

```
0SP
14 13 12 11 10
3 PICK . ( prints 13 )
0 PICK . ( prints 10 )
4 PICK .
```

PICK makes a copy of the Nth item on the stack. The numbering starts with
zero, therefore:

0 PICK is equivalent to DUP

1 PICK is equivalent to OVER

**PICK ( ... v3 v2 v1 v0 N -- ... v3 v2 v1 v0 vN )**

(Warning. The Forth-79 and FIG Forth standards differ from the ANS and
Forth '83 standard in that their PICK numbering starts with one, not zero.)

I have included the stack diagrams for some other useful stack manipulation
words. Try experimenting with them by putting numbers on the stack and
calling them to get a feel for what they do. Again, the text in parentheses
is just a comment and need not be entered.

**DROP ( n -- , remove top of stack )**

**?DUP ( n -- n n | 0 , duplicate only if non-zero, '|' means OR
)**

**-ROT ( a b c -- c a b , rotate top to third position )**

**2SWAP ( a b c d -- c d a b , swap pairs )**

**2OVER ( a b c d -- a b c d a b , leapfrog pair )**

**2DUP ( a b -- a b a b , duplicate pair )**

**2DROP ( a b -- , remove pair )**

**NIP ( a b -- b , remove second item from stack )**

**TUCK ( a b -- b a b , copy top item to third position )**

### Problems:

Start each problem by entering:

```
0SP 11 22 33
```

Then use the stack manipulation words you have learned to end up with the
following numbers on the stack:

```
1) 11 33 22 22
```

```
2) 22 33
```

```
3) 22 33 11 11 22
```

```
4) 11 33 22 33 11
```

```
5) 33 11 22 11 22
```

[Answers to the problems](#Answers to Problems) can be found
at the end of this tutorial.

## Arithmetic

Great joy can be derived from simply moving numbers around on a stack.
Eventually, however, you'll want to do something useful with them. This
section describes how to perform arithmetic operations in Forth.

The Forth arithmetic operators work on the numbers currently on top
of the stack. If you want to add the top two numbers together, use the
Forth word **+** , pronounced "plus". Enter:

```
2 3 + .
2 3 + 10 + .
```

This style of expressing arithmetic operations is called *Reverse Polish
Notation,* or *RPN*. It will already be familiar to those of you
with HP calculators. In the following examples, I have put the algebraic
equivalent representation in a comment.

Some other arithmetic operators are **- \* /** . Enter:

```
30 5 - . ( 25=30-5 )
30 5 / . ( 6=30/5 )
30 5 * . ( 150=30*5 )
30 5 + 7 / . \ 5=(30+5)/7
```

Some combinations of operations are very common and have been coded in
assembly language for speed. For example, **2\*** is short for 2 \* .
You should use these whenever possible to increase the speed of your program.
These include:

```
1+ 1- 2+ 2- 2* 2/
```

Try entering:

```
10 1- .
7 2* 1+ . ( 15=7*2+1 )
```

One thing that you should be aware of is that when you are doing division
with integers using / , the remainder is lost. Enter:

```
15 5 / .
17 5 / .
```

This is true in all languages on all computers. Later we will examine **/MOD**
and **MOD** which do give the remainder.

## Defining a New Word

It's now time to write a *small program* in Forth. You can do this
by defining a new word that is a combination of words we have already learned.
Let's define and test a new word that takes the average of two numbers.

We will make use of two new words, **:** ( "colon"), and **;** (
"semicolon") . These words start and end a typical *Forth definition*.
Enter:

```
: AVERAGE ( a b -- avg ) + 2/ ;
```

Congratulations. You have just written a Forth program. Let's look more
closely at what just happened. The colon told Forth to add a new word to
its list of words. This list is called the Forth dictionary. The name of
the new word will be whatever name follows the colon. Any Forth words entered
after the name will be compiled into the new word. This continues until
the semicolon is reached which finishes the definition.

Let's test this word by entering:

```
10 20 AVERAGE . ( should print 15 )
```

Once a word has been defined, it can be used to define more words. Let's
write a word that tests our word.. Enter:

```
: TEST ( --) 50 60 AVERAGE . ;
TEST
```

Try combining some of the words you have learned into new Forth definitions
of your choice. If you promise not to be overwhelmed, you can get a list
of the words that are available for programming by entering:

```
WORDS
```

Don't worry, only a small fraction of these will be used directly in your
programs.

## More Arithmetic

When you need to know the remainder of a divide operation. /MOD will return
the remainder as well as the quotient. the word MOD will only return the
remainder. Enter:

```
0SP
53 10 /MOD .S
0SP
7 5 MOD .S
```

Two other handy words are **MIN** and **MAX** . They accept two numbers
and return the MINimum or MAXimum value respectively. Try entering the
following:

```
56 34 MAX .
56 34 MIN .
-17 0 MIN .
```

Some other useful words are:

**ABS ( n -- abs(n) , absolute value of n )**

**NEGATE ( n -- -n , negate value, faster then -1 \* )**

**LSHIFT ( n c -- n<<c , left shift of n )**

**RSHIFT ( n c -- n>>c , logical right shift of n )**

**ARSHIFT ( n c -- n>>c ) , arithmetic right shift of n )**

ARSHIFT or LSHIFT can be used if you have to multiply quickly by a power
of 2 . A right shift is like doing a divide by 2. This is often faster
than doing a regular multiply or divide. Try entering:

```
: 256* 8 LSHIFT ;
3 256* .
```

### Arithmetic Overflow

If you are having problems with your calculation overflowing the 32-bit
precision of the stack, then you can use **\*/** . This produces an intermediate
result that is 64 bits long. Try the following three methods of doing the
same calculation. Only the one using \*/ will yield the correct answer,
5197799.

```
34867312 99154 * 665134 / .
34867312 665134 / 99154 * .
34867312 99154 665134 */ .
```

#### Convert Algebraic Expressions to Forth

How do we express complex algebraic expressions in Forth? For example:
20 + (3 \* 4)

To convert this to Forth you must order the operations in the order
of evaluation. In Forth, therefore, this would look like:

```
3 4 * 20 +
```

Evaluation proceeds from left to right in Forth so there is no ambiguity.
Compare the following algebraic expressions and their Forth equivalents:
(Do **not** enter these!)

```
(100+50)/2 ==> 100 50 + 2/
((2*7) + (13*5)) ==> 2 7 * 13 5 * +
```

If any of these expressions puzzle you, try entering them one word at a
time, while viewing the stack with .S .

### Problems:

Convert the following algebraic expressions to their equivalent Forth expressions.
(Do **not** enter these because they are not Forth code!)

```
(12 * ( 20 - 17 ))
```

```
(1 - ( 4 * (-18) / 6) )
```

```
( 6 * 13 ) - ( 4 * 2 * 7 )
```

Use the words you have learned to write these new words:

```
SQUARE ( N -- N*N , calculate square )
```

```
DIFF.SQUARES ( A B -- A*A-B*B , difference of squares )
```

```
AVERAGE4 ( A B C D -- [A+B+C+D]/4 )
```

```
HMS>SECONDS ( HOURS MINUTES SECONDS -- TOTAL-SECONDS , convert )
```

[Answers to the problems](#Answers to Problems) can be found
at the end of this tutorial.

## Character Input and Output

The numbers on top of the stack can represent anything. The top number
might be how many blue whales are left on Earth or your weight in kilograms.
It can also be an ASCII character. Try entering the following:

```
72 EMIT 105 EMIT
```

You should see the word "Hi" appear before the OK. The 72 is an ASCII 'H'
and 105 is an 'i'. EMIT takes the number on the stack and outputs it as
a character. If you want to find the ASCII value for any character, you
can use the word ASCII . Enter:

```
CHAR W .
CHAR % DUP . EMIT
CHAR A DUP .
32 + EMIT
```

Here is a complete [ASCII chart](http://www.asciitable.com/).

Notice that the word CHAR is a bit unusual because its input comes not
from the stack, but from the following text. In a stack diagram, we represent
that by putting the input in angle brackets, <input>. Here is the stack
diagram for CHAR.

**CHAR ( <char> -- char , get ASCII value of a character )**

Using EMIT to output character strings would be very tedious. Luckily
there is a better way. Enter:

```
: TOFU ." Yummy bean curd!" ;
TOFU
```

The word **."** , pronounced "dot quote", will take everything up to
the next quotation mark and print it to the screen. Make sure you leave
a space after the first quotation mark. When you want to have text begin
on a new line, you can issue a carriage return using the word **CR**
. Enter:

```
: SPROUTS ." Miniature vegetables." ;
: MENU
    CR TOFU CR SPROUTS CR
;
MENU
```

You can emit a blank space with **SPACE** . A number of spaces can be
output with SPACES . Enter:

```
CR TOFU SPROUTS
CR TOFU SPACE SPROUTS
CR 10 SPACES TOFU CR 20 SPACES SPROUTS
```

For character input, Forth uses the word **KEY** which corresponds to
the word EMIT for output. KEY waits for the user to press a key then leaves
its value on the stack. Try the following.

```
: TESTKEY ( -- )
    ." Hit a key: " KEY CR
    ." That = " . CR
;
TESTKEY
```

[Note: On some computers, the input if buffered so you will need to hit
the ENTER key after typing your character.]

**EMIT ( char -- , output character )**

**KEY ( -- char , input character )**

**SPACE ( -- , output a space )**

**SPACES ( n -- , output n spaces )**

**CHAR ( <char> -- char , convert to ASCII )**

**CR ( -- , start new line , carriage return )**

**." ( -- , output " delimited text )**

## Compiling from Files

PForth can read read from ordinary text files so you can use any editor
that you wish to write your programs.

### Sample Program

Enter into your file, the following code.

```
\ Sample Forth Code
\ Author: your name
```

```
: SQUARE ( n -- n*n , square number )
    DUP *
;
```

```
: TEST.SQUARE ( -- )
    CR ." 7 squared = "
    7 SQUARE . CR
;
```

Now save the file to disk.

The text following the **\** character is treated as a comment. This
would be a REM statement in BASIC or a /\*---\*/ in 'C'. The text in parentheses
is also a comment.

### Using INCLUDE

"INCLUDE" in Forth means to compile from a file.

You can compile this file using the INCLUDE command. If you saved your
file as WORK:SAMPLE, then compile it by entering:

```
INCLUDE SAMPLE.FTH
```

Forth will compile your file and tell you how many bytes it has added to
the dictionary. To test your word, enter:

```
TEST.SQUARE
```

Your two words, SQUARE and TEST.SQUARE are now in the Forth dictionary.
We can now do something that is very unusual in a programming language.
We can "uncompile" the code by telling Forth to **FORGET** it. Enter:

```
FORGET SQUARE
```

This removes SQUARE and everything that follows it, ie. TEST.SQUARE, from
the dictionary. If you now try to execute TEST.SQUARE it won't be found.

Now let's make some changes to our file and reload it. Go back into
the editor and make the following changes: (1) Change TEST.SQUARE to use
15 instead of 7 then (2) Add this line right before the definition of SQUARE:

```
ANEW TASK-SAMPLE.FTH
```

Now Save your changes and go back to the Forth window.

You're probably wondering what the line starting with **ANEW** was
for. ANEW is always used at the beginning of a file. It defines a special
marker word in the dictionary before the code. The word typically has "TASK-"
as a prefix followed by the name of the file. When you ReInclude a file,
ANEW will automatically FORGET the old code starting after the ANEW statement.
This allows you to Include a file over and over again without having to
manually FORGET the first word. If the code was not forgotten, the dictionary
would eventually fill up.

If you have a big project that needs lots of files, you can have a file
that will load all the files you need. Sometimes you need some code to
be loaded that may already be loaded. The word **INCLUDE?** will only
load code if it isn't already in the dictionary. In this next example,
I assume the file is on the volume WORK: and called SAMPLE. If not, please
substitute the actual name. Enter:

```
FORGET TASK-SAMPLE.FTH
INCLUDE? SQUARE WORK:SAMPLE
INCLUDE? SQUARE WORK:SAMPLE
```

Only the first INCLUDE? will result in the file being loaded.

## Variables

Forth does not rely as heavily on the use of variables as other compiled
languages. This is because values normally reside on the stack. There are
situations, of course, where variables are required. To create a variable,
use the word **VARIABLE** as follows:

```
VARIABLE MY-VAR
```

This created a variable named MY-VAR . A space in memory is now reserved
to hold its 32-bit value. The word VARIABLE is what's known as a "defining
word" since it creates new words in the dictionary. Now enter:

```
MY-VAR .
```

The number you see is the address, or location, of the memory that was
reserved for MY-VAR. To store data into memory you use the word **!**
, pronounced "store". It looks like an exclamation point, but to a Forth
programmer it is the way to write 32-bit data to memory. To read the value
contained in memory at a given address, use the Forth word **@** , pronounced
"fetch". Try entering the following:

```
513 MY-VAR !
MY-VAR @ .
```

This sets the variable MY-VAR to 513 , then reads the value back and prints
it. The stack diagrams for these words follows:

**@ ( address -- value , FETCH value FROM address in memory )**

**! ( value address -- , STORE value TO address in memory )**

**VARIABLE ( <name> -- , define a 4 byte memory storage location)**

A handy word for checking the value of a variable is **?** , pronounced
"question". Try entering:

```
MY-VAR ?
```

If ? wasn't defined, we could define it as:

```
: ? ( address -- , look at variable )
    @ .
;
```

Imagine you are writing a game and you want to keep track of the highest
score. You could keep the highest score in a variable. When you reported
a new score, you could check it aginst the highest score. Try entering
this code in a file as described in the previous section:

```
VARIABLE HIGH-SCORE
```

```
: REPORT.SCORE ( score -- , print out score )
    DUP CR ." Your Score = " . CR
    HIGH-SCORE @ MAX ( calculate new high )
    DUP ." Highest Score = " . CR
    HIGH-SCORE ! ( update variable )
;
```

Save the file to disk, then compile this code using the INCLUDE word. Test
your word as follows:

```
123 REPORT.SCORE
9845 REPORT.SCORE
534 REPORT.SCORE
```

The Forth words @ and ! work on 32-bit quantities. Some Forths are "16-bit"
Forths. They fetch and store 16-bit quantities. Forth has some words that
will work on 8 and 16-bit values. C@ and C! work characters which are usually
for 8-bit bytes. The 'C' stands for "Character" since ASCII characters
are 8-bit numbers. Use W@ and W! for 16-bit "Words."

Another useful word is **+!** , pronounced "plus store." It adds
a value to a 32-bit value in memory. Try:

```
20 MY-VAR !
5 MY-VAR +!
MY-VAR @ .
```

Forth also provides some other words that are similar to VARIABLE. Look
in the glossary for VALUE and ARRAY. Also look at the section on "[local
variables](pf_ref.php#Local Variables { foo --}?)" which are variables which only exist on the stack while
a Forth word is executing.

*A word of warning about fetching and storing to memory*: You have
now learned enough about Forth to be dangerous. The operation of a computer
is based on having the right numbers in the right place in memory. You
now know how to write new numbers to any place in memory. Since an address
is just a number, you could, but shouldn't, enter:

```
73 253000 ! ( Do NOT do this. )
```

The 253000 would be treated as an address and you would set that memory
location to 73. I have no idea what will happen after that, maybe nothing.
This would be like firing a rifle through the walls of your apartment building.
You don't know who or what you are going to hit. Since you share memory
with other programs including the operating system, you could easily cause
the computer to behave strangely, even crash. Don't let this bother you
too much, however. Crashing a computer, unlike crashing a car, does not
hurt the computer. You just have to reboot. The worst that could happen
is that if you crash while the computer is writing to a disk, you could
lose a file. That's why we make backups. This same potential problem exists
in any powerful language, not just Forth. This might be less likely in
BASIC, however, because BASIC protects you from a lot of things, including
the danger of writing powerful programs.

Another way to get into trouble is to do what's called an "odd address
memory access." The 68000 processor arranges words and longwords, 16 and
32 bit numbers, on even addresses. If you do a **@** or **!** , or
**W@** or **W!** , to an odd address, the 68000 processor will take
exception to this and try to abort.

Forth gives you some protection from this by trapping this exception
and returning you to the OK prompt. If you really need to access data on
an odd address, check out the words **ODD@** and **ODD!** in the
glossary. **C@** and **C!** work fine on both odd and even addresses.

## Constants

If you have a number that is appearing often in your program, we recommend
that you define it as a "constant." Enter:

```
128 CONSTANT MAX_CHARS
MAX_CHARS .
```

We just defined a word called MAX\_CHARS that returns the value on the stack
when it was defined. It cannot be changed unless you edit the program and
recompile. Using **CONSTANT** can improve the readability of your programs
and reduce some bugs. Imagine if you refer to the number 128 very often
in your program, say 8 times. Then you decide to change this number to
256. If you globally change 128 to 256 you might change something you didn't
intend to. If you change it by hand you might miss one, especially if your
program occupies more than one file. Using CONSTANT will make it easy to
change. The code that results is equally as fast and small as putting the
numbers in directly. I recommend defining a constant for almost any number.

## Logical Operators

These next two sections are concerned with decision making. This first
section deals with answering questions like "Is this value too large?"
or "Does the guess match the answer?". The answers to questions like these
are either TRUE or FALSE. Forth uses a 0 to represent **FALSE** and
a -1 to represent **TRUE**. TRUE and FALSE have been capitalized because
they have been defined as Forth constants. Try entering:

```
23 71 = .
18 18 = .
```

You will notice that the first line printed a 0, or FALSE, and the second
line a -1, or TRUE. The equal sign in Forth is used as a question, not
a statement. It asks whether the top two items on the stack are equal.
It does not set them equal. There are other questions that you can ask.
Enter:

```
23 198 < .
23 198 > .
254 15 > .
```

In California, the drinking age for alcohol is 21. You could write a simple
word now to help bartenders. Enter:

```
: DRINK? ( age -- flag , can this person drink? )
    20 >
;
```

```
20 DRINK? .
21 DRINK? .
43 DRINK? .
```

The word FLAG in the stack diagram above refers to a logical value.

Forth provides special words for comparing a number to 0. They are **0=**
**0>** and **0<** . Using 0> is faster than calling 0 and > separately.
Enter:

23 0> . ( print -1 )

-23 0> . ( print 0 )

23 0= . ( print 0 )

For more complex decisions, you can use the *Boolean* operators **OR**
, **AND** , and **NOT** . OR returns a TRUE if either one or both
of the top two stack items are true.

```
TRUE TRUE OR .
TRUE FALSE OR .
FALSE FALSE OR .
```

AND only returns a TRUE if both of them are true.

```
TRUE TRUE AND .
TRUE FALSE AND .
```

NOT reverses the value of the flag on the stack. Enter:

```
TRUE .
TRUE NOT .
```

Logical operators can be combined.

```
56 3 > 56 123 < AND .
23 45 = 23 23 = OR .
```

Here are stack diagrams for some of these words. See the glossary for a
more complete list.

**< ( a b -- flag , flag is true if A is less than B )**

**> ( a b -- flag , flag is true if A is greater than B )**

**= ( a b -- flag , flag is true if A is equal to B )**

**0= ( a -- flag , true if a equals zero )**

**OR ( a b -- a||b , perform logical OR of bits in A and B )**

**AND ( a b -- a&b , perform logical AND of bits in A and B
)**

**NOT ( flag -- opposite-flag , true if false, false if true )**

### Problems:

1) Write a word called LOWERCASE? that returns TRUE if the number on top
of the stack is an ASCII lowercase character. An ASCII 'a' is 97 . An ASCII
'z' is 122 . Test using the characters " A ` a q z { ".

```
CHAR A LOWERCASE? . ( should print 0 )
CHAR a LOWERCASE? . ( should print -1 )
```

[Answers to the problems](#Answers to Problems) can be found
at the end of this tutorial.

## Conditionals - IF ELSE THEN CASE

You will now use the TRUE and FALSE flags you learned to generate in the
last section. The "flow of control" words accept flags from the stack,
and then possibly "branch" depending on the value. Enter the following
code.

```
: .L ( flag -- , print logical value )
    IF ." True value on stack!"
    ELSE ." False value on stack!"
    THEN
;
```

```
0 .L
FALSE .L
TRUE .L
23 7 < .L
```

You can see that when a TRUE was on the stack, the first part got executed.
If a FALSE was on the stack, then the first part was skipped, and the second
part was executed. One thing you will find interesting is that if you enter:

```
23 .L
```

the value on the stack will be treated as true. The flow of control words
consider any value that does not equal zero to be TRUE.

The **ELSE** word is optional in the **IF...THEN** construct.
Try the following:

```
: BIGBUCKS? ( amount -- )
    1000 >
    IF ." That's TOO expensive!"
    THEN
;
```

```
531 BIGBUCKS?
1021 BIGBUCKS?
```

Many Forths also support a **CASE** statement similar to switch() in
'C'. Enter:

```
: TESTCASE ( N -- , respond appropriately )
    CASE
        0 OF ." Just a zero!" ENDOF
        1 OF ." All is ONE!" ENDOF
        2 OF WORDS ENDOF
        DUP . ." Invalid Input!"
    ENDCASE CR
;
```

```
0 TESTCASE
1 TESTCASE
5 TESTCASE
```

See CASE in the glossary for more information.

### Problems:

1) Write a word called DEDUCT that subtracts a value from a variable containing
your checking account balance. Assume the balance is in dollars. Print
the balance. Print a warning if the balance is negative.

```
VARIABLE ACCOUNT
```

```
: DEDUCT ( n -- , subtract N from balance )
    ????????????????????????????????? ( you fill this in )
;
```

```
300 ACCOUNT ! ( initial funds )
40 DEDUCT ( prints 260 )
200 DEDUCT ( print 60 )
100 DEDUCT ( print -40 and give warning! )
```

[Answers to the problems](#Answers to Problems) can be found
at the end of this tutorial.

## Loops

Another useful pair of words is **BEGIN...UNTIL** . These are used to
loop until a given condition is true. Try this:

```
: COUNTDOWN  ( N -- )
    BEGIN
        DUP . CR       ( print number on top of stack )
        1-  DUP  0<    ( loop until we go negative )
    UNTIL
;
```

```
16 COUNTDOWN
```

This word will count down from N to zero.

If you know how many times you want a loop to execute, you can use the
**DO...LOOP** construct. Enter:

```
: SPELL
    ." ba"
    4 0 DO
        ." na"
    LOOP
;
```

This will print "ba" followed by four occurrences of "na". The ending value
is placed on the stack before the beginning value. Be careful that you
don't pass the values in reverse. Forth will go "the long way around" which
could take awhile. The reason for this order is to make it easier to pass
the loop count into a word on the stack. Consider the following word for
doing character graphics. Enter:

```
: PLOT# ( n -- )
    0 DO
        [CHAR] - EMIT
    LOOP CR
;
```

```
CR 9 PLOT# 37 PLOT#
```

If you want to access the loop counter you can use the word I . Here is
a simple word that dumps numbers and their associated ASCII characters.

```
: .ASCII ( end start -- , dump characters )
    DO
        CR I . I EMIT
    LOOP CR
;
```

```
80 64 .ASCII
```

If you want to leave a DO LOOP before it finishes, you can use the word **LEAVE**. Enter:

```
: TEST.LEAVE  ( -- , show use of leave )
    100 0
    DO
        I . CR  \ print loop index
        I 20 >  \ is I over 20
        IF
            LEAVE
        THEN
    LOOP
;
TEST.LEAVE  \ will print 0 to 20
```

Please consult the manual to learn about the following words **+LOOP**
and **RETURN** . FIXME

Another useful looping construct is the **BEGIN WHILE REPEAT** loop.
This allows you to make a test each time through the loop before you actually
do something. The word WHILE will continue looping if the flag on the stack
is True. Enter:

```
: SUM.OF.N ( N -- SUM[N] , calculate sum of N integers )
    0  \ starting value of SUM
    BEGIN
        OVER 0>   \ Is N greater than zero?
    WHILE
        OVER +  \ add N to sum
        SWAP 1- SWAP  \ decrement N
    REPEAT
    SWAP DROP  \ get rid on N
;
```

```
4 SUM.OF.N    \ prints 10   ( 1+2+3+4 )
```

### Problems:

1) Rewrite SUM.OF.N using a DO LOOP.

2) Rewrite SUM.OF.N using BEGIN UNTIL.

3) For bonus points, write SUM.OF.N without using any looping or conditional
construct!

[Answers to the problems](#Answers to Problems) can be found
at the end of this tutorial.

## Text Input and Output

You learned earlier how to do single character I/O. This section concentrates
on using strings of characters. You can embed a text string in your program
using S". Note that you must follow the S" by one space. The text string
is terminated by an ending " .Enter:

```
: TEST S" Hello world!" ;
TEST .S
```

Note that TEST leaves two numbers on the stack. The first number is the
address of the first character. The second number is the number of characters
in the string. You can print the characters of the string as follows.

```
TEST DROP       \ get rid of number of characters
DUP C@ EMIT     \ prints first character, 'H'
CHAR+ DUP C@ EMIT  \ prints second character, 'e'
\ and so on
```

CHAR+ advances the address to the next character. You can print the entire
string using TYPE.

```
TEST  TYPE
TEST  2/  TYPE   \ print half of string
```

It would be nice if we could simply use a single address to describe a
string and not have to pass the number of characters around. 'C' does this
by putting a zero at the end of the string to show when it ends. Forth
has a different solution. A text string in Forth consists of a character
count in the first byte, followed immediately by the characters themselves.
This type of character string can be created using the Forth word C" ,
pronounced 'c quote'. Enter:

```
: T2 C" Greetings Fred" ;
T2 .
```

The number that was printed was the address of the start of the string.
It should be a byte that contains the number of characters. Now enter:

```
T2 C@ .
```

You should see a 14 printed. Remember that C@ fetches one character/byte
at the address on the stack. You can convert a counted Forth string to
an address and count using COUNT.

```
T2 COUNT .S
TYPE
```

The word **COUNT** extracts the number of characters and their starting
address. COUNT will only work with strings of less than 256 characters,
since 255 is the largest number that can be stored in the count byte. TYPE
will, however, work with longer strings since the length is on the stack.
Their stack diagrams follow:

**CHAR+ ( address -- address' , add the size of one character )**

**COUNT ( $addr -- addr #bytes , extract string information )**

**TYPE ( addr #bytes -- , output characters at addr )**

The $addr is the address of a count byte. The dollar sign is often used
to mark words that relate to strings.

You can easily input a string using the word **ACCEPT**. (You may
want to put these upcoming examples in a file since they are very handy.)
The word **ACCEPT** receives characters from the keyboard and places
them at any specified address. **ACCEPT** takes input characters until
a maximum is reached or an end of line character is entered. **ACCEPT**returns the number of characters entered. You can write a word for
entering text. Enter:

```
: INPUT$ ( -- $addr )
    PAD  1+ ( leave room for byte count )
    127 ACCEPT ( recieve a maximum of 127 chars )
    PAD C! ( set byte count )
    PAD ( return address of string )
;
```

```
INPUT$ COUNT TYPE
```

Enter a string which should then be echoed. You could use this in a program
that writes form letters.

```
: FORM.LETTER ( -- )
    ." Enter customer's name." CR
    INPUT$
    CR ." Dear " DUP COUNT TYPE CR
    ." Your cup that says " COUNT TYPE
    ." is in the mail!" CR
;
```

**ACCEPT ( addr maxbytes -- numbytes , input text, save at address
)**

You can use your word INPUT$ to write a word that will read a number
from the keyboard. Enter:

```
: INPUT# ( -- N true | false )
    INPUT$ ( get string )
    NUMBER? ( convert to a string if valid )
    IF DROP TRUE ( get rid of high cell )
    ELSE FALSE
    THEN
;
```

This word will return a single-precision number and a TRUE, or it will
just return FALSE. The word **NUMBER?** returns a double precision number
if the input string contains a valid number. Double precision numbers are
64-bit so we DROP the top 32 bits to get a single-precision 32 bit number.

## Changing Numeric Base

For day-to-day life, the numbering system we use is decimal, or "base 10." That means each digit get multiplied by a power of 10. Thus a number
like 527 is equal to (5\*100 + 2\*10 + 7\*1). The use of 10 for the numeric
base is a completely arbitrary decision. It no doubt has something to do
with the fact that most people have 10 fingers (including thumbs). The
Babylonians used base 60, which is where we got saddled with the concept
of 60 minutes in an hour. Computer hardware uses base 2, or "binary". The
computer number "1101" is equal to (1\*8 + 1\*4 + 0\*2 + 1\*1). If you add
these up, you get 8+4+1=13 . The binary number "10" is (1\*2 + 0\*1), or 2. Likewise
the numeric string "10" in any base N is N.

Forth makes it very easy to explore different numeric bases because
it can work in any base. Try entering the following:

```
DECIMAL 6 BINARY .
1 1 + .
1101 DECIMAL .
```

Another useful numeric base is *hexadecimal*. which is base 16. One
problem with bases over 10 is that our normal numbering system only has
digits 0 to 9. For hex numbers we use the letters A to F for the digits
10 to 15. Thus the hex number "3E7" is equal to (3\*256 + 14\*16 + 7\*1). Try
entering:

```
DECIMAL 12 HEX .  \ print C
DECIMAL 12 256 *   7 16 * +  10 + .S
DUP BINARY .
HEX .
```

A variable called **BASE** is used to keep track of the current numeric
base. The words **HEX** , **DECIMAL** , and **BINARY** work by changing
this variable. You can change the base to anything you want. Try:

```
7 BASE !
6 1 + .
BASE @ . \ surprise!
```

You are now in base 7 . When you fetched and printed the value of BASE,
it said "10" because 7, in base 7, is "10".

PForth defines a word called .HEX that prints a number as hexadecimal
regardless of the current base.

```
DECIMAL 14 .HEX
```

You could define a word like .HEX for any base. What is needed is a way
to temporarily set the base while a number is printed, then restore it
when we are through. Try the following word:

```
: .BIN ( N -- , print N in Binary )
    BASE @ ( save current base )
    2 BASE ! ( set to binary )
    SWAP . ( print number )
    BASE ! ( restore base )
;
```

```
DECIMAL
13 .BIN
13 .
```

## Answers to Problems

If your answer doesn't exactly match these but it works, don't fret. In
Forth, there are usually many ways to the same thing.

### [Stack Manipulations](#Problems - Stack)

```
1) SWAP DUP
2) ROT DROP
3) ROT DUP 3 PICK
4) SWAP OVER 3 PICK
5) -ROT 2DUP
```

### [Arithmetic](#Problems - Square)

```
(12 * (20 - 17)) ==> 20 17 - 12 *
(1 - (4 * (-18) / 6)) ==> 1 4 -18 * 6 / -
(6 * 13) - (4 * 2 * 7) ==> 6 13 * 4 2 * 7 * -
```

```
: SQUARE ( N -- N*N )
    DUP *
;
```

```
: DIFF.SQUARES ( A B -- A*A-B*B )
	SWAP SQUARE
	SWAP SQUARE -
;
```

```
: AVERAGE4 ( A B C D -- [A+B+C+D]/4 )
    + + + ( add'em up )
    4 /
;
```

```

: HMS>SECONDS ( HOURS MINUTES SECONDS -- TOTAL-SECONDS )
    -ROT SWAP ( -- seconds minutes hours )
    60 * + ( -- seconds total-minutes )
    60 * + ( -- seconds )
;

```

### [Logical Operators](#Problems - Logical)

```
: LOWERCASE? ( CHAR -- FLAG , true if lowercase )
    DUP 123 <
    SWAP 96 > AND
;
```

### [Conditionals](#Problems - Conditionals)

```
: DEDUCT ( n -- , subtract from account )
    ACCOUNT @ ( -- n acc )
    SWAP - DUP ACCOUNT ! ( -- acc' , update variable )
    ." Balance = $" DUP . CR ( -- acc' )
    0< ( are we broke? )
    IF ." Warning!! Your account is overdrawn!" CR
    THEN
;
```

### [Loops](#Problems - Loops)

```
: SUM.OF.N.1 ( N -- SUM[N] )
    0 SWAP \ starting value of SUM
    1+ 0 \ set indices for DO LOOP
    ?DO \ safer than DO if N=0
        I +
    LOOP
;
```

```
: SUM.OF.N.2 ( N -- SUM[N] )
    0 \ starting value of SUM
    BEGIN ( -- N' SUM )
        OVER +
        SWAP 1- SWAP
        OVER 0<
    UNTIL
    SWAP DROP
;
```

```
: SUM.OF.N.3 ( NUM -- SUM[N] , Gauss' method )
    DUP 1+   \ SUM(N) = N*(N+1)/2
    * 2/
;
```

Back to [pForth Home Page](../pforth)

(C) 1997-2015 Mobileer Inc - All Rights Reserved - [Contact Us](/contacts.php)
