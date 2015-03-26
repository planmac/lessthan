LESSTHAN most other languages
=============================

A (description of a) minimalist functional stack/stream-based language

Roots and Design
----------------

LESSTHAN or LT is a programming language that draws on concepts from several other programming languages, most noteably FALSE (http://strlen.com/false/), FORTH, and LISP and various derivatives thereof. The name derives from the '<' operator which plays a significant role. LT uses a FORTHish postfix notation and a push/pop stack, FALSEish single-char (obfuscated!) operators and variables, and LISPish lists (called sequences in LT) and functional programming concepts. Implementation of an interpreter is a work-in-progress and one day I hope to write a compiler!

###Functional programming principles

LT attempts to provide functional programming capability within the context of a stack-based model. In this regard I have attempted to define the language adhering to the following principles:

Anonymous functions: Functions can be declared and called without being bound to a name, often refered to as lambda functions.

Higher-order functions: Functions can take other functions as arguments, and can return functions as results.

Purely functional functions: Functions do not have memory or i/o side-effects other than the result returned (ie. function arguments are immutable).

Lazy evaluation: Evaluation can be delayed until such time as the result is known to be needed, and then evaluated only as far as required. 

Recursion: Functions can call themselves thus iterating over a particular expression (generative recursion) or data structure (structural recursion).

Tail recursion: Recursion is optimised to an iteration if the last operation of a function is a recursive call. 
[NOT SURE HOW TO DO THIS YET!!!]


###Data types 
[NEED TO SHRINK THE DATA TYPES.. 1X STRING/SYMBOL TYPE AND 1X NUMERIC/BOOLEAN TYPE ?]

LT is not strictly typed in that all data is created and manipulated on the data stack (a stack of dword integers), however some operators expect stack items to be of a specific data type. The recognised types are: 

`
chr	A single printable ascii character in the range: '!' (ascii 33) to '~' (ascii 126) or wsp
nul	The Null char ascii 0 or the integer value zero
wsp	One or more chars in the range ascii 1 to ascii 32
var	One lower case letter a..z

int	One or more of the digits 0..9 optionally followed by the unary minus operator '_' 
flt	An int with one period '.' after the first digit and before the optional unary negation suffix '_'

fun	Zero or more chr or wsp chars enclosed in brackets [..] 
seq	Zero or more int, flt, fun or seq delimited by wsp and enclosed in parenthesis ( .. ) 

adr	A dword memory address of a var, fun or seq
bvl	A boolean value, zero for false or any other value for true
bfn	A fun that returns a bvl
`

###Language syntax

As with FALSE and FORTH, there is very little syntax at all: code is a stream of operators, digits, variables, and comments that are evaluated in sequence. Different to FORTH, operators and variables are single char and are generally not separated, so [$%+/] is a function consisting of four operators while [1 1+] is a function consisting of two integers and an operator.  

White space may be inserted anywhere in the code  and has no significance EXCEPT that at least one whitespace char is required between consecutive ints and flts and between items in a seq. Any chars enclosed in braces {..} are considered comments and are ignored. Code may be presented in indented style to depict program logic flow.

The parameter stack (referred to as the stack hereinafter) is a rotating stack of dword memory slots used to pass parameters to operators and return results from operators. The default depth of 32 can be altered by a command line parameter. Pushing more than the max. number of stack items will rotate to the beginning of the param stack area overwriting anything that was there. Popping more than the number of items previously pushed will rotate to the end of the param stack area and continue popping whatever is there. The initial values of param stack items is undefined except for the top stack item which is guaranteed to be nul/zero. 

Operators that expect a truth value will regard zero as false and any other value as true. Operators that return a truth value will return zero for false and minus one for true. Certain operators, such as those wrapped around system calls, return error codes as negative numbers and success as zero. 

The 'quit' operator Q expects a value on the stack and performs an appropriate system call to terminate the current process witht the supplied value as exit code, ie. anything other than 0Q will terminate with an error. Any sub-processes started by the current process will continue unaffected.

###Global variables

The 26 lower case letters 'a' to 'z' are available as dword global variable memory slots. Using the var name in code will push its address onto the stack. A var can be bound to a value at any time by eg. a: and the current value retrieved by eg. a; .  Upper case letters are seq operators.

###Conditional operators

Only two conditional operators are defined, 'ifelse' and 'while'. 
[TRY TO GET AWAY FROM CONDITIONALS - USE RECURSION INSTEAD]

The 'ifelse' operator ? expects a bvl and two funs on the stack, if the bvl is true the first fun is evaluated, if it is false the  second fun is evaluated, thereafter evaluation continues after the ? . The bvl is consumed in the truth test and both funs are consumed regardless of which one was executed. Either fun may be empty [] but they may not be omitted eg. n;0=[[HI!]"][[BYE]"]? will print HI! if n=0 else BYE.

The 'while' operator # expects a bfn and a fun on the stack. If the bfn returns true, the fun is evaluated and then the bfn is evaluated again. If the bfn returns false evaluation continues after the # . The bfn and the fun are consumed whether evaluated or not eg. 11[1-%0>][%.]# will print the digits 10 to 1.

A 'case' construct can be formed using nested 'ifelse' statements and duplicating the case value for each 'ifelse' operator eg. <casevalue> %3=["Three"][%2=["Two"][1=["One"]["Other"]?]?]? . The need for an 'until' construct can normally be avoided.

###Lambda Functions

A fun definition [..] may contain a sequence of zero or more printable ascii characters that should be valid LT operators, digits or variables. The fun definition may also be used to define strings. Definition of a fun pushes its address onto the stack.

A fun may contain embedded funs eg. [%.[10+]!10-] . During definition of the outer fun an embedded fun is included as part of the sequence of chars ie. it is not 'defined'. If the outer fun is evaluated the embedded fun will be defined and its address pushed on the stack.

The 'eval' operator ! expects the adr of a fun on the stack, and evaluates the contained sequence of chars, returning input to the keyboard when complete eg. [10%*]! computes ten squared.

###Bound/named functions and recursion

Lambda functions may be bound to a variable name with eg. [%*]f: and then evaluated by first getting the value of the variable (the fun adr) and calling the 'eval' operator eg. f;! . A var can be used in a fun and its binding is not accessed until the fun is evaluated. This allows recursive calls to named functions.

The ubiquitous factorial function in LT: ` [%0=[\1][%1-f;!*]?]f: ` 
Which can be evaluated as follows: ` 0f;! -> 1  1f;! -> 1  2f;! -> 2  3f;! -> 6  4f;! -> 24  ... `

###Console i/o

LT provides simple i/o operators to accept input and emit output. The 'accept' operator ^ accepts a key from the current input device (default is the keyboard) and pushes its ascii code onto the stack. The 'emitchar' operator , expects an ascii code on the stack and emits it as an ascii char to the current output device (default is the console). The 'emitint' operator . expects a signed integer on the stack, converts it to a stream of ascii chars including a leading minus sign for negatives and emits these to the current output device. The 'emitstring' operator " expects a fun on the stack and emits its contents as a stream of ascii chars to the current output device.
[LOOK INTO IO MONAD THINKING TO REFINE THIS TOPIC]

The interpreter
---------------

The interpreter is a simple REPL routine, ie. Read input up to the first LF char using the syscall read and storing it to an internal buffer, Evaluate each char in the stream of chars, Print the results if any, Loop back for more input. There is no error trapping overhead and eg. inappropriate memory addresses or divide by zero etc. will 'crash' the interpreter. 

The interpreter defaults to accept input from stdin and emit output to stdout. Command line pipes and redirection can be used to alter this behavior but then input/output cannot return to the keyboard/terminal during the session.

Sequences
---------

Sequences in LT are somewhat similar to lists in LISP although they follow the postfix notation, operating in reverse order they seem more like stacks than lists. (The name 'sequence' or 'seq' is borrowed from CLOJURE.)

###Defining sequences

A seq definition ( .. ) may contain zero or more int, flt, fun or seq items delimited by wsp. Consecutive delimiters or a final delimiter will be treated as one delimiter and wsp may include Line Feed chars eg. (  1  <LF>
  2  ) is equivalent to ( 1 2 ) . Entering a seq definition stores its contents as described below and pushes its address onto the stack. 

During definition each item is wrapped in brackets `[..]` and stored as a delayed fun definition, ie. items are not evaluated or defined until needed eg. `( [Hello World!] [%.] 100 )` would be stored as a seq of the following funs: `[[Hello World!]]` and `[[%.]]` and `[100]`. Embedded seqs or funs are treated as a single item eg. `( ( 1 2 ) [1 2+] )` will be stored as the following funs: `[( 1 2 )] and [[1 2+]]`.

Seqs may share state if, for example, an operator returns a part of the original seq, the result may not re-create the items but return a pointer to existing items. This is explained in more detail below.

###Accessing sequence items

When a specific item in a seq is accessed by one of the seq operators, its fun representation, as described above, is pushed onto the stack and the evaluator function ! is called which will push the 'value' of the item onto the stack. An item's value may be the adr of a fun or seq or an int or flt value. 

For example, given the seq `( [Hello World!] [%.] 100 ( 1 2 ) )` , accessing the `[[Hello World!]]` item will construct the fun `[Hello World!]` (which is usefull only as a string) and push its adr on the stack, accessing the `[[%.]]` item will construct the fun `[%.]` and push its adr on the stack, accessing the `[100]` item will evaluate the digits 100 and push the signed int 100 onto the stack, and accessing the `[( 1 2 )]` item will construct the seq `( 1 2 )` and push its adr on the stack.

###Interrogating sequences

The 'length' operator L expects a seq on the stack and returns the number of items in the seq. It does not evaluate items. Items are numbered starting from one at the 'top' of the seq ie. ( n .. 1 ) , and from minus one from the 'bottom' of the seq ie. ( -1 .. -n ) . Referencing outside of this range will throw an untrapped error!

The 'getsetwith' operator G expects a value, an index, a fun and a seq on the stack. It gets and evaluates the ith item in the seq, pushes it on the stack, evaluates the fun, and sets the ith item to the value on the stack. When the fun is evaluated the stack will have the supplied value and the value of the ith item. The fun should leave a meaningful value on the stack. There are three basic methods of using 'getsetwith':
Get ith item without setting it: ` 1[%]( 5 4 2 )G ` will leave 2 on the stack and the seq unchanged.
Set ith item without getting it: ` 3 1[\]( 5 4 2 )G ` will change the first item to 3 and leave stack unchanged.
Get and set ith item:            ` 3 1[$]( 5 4 2 )G ` will change the first item to 3 and leave 2 on the stack. 

The 'findwith' operator F expects a search value, a bfn and a seq on the stack. It iterates through each item in the seq, pushing the search value and the value of the current item onto the stack and evaluating the bfn. The first time the bfn evaluates to true the iteration stops and the index number of the current item is pushed onto the stack. The bfn should consume the search value and the current item value and return a truth value. eg. 30[=]( 30 20 10 )F will return 3.

###Iterating over sequences

The 'rest,top' operator < expects a seq on the stack. It first pushes the adr of the rest of the seq onto the stack, then the value of the top item, eg. ( 30 20 10 )<  will return ( 30 20 ),10 on the stack. 

Evaluating < on the last item in a seq will push an adr of nul/zero and the value of the last item eg. (1)< will push 0,1. Evaluating < on an empty seq will push an adr of minus one and a value of nul/zero, eg. ()< will push -1,0 . Therefore the adr can be tested separately for last-item or empty-seq conditions.

###Constructing and destructing sequences

The 'cons' operator C expects a value and a seq on the stack and logically pushes the value onto the top of the target seq and returns the adr of a new seq. The cons operation creates a new item and links its 'rest' field to the 'top' of the target seq and thus the new and target seq share structure, eg. if a seq is stored in a var with ( 3 2 )a: the following code 1a;C 0a;C will evaluate to ( 3 2 1 ) and ( 3 2 0 ) and all three seqs will share the ( 3 2 ) part, diagramatically:

`
( 3 2 )      <- adr of target seq, stored in a
    |
    +-< 1 )  <- adr of seq pushed by 1a;C
    |
    +-< 0 )  <- adr of seq pushed by 0a;C
`

Similarly the 'join' operator J expects two seqs on the stack and logically links the 'rest' field of the second seq to the 'top' of the target seq and returns the adr of the new seq, eg. ( 2 1 )( 4 3 )J will evaluate to ( 4 3 2 1 ), diagramatically:

`
( 2 1 )	 <- adr of second seq 

( 4 3 )	 <- adr of target seq
            | 
    +-< 2 1 )	<- adr of result seq
`

The 'split' operator S expects an index value and a seq on the stack and logically splits the seq after the ith item into an 'upto' and an 'after' part. A new seq is defined for the 'upto' part and the adr of the i+1th item is returned as the 'after' part, eg. 2( 4 3 2 1 )S will evaluate to ( 4 3 )( 2 1 ), diagramatically:

`
( 4 3 2 1 )    <- adr of orig seq
            |
    | ( 2 1 )	<- adr of 'upto' part 
            |
    +---------- <- adr of 'after' part 
`

###Processing sequences

The 'applyto' operator A expects a fun and a seq on the stack. It iterates through each item in the seq, pushing its value onto the stack and evaluating the fun. No assumptions are made about the result or the state of the stack, ie. the fun may perform calculations, consume the value, or leave results on the stack for use during the next iteration or after all iterations are completed.

The 'mapwith' operator M expects a fun and a seq on the stack. It iterates through each item in the seq, pushing its value onto the stack and evaluating the fun, and then 'cons'ing the value on the stack to a results seq. At each iteration the fun should leave a meaningful result on the stack for the cons. After the last iteration the adr of the results seq is left on the stack.

###Sorting sequences

The 'orderwith' operator O expects a bfn and a seq on the stack. It defines a new seq with items sorted according to the bfn, which should evaluate to true if the next item is greater than the top item and false otherwise, eg. `[>]( 2 1 3 )O` will evaluate to `( 3 2 1 )`. All items are evaluated in order to determine their numerical valuefor the sort algorithm. Items that are funs or seqs will evaluate to a memory address, which may or may not be useful. [A DEEPER SORTING ALGORITHM MAY BE DESIGNED LATER]

The order of a seq may be reversed by passing a fun that always evaluates to false, eg. `[0]( 2 1 3 )O` will evaluate to `( 3 1 2 )`. A fun that always evaluates to true will define a new seq in the same order.

###Comparing sequences

LT provides set theory type operators to compare the items in two seqs. The 'intersect' operator I defines a new seq consisting of items common to both seqs. The 'union' operator U defines a new seq consisting of items in both seqs including those in both only once. The 'difference' operator D defines a new seq consisting of items in only one of the seqs, ie. not intersect.

###Pairs and sequences

The 'bindwith' operator B expects a fun and two seqs on the stack. It iterates through corresponding items in each seq, pushing thier values onto the stack and evaluating the fun, and then 'cons'ing the value on the stack to a results seq. At each iteration the fun should leave a meaningful result on the stack for the cons. After the last iteration the adr of the results seq is left on the stack.

The 'zipwith' operator Z expects a fun and two seqs on the stack and converts the pair of seqs to a seq of pairs. It iterates through corresponding items in the two seqs, pushing their values onto the stack and evaluating the fun, and then 'cons'ing two values on the stack to a results seq. At each iteration the fun should leave two meaningful results on the stack for the cons. After the last iteration the adr of the results seq is left on the stack, eg. `[]( 9 4 1 )( 3 2 1 )Z` will evaluate to `( 9 3 4 2 1 1 )`.

The 'unzipwith' operator Y expects a fun and a seq on the stack and converts the seq of pairs to a pair of seqs. It iterates through pairs of values in the seq, pushing their values onto the stack and evaluating the fun, and then 'cons'ing two values on the stack one to each of two results seqs. At each iteration the fun should leave two meaningful results on the stack for the cons. After the last iteration the adrs of the two results seqs are left on the stack, eg. `[]( 9 3 4 2 1 1 )Y` will evaluate to `( 9 4 1 )( 3 2 1 )`.

###Using `<` to construct S-Expressions

The astute reader would have noticed that the `<` operator evaluates the top item in the seq after pushing the adr of the 'rest' of the seq! This provides the opportunity to build S-Expressions in a (reversed) LISP fasion:
eg. `( 3 2 1 [[.]$A] )f:` creates a function with embedded parameters and stores it to variable f and the code `f;<!` will emit the ints `1 2 3`. This happens as follows: 
`
f;	pushes the adr of the seq on the stack
<	replaces it with the adr of the 'rest' ( 3 2 1 ) 
and constructs the fun [[.]$A] pushing its adr on the stack
!	evaluates this fun:
[.]	defines a simple fun [.]
$	swaps it behind the 'rest' adr, leaving the stack as [.]( 3 2 1 )
A	applies the fun [.] to each item in the list ( 3 2 1 )
which emits the ints 1 2 3
`

Procs and concurrency
---------------------

LT provides a small suite of process operators allowing concurrent processing and inter-process communication. 

###Starting and stopping processes

The 'procwithfun' operator P expects a fun on the stack. It starts a new process and passes it the fun, which the new process will evaluate, and leaves the new process id and the syscall error code on the stack, eg. `[11[1-%0>][%.]#0Q]P` will launch a new process that will print the digits 10 to 1 and quit normally, and returns the process id and a syscall error code. The fun is passed as a stream of ascii chars, ie. it is not evaluated by the originating process.

The 'procwithfile' operator N expects a fun on the stack containing the full path to a text file containing LT code. It starts a new process and passes it the file name, which the new process will evaluate, and leaves the new process id and the syscall error code on the stack, eg. `[myfile.lt]N` will launch a new process that will evaluate the contents of myfile.lt, and returns the process id and a syscall error code.

The 'killproc' operator K expects a value and a process id on the stack. It requests the process to terminate with the value as exit code, and returns the syscall error code.

###Inter-process communication

Interprocess communication is based on a simple echo protocol where the sender sends a message and waits for a response and the receiver waits for a message and sends its pid to acknowledge receipt. Messages are passed as strings in the form of a fun. The sended must decide what to do with the echoed response, and the receiver must decide what to do with the received message. 

The 'echotoproc' operator E expects a fun and a pid on the stack. It sends the fun as a string of chars to the specified process and waits for the process to respond by sending back its pid. Specifying a pid of nul/zero will 'broadcast' the message for any 'listening' process and the first process to 'hear' the message will grab it and respond with its pid.

The 'hearfromproc' operator H expects a timeout value in seconds and a pid on the stack. It waits for the specified timeout period for an 'echotoproc' message from the specified process. If a matching message is available it will grab the message and respond to the sender by sending its pid. The message content string will be defined as a fun and its adr pushed on the stack followed by the sender's pid and an error code of nul/zero for success. If no matching message is recieved within the timeout a non-zero error code will be pushed on the stack and no response will be sent. Specifying a pid of nul/zero will 'listen' for a message from any process and the first process to 'send' a message, iether with the receiver's pid or with a 'broadcast' pid, will be grabbed and a pid response sent.


File input/output
-----------------

LT provides a small suite of file input/output operators that are more like redirection operators. 

Opening a file will redirect subsequent console i/o (ie. from the emit and accept operators) from/to the file until the file is closed, which will return i/o to the console. Opening a second file will NOT close any previously openned file but WILL redirect to the most recently openned file. Opening a file read only will redirect input from the file and opening a file read/write will redirect input and output from/to the file. To redirect to different files for input and output first open the output file read/write and then the input file read only.

There is no explicit method to flush system i/o buffers other than closing the file. There is no method to move the file pointer other than reading from the file.

The 'openro' operator V expects a file name in a fun definition on the stack. It calls the system call to open the file read only and returns the file id and the system error code (nul/zero for success). The 'openrw' operator W expects a file name in a fun definition on the stack. It calls the system call to open the file read/write if the file exists or create the file read/write if it does not exist, and returns the file id and the system error code (nul/zero for success). The 'closefid' operator X expects a file id on the stack. It calls the system call to close the file and returns the system error code (null/zero for success).


language overview
-----------------
`
syntax: pops:           pushes:         Example:        Result:         Description:
-------	---------------	--------------- --------------- ---------------	----------------------------
Console:
Enter	  -	              -               some code¬      ?               Read Evaluate Print Loop 
{..}    -               -               {some comment}                  Ignore up to next '}'
Q       n               -               0Q              <exit normal>   Quit with exit code

Numbers:
0..9	-	 int	 123	 123	 Convert to unsigned int
'<char>	-	 ascii	 'A	 65	 Ascii code of <char>
+	n1,n2	 n1+n2	 1 2+	 3	 Add top two stack items
-	n1,n2	 n1-n2	 1 2-	 -1	 Subtract top from next
*	n1,n2	 n1*n2	 1 2*	 2	 Multiply top two stack items
/	n1,n2	 n1%n2,n1/n2	1 2/	 1 0	 Modulus and quotient of n1/n2
_	n	 -n	 1_	 -1	 Unary minus of top stack item

Logicals:
=	n1,n1	 n1=n2	 1 2=	 0	 True if n1=n2 else false
>	n1,n2	 n1>n2	 1 2>	 0	 True if n1>n2 else false
&	n1,n2	 n1 and n2	1 2&	 0	 Bitwise and top two stack items
|	n1,n2	 n1 or n2	1 2|	 3	 Bitwise or top two stack items
~	n	 not n	 0~	 -1	 Bitwise complement top stack item

Variables:
a..z	-	 adr	 a	 <adr>	 Get address of variable
:	n,adr	 - ???	 1a:	 -	 Poke dword value to address ##WARNING!
;	adr	 n	 a;	 10	 Peek dword value from address

Functions:
[..]	-	 adr	 [1+]	 <adr>	 Define fun and push adr
!	fun|adr	 -	 [10 2*]!	20	 Evaluate function at adr ##WARNING!

Stack manipulation:
\	n	 -	 1\	 Drop top item
%	n	 n,n	 1%	 1 1	 Dup(licate) top item
$	n1,n2	 n2,n1	 1 2$	 2 1	 Swap top two items
`	n1,n2	 n1,n2,n1	1 2`	 1 2 1	 Over copy second item to top
@	n1,n2,n3	n2,n3,n1	1 2 3@	 2 3 1	 Rot(ate) third item to top

Additional stack manipulation axioms:
$\	n1,n2	 n2	 1 2$\	 2	 Nip drop second item
$`    n1,n2	 n2,n1,n2	1 2%@@	 2 1 2	 Tuck copy top item under second
@@	n1,n2,n3	n3,n1,n2	1 2 3@@	 3 1 2	 -Rot(ate) top item to third 

Conditional operators:
?	bvl,iff,elf	-	 a;2=[f;!][]?	if a=2 then f()	IF bvl eval iff else eval elf
#	bfn,fun	 -	 1[%99>~][1+]#	count 1 to 99	While bfn eval fun

Input/output:
.	n	 -	 10.	 <emit 10>	Emit as signed integer
,	ascii	 -	 10,	 <emit LF>	Emit as ascii char
"	fun	 -	 [Hi!]"	 <emit Hi!>	Emit fun as string
^	-	 ascii	 ^	 <ascii>	 Accept key and push ascii code

Sequences:
( .. )	-	 adr	 ( [Hi!] 2 3 )	<adr>	 Define seq and push adr

<	seq1	 seq2,n	 ( 3 2 1 )<	( 3 2 ) 1	Push adr of rest and eval top
C	n,seq1	 seq2	 1( 3 2 )C	( 3 2 1 )	Cons value n onto top of seq
L	seq	 n	 ( 3 2 1 )L	3	 Length of seq / no. of items
G	i,seq	 n	 3( 3 2 1)I	3	 Get and/or set ith item in seq

A	fun,seq	 n?	 [.]( 3 2 1 )A	<emit 1 2 3>	Apply fun to each item in seq 
M	fun,seq1	seq2	 [%*]( 3 2 1 )M	( 9 4 1 )	Map fun to each and return seq

J	seq1,seq2	seq3	 ( 3 2 )( 1 )J	( 3 2 1 )	Join seq1 to top of seq2
S	i,seq1	 seq2,seq3	1( 3 2 1 )S	( 3 2 )( 1 )	Split seq1 after ith item
O	seq1	 seq2	 ( 1 3 2 )O	( 3 2 1 )	Order items using >
R	seq1	 seq2	 ( 3 2 1 )R	( 1 2 3 )	Reverse order of items

I	seq1,seq2	seq3	 ( 2 1 )( 1 0 )I	( 1 )	 Intersect of seqs (in both)
U	seq1,seq2	seq3	 ( 2 1 )( 1 0 )U	( 2 1 0 )	Union of seqs (no duplicates)
D	seq1,seq2	seq3	 ( 2 1 )( 1 0 )D	( 2 0 )	 Difference of seqs (not in both)

Y	fun,seq1	seq2,seq3	[T]( 1 1 2 4 )Y	 ( 1 2 )( 1 4 )	Unzip pairs to seqs on fun
Z	fun,seq1,seq2	seq3	 [T]( 1 2 )( 1 4 )Z  ( 1 1 2 4 )	Zip seqs to pairs on fun
B	fun,seq1,seq2	seq3	 [+]( 3 2 )( 2 2 )  ( 5 4 )	Bind seqs with fun

Processes:
P	fun	 pid,per	 [..]P	 1024 0	 Start proc with fun
N	[filename]	pid,per	 [code.lt]N	1024 0	 Start proc with file
E	fun,pid1	pid2,per	[HI!]1024E	1024 0	 Echo fun to process pid
H	n,pid	 fun,pid2,per	60 0H	 [HI!]800 0	Hear fun from process pid
K	pid	 per	 1024K	 0	 Kill process pid

File input/output:
V	[filename]	fid,fer	 [myfile.txt]V	1024 0	 Open file with readonly
W	[filename]	fid,fer	 [myfile.txt]V	1024 0	 Open/create file readwrite
U	fid	 fer	 1024U	 0	 Close file fid
------------------------------------------------------------------------------------------------------------
Alternative: use ` for pick, then different axioms:
`	nn..ni..n1,i	nn..ni..n1,ni	3 2 1 3`	3 2 1 3	 Pick ith item from rest of stack
%@@    n1,n2	 n2,n1,n2	1 2%@@	 2 1 2	 Tuck copy top item under second
$%@@  n1,n2	 n1,n2,n1	1 2$%@@	 1 2 1	 Over copy second item to top (or 2` )
`

Memory model
------------

###Funs in memory

Funs are created in a revolving cache area. The cache default size of 2048 bytes can be altered by a command line option. Once the cache is full, new funs are created starting again at the beginning of the cache overwriting anything that was there.  If a new fun will not fit in the remaining area of cache its storage is restarted from the beginning of the cache. An attempt to create a fun bigger than the cache will cause an untrapped error!

For illustrative purposes three funs represented by [xxx..xxx] [yyy..yyy] and [abcdefghi] in the cache, as follows:

                                                                                2048 bytes
              +-------------------------------- ~~ ---------------------------------------+
Overwritten:  |xxxxxxxxxx                      ~~                                        |
Current value:|abcdefghi0xxxxxxxxxx0yyyyyyyyyyy ~~ yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy0abcde|
              +-------------------------------- ~~ ----------------------------------^----+
              ^        ^                                                          |    ^
            cache    cache                                                        |  cache
              top      ptr                      This  list did  not fit  and so \  |  end
                                                restarted  at  the top  of  the |___/
                                                cache overwriting prev contents /


Funs are stored as an asciz string: a sequence of ascii chars terminated by a 'NUL'. Funs are immutable after construction. Fun adrs may be stored in variables with eg. [abc]a: however the fun itself remains in the cache and is therefore transient.

-----------------------------------------------------------------------
input:  [  c  h  a  r  l  i  s  t    ]

hex:        63  68  61  72  6C  69  73  74  00
-----------------------------------------------------------------------
            ^                              ^
          fun                            'NUL'
          addr                        terminator  


##Seqs in memory

Seqs are created in a revolving seq stack area. The seq stack default size of 512 dwords can be altered by a command line option. Once the seq stack is full adding additional items continues from the beginning of the seq stack area overwriting anything that was there, ie. a seq definition can wrap from the end to the beginning of the seq stack area. 

Seqs are stored as a nul initiated list of addresses of seq stack items and the adr of the seq is the adr of the last item adr 'pushed' onto the seq stack. ie. a seq definition pushes a nul dword to mark the 'bottom' of the seq and then pushes the adr of each item's fun definition. Items are accessed by logically 'popping' them from the seq stack in reverse order until the initiating nul/zero is reached.

Seq adrs may be stored in variables with eg. ( 2 1 )a: however the seq itself remains in the seq stack and is therefore transient.

For illustrative purposes three seqs represented by ( x .. x ) ( y .. y ) and ( a b c d e f g h i ) in the seq stack, as follows: 

                                                                                512 dwords
              +-------------------------------- ~~ ---------------------------------------+
Overwritten:  |0xxxxxxxxx                      ~~                                        |
Current value:|fghixxxxxxxxxxxxxxxx0yyyyyyyyyyy ~~ yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy0abcde|
              +-------------------------------- ~~ ---------------------------------^-----+
              ^  ^^                                                                ^
            stack |\__cache                                                        |  stack
            top  |  ptr                          This seq wraps to the top of \___|    end
                  |                                the stack and continues      /
              seq adr


###Optimising memory useage

LT does not provide garbage collection although using a revolving fun cache and a revolving seq stack eliminates the possibility of over/under flow errors or out-of-memory conditions (at the expense of potentially overwriting required data). It is up to the programmer to optimise the size of the fun cache and seq stack (and the parameter stack) for the particular application. To assist, the interpreter can track and provide useage data. If the interpreter is passed the -u command line option and then terminates gracefully with the 'quit' operator, it will print the following stats to stdout:

Termination memory state:
Param stack:	depth xx of yy dwords
Fun cache:	depth xx of yy bytes after zz revolutions 
Seq stack:	depth xx of yy dwords after zz revolutions

The 'revolutions' counts how many times the fun cache and seq stack have revolved. Anything more than zero implies that some data was overwritten (which may or may not be a problem). The 'depth' indicates how much space was used (in the last revolution) of the param stack, the fun cache and the seq stack, and the 'of' indicates the size for this invocation.

This data can be used to optimise memory allocations with the '-p xx' param stack size in dwords, '-f xx' fun cache size in bytes, and '-s xx' seq stack size in dwords command line parameters.

Licence
=======

You are free to do whatever you like with LESSTHAN. Your feedback, advice and contributions are very welcome!

ENJOY!

Peter MacDonald
<petermac@gmail.com>

Originated: April 2008
Revised: March 2015
