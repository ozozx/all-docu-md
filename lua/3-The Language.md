This section describes the lexis, the syntax, and the semantics of Lua. In other words, this section describes which tokens are valid, how they can be combined, and what their combinations mean.

Language constructs will be explained using the usual extended BNF notation, in which {_a_} means 0 or more _a_'s, and [_a_] means an optional _a_. Non-terminals are shown like non-terminal, keywords are shown like **kword**, and other terminal symbols are shown like ‘**=**’. The complete syntax of Lua can be found in [[9-The Complete Syntax of Lua|§9]] at the end of this manual.

## 3.1 – Lexical Conventions

Lua is a free-form language. It ignores spaces and comments between lexical elements (tokens), except as delimiters between two tokens. In source code, Lua recognizes as spaces the standard ASCII whitespace characters space, form feed, newline, carriage return, horizontal tab, and vertical tab.

_Names_ (also called _identifiers_) in Lua can be any string of Latin letters, Arabic-Indic digits, and underscores, not beginning with a digit and not being a reserved word. Identifiers are used to name variables, table fields, and labels.

The following _keywords_ are reserved and cannot be used as names:
```
and       break     do        else      elseif    end
false     for       function  global    goto      if
in        local     nil       not       or        repeat
return    then      true      until     while
```
Lua is a case-sensitive language: `and` is a reserved word, but `And` and `AND` are two different, valid names. As a convention, programs should avoid creating names that start with an underscore followed by one or more uppercase letters (such as <code>[[6-The Standard Libraries#`_VERSION`|_VERSION]]</code>).

The following strings denote other tokens:
```
+     -     *     /     %     ^     #
&     ~     |     <<    >>    //
==    ~=    <=    >=    <     >     =
(     )     {     }     [     ]     ::
;     :     ,     .     ..    ...
```
A _short literal string_ can be delimited by matching single or double quotes, and can contain the following C-like escape sequences: '`\a`' (bell), '`\b`' (backspace), '`\f`' (form feed), '`\n`' (newline), '`\r`' (carriage return), '`\t`' (horizontal tab), '`\v`' (vertical tab), '`\\`' (backslash), '`\"`' (quotation mark [double quote]), and '`\'`' (apostrophe [single quote]). A backslash followed by a line break results in a newline in the string. The escape sequence '`\z`' skips the following span of whitespace characters, including line breaks; it is particularly useful to break and indent a long literal string into multiple lines without adding the newlines and spaces into the string contents. A short literal string cannot contain unescaped line breaks nor escapes not forming a valid escape sequence.

We can specify any byte in a short literal string, including embedded zeros, by its numeric value. This can be done with the escape sequence \x_XX_, where _XX_ is a sequence of exactly two hexadecimal digits, or with the escape sequence \\_ddd_, where _ddd_ is a sequence of up to three decimal digits. (Note that if a decimal escape sequence is to be followed by a digit, it must be expressed using exactly three digits.)

The UTF-8 encoding of a Unicode character can be inserted in a literal string with the escape sequence \u{_XXX_} (with mandatory enclosing braces), where _XXX_ is a sequence of one or more hexadecimal digits representing the character code point. This code point can be any value less than _2<sup>31</sup>_. (Lua uses the original UTF-8 specification here, which is not restricted to valid Unicode code points.)

Literal strings can also be defined using a long format enclosed by _long brackets_. We define an _opening long bracket of level _n__ as an opening square bracket followed by _n_ equal signs followed by another opening square bracket. So, an opening long bracket of level 0 is written as `[[`, an opening long bracket of level 1 is written as `[=[`, and so on. A _closing long bracket_ is defined similarly; for instance, a closing long bracket of level 4 is written as `]====]`. A _long literal_ starts with an opening long bracket of any level and ends at the first closing long bracket of the same level. It can contain any text except a closing bracket of the same level. Literals in this bracketed form can run for several lines, do not interpret any escape sequences, and ignore long brackets of any other level. Any kind of end-of-line sequence (carriage return, newline, carriage return followed by newline, or newline followed by carriage return) is converted to a simple newline. When the opening long bracket is immediately followed by a newline, the newline is not included in the string.

As an example, in a system using ASCII (in which '`a`' is coded as 97, newline is coded as 10, and '`1`' is coded as 49), the five literal strings below denote the same string:

&nbsp;&nbsp;&nbsp;a = 'alo\n123"'
&nbsp;&nbsp;&nbsp;&nbsp;a = "alo\n123\""
&nbsp;&nbsp;&nbsp;&nbsp;a = '\97lo\10\04923"'
&nbsp;&nbsp;&nbsp;&nbsp;a = [[alo
&nbsp;&nbsp;&nbsp;&nbsp;123"]]
&nbsp;&nbsp;&nbsp;&nbsp;a = [\==[
&nbsp;&nbsp;&nbsp;&nbsp;alo
&nbsp;&nbsp;&nbsp;&nbsp;123"]\==]

Any byte in a literal string not explicitly affected by the previous rules represents itself. However, Lua opens files for parsing in text mode, and the system's file functions may have problems with some control characters. So, it is safer to represent binary data as a quoted literal with explicit escape sequences for the non-text characters.

A _numeric constant_ (or _numeral_) can be written with an optional fractional part and an optional decimal exponent, marked by a letter '`e`' or '`E`'. Lua also accepts hexadecimal constants, which start with `0x` or `0X`. Hexadecimal constants also accept an optional fractional part plus an optional binary exponent, marked by a letter '`p`' or '`P`' and written in decimal. (For instance, `0x1.fp10` denotes 1984, which is _0x1f / 16_ multiplied by _210_.)

A numeric constant with a radix point or an exponent denotes a float; otherwise, if its value fits in an integer or it is a hexadecimal constant, it denotes an integer; otherwise (that is, a decimal integer numeral that overflows), it denotes a float. Hexadecimal numerals with neither a radix point nor an exponent always denote an integer value; if the value overflows, it _wraps around_ to fit into a valid integer.

Examples of valid integer constants are
```
3   345   0xff   0xBEBADA
```
Examples of valid float constants are
```
3.0     3.1416     314.16e-2     0.31416E1     34e1
0x0.1E  0xA23p-4   0X1.921FB54442D18P+1
```
A _comment_ starts with a double hyphen (`--`) anywhere outside a string. If the text immediately after `--` is not an opening long bracket, the comment is a _short comment_, which runs until the end of the line. Otherwise, it is a _long comment_, which runs until the corresponding closing long bracket.

## 3.2 – Variables

Variables are places that store values. There are three kinds of variables in Lua: global variables, local variables, and table fields.

A single name can denote a global variable or a local variable (or a function's formal parameter, which is a particular kind of local variable) (see [[2-Basic Concepts#2.2 – Scopes, Variables, and Environments|§2.2]]):

&nbsp;&nbsp;&nbsp;&nbsp;var ::= Name

Name denotes identifiers (see [[#3.1 – Lexical Conventions|§3.1]]).

Because variables are _lexically scoped_, local variables can be freely accessed by functions defined inside their scope (see [[2-Basic Concepts#2.2 – Scopes, Variables, and Environments|§2.2]]).

Before the first assignment to a variable, its value is **nil**.

Square brackets are used to index a table:

&nbsp;&nbsp;&nbsp;&nbsp;var ::= prefixexp ‘**[**’ exp ‘**]**’

The meaning of accesses to table fields can be changed via metatables (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

The syntax `var.Name` is just syntactic sugar for `var["Name"]`:

&nbsp;&nbsp;&nbsp;&nbsp;var ::= prefixexp ‘**.**’ Name

An access to a global variable `x` is equivalent to `_ENV.x`.

## 3.3 – Statements

Lua supports an almost conventional set of statements, similar to those in other conventional languages. This set includes blocks, assignments, control structures, function calls, and variable declarations.

### 3.3.1 – Blocks

A block is a list of statements, which are executed sequentially:

&nbsp;&nbsp;&nbsp;&nbsp;block ::= {stat}

Lua has _empty statements_ that allow you to separate statements with semicolons, start a block with a semicolon or write two semicolons in sequence:

&nbsp;&nbsp;&nbsp;&nbsp;stat ::= ‘**;**’

Both function calls and assignments can start with an open parenthesis. This possibility leads to an ambiguity in Lua's grammar. Consider the following fragment:

&nbsp;&nbsp;&nbsp;a = b + c
&nbsp;&nbsp;&nbsp;&nbsp;(print or io.write)('done')

The grammar could see this fragment in two ways:

&nbsp;&nbsp;&nbsp;a = b + c(print or io.write)('done')
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;a = b + c; (print or io.write)('done')

The current parser always sees such constructions in the first way, interpreting the open parenthesis as the start of the arguments to a call. To avoid this ambiguity, it is a good practice to always precede with a semicolon statements that start with a parenthesis:

&nbsp;&nbsp;&nbsp;;(print or io.write)('done')

A block can be explicitly delimited to produce a single statement:

&nbsp;&nbsp;&nbsp;&nbsp;stat ::= **do** block **end**

Explicit blocks are useful to control the scope of variable declarations. Explicit blocks are also sometimes used to add a **return** statement in the middle of another block (see [[#3.3.4 – Control Structures|§3.3.4]]).

### 3.3.2 – Chunks

The unit of compilation of Lua is called a _chunk_. Syntactically, a chunk is simply a block:

&nbsp;&nbsp;&nbsp;&nbsp;chunk ::= block

Lua handles a chunk as the body of an anonymous function with a variable number of arguments (see [[#3.4.11 – Function Definitions|§3.4.11]]). As such, chunks can define local variables, receive arguments, and return values. Moreover, such anonymous function is compiled as in the scope of an external local variable called `_ENV` (see [[2-Basic Concepts#2.2 – Scopes, Variables, and Environments|§2.2]]). The resulting function always has `_ENV` as its only external variable, even if it does not use that variable.

A chunk can be stored in a file or in a string inside the host program. To execute a chunk, Lua first _loads_ it, precompiling the chunk's code into instructions for a virtual machine, and then Lua executes the compiled code with an interpreter for the virtual machine.

Chunks can also be precompiled into binary form; see the program `luac` and the function <code>[[6-The Standard Libraries#`string.dump (function [, strip])`|string.dump]]</code> for details. Programs in source and compiled forms are interchangeable; Lua automatically detects the file type and acts accordingly (see <code>[[6-The Standard Libraries#`load (chunk [, chunkname [, mode [, env ])`|load]]</code>). Be aware that, unlike source code, maliciously crafted binary chunks can crash the interpreter.

### 3.3.3 – Assignment

Lua allows multiple assignments. Therefore, the syntax for assignment defines a list of variables on the left side and a list of expressions on the right side. The elements in both lists are separated by commas:

&nbsp;&nbsp;&nbsp;stat ::= varlist ‘**=**’ explist
&nbsp;&nbsp;&nbsp;&nbsp;varlist ::= var {‘**,**’ var}
&nbsp;&nbsp;&nbsp;&nbsp;explist ::= exp {‘**,**’ exp}

Expressions are discussed in [[#3.4 – Expressions|§3.4]].

Before the assignment, the list of values is _adjusted_ to the length of the list of variables (see [[#3.4.12 – Lists of Expressions, Multiple Results, and Adjustment|§3.4.12]]).

If a variable is both assigned and read inside a multiple assignment, Lua ensures that all reads get the value of the variable before the assignment. Thus the code
```lua
i = 3
i, a[i] = i+1, 20
```
sets `a[3]` to 20, without affecting `a[4]` because the `i` in `a[i]` is evaluated (to 3) before it is assigned 4. Similarly, the line
```lua
x, y = y, x
```
exchanges the values of `x` and `y`, and
```lua
x, y, z = y, z, x
```
cyclically permutes the values of `x`, `y`, and `z`.

Note that this guarantee covers only accesses syntactically inside the assignment statement. If a function or a metamethod called during the assignment changes the value of a variable, Lua gives no guarantees about the order of that access.

An assignment to a global name `x = val` is equivalent to the assignment `_ENV.x = val` (see [[2-Basic Concepts#2.2 – Scopes, Variables, and Environments|§2.2]]).

The meaning of assignments to table fields and global variables (which are actually table fields, too) can be changed via metatables (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

### 3.3.4 – Control Structures

The control structures **if**, **while**, and **repeat** have the usual meaning and familiar syntax:

&nbsp;&nbsp;&nbsp;stat ::= **while** exp **do** block **end**
&nbsp;&nbsp;&nbsp;&nbsp;stat ::= **repeat** block **until** exp
&nbsp;&nbsp;&nbsp;&nbsp;stat ::= **if** exp **then** block {**elseif** exp **then** block} [**else** block] **end**

Lua also has a **for** statement, in two flavors (see [[#3.3.5 – For Statement|§3.3.5]]).

The condition expression of a control structure can return any value. Both **false** and **nil** test false. All values different from **nil** and **false** test true. In particular, the number 0 and the empty string also test true.

In the **repeat**–**until** loop, the inner block does not end at the **until** keyword, but only after the condition. So, the condition can refer to local variables declared inside the loop block.

The **goto** statement transfers the program control to a label. For syntactical reasons, labels in Lua are considered statements too:

&nbsp;&nbsp;&nbsp;stat ::= **goto** Name
&nbsp;&nbsp;&nbsp;&nbsp;stat ::= label
&nbsp;&nbsp;&nbsp;&nbsp;label ::= ‘**::**’ Name ‘**::**’

A label is visible in the entire block where it is defined, except inside nested functions. A goto can jump to any visible label as long as it does not enter into the scope of a variable declaration. A label should not be declared where a previous label with the same name is visible, even if this other label has been declared in an enclosing block.

The **break** statement terminates the execution of a **while**, **repeat**, or **for** loop, skipping to the next statement after the loop:

&nbsp;&nbsp;&nbsp;stat ::= **break**

A **break** ends the innermost enclosing loop.

The **return** statement is used to return values from a function or a chunk (which is handled as an anonymous function). Functions can return more than one value, so the syntax for the **return** statement is

&nbsp;&nbsp;&nbsp;stat ::= **return** [explist] [‘**;**’]

The **return** statement can only be written as the last statement of a block. If it is necessary to **return** in the middle of a block, then an explicit inner block can be used, as in the idiom `do return end`, because now **return** is the last statement in its (inner) block.

### 3.3.5 – For Statement

The **for** statement has two forms: one numerical and one generic.

#### The numerical **for** loop

The numerical **for** loop repeats a block of code while a control variable goes through an arithmetic progression. It has the following syntax:

&nbsp;&nbsp;&nbsp;stat ::= **for** Name ‘**=**’ exp ‘**,**’ exp [‘**,**’ exp] **do** block **end**

The given identifier (Name) defines the control variable, which is a new read-only (`const`) variable local to the loop body (_block_).

The loop starts by evaluating once the three control expressions. Their values are called respectively the _initial value_, the _limit_, and the _step_. If the step is absent, it defaults to 1.

If both the initial value and the step are integers, the loop is done with integers; note that the limit may not be an integer. Otherwise, the three values are converted to floats and the loop is done with floats. Beware of floating-point accuracy in this case.

After that initialization, the loop body is repeated with the value of the control variable going through an arithmetic progression, starting at the initial value, with a common difference given by the step. A negative step makes a decreasing sequence; a step equal to zero raises an error. The loop continues while the value is less than or equal to the limit (greater than or equal to for a negative step). If the initial value is already greater than the limit (or less than, if the step is negative), the body is not executed.

For integer loops, the control variable never wraps around; instead, the loop ends in case of an overflow.

#### The generic **for** loop

The generic **for** statement works over functions, called _iterators_. On each iteration, the iterator function is called to produce a new value, stopping when this new value is **nil**. The generic **for** loop has the following syntax:

&nbsp;&nbsp;&nbsp;stat ::= **for** namelist **in** explist **do** block **end**
&nbsp;&nbsp;&nbsp;&nbsp;namelist ::= Name {‘**,**’ Name}

A **for** statement like

&nbsp;&nbsp;&nbsp;&nbsp;for _var_1_, ···, _var_n_ in _explist_ do _body_ end

works as follows.

The names _var_i_ declare loop variables local to the loop body. The first of these variables is the _control variable_, which is a read-only (`const`) variable.

The loop starts by evaluating _explist_ to produce four values: an _iterator function_, a _state_, an initial value for the control variable, and a _closing value_.

Then, at each iteration, Lua calls the iterator function with two arguments: the state and the control variable. The results from this call are then assigned to the loop variables, following the rules of multiple assignments (see [[#3.3.3 – Assignment|§3.3.3]]). If the control variable becomes **nil**, the loop terminates. Otherwise, the body is executed and the loop goes to the next iteration.

The closing value behaves like a to-be-closed variable (see [[#3.3.8 – To-be-closed Variables|§3.3.8]]), which can be used to release resources when the loop ends. Otherwise, it does not interfere with the loop.

### 3.3.6 – Function Calls as Statements

To allow possible side-effects, function calls can be executed as statements:

&nbsp;&nbsp;&nbsp;stat ::= functioncall

In this case, all returned values are thrown away. Function calls are explained in [[#3.4.10 – Function Calls|§3.4.10]].

### 3.3.7 – Variable Declarations

Local and global variables can be declared anywhere inside a block. The declaration can include an initialization:

&nbsp;&nbsp;&nbsp;stat ::= **local** attnamelist [‘**=**’ explist]
&nbsp;&nbsp;&nbsp;&nbsp;stat ::= **global** attnamelist [‘**=**’ explist]

If there is no initialization, local variables are initialized with **nil**; global variables are left unchanged. Otherwise, the initialization gets the same adjustment of a multiple assignment (see [[#3.3.3 – Assignment|§3.3.3]]). Moreover, for global variables, the initialization will raise a runtime error if the variable is already defined, that is, it has a non-nil value.

The list of names may be prefixed by an attribute (a name between angle brackets) and each variable name may be postfixed by an attribute:

&nbsp;&nbsp;&nbsp;attnamelist ::=  [attrib] Name [attrib] {‘**,**’ Name [attrib]}
&nbsp;&nbsp;&nbsp;&nbsp;attrib ::= ‘**<**’ Name ‘**>**’

A prefixed attribute applies to all names in the list; a postfixed attribute applies to its particular name. There are two possible attributes: `const`, which declares a _constant_ or _read-only_ variable, that is, a variable that cannot be used as the left-hand side of an assignment, and `close`, which declares a to-be-closed variable (see [[#3.3.8 – To-be-closed Variables|§3.3.8]]). Only local variables can have the `close` attribute. A list of variables can contain at most one to-be-closed variable.

Lua offers also a collective declaration for global variables:

&nbsp;&nbsp;&nbsp;stat ::= **global** [attrib] ‘*****’

This special form implicitly declares as globals all names not explicitly declared previously. In particular, `global<const> *` implicitly declares as read-only globals all names not explicitly declared previously; see the following example:
```lua
global X
global<const> *
print(math.pi)   -- Ok, 'print' and 'math' are read-only
X = 1            -- Ok, declared as read-write
Y = 1            -- Error, Y is read-only
```
As noted in [[2-Basic Concepts#2.2 – Scopes, Variables, and Environments|§2.2]], all chunks start with an implicit declaration `global *`, but this preambular declaration becomes void inside the scope of any other **global** declaration. Therefore, a program that does not use global declarations or start with `global *` has free read-write access to any global; a program that starts with `global<const> *` has free read-only access to any global; and a program that starts with any other global declaration (e.g., `global none`) can only refer to declared variables.

Note that, for global variables, the effect of any declaration is only syntactical (except for the optional assignment):
```lua
global X <const>, _G
X = 1           -- ERROR
_ENV.X = 1      -- Ok
_G.print(X)     -- Ok
foo()           -- 'foo' can freely change any global
```
A chunk is also a block (see [[#3.3.2 – Chunks|§3.3.2]]), and so variables can be declared in a chunk outside any explicit block.

The visibility rules for variable declarations are explained in [[2-Basic Concepts#2.2 – Scopes, Variables, and Environments|§2.2]].

### 3.3.8 – To-be-closed Variables

A to-be-closed variable behaves like a constant local variable, except that its value is _closed_ whenever the variable goes out of scope, including normal block termination, exiting its block by **break**/**goto**/**return**, or exiting by an error.

Here, to _close_ a value means to call its `__close` metamethod. When calling the metamethod, the value itself is passed as the first argument. If there was an error, the error object that caused the exit is passed as a second argument; otherwise, there is no second argument.

The value assigned to a to-be-closed variable must have a `__close` metamethod or be a false value. (**nil** and **false** are ignored as to-be-closed values.)

If several to-be-closed variables go out of scope at the same event, they are closed in the reverse order that they were declared.

If there is any error while running a closing method, that error is handled like an error in the regular code where the variable was defined. After an error, the other pending closing methods will still be called.

If a coroutine yields and is never resumed again, some variables may never go out of scope, and therefore they will never be closed. (These variables are the ones created inside the coroutine and in scope at the point where the coroutine yielded.) Similarly, if a coroutine ends with an error, it does not unwind its stack, so it does not close any variable. In both cases, you can either use finalizers or call <code>[[6-The Standard Libraries#`coroutine.close ([co])`|coroutine.close]]</code> to close the variables. However, if the coroutine was created through <code>[[6-The Standard Libraries#`coroutine.wrap (f)`|coroutine.warp]]</code>, then its corresponding function will close the coroutine in case of errors.

## 3.4 – Expressions

The basic expressions in Lua are the following:

&nbsp;&nbsp;&nbsp;exp ::= prefixexp
&nbsp;&nbsp;&nbsp;&nbsp;exp ::= **nil** | **false** | **true**
&nbsp;&nbsp;&nbsp;&nbsp;exp ::= Numeral
&nbsp;&nbsp;&nbsp;&nbsp;exp ::= LiteralString
&nbsp;&nbsp;&nbsp;&nbsp;exp ::= functiondef
&nbsp;&nbsp;&nbsp;&nbsp;exp ::= tableconstructor
&nbsp;&nbsp;&nbsp;&nbsp;exp ::= ‘**...**’
&nbsp;&nbsp;&nbsp;&nbsp;exp ::= exp binop exp
&nbsp;&nbsp;&nbsp;&nbsp;exp ::= unop exp
&nbsp;&nbsp;&nbsp;&nbsp;prefixexp ::= var | functioncall | ‘**(**’ exp ‘**)**’

Numerals and literal strings are explained in [[#3.1 – Lexical Conventions|§3.1]]; variables are explained in [[#3.2 – Variables|§3.2]]; function definitions are explained in [[#3.4.11 – Function Definitions|§3.4.11]]; function calls are explained in [[#3.4.10 – Function Calls|§3.4.10]]; table constructors are explained in [[#3.4.9 – Table Constructors|§3.4.9]]. Vararg expressions, denoted by three dots ('`...`'), can only be used when directly inside a variadic function; they are explained in [[#3.4.11 – Function Definitions|§3.4.11]].

Binary operators comprise arithmetic operators (see [[#3.4.1 – Arithmetic Operators|§3.4.1]]), bitwise operators (see [[#3.4.2 – Bitwise Operators|§3.4.2]]), relational operators (see [[#3.4.4 – Relational Operators|§3.4.4]]), logical operators (see [[#3.4.5 – Logical Operators|§3.4.5]]), and the concatenation operator (see [[#3.4.6 – Concatenation|§3.4.6]]). Unary operators comprise the unary minus (see [[#3.4.1 – Arithmetic Operators|§3.4.1]]), the unary bitwise NOT (see [[#3.4.2 – Bitwise Operators|§3.4.2]]), the unary logical **not** (see [[#3.4.5 – Logical Operators|§3.4.5]]), and the unary _length operator_ (see [[#3.4.7 – The Length Operator|§3.4.7]]).

### 3.4.1 – Arithmetic Operators

Lua supports the following arithmetic operators:

- **`+`**: addition
- **`-`**: subtraction
- **`*`**: multiplication
- **`/`**: float division
- **`//`**: floor division
- **`%`**: modulo
- **`^`**: exponentiation
- **`-`**: unary minus

With the exception of exponentiation and float division, the arithmetic operators work as follows: If both operands are integers, the operation is performed over integers and the result is an integer. Otherwise, if both operands are numbers, then they are converted to floats, the operation is performed following the machine's rules for floating-point arithmetic (usually the IEEE 754 standard), and the result is a float. (The string library coerces strings to numbers in arithmetic operations; see [[#3.4.3 – Coercions and Conversions|§3.4.3]] for details.)

Exponentiation and float division (`/`) always convert their operands to floats and the result is always a float. Exponentiation uses the ISO C function `pow`, so that it works for non-integer exponents too.

Floor division (`//`) is a division that rounds the quotient towards minus infinity, resulting in the floor of the division of its operands.

Modulo is defined as the remainder of a division that rounds the quotient towards minus infinity (floor division).

In case of overflows in integer arithmetic, all operations _wrap around_.

### 3.4.2 – Bitwise Operators

Lua supports the following bitwise operators:

- **`&`**: bitwise AND
- **`|`**: bitwise OR
- **`~`**: bitwise exclusive OR
- **`>>`**: right shift
- **`<<`**: left shift
- **`~`**: unary bitwise NOT

All bitwise operations convert its operands to integers (see [[#3.4.3 – Coercions and Conversions|§3.4.3]]), operate on all bits of those integers, and result in an integer.

Both right and left shifts fill the vacant bits with zeros. Negative displacements shift to the other direction; displacements with absolute values equal to or higher than the number of bits in an integer result in zero (as all bits are shifted out).

### 3.4.3 – Coercions and Conversions

Lua provides some automatic conversions between some types and representations at run time. Bitwise operators always convert float operands to integers. Exponentiation and float division always convert integer operands to floats. All other arithmetic operations applied to mixed numbers (integers and floats) convert the integer operand to a float. The C API also converts both integers to floats and floats to integers, as needed. Moreover, string concatenation accepts numbers as arguments, besides strings.

In a conversion from integer to float, if the integer value has an exact representation as a float, that is the result. Otherwise, the conversion gets the nearest higher or the nearest lower representable value. This kind of conversion never fails.

The conversion from float to integer checks whether the float has an exact representation as an integer (that is, the float has an integral value and it is in the range of integer representation). If it does, that representation is the result. Otherwise, the conversion fails.

Several places in Lua coerce strings to numbers when necessary. In particular, the string library sets metamethods that try to coerce strings to numbers in all arithmetic operations. If the conversion fails, the library calls the metamethod of the other operand (if present) or it raises an error. Note that bitwise operators do not do this coercion.

It is always a good practice not to rely on the implicit coercions from strings to numbers, as they are not always applied; in particular, `"1"==1` is false and `"1"<1` raises an error (see [[#3.4.4 – Relational Operators|§3.4.4]]). These coercions exist mainly for compatibility and may be removed in future versions of the language.

A string is converted to an integer or a float following its syntax and the rules of the Lua lexer. The string may have also leading and trailing whitespaces and a sign. All conversions from strings to numbers accept both a dot and the current locale mark as the radix character. (The Lua lexer, however, accepts only a dot.) If the string is not a valid numeral, the conversion fails. If necessary, the result of this first step is then converted to a specific number subtype following the previous rules for conversions between floats and integers.

The conversion from numbers to strings uses a non-specified human-readable format. To convert numbers to strings in any specific way, use the function <code>[[6-The Standard Libraries#`string.format (formatstring, ···)`|string.format]]</code>.

### 3.4.4 – Relational Operators

Lua supports the following relational operators:

- **`​==`**: equality
- **`~=`**: inequality
- **`<`**: less than
- **`>`**: greater than
- **`<=`**: less or equal
- **`>=`**: greater or equal

These operators always result in **false** or **true**.

Equality (`​==`) first compares the type of its operands. If the types are different, then the result is **false**. Otherwise, the values of the operands are compared. Strings are equal if they have the same byte content. Numbers are equal if they denote the same mathematical value.

Tables, userdata, and threads are compared by reference: two objects are considered equal only if they are the same object. Every time you create a new object (a table, a userdata, or a thread), this new object is different from any previously existing object. A function is always equal to itself. Functions with any detectable difference (different behavior, different definition) are always different. Functions created at different times but with no detectable differences may be classified as equal or not (depending on internal caching details).

You can change the way that Lua compares tables and userdata by using the `__eq` metamethod (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

Equality comparisons do not convert strings to numbers or vice versa. Thus, `"0"==0` evaluates to **false**, and `t[0]` and `t["0"]` denote different entries in a table.

The operator `~=` is exactly the negation of equality (`​==`).

The order operators work as follows. If both arguments are numbers, then they are compared according to their mathematical values, regardless of their subtypes. Otherwise, if both arguments are strings, then their values are compared according to the current locale. Otherwise, Lua tries to call the `__lt` or the `__le` metamethod (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]). A comparison `a > b` is translated to `b < a` and `a >= b` is translated to `b <= a`.

Following the IEEE 754 standard, the special value NaN is considered neither less than, nor equal to, nor greater than any value, including itself.

### 3.4.5 – Logical Operators

The logical operators in Lua are **and**, **or**, and **not**. Like the control structures (see [[#3.3.4 – Control Structures|§3.3.4]]), all logical operators consider both **false** and **nil** as false and anything else as true.

The negation operator **not** always returns **false** or **true**. The conjunction operator **and** returns its first argument if this value is **false** or **nil**; otherwise, **and** returns its second argument. The disjunction operator **or** returns its first argument if this value is different from **nil** and **false**; otherwise, **or** returns its second argument. Both **and** and **or** use short-circuit evaluation; that is, the second operand is evaluated only if necessary. Here are some examples:
```
10 or 20            --> 10
10 or error()       --> 10
nil or "a"          --> "a"
nil and 10          --> nil
false and error()   --> false
false and nil       --> false
false or nil        --> nil
10 and 20           --> 20
```
### 3.4.6 – Concatenation

The string concatenation operator in Lua is denoted by two dots ('`..`'). If both operands are strings or numbers, then the numbers are converted to strings in a non-specified format (see [[#3.4.3 – Coercions and Conversions|§3.4.3]]). Otherwise, the `__concat` metamethod is called (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

### 3.4.7 – The Length Operator

The length operator is denoted by the unary prefix operator `#`.

The length of a string is its number of bytes. (That is the usual meaning of string length when each character is one byte.)

The length operator applied on a table returns a border in that table. A _border_ in a table `t` is any non-negative integer that satisfies the following condition:
```lua
(border == 0 or t[border] ~= nil) and
(t[border + 1] == nil or border == math.maxinteger)
```
In words, a border is any positive integer index present in the table that is followed by an absent index, plus two limit cases: zero, when index 1 is absent; and the maximum value for an integer, when that index is present. Note that keys that are not positive integers do not interfere with borders.

A table with exactly one border is called a _sequence_. For instance, the table `{10,20,30,40,50}` is a sequence, as it has only one border (5). The table `{10,20,30,nil,50}` has two borders (3 and 5), and therefore it is not a sequence. (The **nil** at index 4 is called a _hole_.) The table `{nil,20,30,nil,nil,60,nil}` has three borders (0, 3, and 6), so it is not a sequence, too. The table `{}` is a sequence with border 0.

When `t` is a sequence, `#t` returns its only border, which corresponds to the intuitive notion of the length of the sequence. When `t` is not a sequence, `#t` can return any of its borders. (The exact one depends on details of the internal representation of the table, which in turn can depend on how the table was populated and the memory addresses of its non-numeric keys.)

The computation of the length of a table has a guaranteed worst time of _O(log n)_, where _n_ is the largest integer key in the table.

A program can modify the behavior of the length operator for any value but strings through the `__len` metamethod (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

### 3.4.8 – Precedence

Operator precedence in Lua follows the table below, from lower to higher priority:
```
or
and
<     >     <=    >=    ~=    ==
|
~
&
<<    >>
..
+     -
*     /     //    %
unary operators (not   #     -     ~)
^
```
As usual, you can use parentheses to change the precedences of an expression. The concatenation ('`..`') and exponentiation ('`^`') operators are right associative. All other binary operators are left associative.

### 3.4.9 – Table Constructors

Table constructors are expressions that create tables. Every time a constructor is evaluated, a new table is created. A constructor can be used to create an empty table or to create a table and initialize some of its fields. The general syntax for constructors is

&nbsp;&nbsp;&nbsp;tableconstructor ::= ‘**{**’ [fieldlist] ‘**}**’
&nbsp;&nbsp;&nbsp;&nbsp;fieldlist ::= field {fieldsep field} [fieldsep]
&nbsp;&nbsp;&nbsp;&nbsp;field ::= ‘**[**’ exp ‘**]**’ ‘**=**’ exp | Name ‘**=**’ exp | exp
&nbsp;&nbsp;&nbsp;&nbsp;fieldsep ::= ‘**,**’ | ‘**;**’

Each field of the form `[exp1] = exp2` adds to the new table an entry with key `exp1` and value `exp2`. A field of the form `name = exp` is equivalent to `["name"] = exp`. Fields of the form `exp` are equivalent to `[i] = exp`, where `i` are consecutive integers starting with 1; fields in the other formats do not affect this counting. For example,
```lua
a = { [f(1)] = g; "x", "y"; x = 1, f(x), [30] = 23; 45 }
```
is equivalent to
```lua
do
  local t = {}
  t[f(1)] = g
  t[1] = "x"         -- 1st exp
  t[2] = "y"         -- 2nd exp
  t.x = 1            -- t["x"] = 1
  t[3] = f(x)        -- 3rd exp
  t[30] = 23
  t[4] = 45          -- 4th exp
  a = t
end
```
The order of the assignments in a constructor is undefined. (This order would be relevant only when there are repeated keys.)

If the last field in the list has the form `exp` and the expression is a multires expression, then all values returned by this expression enter the list consecutively (see [[#3.4.12 – Lists of Expressions, Multiple Results, and Adjustment|§3.4.12]]).

The field list can have an optional trailing separator, as a convenience for machine-generated code.

### 3.4.10 – Function Calls

A function call in Lua has the following syntax:
```
functioncall ::= prefixexp args
```
In a function call, first prefixexp and args are evaluated. If the value of prefixexp has type _function_, then this function is called with the given arguments. Otherwise, if present, the prefixexp `__call` metamethod is called: its first argument is the value of prefixexp, followed by the original call arguments (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

The form

&nbsp;&nbsp;&nbsp;functioncall ::= prefixexp ‘**:**’ Name args

can be used to emulate methods. A call `v:name(_args_)` is syntactic sugar for `v.name(v, _args_)`, except that `v` is evaluated only once.

Arguments have the following syntax:

&nbsp;&nbsp;&nbsp;args ::= ‘**(**’ [explist] ‘**)**’
&nbsp;&nbsp;&nbsp;&nbsp;args ::= tableconstructor
&nbsp;&nbsp;&nbsp;&nbsp;args ::= LiteralString

All argument expressions are evaluated before the call. A call of the form `f{`_`fields`_`}` is syntactic sugar for `f({`_`fields`_`})`; that is, the argument list is a single new table. A call of the form `f'`_`string`_`'` (or `f"`_`string`_`"` or `f[[`_`string`_`]]`) is syntactic sugar for `f('`_`string`_`')`; that is, the argument list is a single literal string.

A call of the form `return _functioncall_` not in the scope of a to-be-closed variable is called a _tail call_. Lua implements _proper tail calls_ (or _proper tail recursion_): In a tail call, the called function reuses the stack entry of the calling function. Therefore, there is no limit on the number of nested tail calls that a program can execute. However, a tail call erases any debug information about the calling function. Note that a tail call only happens with a particular syntax, where the **return** has one single function call as argument, and it is outside the scope of any to-be-closed variable. This syntax makes the calling function return exactly the returns of the called function, without any intervening action. So, none of the following examples are tail calls:
```lua
return (f(x))        -- results adjusted to 1
return 2 * f(x)      -- result multiplied by 2
return x, f(x)       -- additional results
f(x); return         -- results discarded
return x or f(x)     -- results adjusted to 1
```
### 3.4.11 – Function Definitions

The syntax for a function definition is

&nbsp;&nbsp;&nbsp;functiondef ::= **function** funcbody
&nbsp;&nbsp;&nbsp;&nbsp;funcbody ::= ‘**(**’ [parlist] ‘**)**’ block **end**

The following syntactic sugar simplifies function definitions:

&nbsp;&nbsp;&nbsp;stat ::= **function** funcname funcbody
&nbsp;&nbsp;&nbsp;&nbsp;stat ::= **local** **function** Name funcbody
&nbsp;&nbsp;&nbsp;&nbsp;stat ::= **global** **function** Name funcbody
&nbsp;&nbsp;&nbsp;&nbsp;funcname ::= Name {‘**.**’ Name} [‘**:**’ Name]

The statement

&nbsp;&nbsp;&nbsp;function f () _body_ end

translates to

&nbsp;&nbsp;&nbsp;f = function () _body_ end

The statement

&nbsp;&nbsp;&nbsp;function t.a.b.c.f () _body_ end

translates to

&nbsp;&nbsp;&nbsp;t.a.b.c.f = function () _body_ end

The statement

&nbsp;&nbsp;&nbsp;local function f () _body_ end

translates to

&nbsp;&nbsp;&nbsp;local f; f = function () _body_ end

not to

&nbsp;&nbsp;&nbsp;local f = function () _body_ end

(This only makes a difference when the body of the function contains recursive references to `f`.) Similarly, the statement

&nbsp;&nbsp;&nbsp;global function f () _body_ end

translates to

&nbsp;&nbsp;&nbsp;global f; global f = function () _body_ end

The second **global** makes the assignment an initialization, which will raise an error if that global is already defined.

The _colon_ syntax is used to emulate _methods_, adding an implicit extra parameter `self` to the function. Thus, the statement

&nbsp;&nbsp;&nbsp;function t.a.b.c:f (_params_) _body_ end

is syntactic sugar for

&nbsp;&nbsp;&nbsp;t.a.b.c.f = function (self, _params_) _body_ end

A function definition is an executable expression, whose value has type _function_. When Lua precompiles a chunk, all its function bodies are precompiled too, but they are not created yet. Then, whenever Lua executes the function definition, the function is _instantiated_ (or _closed_). This function instance, or _closure_, is the final value of the expression.

Results are returned using the **return** statement (see [[#3.3.4 – Control Structures|§3.3.4]]). If control reaches the end of a function without encountering a **return** statement, then the function returns with no results.

There is a system-dependent limit on the number of values that a function may return. This limit is guaranteed to be at least 1000.

#### Parameters

Parameters act as local variables that are initialized with the argument values:

&nbsp;&nbsp;&nbsp;parlist ::= namelist [‘**,**’ varargparam] | varargparam
&nbsp;&nbsp;&nbsp;&nbsp;varargparam ::= ‘**...**’ [Name]

When a Lua function is called, it adjusts its list of arguments to the length of its list of parameters (see [[#3.4.12 – Lists of Expressions, Multiple Results, and Adjustment|§3.4.12]]), unless the function is a _variadic function_, which is indicated by three dots ('`...`') at the end of its parameter list. A variadic function does not adjust its argument list; instead, it collects all extra arguments and supplies them to the function through a _vararg table_. In that table, the values at indices 1, 2, etc. are the extra arguments, and the value at index "`n`" is the number of extra arguments.

As an example, consider the following definitions:
```lua
function f(a, b) end
function g(a, b, ...) end
function r() return 1,2,3 end
```
Then, we have the following mapping from arguments to parameters and to the vararg table:
```lua
CALL             PARAMETERS

f(3)             a=3, b=nil
f(3, 4)          a=3, b=4
f(3, 4, 5)       a=3, b=4
f(r(), 10)       a=1, b=10
f(r())           a=1, b=2

g(3)             a=3, b=nil, va. table ->  {n = 0}
g(3, 4)          a=3, b=4,   va. table ->  {n = 0}
g(3, 4, 5, 8)    a=3, b=4,   va. table ->  {5, 8, n = 2}
g(5, r())        a=5, b=1,   va. table ->  {2, 3, n = 2}
```
A vararg table in a variadic function can have an optional name, given after the three dots. When present, that name denotes a read-only local variable that refers to the vararg table. If the vararg table does not have a name, it can only be accessed through a vararg expression.

A vararg expression is also written as three dots, and its value is a list of the values in the vararg table, from 1 to the integer value at index "`n`". (Therefore, if the code does not modify the vararg table, this list corresponds to the extra arguments in the function call.) This list behaves like the results from a function with multiple results (see [[#3.4.12 – Lists of Expressions, Multiple Results, and Adjustment|§3.4.12]]).

As an optimization, if the vararg table satisfies some conditions, the code does not create an actual table and instead translates the indexing expressions and the vararg expressions into accesses to the internal vararg data. The conditions are as follows: If the vararg table has a name, that name is not an upvalue in a nested function and it is used only as the base table in the syntactic constructions `t[exp]` or `t.id`. Note that an anonymous vararg table always satisfy these conditions.

### 3.4.12 – Lists of Expressions, Multiple Results, and Adjustment

Both function calls and vararg expressions can result in multiple values. These expressions are called _multires expressions_.

When a multires expression is used as the last element of a list of expressions, all results from the expression are added to the list of values produced by the list of expressions. Note that a single expression in a place that expects a list of expressions is the last expression in that (singleton) list.

These are the places where Lua expects a list of expressions:

- A **return** statement, for instance `return e1,e2,e3` (see [[#3.3.4 – Control Structures|§3.3.4]]).
- A table constructor, for instance `{e1,e2,e3}` (see [[#3.4.9 – Table Constructors|§3.4.9]]).
- The arguments of a function call, for instance `foo(e1,e2,e3)` (see [[#3.4.10 – Function Calls|§3.4.10]]).
- A multiple assignment, for instance `a,b,c = e1,e2,e3` (see [[#3.3.3 – Assignment|§3.3.3]]).
- A local or global declaration, which is similar to a multiple assignment.
- The initial values in a generic **for** loop, for instance `for k in e1,e2,e3 do ... end` (see [[#3.3.5 – For Statement|§3.3.5]]).

In the last four cases, the list of values from the list of expressions must be _adjusted_ to a specific length: the number of parameters in a call to a non-variadic function (see [[#3.4.11 – Function Definitions|§3.4.11]]), the number of variables in a multiple assignment or a declaration, and exactly four values for a generic **for** loop. The _adjustment_ follows these rules: If there are more values than needed, the extra values are thrown away; if there are fewer values than needed, the list is extended with **nil**'s. When the list of expressions ends with a multires expression, all results from that expression enter the list of values before the adjustment.

When a multires expression is used in a list of expressions without being the last element, or in a place where the syntax expects a single expression, Lua adjusts the result list of that expression to one element. As a particular case, the syntax expects a single expression inside a parenthesized expression; therefore, adding parentheses around a multires expression forces it to produce exactly one result.

We seldom need to use a vararg expression in a place where the syntax expects a single expression. (Usually it is simpler to add a regular parameter before the variadic part and use that parameter.) When there is such a need, we recommend assigning the vararg expression to a single variable and using that variable in its place.

Here are some examples of uses of multires expressions. In all cases, when the construction needs "the n-th result" and there is no such result, it uses a **nil**.

```lua
print(x, f())      -- prints x and all results from f().
print(x, (f()))    -- prints x and the first result from f().
print(f(), x)      -- prints the first result from f() and x.
print(1 + f())     -- prints 1 added to the first result from f().
local x = ...      -- x gets the first vararg argument.
x,y = ...          -- x gets the first vararg argument,
                   -- y gets the second vararg argument.
x,y,z = w, f()     -- x gets w, y gets the first result from f(),
                   -- z gets the second result from f().
x,y,z = f()        -- x gets the first result from f(),
                   -- y gets the second result from f(),
                   -- z gets the third result from f().
x,y,z = f(), g()   -- x gets the first result from f(),
                   -- y gets the first result from g(),
                   -- z gets the second result from g().
x,y,z = (f())      -- x gets the first result from f(), y and z get nil.
return f()         -- returns all results from f().
return x, ...      -- returns x and all received vararg arguments.
return x,y,f()     -- returns x, y, and all results from f().
{f()}              -- creates a list with all results from f().
{...}              -- creates a list with all vararg arguments.
{f(), 5}           -- creates a list with the first result from f() and 5.
```