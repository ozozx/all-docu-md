This section describes the basic concepts of the language.

## 2.1 – Values and Types

Lua is a dynamically typed language. This means that variables do not have types; only values do. There are no type definitions in the language. All values carry their own type.

All values in Lua are first-class values. This means that all values can be stored in variables, passed as arguments to other functions, and returned as results.

There are eight basic types in Lua: _nil_, _boolean_, _number_, _string_, _function_, _userdata_, _thread_, and _table_. The type _nil_ has one single value, **nil**, whose main property is to be different from any other value; it often represents the absence of a useful value. The type _boolean_ has two values, **false** and **true**. Both **nil** and **false** make a condition false; they are collectively called _false values_. Any other value makes a condition true. Despite its name, **false** is frequently used as an alternative to **nil**, with the key difference that **false** behaves like a regular value in a table, while a **nil** in a table represents an absent key.

The type _number_ represents both integer numbers and real (floating-point) numbers, using two subtypes: _integer_ and _float_. Standard Lua uses 64-bit integers and double-precision (64-bit) floats, but you can also compile Lua so that it uses 32-bit integers and/or single-precision (32-bit) floats. The option with 32 bits for both integers and floats is particularly attractive for small machines and embedded systems. (See macro `LUA_32BITS` in file `luaconf.h`.)

Unless stated otherwise, any overflow when manipulating integer values _wrap around_, according to the usual rules of two's complement arithmetic. (In other words, the actual result is the unique representable integer that is equal modulo _2n_ to the mathematical result, where _n_ is the number of bits of the integer type.)

Lua has explicit rules about when each subtype is used, but it also converts between them automatically as needed (see [[3-The Language#3.4.3 – Coercions and Conversions|§3.4.3]]). Therefore, the programmer may choose to mostly ignore the difference between integers and floats or to assume complete control over the representation of each number.

The type _string_ represents immutable sequences of bytes. Lua is 8-bit clean: strings can contain any 8-bit value, including embedded zeros ('`\0`'). Lua is also encoding-agnostic; it makes no assumptions about the contents of a string. The length of any string in Lua must fit in a Lua integer, and the string plus a small header must fit in `size_t`.

Lua can call (and manipulate) functions written in Lua and functions written in C (see [[3-The Language#3.4.10 – Function Calls|§3.4.10]]). Both are represented by the type _function_.

The type _userdata_ is provided to allow arbitrary C data to be stored in Lua variables. A userdata value represents a block of raw memory. There are two kinds of userdata: _full userdata_, which is an object with a block of memory managed by Lua, and _light userdata_, which is simply a C pointer value. Userdata has no predefined operations in Lua, except assignment and identity test. By using _metatables_, the programmer can define operations for full userdata values (see [[#2.4 – Metatables and Metamethods|§2.4]]). Userdata values cannot be created or modified in Lua, only through the C API. This guarantees the integrity of data owned by the host program and C libraries.

The type _thread_ represents independent threads of execution and it is used to implement coroutines (see [[#2.6 – Coroutines|§2.6]]). Lua threads are not related to operating-system threads. Lua supports coroutines on all systems, even those that do not support threads natively.

The type _table_ implements associative arrays, that is, arrays that can have as indices not only numbers, but any Lua value except **nil** and NaN. (_Not a Number_ is a special floating-point value used by the IEEE 754 standard to represent undefined numerical results, such as `0/0`.) Tables can be _heterogeneous_; that is, they can contain values of all types (except **nil**). Any key associated to the value **nil** is not considered part of the table. Conversely, any key that is not part of a table has an associated value **nil**.

Tables are the sole data-structuring mechanism in Lua; they can be used to represent ordinary arrays, lists, symbol tables, sets, records, graphs, trees, etc. To represent records, Lua uses the field name as an index. The language supports this representation by providing `a.name` as syntactic sugar for `a["name"]`. There are several convenient ways to create tables in Lua (see [[3-The Language#3.4.9 – Table Constructors|§3.4.9]].

Like indices, the values of table fields can be of any type. In particular, because functions are first-class values, table fields can contain functions. Thus tables can also carry _methods_ (see [[3-The Language#3.4.11 – Function Definitions|§3.4.11]]).

The indexing of tables follows the definition of raw equality in the language. The expressions `a[i]` and `a[j]` denote the same table element if and only if `i` and `j` are raw equal (that is, equal without metamethods). In particular, floats with integral values are equal to their respective integers (e.g., `1.0 == 1`). To avoid ambiguities, any float used as a key that is equal to an integer is converted to that integer. For instance, if you write `a[2.0] = true`, the actual key inserted into the table will be the integer `2`.

Tables, functions, threads, and (full) userdata values are _objects_: variables do not actually _contain_ these values, only _references_ to them. Assignment, parameter passing, and function returns always manipulate references to such values; these operations do not imply any kind of copy.

The library function <code>[[6-The Standard Libraries#`type (v)`|type]]</code> returns a string describing the type of a given value (see <code>[[6-The Standard Libraries#`type (v)`|type]]</code>).

## 2.2 – Scopes, Variables, and Environments

A variable name refers to a global or a local variable according to the declaration that is in context at that point of the code. (For the purposes of this discussion, a function's formal parameter is equivalent to a local variable.)

All chunks start with an implicit declaration `global *`, which declares all free names as global variables; this preambular declaration becomes void inside the scope of any other **global** declaration, as the following example illustrates:

     X = 1       -- Ok, global by default
     do
       global Y  -- voids the implicit initial declaration
       Y = 1     -- Ok, Y declared as global
       X = 1     -- ERROR, X not declared
     end
     X = 2       -- Ok, global by default again

So, outside any global declaration, Lua works as global-by-default. Inside any global declaration, Lua works without a default: All variables must be declared.

Lua is a lexically scoped language. The scope of a variable declaration begins at the first statement after the declaration and lasts until the last non-void statement of the innermost block that includes the declaration. (_Void statements_ are labels and empty statements.)

A declaration shadows any declaration for the same name that is in context at the point of the declaration. Inside this shadow, any outer declaration for that name is void. See the next example:

     global print, x
     x = 10                -- global variable
     do                    -- new block
       local x = x         -- new 'x', with value 10
       print(x)            --> 10
       x = x+1
       do                  -- another block
         local x = x+1     -- another 'x'
         print(x)          --> 12
       end
       print(x)            --> 11
     end
     print(x)              --> 10  (the global one)

Notice that, in a declaration like `local x = x`, the new `x` being declared is not in scope yet, and so the `x` on the right-hand side refers to the outside variable.

Because of the lexical scoping rules, local variables can be freely accessed by functions defined inside their scope. A local variable used by an inner function is called an _upvalue_ (or _external local variable_, or simply _external variable_) inside the inner function.

Notice that each execution of a **local** statement defines new local variables. Consider the following example:

     a = {}
     local x = 20
     for i = 1, 10 do
       local y = 0
       a[i] = function () y = y + 1; return x + y end
     end

The loop creates ten closures (that is, ten instances of the anonymous function). Each of these closures uses a different `y` variable, while all of them share the same `x`.

As we will discuss further in [[3-The Language#3.2 – Variables|§3.2]] and [[3-The Language#3.3.3 – Assignment|§3.3.3]], any reference to a global variable `var` is syntactically translated to `_ENV.var`. Moreover, every chunk is compiled in the scope of an external local variable named `_ENV` (see [[3-The Language#3.3.2 – Chunks|§3.3.2]]), so `_ENV` itself is never a free name in a chunk.

Despite the existence of this external `_ENV` variable and the translation of free names, `_ENV` is a regular name. In particular, you can define new variables and parameters with that name. (However, you should not define `_ENV` as a global variable, otherwise `_ENV.var` would translate to `_ENV._ENV.var` and so on, in an infinite loop.) Each reference to a global variable name uses the `_ENV` that is visible at that point in the program.

Any table used as the value of `_ENV` is called an _environment_.

Lua keeps a distinguished environment called the _global environment_. This value is kept at a special index in the C registry (see [[4-The Application Program Interface#4.3 – Registry|§4.3]]). In Lua, the global variable <code>[[6-The Standard Libraries#`_G`|_G]]</code> is initialized with this same value. (<code>[[6-The Standard Libraries#`_G`|_G]]</code> is never used internally, so changing its value will affect only your own code.)

When Lua loads a chunk, the default value for its `_ENV` variable is the global environment (see <code>[[6-The Standard Libraries#`load (chunk [, chunkname [, mode [, env ])`|load]]</code>). Therefore, by default, global variables in Lua code refer to entries in the global environment and, therefore, they act as conventional global variables. Moreover, all standard libraries are loaded in the global environment and some functions there operate on that environment. You can use <code>[[6-The Standard Libraries#`load (chunk [, chunkname [, mode [, env ])`|load]]</code> (or <code>[[6-The Standard Libraries#`loadfile ([filename [, mode [, env ])`|loadfile]]</code>) to load a chunk with a different environment. (In C, you have to load the chunk and then change the value of its first upvalue; see <code>[[4-The Application Program Interface#`lua_setupvalue`|lua_setupvalue]]</code>.)

## 2.3 – Error Handling

Several operations in Lua can _raise_ an error. An error interrupts the normal flow of the program, which can continue by _catching_ the error.

Lua code can explicitly raise an error by calling the <code>[[6-The Standard Libraries#`error (message [, level])`|error]]</code> function. (This function never returns.)

To catch errors in Lua, you can do a _protected call_, using <code>[[6-The Standard Libraries#`pcall (f [, arg1, ···])`|pcall]]</code> (or <code>[[6-The Standard Libraries#`xpcall (f, msgh [, arg1, ···])`|xpcall]]</code>). The function <code>[[6-The Standard Libraries#`pcall (f [, arg1, ···])`|pcall]]</code> calls a given function in _protected mode_. Any error while running the function stops its execution, and control returns immediately to `pcall`, which returns a status code.

Because Lua is an embedded extension language, Lua code starts running by a call from C code in the host program. (When you use Lua standalone, the `lua` application is the host program.) Usually, this call is protected; so, when an otherwise unprotected error occurs during the compilation or execution of a Lua chunk, control returns to the host, which can take appropriate measures, such as printing an error message.

Whenever there is an error, an _error object_ is propagated with information about the error. Lua itself only generates errors whose error object is a string, but programs can generate errors with any value as the error object, except **nil**. (Lua will change a **nil** as error object to a string message.) It is up to the Lua program or its host to handle such error objects. For historical reasons, an error object is often called an _error message_, even though it does not have to be a string.

When you use <code>[[6-The Standard Libraries#`xpcall (f, msgh [, arg1, ···])`|xpcall]]</code> (or <code>[[4-The Application Program Interface#`lua_pcall`|lua_pcall]]</code>, in C) you can give a _message handler_ to be called in case of errors. This function is called with the original error object and returns a new error object. It is called before the error unwinds the stack, so that it can gather more information about the error, for instance by inspecting the stack and creating a stack traceback. This message handler is still protected by the protected call; so, an error inside the message handler will call the message handler again. If this loop goes on for too long, Lua breaks it and returns an appropriate message. The message handler is called only for regular runtime errors. It is not called for memory-allocation errors nor for errors while running finalizers or other message handlers.

Lua also offers a system of _warnings_ (see <code>[[6-The Standard Libraries#`warn (msg1, ···)`|warn]]</code>). Unlike errors, warnings do not interfere in any way with program execution. They typically only generate a message to the user, although this behavior can be adapted from C (see <code>[[4-The Application Program Interface#`lua_setwarnf`|lua_setwarnf]]</code>).

## 2.4 – Metatables and Metamethods

Every value in Lua can have a _metatable_. This _metatable_ is an ordinary Lua table that defines the behavior of the original value under certain events. You can change several aspects of the behavior of a value by setting specific fields in its metatable. For instance, when a non-numeric value is the operand of an addition, Lua checks for a function in the field `__add` of the value's metatable. If it finds one, Lua calls this function to perform the addition.

The key for each event in a metatable is a string with the event name prefixed by two underscores; the corresponding value is called a _metavalue_. For most events, the metavalue must be a function, which is then called a _metamethod_. In the previous example, the key is the string "`__add`" and the metamethod is the function that performs the addition. Unless stated otherwise, a metamethod can in fact be any callable value, which is either a function or a value with a `__call` metamethod.

You can query the metatable of any value using the <code>[[6-The Standard Libraries#`getmetatable (object)`|getmetatable]]</code> function. Lua queries metamethods in metatables using a raw access (see <code>[[6-The Standard Libraries#`rawget (table, index)`|rawget]]</code>.

You can replace the metatable of tables using the <code>[[6-The Standard Libraries#`setmetatable (table, metatable)`|setmetatable]]</code> function. You cannot change the metatable of other types from Lua code, except by using the debug library ([[6-The Standard Libraries#6.11 – The Debug Library|§6.11]]).

Tables and full userdata have individual metatables, although multiple tables and userdata can share their metatables. Values of all other types share one single metatable per type; that is, there is one single metatable for all numbers, one for all strings, etc. By default, a value has no metatable, but the string library sets a metatable for the string type (see [[6-The Standard Libraries#6.5 – String Manipulation|§6.5]]).

A detailed list of operations controlled by metatables is given next. Each event is identified by its corresponding key. By convention, all metatable keys used by Lua are composed by two underscores followed by lowercase Latin letters.

- **`__add`**: the addition (`+`) operation. If any operand for an addition is not a number, Lua will try to call a metamethod. It starts by checking the first operand (even if it is a number); if that operand does not define a metamethod for `__add`, then Lua will check the second operand. If Lua can find a metamethod, it calls the metamethod with the two operands as arguments, and the result of the call (adjusted to one value) is the result of the operation. Otherwise, if no metamethod is found, Lua raises an error.
- **`__sub`**: the subtraction (`-`) operation. Behavior similar to the addition operation.
- **`__mul`**: the multiplication (`*`) operation. Behavior similar to the addition operation.
- **`__div`**: the division (`/`) operation. Behavior similar to the addition operation.
- **`__mod`**: the modulo (`%`) operation. Behavior similar to the addition operation.
- **`__pow`**: the exponentiation (`^`) operation. Behavior similar to the addition operation.
- **`__unm`**: the negation (unary `-`) operation. Behavior similar to the addition operation.
- **`__idiv`**: the floor division (`//`) operation. Behavior similar to the addition operation.
- **`__band`**: the bitwise AND (`&`) operation. Behavior similar to the addition operation, except that Lua will try a metamethod if any operand is neither an integer nor a float coercible to an integer (see [[3-The Language#3.4.3 – Coercions and Conversions|§3.4.3]]).
- **`__bor`**: the bitwise OR (`|`) operation. Behavior similar to the bitwise AND operation.
- **`__bxor`**: the bitwise exclusive OR (binary `~`) operation. Behavior similar to the bitwise AND operation.
- **`__bnot`**: the bitwise NOT (unary `~`) operation. Behavior similar to the bitwise AND operation.
- **`__shl`**: the bitwise left shift (`<<`) operation. Behavior similar to the bitwise AND operation.
- **`__shr`**: the bitwise right shift (`>>`) operation. Behavior similar to the bitwise AND operation.
- **`__concat`**: the concatenation (`..`) operation. Behavior similar to the addition operation, except that Lua will try a metamethod if any operand is neither a string nor a number (which is always coercible to a string).
- **`__len`**: the length (`#`) operation. If the object is not a string, Lua will try its metamethod. If there is a metamethod, Lua calls it with the object as argument, and the result of the call (always adjusted to one value) is the result of the operation. If there is no metamethod but the object is a table, then Lua uses the table length operation (see [[3-The Language#3.4.7 – The Length Operator|§3.4.7]]). Otherwise, Lua raises an error.
- **`__eq`**: the equal (`==`) operation. Behavior similar to the addition operation, except that Lua will try a metamethod only when the values being compared are either both tables or both full userdata and they are not primitively equal. The result of the call is always converted to a boolean.
- **`__lt`**: the less than (`<`) operation. Behavior similar to the addition operation, except that Lua will try a metamethod only when the values being compared are neither both numbers nor both strings. Moreover, the result of the call is always converted to a boolean.
- **`__le`**: the less equal (`<=`) operation. Behavior similar to the less than operation.
- **`__index`**: The indexing access operation `table[key]`. This event happens when `table` is not a table or when `key` is not present in `table`. The metavalue is looked up in the metatable of `table`.
    
    The metavalue for this event can be either a function, a table, or any value with an `__index` metavalue. If it is a function, it is called with `table` and `key` as arguments, and the result of the call (adjusted to one value) is the result of the operation. Otherwise, the final result is the result of indexing this metavalue with `key`. This indexing is regular, not raw, and therefore can trigger another `__index` metavalue.
    
- **`__newindex`**: The indexing assignment `table[key] = value`. Like the index event, this event happens when `table` is not a table or when `key` is not present in `table`. The metavalue is looked up in the metatable of `table`.
    
    Like with indexing, the metavalue for this event can be either a function, a table, or any value with an `__newindex` metavalue. If it is a function, it is called with `table`, `key`, and `value` as arguments. Otherwise, Lua repeats the indexing assignment over this metavalue with the same key and value. This assignment is regular, not raw, and therefore can trigger another `__newindex` metavalue.
    
    Whenever a `__newindex` metavalue is invoked, Lua does not perform the primitive assignment. If needed, the metamethod itself can call <code>[[6-The Standard Libraries#`rawset (table, index, value)`|rawset]]</code> to do the assignment.
    
- **`__call`**: The call operation `func(args)`. This event happens when Lua tries to call a non-function value (that is, `func` is not a function). The metamethod is looked up in `func`. If present, the metamethod is called with `func` as its first argument, followed by the arguments of the original call (`args`). All results of the call are the results of the operation. This is the only metamethod that allows multiple results.

In addition to the previous list, the interpreter also respects the following keys in metatables: `__gc` (see [[2-Basic Concepts#2.5.3 – Garbage-Collection Metamethods|§2.5.3]]), `__close` (see [[3-The Language#3.3.8 – To-be-closed Variables|§3.3.8]]), `__mode` (see [[2-Basic Concepts#2.5.4 – Weak Tables|§2.5.4]]), and `__name`. (The entry `__name`, when it contains a string, may be used by <code>[[6-The Standard Libraries#`tostring (v)`|tostring]]</code> and in error messages.)

For the unary operators (negation, length, and bitwise NOT), the metamethod is computed and called with a dummy second operand, equal to the first one. This extra operand is only to simplify Lua's internals (by making these operators behave like a binary operation) and may be removed in future versions. For most uses this extra operand is irrelevant.

Because metatables are regular tables, they can contain arbitrary fields, not only the event names defined above. Some functions in the standard library (e.g., <code>[[6-The Standard Libraries#`tostring (v)`|tostring]]</code>) use other fields in metatables for their own purposes.

It is a good practice to add all needed metamethods to a table before setting it as a metatable of some object. In particular, the `__gc` metamethod works only when this order is followed (see [[2-Basic Concepts#2.5.3 – Garbage-Collection Metamethods|§2.5.3]]). It is also a good practice to set the metatable of an object right after its creation.

## 2.5 – Garbage Collection

Lua performs automatic memory management. This means that you do not have to worry about allocating memory for new objects or freeing it when the objects are no longer needed. Lua manages memory automatically by running a _garbage collector_ to collect all _dead_ objects. All memory used by Lua is subject to automatic management: strings, tables, userdata, functions, threads, internal structures, etc.

An object is considered _dead_ as soon as the collector can be sure the object will not be accessed again in the normal execution of the program. ("Normal execution" here excludes finalizers, which resurrect dead objects (see [[2-Basic Concepts#2.5.3 – Garbage-Collection Metamethods|§2.5.3]]), and it excludes also some operations using the debug library.) Note that the time when the collector can be sure that an object is dead may not coincide with the programmer's expectations. The only guarantees are that Lua will not collect an object that may still be accessed in the normal execution of the program, and it will eventually collect an object that is inaccessible from Lua. (Here, _inaccessible from Lua_ means that neither a variable nor another live object refer to the object.) Because Lua has no knowledge about C code, it never collects objects accessible through the registry (see [[4-The Application Program Interface#4.3 – Registry|§4.3]]), which includes the global environment (see [[2-Basic Concepts#2.2 – Scopes, Variables, and Environments|§2.2]]) and the main thread.

The garbage collector (GC) in Lua can work in two modes: incremental and generational.

The default GC mode with the default parameters are adequate for most uses. However, programs that waste a large proportion of their time allocating and freeing memory can benefit from other settings. Keep in mind that the GC behavior is non-portable both across platforms and across different Lua releases; therefore, optimal settings are also non-portable.

You can change the GC mode and parameters by calling <code>[[4-The Application Program Interface#`lua_gc`|lua_gc]]</code> in C or <code>[[6-The Standard Libraries#`collectgarbage ([opt [, arg )`|collectgarbage]]</code> in Lua. You can also use these functions to control the collector directly, for instance to stop or restart it.

### 2.5.1 – Incremental Garbage Collection

In incremental mode, each GC cycle performs a mark-and-sweep collection in small steps interleaved with the program's execution. In this mode, the collector uses three numbers to control its garbage-collection cycles: the _garbage-collector pause_, the _garbage-collector step multiplier_, and the _garbage-collector step size_.

The garbage-collector pause controls how long the collector waits before starting a new cycle. The collector starts a new cycle when the number of bytes hits _n%_ of the total after the previous collection. Larger values make the collector less aggressive. Values equal to or less than 100 mean the collector will not wait to start a new cycle. A value of 200 means that the collector waits for the total number of bytes to double before starting a new cycle.

The garbage-collector step size controls the size of each incremental step, specifically how many bytes the interpreter allocates before performing a step: A value of _n_ means the interpreter will allocate approximately _n_ bytes between steps.

The garbage-collector step multiplier controls how much work each incremental step does. A value of _n_ means the interpreter will execute _n%_ _units of work_ for each word allocated. A unit of work corresponds roughly to traversing one slot or sweeping one object. Larger values make the collector more aggressive. Beware that values too small can make the collector too slow to ever finish a cycle. As a special case, a zero value means unlimited work, effectively producing a non-incremental, stop-the-world collector.

### 2.5.2 – Generational Garbage Collection

In generational mode, the collector does frequent _minor_ collections, which traverses only objects recently created. If after a minor collection the number of bytes is above a limit, the collector shifts to a _major_ collection, which traverses all objects. The collector will then stay doing major collections until it detects that the program is generating enough garbage to justify going back to minor collections.

The generational mode uses three parameters: the _minor multiplier_, the _minor-major multiplier_, and the _major-minor multiplier_.

The minor multiplier controls the frequency of minor collections. For a minor multiplier _x_, a new minor collection will be done when the number of bytes grows _x%_ larger than the number in use just after the last major collection. For instance, for a multiplier of 20, the collector will do a minor collection when the number of bytes gets 20% larger than the total after the last major collection.

The minor-major multiplier controls the shift to major collections. For a multiplier _x_, the collector will shift to a major collection when the number of bytes from old objects grows _x%_ larger than the total after the previous major collection. For instance, for a multiplier of 100, the collector will do a major collection when the number of old bytes gets larger than twice the total after the previous major collection. As a special case, a value of 0 stops the collector from doing major collections.

The major-minor multiplier controls the shift back to minor collections. For a multiplier _x_, the collector will shift back to minor collections after a major collection collects at least _x%_ of the bytes allocated during the last cycle. In particular, for a multiplier of 0, the collector will immediately shift back to minor collections after doing one major collection.

### 2.5.3 – Garbage-Collection Metamethods

You can set garbage-collector metamethods for tables and, using the C API, for full userdata (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]). These metamethods, called _finalizers_, are called when the garbage collector detects that the corresponding table or userdata is dead. Finalizers allow you to coordinate Lua's garbage collection with external resource management such as closing files, network or database connections, or freeing your own memory.

For an object (table or userdata) to be finalized when collected, you must _mark_ it for finalization. You mark an object for finalization when you set its metatable and the metatable has a `__gc` metamethod. Note that if you set a metatable without a `__gc` field and later create that field in the metatable, the object will not be marked for finalization.

When a marked object becomes dead, it is not collected immediately by the garbage collector. Instead, Lua puts it in a list. After the collection, Lua goes through that list. For each object in the list, it checks the object's `__gc` metamethod: If it is present, Lua calls it with the object as its single argument.

At the end of each garbage-collection cycle, the finalizers are called in the reverse order that the objects were marked for finalization, among those collected in that cycle; that is, the first finalizer to be called is the one associated with the object marked last in the program. The execution of each finalizer may occur at any point during the execution of the regular code.

Because the object being collected must still be used by the finalizer, that object (and other objects accessible only through it) must be _resurrected_ by Lua. Usually, this resurrection is transient, and the object memory is freed in the next garbage-collection cycle. However, if the finalizer stores the object in some global place (e.g., a global variable), then the resurrection is permanent. Moreover, if the finalizer marks a finalizing object for finalization again, its finalizer will be called again in the next cycle where the object is dead. In any case, the object memory is freed only in a GC cycle where the object is dead and not marked for finalization.

When you close a state (see <code>[[4-The Application Program Interface#`lua_close`|lua_close]]</code>), Lua calls the finalizers of all objects marked for finalization, following the reverse order that they were marked. If any finalizer marks objects for collection during that phase, these marks have no effect.

Finalizers cannot yield nor run the garbage collector. Because they can run in unpredictable times, it is good practice to restrict each finalizer to the minimum necessary to properly release its associated resource.

Any error while running a finalizer generates a warning; the error is not propagated.

### 2.5.4 – Weak Tables

A _weak table_ is a table whose elements are _weak references_. A weak reference is ignored by the garbage collector. In other words, if the only references to an object are weak references, then the garbage collector will collect that object.

A weak table can have weak keys, weak values, or both. A table with weak values allows the collection of its values, but prevents the collection of its keys. A table with both weak keys and weak values allows the collection of both keys and values. In any case, if either the key or the value is collected, the whole pair is removed from the table. The weakness of a table is controlled by the `__mode` field of its metatable. This metavalue, if present, must be one of the following strings: "`k`", for a table with weak keys; "`v`", for a table with weak values; or "`kv`", for a table with both weak keys and values.

A table with weak keys and strong values is also called an _ephemeron table_. In an ephemeron table, a value is considered reachable only if its key is reachable. In particular, if the only reference to a key comes through its value, the pair is removed.

Any change in the weakness of a table may take effect only at the next collect cycle. In particular, if you change the weakness to a stronger mode, Lua may still collect some items from that table before the change takes effect.

Only objects that have an explicit construction are removed from weak tables. Values, such as numbers and light C functions, are not subject to garbage collection, and therefore are not removed from weak tables (unless their associated values are collected). Although strings are subject to garbage collection, they do not have an explicit construction and their equality is by value; they behave more like values than like objects. Therefore, they are not removed from weak tables.

Resurrected objects (that is, objects being finalized and objects accessible only through objects being finalized) have a special behavior in weak tables. They are removed from weak values before running their finalizers, but are removed from weak keys only in the next collection after running their finalizers, when such objects are actually freed. This behavior allows the finalizer to access properties associated with the object through weak tables.

If a weak table is among the resurrected objects in a collection cycle, it may not be properly cleared until the next cycle.

## 2.6 – Coroutines

Lua supports coroutines, also called _collaborative multithreading_. A coroutine in Lua represents an independent thread of execution. Unlike threads in multithread systems, however, a coroutine only suspends its execution by explicitly calling a yield function.

You create a coroutine by calling <code>[[6-The Standard Libraries#`coroutine.create (f)`|coroutine.create]]</code>. Its sole argument is a function that is the main function of the coroutine. The `create` function only creates a new coroutine and returns a handle to it (an object of type _thread_); it does not start the coroutine.

You execute a coroutine by calling <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code>. When you first call <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code>, passing as its first argument a thread returned by <code>[[6-The Standard Libraries#`coroutine.create (f)`|coroutine.create]]</code>, the coroutine starts its execution by calling its main function. Extra arguments passed to <code>[[6-The Standard Libraries#`coroutine.create (f)`|coroutine.create]]</code> are passed as arguments to that function. After the coroutine starts running, it runs until it terminates or _yields_.

A coroutine can terminate its execution in two ways: normally, when its main function returns (explicitly or implicitly, after the last instruction); and abnormally, if there is an unprotected error. In case of normal termination, <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code> returns **true**, plus any values returned by the coroutine main function. In case of errors, <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code> returns **false** plus the error object. In this case, the coroutine does not unwind its stack, so that it is possible to inspect it after the error with the debug API.

A coroutine yields by calling <code>[[6-The Standard Libraries#`coroutine.yield (···)`|coroutine.yield]]</code>. When a coroutine yields, the corresponding <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code> returns immediately, even if the yield happens inside nested function calls (that is, not in the main function, but in a function directly or indirectly called by the main function). In the case of a yield, <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code> also returns **true**, plus any values passed to <code>[[6-The Standard Libraries#`coroutine.yield (···)`|coroutine.yield]]</code>. The next time you resume the same coroutine, it continues its execution from the point where it yielded, with the call to <code>[[6-The Standard Libraries#`coroutine.yield (···)`|coroutine.yield]]</code> returning any extra arguments passed to <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code>.

Like <code>[[6-The Standard Libraries#`coroutine.create (f)`|coroutine.create]]</code>, the <code>[[6-The Standard Libraries#`coroutine.wrap (f)`|coroutine.warp]]</code> function also creates a coroutine, but instead of returning the coroutine itself, it returns a function that, when called, resumes the coroutine. Any arguments passed to this function go as extra arguments to <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code>. <code>[[6-The Standard Libraries#`coroutine.wrap (f)`|coroutine.warp]]</code> returns all the values returned by <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code>, except the first one (the boolean error code). Unlike <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code>, the function created by <code>[[6-The Standard Libraries#`coroutine.wrap (f)`|coroutine.warp]]</code> propagates any error to the caller. In this case, the function also closes the coroutine (see <code>[[6-The Standard Libraries#`coroutine.close ([co])`|coroutine.close]]</code>).

As an example of how coroutines work, consider the following code:

     function foo (a)
       print("foo", a)
       return coroutine.yield(2*a)
     end
     
     co = coroutine.create(function (a,b)
           print("co-body", a, b)
           local r = foo(a+1)
           print("co-body", r)
           local r, s = coroutine.yield(a+b, a-b)
           print("co-body", r, s)
           return b, "end"
     end)
     
     print("main", coroutine.resume(co, 1, 10))
     print("main", coroutine.resume(co, "r"))
     print("main", coroutine.resume(co, "x", "y"))
     print("main", coroutine.resume(co, "x", "y"))

When you run it, it produces the following output:

     co-body 1       10
     foo     2
     main    true    4
     co-body r
     main    true    11      -9
     co-body x       y
     main    true    10      end
     main    false   cannot resume dead coroutine

You can also create and manipulate coroutines through the C API: see functions <code>[[4-The Application Program Interface#`lua_newthread`|lua_newthread]]</code>, <code>[[6-The Standard Libraries#`coroutine.resume (co [, val1, ···])`|coroutine.resume]]</code>, and <code>[[6-The Standard Libraries#`coroutine.yield (···)`|coroutine.yield]]</code>.