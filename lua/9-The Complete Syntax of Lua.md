Here is the complete syntax of Lua in extended BNF. As usual in extended BNF, {A} means 0 or more As, and [A] means an optional A. (For operator precedences, see [[3-The Language#3.4.8 – Precedence|§3.4.8]]; for a description of the terminals Name, Numeral, and LiteralString, see [[3-The Language#3.1 – Lexical Conventions|§3.1]].)

&nbsp;&nbsp;&nbsp;&nbsp;chunk ::= block

&nbsp;&nbsp;&nbsp;&nbsp;block ::= {stat} [retstat]

&nbsp;&nbsp;&nbsp;&nbsp;stat ::=  ‘**;**’ | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; varlist ‘**=**’ explist | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; functioncall | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; label | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **break** | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **goto** Name | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **do** block **end** | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **while** exp **do** block **end** | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **repeat** block **until** exp | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **if** exp **then** block {**elseif** exp **then** block} [**else** block] **end** | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **for** Name ‘**=**’ exp ‘**,**’ exp [‘**,**’ exp] **do** block **end** | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **for** namelist **in** explist **do** block **end** | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **function** funcname funcbody | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **local** **function** Name funcbody | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **global** **function** Name funcbody | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **local** attnamelist [‘**=**’ explist] | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **global** attnamelist | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **global** [attrib] ‘*****’ 

&nbsp;&nbsp;&nbsp;&nbsp;attnamelist ::=  [attrib] Name [attrib] {‘**,**’ Name [attrib]}

&nbsp;&nbsp;&nbsp;&nbsp;attrib ::= ‘**<**’ Name ‘**>**’

&nbsp;&nbsp;&nbsp;&nbsp;retstat ::= **return** [explist] [‘**;**’]

&nbsp;&nbsp;&nbsp;&nbsp;label ::= ‘**::**’ Name ‘**::**’

&nbsp;&nbsp;&nbsp;&nbsp;funcname ::= Name {‘**.**’ Name} [‘**:**’ Name]

&nbsp;&nbsp;&nbsp;&nbsp;varlist ::= var {‘**,**’ var}

&nbsp;&nbsp;&nbsp;&nbsp;var ::=  Name | prefixexp ‘**[**’ exp ‘**]**’ | prefixexp ‘**.**’ Name 

&nbsp;&nbsp;&nbsp;&nbsp;namelist ::= Name {‘**,**’ Name}

&nbsp;&nbsp;&nbsp;&nbsp;explist ::= exp {‘**,**’ exp}

&nbsp;&nbsp;&nbsp;&nbsp;exp ::=  **nil** | **false** | **true** | Numeral | LiteralString | ‘**...**’ | functiondef | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; prefixexp | tableconstructor | exp binop exp | unop exp 

&nbsp;&nbsp;&nbsp;&nbsp;prefixexp ::= var | functioncall | ‘**(**’ exp ‘**)**’

&nbsp;&nbsp;&nbsp;&nbsp;functioncall ::=  prefixexp args | prefixexp ‘**:**’ Name args 

&nbsp;&nbsp;&nbsp;&nbsp;args ::=  ‘**(**’ [explist] ‘**)**’ | tableconstructor | LiteralString 

&nbsp;&nbsp;&nbsp;&nbsp;functiondef ::= **function** funcbody

&nbsp;&nbsp;&nbsp;&nbsp;funcbody ::= ‘**(**’ [parlist] ‘**)**’ block **end**

&nbsp;&nbsp;&nbsp;&nbsp;parlist ::= namelist [‘**,**’ varargparam] | varargparam

&nbsp;&nbsp;&nbsp;&nbsp;varargparam ::= ‘**...**’ [Name]

&nbsp;&nbsp;&nbsp;&nbsp;tableconstructor ::= ‘**{**’ [fieldlist] ‘**}**’

&nbsp;&nbsp;&nbsp;&nbsp;fieldlist ::= field {fieldsep field} [fieldsep]

&nbsp;&nbsp;&nbsp;&nbsp;field ::= ‘**[**’ exp ‘**]**’ ‘**=**’ exp | Name ‘**=**’ exp | exp

&nbsp;&nbsp;&nbsp;&nbsp;fieldsep ::= ‘**,**’ | ‘**;**’

&nbsp;&nbsp;&nbsp;&nbsp;binop ::=  ‘**+**’ | ‘**-**’ | ‘*****’ | ‘**/**’ | ‘**//**’ | ‘**^**’ | ‘**%**’ | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ‘**&**’ | ‘**~**’ | ‘**|**’ | ‘**>>**’ | ‘**<<**’ | ‘**..**’ | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ‘**<**’ | ‘**<=**’ | ‘**>**’ | ‘**>=**’ | ‘**=​=**’ | ‘**~=**’ | 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **and** | **or**

&nbsp;&nbsp;&nbsp;&nbsp;unop ::= ‘**-**’ | **not** | ‘**#**’ | ‘**~**’
