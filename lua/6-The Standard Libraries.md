The standard Lua libraries provide useful functions that are implemented in C through the C API. Some of these functions provide essential services to the language (e.g., <code>[[#`type (v)`|type]]</code> and <code>[[#`getmetatable (object)`|getmetatable]]</code>); others provide access to outside services (e.g., I/O); and others could be implemented in Lua itself, but that for different reasons deserve an implementation in C (e.g., <code>[[#`table.sort (list [, comp])`|table.sort]]</code>).

All libraries are implemented through the official C API and are provided as separate C modules. Unless otherwise noted, these library functions do not adjust its number of arguments to its expected parameters. For instance, a function documented as `foo(arg)` should not be called without an argument.

The notation **fail** means a false value representing some kind of failure. (Currently, **fail** is equal to **nil**, but that may change in future versions. The recommendation is to always test the success of these functions with `(not status)`, instead of `(status == nil)`.)

Currently, Lua has the following standard libraries:

- basic library ([[#6.2 – Basic Functions|§6.2]]);
- coroutine library ([[#6.3 – Coroutine Manipulation|§6.3]]);
- package library ([[#6.4 – Modules|§6.4]]);
- string manipulation ([[#6.5 – String Manipulation|§6.5]]);
- basic UTF-8 support ([[#6.6 – UTF-8 Support|§6.6]]);
- table manipulation ([[#6.7 – Table Manipulation|§6.7]]);
- mathematical functions ([[#6.8 – Mathematical Functions|§6.8]]) (sin, log, etc.);
- input and output ([[#6.9 – Input and Output Facilities|§6.9]]);
- operating system facilities ([[#6.10 – Operating System Facilities|§6.10]]);
- debug facilities ([[#6.11 – The Debug Library|§6.11]]).

Except for the basic and the package libraries, each library provides all its functions as fields of a global table or as methods of its objects.

## 6.1 – Loading the Libraries in C code

A C host program must explicitly load the standard libraries into a state, if it wants its scripts to use them. For that, the host program can call the function <code>[[#`luaL_openlibs`|luaL_openlibs]]</code>. Alternatively, the host can select which libraries to open, by using <code>[[#`luaL_openselectedlibs`|luaL_openselectedlibs]]</code>. Both functions are declared in the header file `lualib.h`.

The stand-alone interpreter `lua` (see [[7-Lua Standalone|§7]]) already opens all standard libraries.

### `luaL_openlibs`

[-0, +0, _e_]

void luaL_openlibs (lua_State *L);

Opens all standard Lua libraries into the given state.

### `luaL_openselectedlibs`

[-0, +0, _e_]

void luaL_openselectedlibs (lua_State *L, int load, int preload);

Opens (loads) and preloads selected standard libraries into the state `L`. (To _preload_ means to add the library loader into the table <code>[[#`package.preload`|package.preload]]</code>, so that the library can be required later by the program. Keep in mind that <code>[[#`require (modname)`|require]]</code> itself is provided by the _package_ library. If a program does not load that library, it will be unable to require anything.)

The integer `load` selects which libraries to load; the integer `preload` selects which to preload, among those not loaded. Both are masks formed by a bitwise OR of the following constants:

- **`LUA_GLIBK`** : the basic library.
- **`LUA_LOADLIBK`** : the package library.
- **`LUA_COLIBK`** : the coroutine library.
- **`LUA_STRLIBK`** : the string library.
- **`LUA_UTF8LIBK`** : the UTF-8 library.
- **`LUA_TABLIBK`** : the table library.
- **`LUA_MATHLIBK`** : the mathematical library.
- **`LUA_IOLIBK`** : the I/O library.
- **`LUA_OSLIBK`** : the operating system library.
- **`LUA_DBLIBK`** : the debug library.

## 6.2 – Basic Functions

The basic library provides core functions to Lua. If you do not include this library in your application, you should check carefully whether you need to provide implementations for some of its facilities.

### `assert (v [, message])`

Raises an error if the value of its argument `v` is false (i.e., **nil** or **false**); otherwise, returns all its arguments. In case of error, `message` is the error object; when absent, it defaults to "`assertion failed!`"

### `collectgarbage ([opt [, arg]])`

This function is a generic interface to the garbage collector. It performs different functions according to its first argument, `opt`:

- **"`collect`"**: Performs a full garbage-collection cycle. This is the default option.
- **"`stop`"**: Stops automatic execution of the garbage collector. The collector will run only when explicitly invoked, until a call to restart it.
- **"`restart`"**: Restarts automatic execution of the garbage collector.
- **"`count`"**: Returns the total memory in use by Lua in Kbytes. The value has a fractional part, so that it multiplied by 1024 gives the exact number of bytes in use by Lua.
- **"`step`"**: Performs a garbage-collection step. This option may be followed by an extra argument, an integer with the step size.
    
    If the size is a positive `n`, the collector acts as if `n` new bytes have been allocated. If the size is zero, the collector performs a basic step. In incremental mode, a basic step corresponds to the current step size. In generational mode, a basic step performs a full minor collection or an incremental step, if the collector has scheduled one.
    
    In incremental mode, the function returns **true** if the step finished a collection cycle. In generational mode, the function returns **true** if the step finished a major collection.
    
- **"`isrunning`"**: Returns a boolean that tells whether the collector is running (i.e., not stopped).
- **"`incremental`"**: Changes the collector mode to incremental and returns the previous mode.
- **"`generational`"**: Changes the collector mode to generational and returns the previous mode.
- **"`param`"**: Changes and/or retrieves the values of a parameter of the collector. This option must be followed by one or two extra arguments: The name of the parameter being changed or retrieved (a string) and an optional new value for that parameter, an integer in the range _[0,100000]_. The first argument must have one of the following values:
    
    - **"`minormul`"**: The minor multiplier.
    - **"`majorminor`"**: The major-minor multiplier.
    - **"`minormajor`"**: The minor-major multiplier.
    - **"`pause`"**: The garbage-collector pause.
    - **"`stepmul`"**: The step multiplier.
    - **"`stepsize`"**: The step size.
    
    The call always returns the previous value of the parameter. If the call does not give a new value, the value is left unchanged.
    
    Lua stores these values in a compressed format, so, the value returned as the previous value may not be exactly the last value set.
    

See [[2-Basic Concepts#2.5 – Garbage Collection|§2.5]] for more details about garbage collection and some of these options.

This function should not be called by a finalizer.

### `dofile ([filename])`

Opens the named file and executes its content as a Lua chunk, returning all values returned by the chunk. When called without arguments, `dofile` executes the content of the standard input (`stdin`). In case of errors, `dofile` propagates the error to its caller. (That is, `dofile` does not run in protected mode.)

### `error (message [, level])`

Raises an error (see [[2-Basic Concepts#2.3 – Error Handling|§2.3]]) with `message` as the error object. This function never returns.

Usually, `error` adds some information about the error position at the beginning of the message, if the message is a string. The `level` argument specifies how to get the error position. With level 1 (the default), the error position is where the `error` function was called. Level 2 points the error to where the function that called `error` was called; and so on. Passing a level 0 avoids the addition of error position information to the message.

### `_G`

A global variable (not a function) that holds the global environment (see [[2-Basic Concepts#2.2 – Scopes, Variables, and Environments|§2.2]]). Lua itself does not use this variable; changing its value does not affect any environment, nor vice versa.

### `getmetatable (object)`

If `object` does not have a metatable, returns **nil**. Otherwise, if the object's metatable has a `__metatable` field, returns the associated value. Otherwise, returns the metatable of the given object.

### `ipairs (t)`

Returns three values (an iterator function, the value `t`, and 0) so that the construction

     for i,v in ipairs(t) do _body_ end

will iterate over the key–value pairs (`1,t[1]`), (`2,t[2]`), ..., up to the first absent index.

### `load (chunk [, chunkname [, mode [, env]]])`

Loads a chunk.

If `chunk` is a string, the chunk is this string. If `chunk` is a function, `load` calls it repeatedly to get the chunk pieces. Each call to `chunk` must return a string that concatenates with previous results. A return of an empty string, **nil**, or no value signals the end of the chunk.

If there are no syntactic errors, `load` returns the compiled chunk as a function; otherwise, it returns **fail** plus the error message.

When you load a main chunk, the resulting function will always have exactly one upvalue, the `_ENV` variable (see [[2-Basic Concepts#2.2 – Scopes, Variables, and Environments|§2.2]]). However, when you load a binary chunk created from a function (see <code>[[#`string.dump (function [, strip])`|string.dump]]</code>), the resulting function can have an arbitrary number of upvalues, and there is no guarantee that its first upvalue will be the `_ENV` variable. (A non-main function may not even have an `_ENV` upvalue.)

Regardless, if the resulting function has any upvalues, its first upvalue is set to the value of `env`, if that parameter is given, or to the value of the global environment. Other upvalues are initialized with **nil**. All upvalues are fresh, that is, they are not shared with any other function.

`chunkname` is used as the name of the chunk for error messages and debug information (see [[4-The Application Program Interface#4.7 – The Debug Interface|§4.7]]). When absent, it defaults to `chunk`, if `chunk` is a string, or to "`=(load)`" otherwise.

The string `mode` controls whether the chunk can be text or binary (that is, a precompiled chunk). It may be the string "`b`" (only binary chunks), "`t`" (only text chunks), or "`bt`" (both binary and text). The default is "`bt`".

Lua does not check the consistency of binary chunks. Maliciously crafted binary chunks can crash the interpreter. You can use the `mode` parameter to prevent loading binary chunks.

### `loadfile ([filename [, mode [, env]]])`

Similar to <code>[[#`load (chunk [, chunkname [, mode [, env ])`|load]]</code>, but gets the chunk from file `filename` or from the standard input, if no file name is given.

### `next (table [, index])`

Allows a program to traverse all fields of a table. Its first argument is a table and its second argument is an index in this table. A call to `next` returns the next index of the table and its associated value. When called with **nil** as its second argument, `next` returns an initial index and its associated value. When called with the last index, or with **nil** in an empty table, `next` returns **nil**. If the second argument is absent, then it is interpreted as **nil**. In particular, you can use `next(t)` to check whether a table is empty.

The order in which the indices are enumerated is not specified, _even for numeric indices_. (To traverse a table in numerical order, use a numerical **for**.)

You should not assign any value to a non-existent field in a table during its traversal. You may however modify existing fields. In particular, you may set existing fields to nil.

### `pairs (t)`

If `t` has a metamethod `__pairs`, calls it with `t` as argument and returns the first four results from the call.

Otherwise, returns the <code>[[#`next (table [, index])`|next]]</code> function, the table `t`, plus two **nil** values, so that the construction

     for k,v in pairs(t) do _body_ end

will iterate over all key–value pairs of table `t`.

See function <code>[[#`next (table [, index])`|next]]</code> for the caveats of modifying the table during its traversal.

### `pcall (f [, arg1, ···])`

Calls the function `f` with the given arguments in _protected mode_. This means that any error inside `f` is not propagated; instead, `pcall` catches the error and returns a status code. Its first result is the status code (a boolean), which is **true** if the call succeeds without errors. In such case, `pcall` also returns all results from the call, after this first result. In case of any error, `pcall` returns **false** plus the error object. Note that errors caught by `pcall` do not call a message handler.

### `print (···)`

Receives any number of arguments and prints their values to `stdout`, converting each argument to a string following the same rules of <code>[[#`tostring (v)`|tostring]]</code>.

The function `print` is not intended for formatted output, but only as a quick way to show a value, for instance for debugging. For complete control over the output, use <code>[[#`string.format (formatstring, ···)`|string.format]]</code> and <code>[[#`io.write (···)`|io.write]]</code>.

### `rawequal (v1, v2)`

Checks whether `v1` is equal to `v2`, without invoking the `__eq` metamethod. Returns a boolean.

### `rawget (table, index)`

Gets the real value of `table[index]`, without using the `__index` metavalue. `table` must be a table; `index` may be any value.

### `rawlen (v)`

Returns the length of the object `v`, which must be a table or a string, without invoking the `__len` metamethod. Returns an integer.

### `rawset (table, index, value)`

Sets the real value of `table[index]` to `value`, without using the `__newindex` metavalue. `table` must be a table, `index` any value different from **nil** and NaN, and `value` any Lua value.

This function returns `table`.

### `select (index, ···)`

If `index` is a number, returns all arguments after argument number `index`; a negative number indexes from the end (-1 is the last argument). Otherwise, `index` must be the string `"#"`, and `select` returns the total number of extra arguments it received.

### `setmetatable (table, metatable)`

Sets the metatable for the given table. If `metatable` is **nil**, removes the metatable of the given table. If the original metatable has a `__metatable` field, raises an error.

This function returns `table`.

To change the metatable of other types from Lua code, you must use the debug library ([[#6.11 – The Debug Library|§6.11]]).

### `tonumber (e [, base])`

When called with no `base`, `tonumber` tries to convert its argument to a number. If the argument is already a number or a string convertible to a number, then `tonumber` returns this number; otherwise, it returns **fail**.

The conversion of strings can result in integers or floats, according to the lexical conventions of Lua (see [[3-The Language#3.1 – Lexical Conventions|§3.1]]). The string may have leading and trailing spaces and a sign.

When called with `base`, then `e` must be a string to be interpreted as an integer numeral in that base. The base may be any integer between 2 and 36, inclusive. In bases above 10, the letter '`A`' (in either upper or lower case) represents 10, '`B`' represents 11, and so forth, with '`Z`' representing 35. If the string `e` is not a valid numeral in the given base, the function returns **fail**.

### `tostring (v)`

Receives a value of any type and converts it to a string in a human-readable format.

If the metatable of `v` has a `__tostring` field, then `tostring` calls the corresponding value with `v` as argument, and uses the result of the call as its result. Otherwise, if the metatable of `v` has a `__name` field with a string value, `tostring` may use that string in its final result.

For complete control of how numbers are converted, use <code>[[#`string.format (formatstring, ···)`|string.format]]</code>.

### `type (v)`

Returns the type of its only argument, coded as a string. The possible results of this function are "`nil`" (a string, not the value **nil**), "`number`", "`string`", "`boolean`", "`table`", "`function`", "`thread`", and "`userdata`".

### `_VERSION`

A global variable (not a function) that holds a string containing the running Lua version. The current value of this variable is "`Lua 5.5`".

### `warn (msg1, ···)`

Emits a warning with a message composed by the concatenation of all its arguments (which should be strings).

By convention, a one-piece message starting with '`@`' is intended to be a _control message_, which is a message to the warning system itself. In particular, the standard warning function in Lua recognizes the control messages "`@off`", to stop the emission of warnings, and "`@on`", to (re)start the emission; it ignores unknown control messages.

### `xpcall (f, msgh [, arg1, ···])`

This function is similar to <code>[[#`pcall (f [, arg1, ···])`|pcall]]</code>, except that it sets a new message handler `msgh`.

## 6.3 – Coroutine Manipulation

This library comprises the operations to manipulate coroutines, which come inside the table `coroutine`. See [[2-Basic Concepts#2.6 – Coroutines|§2.6]] for a general description of coroutines.

### `coroutine.close ([co])`

Closes coroutine `co`, that is, closes all its pending to-be-closed variables and puts the coroutine in a dead state. The default for `co` is the running coroutine.

The given coroutine must be dead, suspended, or be the running coroutine. For the running coroutine, this function does not return. Instead, the resume that (re)started the coroutine returns.

For other coroutines, in case of error (either the original error that stopped the coroutine or errors in closing methods), this function returns **false** plus the error object; otherwise it returns **true**.

### `coroutine.create (f)`

Creates a new coroutine, with body `f`. `f` must be a function. Returns this new coroutine, an object with type `"thread"`.

### `coroutine.isyieldable ([co])`

Returns **true** when the coroutine `co` can yield. The default for `co` is the running coroutine.

A coroutine is yieldable if it is not the main thread and it is not inside a non-yieldable C function.

### `coroutine.resume (co [, val1, ···])`

Starts or continues the execution of coroutine `co`. The first time you resume a coroutine, it starts running its body. The values `val1`, ... are passed as the arguments to the body function. If the coroutine has yielded, `resume` restarts it; the values `val1`, ... are passed as the results from the yield.

If the coroutine runs without any errors, `resume` returns **true** plus any values passed to `yield` (when the coroutine yields) or any values returned by the body function (when the coroutine terminates). If there is any error, `resume` returns **false** plus the error message.

### `coroutine.running ()`

Returns the running coroutine plus a boolean, **true** when the running coroutine is the main one.

### `coroutine.status (co)`

Returns the status of the coroutine `co`, as a string: `"running"`, if the coroutine is running (that is, it is the one that called `status`); `"suspended"`, if the coroutine is suspended in a call to `yield`, or if it has not started running yet; `"normal"` if the coroutine is active but not running (that is, it has resumed another coroutine); and `"dead"` if the coroutine has finished its body function, or if it has stopped with an error.

### `coroutine.wrap (f)`

Creates a new coroutine, with body `f`; `f` must be a function. Returns a function that resumes the coroutine each time it is called. Any arguments passed to this function behave as the extra arguments to `resume`. The function returns the same values returned by `resume`, except the first boolean. In case of error, the function closes the coroutine and propagates the error.

### `coroutine.yield (···)`

Suspends the execution of the calling coroutine. Any arguments to `yield` are passed as extra results to `resume`.

## 6.4 – Modules

The package library provides basic facilities for loading modules in Lua. It exports one function directly in the global environment: <code>[[#`require (modname)`|require]]</code>. Everything else is exported in the table `package`.

### `require (modname)`

Loads the given module. The function starts by looking into the <code>[[#`package.loaded`]]</code> table to determine whether `modname` is already loaded. If it is, then `require` returns the value stored at `package.loaded[modname]`. (The absence of a second result in this case signals that this call did not have to load the module.) Otherwise, it tries to find a _loader_ for the module.

To find a loader, `require` is guided by the table <code>[[#`package.searchers`|package.searchers]]</code>. Each item in this table is a search function, that searches for the module in a particular way. By changing this table, we can change how `require` looks for a module. The following explanation is based on the default configuration for <code>[[#`package.searchers`|package.searchers]]</code>.

First `require` queries `package.preload[modname]`. If it has a value, this value (which must be a function) is the loader. Otherwise `require` searches for a Lua loader using the path stored in <code>[[#`package.path`|package.path]]</code>. If that also fails, it searches for a C loader using the path stored in <code>[[#`package.cpath`|package.cpath]]</code>. If that also fails, it tries an _all-in-one_ loader (see <code>[[#`package.searchers`|package.searchers]]</code>).

Once a loader is found, `require` calls the loader with two arguments: `modname` and an extra value, a _loader data_, also returned by the searcher. The loader data can be any value useful to the module; for the default searchers, it indicates where the loader was found. (For instance, if the loader came from a file, this extra value is the file path.) If the loader returns any non-nil value, `require` assigns the returned value to `package.loaded[modname]`. If the loader does not return a non-nil value and has not assigned any value to `package.loaded[modname]`, then `require` assigns **true** to this entry. In any case, `require` returns the final value of `package.loaded[modname]`. Besides that value, `require` also returns as a second result the loader data returned by the searcher, which indicates how `require` found the module.

If there is any error loading or running the module, or if it cannot find any loader for the module, then `require` raises an error.

### `package.config`

A string describing some compile-time configurations for packages. This string is a sequence of lines:

- The first line is the directory separator string. Default is '`\`' for Windows and '`/`' for all other systems.
- The second line is the character that separates templates in a path. Default is '`;`'.
- The third line is the string that marks the substitution points in a template. Default is '`?`'.
- The fourth line is a string that, in a path in Windows, is replaced by the executable's directory. Default is '`!`'.
- The fifth line is a mark to ignore all text after it when building the `luaopen_` function name. Default is '`-`'.

### `package.cpath`

A string with the path used by <code>[[#`require (modname)`|require]]</code> to search for a C loader.

Lua initializes the C path <code>[[#`package.cpath`|package.cpath]]</code> in the same way it initializes the Lua path <code>[[#`package.path`|package.path]]</code>, using the environment variable `LUA_CPATH_5_5`, or the environment variable `LUA_CPATH`, or a default path defined in `luaconf.h`.

### `package.loaded`

A table used by <code>[[#`require (modname)`|require]]</code> to control which modules are already loaded. When you require a module `modname` and `package.loaded[modname]` is not false, <code>[[#`require (modname)`|require]]</code> simply returns the value stored there.

This variable is only a reference to the real table; assignments to this variable do not change the table used by <code>[[#`require (modname)`|require]]</code>. The real table is stored in the C registry (see [[4-The Application Program Interface#4.3 – Registry|§4.3]]), indexed by the key `LUA_LOADED_TABLE`, a string.

### `package.loadlib (libname, funcname)`

Dynamically links the host program with the C library `libname`.

If `funcname` is "`*`", then it only links with the library, making the symbols exported by the library available to other dynamically linked libraries. Otherwise, it looks for a function `funcname` inside the library and returns this function as a C function. So, `funcname` must follow the <code>[[4-The Application Program Interface#`lua_CFunction`|lua_CFunction]]</code> prototype (see <code>[[4-The Application Program Interface#`lua_CFunction`|lua_CFunction]]</code>).

This is a low-level function. It completely bypasses the package and module system. Unlike <code>[[#`require (modname)`|require]]</code>, it does not perform any path searching and does not automatically adds extensions. `libname` must be the complete file name of the C library, including if necessary a path and an extension. `funcname` must be the exact name exported by the C library (which may depend on the C compiler and linker used).

This functionality is not supported by ISO C. As such, `loadlib` is only available on some platforms: Linux, Windows, Mac OS X, Solaris, BSD, plus other Unix systems that support the `dlfcn` standard.

This function is inherently insecure, as it allows Lua to call any function in any readable dynamic library in the system. (Lua calls any function assuming the function has a proper prototype and respects a proper protocol (see <code>[[4-The Application Program Interface#`lua_CFunction`|lua_CFunction]]</code>). Therefore, calling an arbitrary function in an arbitrary dynamic library more often than not results in an access violation.)

### `package.path`

A string with the path used by <code>[[#`require (modname)`|require]]</code> to search for a Lua loader.

At start-up, Lua initializes this variable with the value of the environment variable `LUA_PATH_5_5` or the environment variable `LUA_PATH` or with a default path defined in `luaconf.h`, if those environment variables are not defined. A "`;;`" in the value of the environment variable is replaced by the default path.

### `package.preload`

A table to store loaders for specific modules (see <code>[[#`require (modname)`|require]]</code>).

This variable is only a reference to the real table; assignments to this variable do not change the table used by <code>[[#`require (modname)`|require]]</code>. The real table is stored in the C registry (see [[4-The Application Program Interface#4.3 – Registry|§4.3]]), indexed by the key `LUA_PRELOAD_TABLE`, a string.

### `package.searchers`

A table used by <code>[[#`require (modname)`|require]]</code> to control how to find modules.

Each entry in this table is a _searcher function_. When looking for a module, <code>[[#`require (modname)`|require]]</code> calls each of these searchers in ascending order, with the module name (the argument given to <code>[[#`require (modname)`|require]]</code>) as its sole argument. If the searcher finds the module, it returns another function, the module _loader_, plus an extra value, a _loader data_, that will be passed to that loader and returned as a second result by <code>[[#`require (modname)`|require]]</code>. If it cannot find the module, it returns a string explaining why (or **nil** if it has nothing to say).

Lua initializes this table with four searcher functions.

The first searcher simply looks for a loader in the <code>[[#`package.preload`|package.preload]]</code> table.

The second searcher looks for a loader as a Lua library, using the path stored at <code>[[#`package.path`|package.path]]</code>. The search is done as described in function <code>[[#`package.searchpath (name, path [, sep [, rep )`|package.searchpath]]</code>.

The third searcher looks for a loader as a C library, using the path given by the variable <code>[[#`package.cpath`|package.cpath]]</code>. Again, the search is done as described in function <code>[[#`package.searchpath (name, path [, sep [, rep )`|package.searchpath]]</code>. For instance, if the C path is the string

     "./?.so;./?.dll;/usr/local/?/init.so"

the searcher for module `foo` will try to open the files `./foo.so`, `./foo.dll`, and `/usr/local/foo/init.so`, in that order. Once it finds a C library, this searcher first uses a dynamic link facility to link the application with the library. Then it tries to find a C function inside the library to be used as the loader. The name of this C function is the string "`luaopen_`" concatenated with a copy of the module name where each dot is replaced by an underscore. Moreover, if the module name has a hyphen, its suffix after (and including) the first hyphen is removed. For instance, if the module name is `a.b.c-v2.1`, the function name will be `luaopen_a_b_c`.

The fourth searcher tries an _all-in-one loader_. It searches the C path for a library for the root name of the given module. For instance, when requiring `a.b.c`, it will search for a C library for `a`. If found, it looks into it for an open function for the submodule; in our example, that would be `luaopen_a_b_c`. With this facility, a package can pack several C submodules into one single library, with each submodule keeping its original open function.

All searchers except the first one (preload) return as the extra value the file path where the module was found, as returned by <code>[[#`package.searchpath (name, path [, sep [, rep )`|package.searchpath]]</code>. The first searcher always returns the string "`:preload:`".

Searchers should raise no errors and have no side effects in Lua. (They may have side effects in C, for instance by linking the application with a library.)

### `package.searchpath (name, path [, sep [, rep]])`

Searches for the given `name` in the given `path`.

A path is a string containing a sequence of _templates_ separated by semicolons. For each template, the function replaces each interrogation mark (if any) in the template with a copy of `name` wherein all occurrences of `sep` (a dot, by default) were replaced by `rep` (the system's directory separator, by default), and then tries to open the resulting file name.

For instance, if the path is the string

     "./?.lua;./?.lc;/usr/local/?/init.lua"

the search for the name `foo.a` will try to open the files `./foo/a.lua`, `./foo/a.lc`, and `/usr/local/foo/a/init.lua`, in that order.

Returns the resulting name of the first file that it can open in read mode (after closing the file), or **fail** plus an error message if none succeeds. (This error message lists all file names it tried to open.)

## 6.5 – String Manipulation

This library provides generic functions for string manipulation, such as finding and extracting substrings, and pattern matching. When indexing a string in Lua, the first character is at position 1 (not at 0, as in C). Indices are allowed to be negative and are interpreted as indexing backwards, from the end of the string. Thus, the last character is at position -1, and so on.

The string library provides all its functions inside the table `string`. It also sets a metatable for strings where the `__index` field points to the `string` table. Therefore, you can use the string functions in object-oriented style. For instance, `string.byte(s,i)` can be written as `s:byte(i)`.

The string library assumes one-byte character encodings.

### `string.byte (s [, i [, j]])`

Returns the internal numeric codes of the characters `s[i]`, `s[i+1]`, ..., `s[j]`. The default value for `i` is 1; the default value for `j` is `i`. These indices are corrected following the same rules of function <code>[[#`string.sub (s, i [, j])`|string.sub]]</code>.

Numeric codes are not necessarily portable across platforms.

### `string.char (···)`

Receives zero or more integers. Returns a string with length equal to the number of arguments, in which each character has the internal numeric code equal to its corresponding argument.

Numeric codes are not necessarily portable across platforms.

### `string.dump (function [, strip])`

Returns a string containing a binary representation (a _binary chunk_) of the given function, so that a later <code>[[#`load (chunk [, chunkname [, mode [, env ])`|load]]</code> on this string returns a copy of the function (but with new upvalues). If `strip` is a true value, the binary representation may not include all debug information about the function, to save space.

Functions with upvalues have only their number of upvalues saved. When (re)loaded, those upvalues receive fresh instances. (See the <code>[[#`load (chunk [, chunkname [, mode [, env ])`|load]]</code> function for details about how these upvalues are initialized. You can use the debug library to serialize and reload the upvalues of a function in a way adequate to your needs.)

### `string.find (s, pattern [, init [, plain]])`

Looks for the first match of `pattern` (see [[#6.5.1 – Patterns|§6.5.1]]) in the string `s`. If it finds a match, then `find` returns the indices of `s` where this occurrence starts and ends; otherwise, it returns **fail**. A third, optional numeric argument `init` specifies where to start the search; its default value is 1 and can be negative. A **true** as a fourth, optional argument `plain` turns off the pattern matching facilities, so the function does a plain "find substring" operation, with no characters in `pattern` being considered magic.

If the pattern has captures, then in a successful match the captured values are also returned, after the two indices.

### `string.format (formatstring, ···)`

Returns a formatted version of its variable number of arguments following the description given in its first argument, which must be a string. The format string follows the same rules as the ISO C function `sprintf`. The accepted conversion specifiers are `A`, `a`, `c`, `d`, `E`, `e`, `f`, `G`, `g`, `i`, `o`, `p`, `s`, `u`, `X`, `x`, and '`%`', plus a non-C specifier `q`. The accepted flags are '`-`', '`+`', '`#`', '`0`', and ' ' (space). Both width and precision, when present, are limited to two digits.

The specifier `q` formats booleans, nil, numbers, and strings in a way that the result is a valid constant in Lua source code. Booleans and nil are written in the obvious way (`true`, `false`, `nil`). Floats are written in hexadecimal, to preserve full precision. A string is written between double quotes, using escape sequences when necessary to ensure that it can safely be read back by the Lua interpreter. For instance, the call

     string.format('%q', 'a string with "quotes" and \n new line')

may produce the string:

     "a string with \"quotes\" and \
      new line"

This specifier does not support modifiers (flags, width, precision).

The conversion specifiers `A`, `a`, `E`, `e`, `f`, `G`, and `g` all expect a number as argument. The specifiers `c`, `d`, `i`, `o`, `u`, `X`, and `x` expect an integer. When Lua is compiled with a C89 compiler, the specifiers `A` and `a` (hexadecimal floats) do not support modifiers.

The specifier `s` expects a string; if its argument is not a string, it is converted to one following the same rules of <code>[[#`tostring (v)`|tostring]]</code>. If the specifier has any modifier, the corresponding string argument should not contain embedded zeros.

The specifier `p` formats the pointer returned by <code>[[4-The Application Program Interface#`lua_topointer`]]</code>. That gives a unique string identifier for tables, userdata, threads, strings, and functions. For other values (numbers, nil, booleans), this specifier results in a string representing the pointer `NULL`.

### `string.gmatch (s, pattern [, init])`

Returns an iterator function that, each time it is called, returns the next captures from `pattern` (see [[#6.5.1 – Patterns|§6.5.1]]) over the string `s`. If `pattern` specifies no captures, then the whole match is produced in each call. A third, optional numeric argument `init` specifies where to start the search; its default value is 1 and can be negative.

As an example, the following loop will iterate over all the words from string `s`, printing one per line:

     s = "hello world from Lua"
     for w in string.gmatch(s, "%a+") do
       print(w)
     end

The next example collects all pairs `key=value` from the given string into a table:

     t = {}
     s = "from=world, to=Lua"
     for k, v in string.gmatch(s, "(%w+)=(%w+)") do
       t[k] = v
     end

For this function, a caret '`^`' at the start of a pattern does not work as an anchor, as this would prevent the iteration.

### `string.gsub (s, pattern, repl [, n])`

Returns a copy of `s` in which all (or the first `n`, if given) occurrences of the `pattern` (see [[#6.5.1 – Patterns|§6.5.1]]) have been replaced by a replacement string specified by `repl`, which can be a string, a table, or a function. `gsub` also returns, as its second value, the total number of matches that occurred. The name `gsub` comes from _Global SUBstitution_.

If `repl` is a string, then its value is used for replacement. The character `%` works as an escape character: any sequence in `repl` of the form `%_d_`, with _d_ between 1 and 9, stands for the value of the _d_-th captured substring; the sequence `%0` stands for the whole match; the sequence `%%` stands for a single `%`.

If `repl` is a table, then the table is queried for every match, using the first capture as the key.

If `repl` is a function, then this function is called every time a match occurs, with all captured substrings passed as arguments, in order.

In any case, if the pattern specifies no captures, then it behaves as if the whole pattern was inside a capture.

If the value returned by the table query or by the function call is a string or a number, then it is used as the replacement string; otherwise, if it is **false** or **nil**, then there is no replacement (that is, the original match is kept in the string).

Here are some examples:

     x = string.gsub("hello world", "(%w+)", "%1 %1")
     -- x="hello hello world world"
     
     x = string.gsub("hello world", "%w+", "%0 %0", 1)
     -- x="hello hello world"
     
     x = string.gsub("hello world from Lua", "(%w+)%s*(%w+)", "%2 %1")
     -- x="world hello Lua from"
     
     x = string.gsub("home = $HOME, user = $USER", "%$(%w+)", os.getenv)
     -- x="home = /home/roberto, user = roberto"
     
     x = string.gsub("4+5 = $return 4+5$", "%$(.-)%$", function (s)
           return load(s)()
         end)
     -- x="4+5 = 9"
     
     local t = {name="lua", version="5.5"}
     x = string.gsub("$name-$version.tar.gz", "%$(%w+)", t)
     -- x="lua-5.5.tar.gz"

### `string.len (s)`

Receives a string and returns its length. The empty string `""` has length 0. Embedded zeros are counted, so `"a\000bc\000"` has length 5.

### `string.lower (s)`

Receives a string and returns a copy of this string with all uppercase letters changed to lowercase. All other characters are left unchanged. The definition of what an uppercase letter is depends on the current locale.

### `string.match (s, pattern [, init])`

Looks for the first _match_ of the `pattern` (see [[#6.5.1 – Patterns|§6.5.1]]) in the string `s`. If it finds one, then `match` returns the captures from the pattern; otherwise it returns **fail**. If `pattern` specifies no captures, then the whole match is returned. A third, optional numeric argument `init` specifies where to start the search; its default value is 1 and can be negative.

### `string.pack (fmt, v1, v2, ···)`

Returns a binary string containing the values `v1`, `v2`, etc. serialized in binary form (packed) according to the format string `fmt` (see [[#6.5.2 – Format Strings for Pack and Unpack|§6.5.2]]).

### `string.packsize (fmt)`

Returns the length of a string resulting from <code>[[#`string.pack (fmt, v1, v2, ···)`|string.pack]]</code> with the given format. The format string cannot have the variable-length options '`s`' or '`z`' (see [[#6.5.2 – Format Strings for Pack and Unpack|§6.5.2]]).

### `string.rep (s, n [, sep])`

Returns a string that is the concatenation of `n` copies of the string `s` separated by the string `sep`. The default value for `sep` is the empty string (that is, no separator). Returns the empty string if `n` is not positive.

(Note that it is very easy to exhaust the memory of your machine with a single call to this function.)

### `string.reverse (s)`

Returns a string that is the string `s` reversed.

### `string.sub (s, i [, j])`

Returns the substring of `s` that starts at `i` and continues until `j`; `i` and `j` can be negative. If `j` is absent, then it is assumed to be equal to -1 (which is the same as the string length). In particular, the call `string.sub(s,1,j)` returns a prefix of `s` with length `j`, and `string.sub(s,-i)` (for a positive `i`) returns a suffix of `s` with length `i`.

If, after the translation of negative indices, `i` is less than 1, it is corrected to 1. If `j` is greater than the string length, it is corrected to that length. If, after these corrections, `i` is greater than `j`, the function returns the empty string.

### `string.unpack (fmt, s [, pos])`

Returns the values packed in string `s` (see <code>[[#`string.pack (fmt, v1, v2, ···)`|string.pack]]</code>) according to the format string `fmt` (see [[#6.5.2 – Format Strings for Pack and Unpack|§6.5.2]]). An optional `pos` marks where to start reading in `s` (default is 1). After the read values, this function also returns the index of the first unread byte in `s`.

### `string.upper (s)`

Receives a string and returns a copy of this string with all lowercase letters changed to uppercase. All other characters are left unchanged. The definition of what a lowercase letter is depends on the current locale.

### 6.5.1 – Patterns

Patterns in Lua are described by regular strings, which are interpreted as patterns by the pattern-matching functions <code>[[#`string.find (s, pattern [, init [, plain )`|string.find]]</code>, <code>[[#`string.gmatch (s, pattern [, init])`|string.gmatch]]</code>, <code>[[#`string.gsub (s, pattern, repl [, n])`|string.gsub]]</code>, and <code>[[#`string.match (s, pattern [, init])`|string.match]]</code>. This section describes the syntax and the meaning (that is, what they match) of these strings.

#### Character Class:

A _character class_ is used to represent a set of characters. The following combinations are allowed in describing a character class:

- **_x_**: (where _x_ is not one of the _magic characters_ `^$()%.[]*+-?`) represents the character _x_ itself.
- **`.`**: (a dot) represents all characters.
- **`%a`**: represents all letters.
- **`%c`**: represents all control characters.
- **`%d`**: represents all digits.
- **`%g`**: represents all printable characters except space.
- **`%l`**: represents all lowercase letters.
- **`%p`**: represents all punctuation characters.
- **`%s`**: represents all space characters.
- **`%u`**: represents all uppercase letters.
- **`%w`**: represents all alphanumeric characters.
- **`%x`**: represents all hexadecimal digits.
- **`%_x_`**: (where _x_ is any non-alphanumeric character) represents the character _x_. This is the standard way to escape the magic characters. Any non-alphanumeric character (including all punctuation characters, even the non-magical) can be preceded by a '`%`' to represent itself in a pattern.
- **`[_set_]`**: represents the class which is the union of all characters in _set_. A range of characters can be specified by separating the end characters of the range, in ascending order, with a '`-`'. All classes `%`_x_ described above can also be used as components in _set_. All other characters in _set_ represent themselves. For example, `[%w_]` (or `[_%w]`) represents all alphanumeric characters plus the underscore, `[0-7]` represents the octal digits, and `[0-7%l%-]` represents the octal digits plus the lowercase letters plus the '`-`' character.
    
    You can put a closing square bracket in a set by positioning it as the first character in the set. You can put a hyphen in a set by positioning it as the first or the last character in the set. (You can also use an escape for both cases.)
    
    The interaction between ranges and classes is not defined. Therefore, patterns like `[%a-z]` or `[a-%%]` have no meaning.
    
- **`[^_set_]`**: represents the complement of _set_, where _set_ is interpreted as above.

For all classes represented by single letters (`%a`, `%c`, etc.), the corresponding uppercase letter represents the complement of the class. For instance, `%S` represents all non-space characters.

The definitions of letter, space, and other character groups depend on the current locale. In particular, the class `[a-z]` may not be equivalent to `%l`.

#### Pattern Item:

A _pattern item_ can be

- a single character class, which matches any single character in the class;
- a single character class followed by '`*`', which matches sequences of zero or more characters in the class. These repetition items will always match the longest possible sequence;
- a single character class followed by '`+`', which matches sequences of one or more characters in the class. These repetition items will always match the longest possible sequence;
- a single character class followed by '`-`', which also matches sequences of zero or more characters in the class. Unlike '`*`', these repetition items will always match the shortest possible sequence;
- a single character class followed by '`?`', which matches zero or one occurrence of a character in the class. It always matches one occurrence if possible;
- `%_n_`, for _n_ between 1 and 9; such item matches a substring equal to the _n_-th captured string (see below);
- `%b_xy_`, where _x_ and _y_ are two distinct characters; such item matches strings that start with _x_, end with _y_, and where the _x_ and _y_ are _balanced_. This means that, if one reads the string from left to right, counting _+1_ for an _x_ and _-1_ for a _y_, the ending _y_ is the first _y_ where the count reaches 0. For instance, the item `%b()` matches expressions with balanced parentheses.
- `%f[_set_]`, a _frontier pattern_; such item matches an empty string at any position such that the next character belongs to _set_ and the previous character does not belong to _set_. The set _set_ is interpreted as previously described. The beginning and the end of the subject are handled as if they were the character '`\0`'.

#### Pattern:

A _pattern_ is a sequence of pattern items. A caret '`^`' at the beginning of a pattern anchors the match at the beginning of the subject string. A '`$`' at the end of a pattern anchors the match at the end of the subject string. At other positions, '`^`' and '`$`' have no special meaning and represent themselves.

#### Captures:

A pattern can contain sub-patterns enclosed in parentheses; they describe _captures_. When a match succeeds, the substrings of the subject string that match captures are stored (_captured_) for future use. Captures are numbered according to their left parentheses. For instance, in the pattern `"(a*(.)%w(%s*))"`, the part of the string matching `"a*(.)%w(%s*)"` is stored as the first capture, and therefore has number 1; the character matching "`.`" is captured with number 2, and the part matching "`%s*`" has number 3.

As a special case, the capture `()` captures the current string position (a number). For instance, if we apply the pattern `"()aa()"` on the string `"flaaap"`, there will be two captures: 3 and 5.

#### Multiple matches:

The function <code>[[#`string.gsub (s, pattern, repl [, n])`|string.gsub]]</code> and the iterator <code>[[#`string.gmatch (s, pattern [, init])`|string.gmatch]]</code> match multiple occurrences of the given pattern in the subject. For these functions, a new match is considered valid only if it ends at least one byte after the end of the previous match. In other words, the pattern machine never accepts the empty string as a match immediately after another match. As an example, consider the results of the following code:

     > string.gsub("abc", "()a*()", print);
     --> 1   2
     --> 3   3
     --> 4   4

The second and third results come from Lua matching an empty string after '`b`' and another one after '`c`'. Lua does not match an empty string after '`a`', because it would end at the same position of the previous match.

### 6.5.2 – Format Strings for Pack and Unpack

The first argument to <code>[[#`string.pack (fmt, v1, v2, ···)`|string.pack]]</code>, <code>[[#`string.packsize (fmt)`|string.packsize]]</code>, and <code>[[#`string.unpack (fmt, s [, pos])`|string.unpack]]</code> is a format string, which describes the layout of the structure being created or read.

A format string is a sequence of conversion options. The conversion options are as follows:

- **`<`**: sets little endian
- **`>`**: sets big endian
- **`=`**: sets native endian
- **`![_n_]`**: sets maximum alignment to `n` (default is native alignment)
- **`b`**: a signed byte (`char`)
- **`B`**: an unsigned byte (`char`)
- **`h`**: a signed `short` (native size)
- **`H`**: an unsigned `short` (native size)
- **`l`**: a signed `long` (native size)
- **`L`**: an unsigned `long` (native size)
- **`j`**: a `lua_Integer`
- **`J`**: a `lua_Unsigned`
- **`T`**: a `size_t` (native size)
- **`i[_n_]`**: a signed `int` with `n` bytes (default is native size)
- **`I[_n_]`**: an unsigned `int` with `n` bytes (default is native size)
- **`f`**: a `float` (native size)
- **`d`**: a `double` (native size)
- **`n`**: a `lua_Number`
- **`c_n_`**: a fixed-sized string with `n` bytes
- **`z`**: a zero-terminated string
- **`s[_n_]`**: a string preceded by its length coded as an unsigned integer with `n` bytes (default is a `size_t`)
- **`x`**: one byte of padding
- **`X_op_`**: an empty item that aligns according to option `op` (which is otherwise ignored)
- **' '**: (space) ignored

(A "`[_n_]`" means an optional integral numeral.) Except for padding, spaces, and configurations (options "`xX <=>!`"), each option corresponds to an argument in <code>[[#`string.pack (fmt, v1, v2, ···)`|string.pack]]</code> or a result in <code>[[#`string.unpack (fmt, s [, pos])`|string.unpack]]</code>.

For options "`!_n_`", "`s_n_`", "`i_n_`", and "`I_n_`", `n` can be any integer between 1 and 16. All integral options check overflows; <code>[[#`string.pack (fmt, v1, v2, ···)`|string.pack]]</code> checks whether the given value fits in the given size; <code>[[#`string.unpack (fmt, s [, pos])`|string.unpack]]</code> checks whether the read value fits in a Lua integer. For the unsigned options, Lua integers are treated as unsigned values too.

Any format string starts as if prefixed by "`!1=`", that is, with maximum alignment of 1 (no alignment) and native endianness.

Native endianness assumes that the whole system is either big or little endian. The packing functions will not emulate correctly the behavior of mixed-endian formats.

Alignment works as follows: For each option, the format gets extra padding until the data starts at an offset that is a multiple of the minimum between the option size and the maximum alignment; this minimum must be a power of 2. Options "`c`" and "`z`" are not aligned; option "`s`" follows the alignment of its starting integer.

All padding is filled with zeros by <code>[[#`string.pack (fmt, v1, v2, ···)`|string.pack]]</code> and ignored by <code>[[#`string.unpack (fmt, s [, pos])`|string.unpack]]</code>.

## 6.6 – UTF-8 Support

This library provides basic support for UTF-8 encoding. It provides all its functions inside the table `utf8`. This library does not provide any support for Unicode other than the handling of the encoding. Any operation that needs the meaning of a character, such as character classification, is outside its scope.

Unless stated otherwise, all functions that expect a byte position as a parameter assume that the given position is either the start of a byte sequence or one plus the length of the subject string. As in the string library, negative indices count from the end of the string.

Functions that create byte sequences accept all values up to `0x7FFFFFFF`, as defined in the original UTF-8 specification; that implies byte sequences of up to six bytes.

Functions that interpret byte sequences only accept valid sequences (well formed and not overlong). By default, they only accept byte sequences that result in valid Unicode code points, rejecting values greater than `10FFFF` and surrogates. A boolean argument `lax`, when available, lifts these checks, so that all values up to `0x7FFFFFFF` are accepted. (Not well formed and overlong sequences are still rejected.)

### `utf8.char (···)`

Receives zero or more integers, converts each one to its corresponding UTF-8 byte sequence and returns a string with the concatenation of all these sequences.

### `utf8.charpattern`

The pattern (a string, not a function) "`[\0-\x7F\xC2-\xFD][\x80-\xBF]*`" (see [[#6.5.1 – Patterns|§6.5.1]]), which matches exactly one UTF-8 byte sequence, assuming that the subject is a valid UTF-8 string.

### `utf8.codes (s [, lax])`

Returns values so that the construction

     for p, c in utf8.codes(s) do _body_ end

will iterate over all UTF-8 characters in string `s`, with `p` being the position (in bytes) and `c` the code point of each character. It raises an error if it meets any invalid byte sequence.

### `utf8.codepoint (s [, i [, j [, lax]]])`

Returns the code points (as integers) from all characters in `s` that start between byte position `i` and `j` (both included). The default for `i` is 1 and for `j` is `i`. It raises an error if it meets any invalid byte sequence.

### `utf8.len (s [, i [, j [, lax]]])`

Returns the number of UTF-8 characters in string `s` that start between positions `i` and `j` (both inclusive). The default for `i` is 1 and for `j` is -1. If it finds any invalid byte sequence, returns **fail** plus the position of the first invalid byte.

### `utf8.offset (s, n [, i])`

Returns the position of the `n`-th character of `s` (counting from byte position `i`) as two integers: The index (in bytes) where its encoding starts and the index (in bytes) where it ends.

If the specified character is right after the end of `s`, the function behaves as if there was a '`\0`' there. If the specified character is neither in the subject nor right after its end, the function returns **fail**.

A negative `n` gets characters before position `i`. The default for `i` is 1 when `n` is non-negative and `#s + 1` otherwise, so that `utf8.offset(s,-n)` gets the offset of the `n`-th character from the end of the string.

As a special case, when `n` is 0 the function returns the start and end of the encoding of the character that contains the `i`-th byte of `s`.

This function assumes that `s` is a valid UTF-8 string.

## 6.7 – Table Manipulation

This library provides generic functions for table manipulation. It provides all its functions inside the table `table`.

Remember that, whenever an operation needs the length of a table, all caveats about the length operator apply (see [[3-The Language#3.4.7 – The Length Operator|§3.4.7]]). All functions ignore non-numeric keys in the tables given as arguments.

### `table.concat (list [, sep [, i [, j]]])`

Given a list where all elements are strings or numbers, returns the string `list[i]..sep..list[i+1] ··· sep..list[j]`. The default value for `sep` is the empty string, the default for `i` is 1, and the default for `j` is `#list`. If `i` is greater than `j`, returns the empty string.

### `table.create (nseq [, nrec])`

Creates a new empty table, preallocating memory. This preallocation may help performance and save memory when you know in advance how many elements the table will have.

Parameter `nseq` is a hint for how many elements the table will have as a sequence. Optional parameter `nrec` is a hint for how many other elements the table will have; its default is zero.

### `table.insert (list, [pos,] value)`

Inserts element `value` at position `pos` in `list`, shifting up the elements `list[pos],list[pos+1],···,list[#list]`. The default value for `pos` is `#list+1`, so that a call `table.insert(t,x)` inserts `x` at the end of the list `t`.

### `table.move (a1, f, e, t [,a2])`

Moves elements from the table `a1` to the table `a2`, performing the equivalent to the following multiple assignment: `a2[t],··· = a1[f],···,a1[e]`. The default for `a2` is `a1`. The destination range can overlap with the source range. The number of elements to be moved must fit in a Lua integer. If `f` is larger than `e`, nothing is moved.

Returns the destination table `a2`.

### `table.pack (···)`

Returns a new table with all arguments stored into keys 1, 2, etc. and with a field "`n`" with the total number of arguments. Note that the resulting table may not be a sequence, if some arguments are **nil**.

### `table.remove (list [, pos])`

Removes from `list` the element at position `pos`, returning the value of the removed element. When `pos` is an integer between 1 and `#list`, it shifts down the elements `list[pos+1],list[pos+2],···,list[#list]` and erases element `list[#list]`; The index `pos` can also be 0 when `#list` is 0, or `#list + 1`.

The default value for `pos` is `#list`, so that a call `table.remove(l)` removes the last element of the list `l`.

### `table.sort (list [, comp])`

Sorts the list elements in a given order, _in-place_, from `list[1]` to `list[#list]`. If `comp` is given, then it must be a function that receives two list elements and returns true when the first element must come before the second in the final order, so that, after the sort, `i <= j` implies `not comp(list[j],list[i])`. If `comp` is not given, then the standard Lua operator `<` is used instead.

The `comp` function must define a consistent order; more formally, the function must define a strict weak order. (A weak order is similar to a total order, but it can equate different elements for comparison purposes.)

The sort algorithm is not stable: Different elements considered equal by the given order may have their relative positions changed by the sort.

### `table.unpack (list [, i [, j]])`

Returns the elements from the given list. This function is equivalent to

     return list[i], list[i+1], ···, list[j]

By default, `i` is 1 and `j` is `#list`.

## 6.8 – Mathematical Functions

This library provides basic mathematical functions. It provides all its functions and constants inside the table `math`. Functions with the annotation "`integer/float`" give integer results for integer arguments and float results for non-integer arguments. The rounding functions <code>[[#`math.ceil (x)`|math.ciel]]</code>, <code>[[#`math.floor (x)`|math.floor]]</code> , and <code>[[#`math.modf (x)`|math.modf]]</code> return an integer when the result fits in the range of an integer, or a float otherwise.

### `math.abs (x)`

Returns the maximum value between `x` and `-x`. (integer/float)

### `math.acos (x)`

Returns the arc cosine of `x` (in radians).

### `math.asin (x)`

Returns the arc sine of `x` (in radians).

### `math.atan (y [, x])`

Returns the arc tangent of `y/x` (in radians), using the signs of both arguments to find the quadrant of the result. It also handles correctly the case of `x` being zero.

The default value for `x` is 1, so that the call `math.atan(y)` returns the arc tangent of `y`.

### `math.ceil (x)`

Returns the smallest integral value greater than or equal to `x`.

### `math.cos (x)`

Returns the cosine of `x` (assumed to be in radians).

### `math.deg (x)`

Converts the angle `x` from radians to degrees.

### `math.exp (x)`

Returns the value _ex_ (where `e` is the base of natural logarithms).

### `math.floor (x)`

Returns the largest integral value less than or equal to `x`.

### `math.fmod (x, y)`

Returns the remainder of the division of `x` by `y` that rounds the quotient towards zero. (integer/float)

### `math.frexp (x)`

Returns two numbers `m` and `e` such that _x = m2e_, where `e` is an integer. When `x` is zero, NaN, +inf, or -inf, `m` is equal to `x`; otherwise, the absolute value of `m` is in the range _[0.5, 1)_ .

### `math.huge`

The float value `HUGE_VAL`, a value greater than any other numeric value.

### `math.ldexp (m, e)`

Returns _m2e_, where `e` is an integer.

### `math.log (x [, base])`

Returns the logarithm of `x` in the given base. The default for `base` is _e_ (so that the function returns the natural logarithm of `x`).

### `math.max (x, ···)`

Returns the argument with the maximum value, according to the Lua operator `<`.

### `math.maxinteger`

An integer with the maximum value for an integer.

### `math.min (x, ···)`

Returns the argument with the minimum value, according to the Lua operator `<`.

### `math.mininteger`

An integer with the minimum value for an integer.

### `math.modf (x)`

Returns the integral part of `x` and the fractional part of `x`. Its second result is always a float.

### `math.pi`

The value of _π_.

### `math.rad (x)`

Converts the angle `x` from degrees to radians.

### `math.random ([m [, n]])`

When called without arguments, returns a pseudo-random float with uniform distribution in the range _[0, 1)_. When called with two integers `m` and `n`, `math.random` returns a pseudo-random integer with uniform distribution in the range _[m, n]_. The call `math.random(n)`, for a positive `n`, is equivalent to `math.random(1,n)`. The call `math.random(0)` produces an integer with all bits (pseudo)random.

This function uses the `xoshiro256**` algorithm to produce pseudo-random 64-bit integers, which are the results of calls with argument 0. Other results (ranges and floats) are unbiased extracted from these integers.

Lua initializes its pseudo-random generator with the equivalent of a call to <code>[[#`math.randomseed ([x [, y )`|math.randomseed]]</code> with no arguments, so that `math.random` should generate different sequences of results each time the program runs.

### `math.randomseed ([x [, y]])`

When called with at least one argument, the integer parameters `x` and `y` are joined into a _seed_ that is used to reinitialize the pseudo-random generator; equal seeds produce equal sequences of numbers. The default for `y` is zero.

When called with no arguments, Lua generates a seed with a weak attempt for randomness.

This function returns the two seed components that were effectively used, so that setting them again repeats the sequence.

To ensure a required level of randomness to the initial state (or contrarily, to have a deterministic sequence, for instance when debugging a program), you should call <code>[[#`math.randomseed ([x [, y )`|math.randomseed]]</code> with explicit arguments.

### `math.sin (x)`

Returns the sine of `x` (assumed to be in radians).

### `math.sqrt (x)`

Returns the square root of `x`. (You can also use the expression `x^0.5` to compute this value.)

### `math.tan (x)`

Returns the tangent of `x` (assumed to be in radians).

### `math.tointeger (x)`

If the value `x` is convertible to an integer, returns that integer. Otherwise, returns **fail**.

### `math.type (x)`

Returns "`integer`" if `x` is an integer, "`float`" if it is a float, or **fail** if `x` is not a number.

### `math.ult (m, n)`

Returns a boolean, **true** if and only if integer `m` is below integer `n` when they are compared as unsigned integers.

## 6.9 – Input and Output Facilities

The I/O library provides two different styles for file manipulation. The first one uses implicit file handles; that is, there are operations to set a default input file and a default output file, and all input/output operations are done over these default files. The second style uses explicit file handles.

When using implicit file handles, all operations are supplied by table `io`. When using explicit file handles, the operation <code>[[#`io.open (filename [, mode])`|io.open]]</code> returns a file handle and then all operations are supplied as methods of the file handle.

The metatable for file handles provides metamethods for `__gc` and `__close` that try to close the file when called.

The table `io` also provides three predefined file handles with their usual meanings from C: `io.stdin`, `io.stdout`, and `io.stderr`. The I/O library never closes these files.

Unless otherwise stated, all I/O functions return **fail** on failure, plus an error message as a second result and a system-dependent error code as a third result, and some non-false value on success. On non-POSIX systems, the computation of the error message and error code in case of errors may be not thread safe, because they rely on the global C variable `errno`.

### `io.close ([file])`

Equivalent to `file:close()`. Without a `file`, closes the default output file.

### `io.flush ()`

Equivalent to `io.output():flush()`.

### `io.input ([file])`

When called with a file name, it opens the named file (in text mode), and sets its handle as the default input file. When called with a file handle, it simply sets this file handle as the default input file. When called without arguments, it returns the current default input file.

In case of errors this function raises the error, instead of returning an error code.

### `io.lines ([filename, ···])`

Opens the given file name in read mode and returns an iterator function that works like `file:lines(···)` over the opened file. When the iterator function fails to read any value, it automatically closes the file. Besides the iterator function, `io.lines` returns three other values: two **nil** values as placeholders, plus the created file handle. Therefore, when used in a generic **for** loop, the file is closed also if the loop is interrupted by an error or a **break**.

The call `io.lines()` (with no file name) is equivalent to `io.input():lines("l")`; that is, it iterates over the lines of the default input file. In this case, the iterator does not close the file when the loop ends.

In case of errors opening the file, this function raises the error, instead of returning an error code.

### `io.open (filename [, mode])`

This function opens a file, in the mode specified in the string `mode`. In case of success, it returns a new file handle.

The `mode` string can be any of the following:

- **"`r`"**: read mode (the default);
- **"`w`"**: write mode;
- **"`a`"**: append mode;
- **"`r+`"**: update mode, all previous data is preserved;
- **"`w+`"**: update mode, all previous data is erased;
- **"`a+`"**: append update mode, previous data is preserved, writing is only allowed at the end of file.

The `mode` string can also have a '`b`' at the end, which is needed in some systems to open the file in binary mode.

### `io.output ([file])`

Similar to <code>[[#`io.input ([file])`|io.input]]</code>, but operates over the default output file.

### `io.popen (prog [, mode])`

This function is system dependent and is not available on all platforms.

Starts the program `prog` in a separated process and returns a file handle that you can use to read data from this program (if `mode` is `"r"`, the default) or to write data to this program (if `mode` is `"w"`).

### `io.read (···)`

Equivalent to `io.input():read(···)`.

### `io.tmpfile ()`

In case of success, returns a handle for a temporary file. This file is opened in update mode and it is automatically removed when the program ends.

### `io.type (obj)`

Checks whether `obj` is a valid file handle. Returns the string `"file"` if `obj` is an open file handle, `"closed file"` if `obj` is a closed file handle, or **fail** if `obj` is not a file handle.

### `io.write (···)`

Equivalent to `io.output():write(···)`.

### `file:close ()`

Closes `file`. Note that files are automatically closed when their handles are garbage collected, but that takes an unpredictable amount of time to happen.

When closing a file handle created with <code>[[#`io.popen (prog [, mode])`|io.popen]]</code>, <code>[[#`file close ()`|file:close]]</code> returns the same values returned by <code>[[#`os.execute ([command])`|os.execute]]</code>.

### `file:flush ()`

Saves any written data to `file`.

### `file:lines (···)`

Returns an iterator function that, each time it is called, reads the file according to the given formats. When no format is given, uses "`l`" as a default. As an example, the construction

     for c in file:lines(1) do _body_ end

will iterate over all characters of the file, starting at the current position. Unlike <code>[[#`io.lines ([filename, ···])`|io.lines]]</code>, this function does not close the file when the loop ends.

### `file:read (···)`

Reads the file `file`, according to the given formats, which specify what to read. For each format, the function returns a string or a number with the characters read, or **fail** if it cannot read data with the specified format. (In this latter case, the function does not read subsequent formats.) When called without arguments, it uses a default format that reads the next line (see below).

The available formats are

- **"`n`"**: reads a numeral and returns it as a float or an integer, following the lexical conventions of Lua. (The numeral may have leading whitespaces and a sign.) This format always reads the longest input sequence that is a valid prefix for a numeral; if that prefix does not form a valid numeral (e.g., an empty string, "`0x`", or "`3.4e-`") or it is too long (more than 200 characters), it is discarded and the format returns **fail**.
- **"`a`"**: reads the whole file, starting at the current position. On end of file, it returns the empty string; this format never fails.
- **"`l`"**: reads the next line skipping the end of line, returning **fail** on end of file. This is the default format.
- **"`L`"**: reads the next line keeping the end-of-line character (if present), returning **fail** on end of file.
- **_number_**: reads a string with up to this number of bytes, returning **fail** on end of file. If `number` is zero, it reads nothing and returns an empty string, or **fail** on end of file.

The formats "`l`" and "`L`" should be used only for text files.

### `file:seek ([whence [, offset]])`

Sets and gets the file position, measured from the beginning of the file, to the position given by `offset` plus a base specified by the string `whence`, as follows:

- **"`set`"**: base is position 0 (beginning of the file);
- **"`cur`"**: base is current position;
- **"`end`"**: base is end of file;

In case of success, `seek` returns the final file position, measured in bytes from the beginning of the file. If `seek` fails, it returns **fail**, plus a string describing the error.

The default value for `whence` is `"cur"`, and for `offset` is 0. Therefore, the call `file:seek()` returns the current file position, without changing it; the call `file:seek("set")` sets the position to the beginning of the file (and returns 0); and the call `file:seek("end")` sets the position to the end of the file, and returns its size.

### `file:setvbuf (mode [, size])`

Sets the buffering mode for a file. There are three available modes:

- **"`no`"**: no buffering.
- **"`full`"**: full buffering.
- **"`line`"**: line buffering.

For the last two cases, `size` is a hint for the size of the buffer, in bytes. The default is an appropriate size.

The specific behavior of each mode is non portable; check the underlying ISO C function `setvbuf` in your platform for more details.

### `file:write (···)`

Writes the value of each of its arguments to `file`. The arguments must be strings or numbers.

In case of success, this function returns `file`. Otherwise, it returns four values: **fail**, the error message, the error code, and the number of bytes it was able to write.

## 6.10 – Operating System Facilities

This library is implemented through table `os`.

### `os.clock ()`

Returns an approximation of the amount in seconds of CPU time used by the program, as returned by the underlying ISO C function `clock`.

### `os.date ([format [, time]])`

Returns a string or a table containing date and time, formatted according to the given string `format`.

If the `time` argument is present, this is the time to be formatted (see the <code>[[#`os.time ([table])`|os.time]]</code>) function for a description of this value). Otherwise, `date` formats the current time.

If `format` starts with '`!`', then the date is formatted in Coordinated Universal Time. After this optional character, if `format` is the string "`*t`", then `date` returns a table with the following fields: `year`, `month` (1–12), `day` (1–31), `hour` (0–23), `min` (0–59), `sec` (0–61, due to leap seconds), `wday` (weekday, 1–7, Sunday is 1), `yday` (day of the year, 1–366), and `isdst` (daylight saving flag, a boolean). This last field may be absent if the information is not available.

If `format` is not "`*t`", then `date` returns the date as a string, formatted according to the same rules as the ISO C function `strftime`.

If `format` is absent, it defaults to "`%c`", which gives a human-readable date and time representation using the current locale.

On non-POSIX systems, this function may be not thread safe because of its reliance on C function `gmtime` and C function `localtime`.

### `os.difftime (t2, t1)`

Returns the difference, in seconds, from time `t1` to time `t2` (where the times are values returned by <code>[[#`os.time ([table])`|os.time]]</code>). In POSIX, Windows, and some other systems, this value is exactly `t2`_-_`t1`.

### `os.execute ([command])`

This function is equivalent to the ISO C function `system`. It passes `command` to be executed by an operating system shell. Its first result is **true** if the command terminated successfully, or **fail** otherwise. After this first result the function returns a string plus a number, as follows:

- **"`exit`"**: the command terminated normally; the following number is the exit status of the command.
- **"`signal`"**: the command was terminated by a signal; the following number is the signal that terminated the command.

When called without a `command`, `os.execute` returns a boolean that is true if a shell is available.

### `os.exit ([code [, close]])`

Calls the ISO C function `exit` to terminate the host program. If `code` is **true**, the returned status is `EXIT_SUCCESS`; if `code` is **false**, the returned status is `EXIT_FAILURE`; if `code` is a number, the returned status is this number. The default value for `code` is **true**.

If the optional second argument `close` is true, the function closes the Lua state before exiting (see <code>[[4-The Application Program Interface#`lua_close`|lua_close]]</code>).

### `os.getenv (varname)`

Returns the value of the process environment variable `varname` or **fail** if the variable is not defined.

### `os.remove (filename)`

Deletes the file (or empty directory, on POSIX systems) with the given name. If this function fails, it returns **fail** plus a string describing the error and the error code. Otherwise, it returns true.

### `os.rename (oldname, newname)`

Renames the file or directory named `oldname` to `newname`. If this function fails, it returns **fail**, plus a string describing the error and the error code. Otherwise, it returns true.

### `os.setlocale (locale [, category])`

Sets the current locale of the program. `locale` is a system-dependent string specifying a locale; `category` is an optional string describing which category to change: `"all"`, `"collate"`, `"ctype"`, `"monetary"`, `"numeric"`, or `"time"`; the default category is `"all"`. The function returns the name of the new locale, or **fail** if the request cannot be honored.

If `locale` is the empty string, the current locale is set to an implementation-defined native locale. If `locale` is the string "`C`", the current locale is set to the standard C locale.

When called with **nil** as the first argument, this function only returns the name of the current locale for the given category.

This function may be not thread safe because of its reliance on C function `setlocale`.

### `os.time ([table])`

Returns the current local time when called without arguments, or a time representing the local date and time specified by the given table. This table must have fields `year`, `month`, and `day`, and may have fields `hour` (default is 12), `min` (default is 0), `sec` (default is 0), and `isdst` (default is **nil**). Other fields are ignored. For a description of these fields, see the <code>[[#`os.date ([format [, time )`|os.date]]</code> function.

When the function is called, the values in these fields do not need to be inside their valid ranges. For instance, if `sec` is -10, it means 10 seconds before the time specified by the other fields; if `hour` is 1000, it means 1000 hours after the time specified by the other fields.

The returned value is a number, whose meaning depends on your system. In POSIX, Windows, and some other systems, this number counts the number of seconds since some given start time (the "epoch"). In other systems, the meaning is not specified, and the number returned by `time` can be used only as an argument to <code>[[#`os.date ([format [, time )`|os.date]]</code> and <code>[[#`os.difftime (t2, t1)`|os.difftime]]</code>.

When called with a table, `os.time` also normalizes all the fields documented in the <code>[[#`os.date ([format [, time )`|os.date]]</code> function, so that they represent the same time as before the call but with values inside their valid ranges.

### `os.tmpname ()`

Returns a string with a file name that can be used for a temporary file. The file must be explicitly opened before its use and explicitly removed when no longer needed.

In POSIX systems, this function also creates a file with that name, to avoid security risks. (Someone else might create the file with wrong permissions in the time between getting the name and creating the file.) You still have to open the file to use it and to remove it (even if you do not use it).

When possible, you may prefer to use <code>[[#`io.tmpfile ()`|io.tmpfile]]</code>, which automatically removes the file when the program ends.

## 6.11 – The Debug Library

This library provides the functionality of the debug interface ([[4-The Application Program Interface#4.7 – The Debug Interface|§4.7]]) to Lua programs. You should exert care when using this library. Several of its functions violate basic assumptions about Lua code (e.g., that variables local to a function cannot be accessed from outside; that userdata metatables cannot be changed by Lua code; that Lua programs do not crash) and therefore can compromise otherwise secure code. Moreover, some functions in this library may be slow.

All functions in this library are provided inside the `debug` table. All functions that operate over a thread have an optional first argument which is the thread to operate over. The default is always the current thread.

### `debug.debug ()`

Enters an interactive mode with the user, running each string that the user enters. Using simple commands and other debug facilities, the user can inspect global and local variables, change their values, evaluate expressions, and so on. A line containing only the word `cont` finishes this function, so that the caller continues its execution.

Note that commands for `debug.debug` are not lexically nested within any function and so have no direct access to local variables.

### `debug.gethook ([thread])`

Returns the current hook settings of the thread, as three values: the current hook function, the current hook mask, and the current hook count, as set by the <code>[[#`debug.sethook ([thread,] hook, mask [, count])`|debug.sethook]]</code> function.

Returns **fail** if there is no active hook.

### `debug.getinfo ([thread,] f [, what])`

Returns a table with information about a function. You can give the function directly or you can give a number as the value of `f`, which means the function running at level `f` of the call stack of the given thread: level 0 is the current function (`getinfo` itself); level 1 is the function that called `getinfo` (except for tail calls, which do not count in the stack); and so on. If `f` is a number greater than the number of active functions, then `getinfo` returns **fail**.

The returned table can contain all the fields returned by <code>[[4-The Application Program Interface#`lua_getinfo`|lua_getinfo]]</code>, with the string `what` describing which fields to fill in. The default for `what` is to get all information available, except the table of valid lines. The option '`f`' adds a field named `func` with the function itself. The option '`L`' adds a field named `activelines` with the table of valid lines, provided the function is a Lua function. If the function has no debug information, the table is empty.

For instance, the expression `debug.getinfo(1,"n").name` returns a name for the current function, if a reasonable name can be found, and the expression `debug.getinfo(print)` returns a table with all available information about the <code>[[#`print (···)`|print]]</code> function.

### `debug.getlocal ([thread,] f, local)`

This function returns the name and the value of the local variable with index `local` of the function at level `f` of the stack. This function accesses not only explicit local variables, but also parameters and temporary values.

The first parameter or local variable has index 1, and so on, following the order that they are declared in the code, counting only the variables that are active in the current scope of the function. Compile-time constants may not appear in this listing, if they were optimized away by the compiler. Negative indices refer to vararg arguments; -1 is the first vararg argument. These negative indices are only available when the vararg table has been optimized away; otherwise, the vararg arguments are available in the vararg table.

The function returns **fail** if there is no variable with the given index, and raises an error when called with a level out of range. (You can call <code>[[#`debug.getinfo ([thread,] f [, what])`|debug.getinfo]]</code> to check whether the level is valid.)

Variable names starting with '`(`' (open parenthesis) represent variables with no known names (internal variables such as loop control variables, and variables from chunks saved without debug information).

The parameter `f` may also be a function. In that case, `getlocal` returns only the name of function parameters.

### `debug.getmetatable (value)`

Returns the metatable of the given `value` or **nil** if it does not have a metatable.

### `debug.getregistry ()`

Returns the registry table (see [[4-The Application Program Interface#4.3 – Registry|§4.3]]).

### `debug.getupvalue (f, up)`

This function returns the name and the value of the upvalue with index `up` of the function `f`. The function returns **fail** if there is no upvalue with the given index.

(For Lua functions, upvalues are the external local variables that the function uses, and that are consequently included in its closure.)

For C functions, this function uses the empty string `""` as a name for all upvalues.

Variable name '`?`' (interrogation mark) represents variables with no known names (variables from chunks saved without debug information).

### `debug.getuservalue (u, n)`

Returns the `n`-th user value associated to the userdata `u` plus a boolean, **false** if the userdata does not have that value.

### `debug.sethook ([thread,] hook, mask [, count])`

Sets the given function as the debug hook. The string `mask` and the number `count` describe when the hook will be called. The string mask may have any combination of the following characters, with the given meaning:

- **'`c`'**: the hook is called every time Lua calls a function;
- **'`r`'**: the hook is called every time Lua returns from a function;
- **'`l`'**: the hook is called every time Lua enters a new line of code.

Moreover, with a `count` different from zero, the hook is called also after every `count` instructions.

When called without arguments, <code>[[#`debug.sethook ([thread,] hook, mask [, count])`|debug.sethook]]</code> turns off the hook.

When the hook is called, its first parameter is a string describing the event that has triggered its call: `"call"`, `"tail call"`, `"return"`, `"line"`, and `"count"`. For line events, the hook also gets the new line number as its second parameter. Inside a hook, you can call `getinfo` with level 2 to get more information about the running function. (Level 0 is the `getinfo` function, and level 1 is the hook function.)

### `debug.setlocal ([thread,] level, local, value)`

This function assigns the value `value` to the local variable with index `local` of the function at level `level` of the stack. The function returns **fail** if there is no local variable with the given index, and raises an error when called with a `level` out of range. (You can call `getinfo` to check whether the level is valid.) Otherwise, it returns the name of the local variable.

See <code>[[#`debug.getlocal ([thread,] f, local)`|debug.getlocal]]</code> for more information about variable indices and names.

### `debug.setmetatable (value, table)`

Sets the metatable for the given `value` to the given `table` (which can be **nil**). Returns `value`.

### `debug.setupvalue (f, up, value)`

This function assigns the value `value` to the upvalue with index `up` of the function `f`. The function returns **fail** if there is no upvalue with the given index. Otherwise, it returns the name of the upvalue.

See <code>[[#`debug.getupvalue (f, up)`|debug.getupvalue]]</code> for more information about upvalues.

### `debug.setuservalue (udata, value, n)`

Sets the given `value` as the `n`-th user value associated to the given `udata`. `udata` must be a full userdata.

Returns `udata`, or **fail** if the userdata does not have that value.

### `debug.traceback ([thread,] [message [, level]])`

If `message` is present but is neither a string nor **nil**, this function returns `message` without further processing. Otherwise, it returns a string with a traceback of the call stack. The optional `message` string is appended at the beginning of the traceback. An optional `level` number tells at which level to start the traceback (default is 1, the function calling `traceback`).

### `debug.upvalueid (f, n)`

Returns a unique identifier (as a light userdata) for the upvalue numbered `n` from the given function.

These unique identifiers allow a program to check whether different closures share upvalues. Lua closures that share an upvalue (that is, that access a same external local variable) will return identical ids for those upvalue indices.

### `debug.upvaluejoin (f1, n1, f2, n2)`

Make the `n1`-th upvalue of the Lua closure `f1` refer to the `n2`-th upvalue of the Lua closure `f2`.
