This section describes the C API for Lua, that is, the set of C functions available to the host program to communicate with Lua. All API functions and related types and constants are declared in the header file `lua.h`.

Even when we use the term "function", any facility in the API may be provided as a macro instead. Except where stated otherwise, all such macros use each of their arguments exactly once (except for the first argument, which is always a Lua state), and so do not generate any hidden side-effects.

As in most C libraries, the Lua API functions do not check their arguments for validity or consistency. However, you can change this behavior by compiling Lua with the macro `LUA_USE_APICHECK` defined.

The Lua library is fully reentrant: it has no global variables. It keeps all information it needs in a dynamic structure, called the _Lua state_.

Each Lua state has one or more threads, which correspond to independent, cooperative lines of execution. The type <code>[[#`lua_State`|lua_State]]</code> (despite its name) refers to a thread. (Indirectly, through the thread, it also refers to the Lua state associated to the thread.)

A pointer to a thread must be passed as the first argument to every function in the library, except to <code>[[#`lua_newstate`|lua_newstate]]</code>, which creates a Lua state from scratch and returns a pointer to the _main thread_ in the new state.

## 4.1 – The Stack

Lua uses a _virtual stack_ to pass values to and from C. Each element in this stack represents a Lua value (**nil**, number, string, etc.). Functions in the API can access this stack through the Lua state parameter that they receive.

Whenever Lua calls C, the called function gets a new stack, which is independent of previous stacks and of stacks of C functions that are still active. This stack initially contains any arguments to the C function and it is where the C function can store temporary Lua values and must push its results to be returned to the caller (see <code>[[#`lua_CFunction`|lua_CFunction]]</code>).

For convenience, most query operations in the API do not follow a strict stack discipline. Instead, they can refer to any element in the stack by using an _index_: A positive index represents an absolute stack position, starting at 1 as the bottom of the stack; a negative index represents an offset relative to the top of the stack. More specifically, if the stack has _n_ elements, then index 1 represents the first element (that is, the element that was pushed onto the stack first) and index _n_ represents the last element; index -1 also represents the last element (that is, the element at the top) and index _-n_ represents the first element.

### 4.1.1 – Stack Size

When you interact with the Lua API, you are responsible for ensuring consistency. In particular, _you are responsible for controlling stack overflow_. When you call any API function, you must ensure the stack has enough room to accommodate the results.

There is one exception to the above rule: When you call a Lua function without a fixed number of results (see <code>[[#`lua_call`|lua_call]]</code>), Lua ensures that the stack has enough space for all results. However, it does not ensure any extra space. So, before pushing anything on the stack after such a call you should use <code>[[#`lua_checkstack`|lua_checkstack]]</code>.

Whenever Lua calls C, it ensures that the stack has space for at least `LUA_MINSTACK` extra elements; that is, you can safely push up to `LUA_MINSTACK` values into it. `LUA_MINSTACK` is defined as 20, so that usually you do not have to worry about stack space unless your code has loops pushing elements onto the stack. Whenever necessary, you can use the function <code>[[#`lua_checkstack`|lua_checkstack]]</code> to ensure that the stack has enough space for pushing new elements.

### 4.1.2 – Valid and Acceptable Indices

Any function in the API that receives stack indices works only with _valid indices_ or _acceptable indices_.

A _valid index_ is an index that refers to a position that stores a modifiable Lua value. It comprises stack indices between 1 and the stack top (`1 ≤ abs(index) ≤ top`) plus _pseudo-indices_, which represent some positions that are accessible to C code but that are not in the stack. Pseudo-indices are used to access the registry (see [[#4.3 – Registry|§4.3]]) and the upvalues of a C function (see [[#4.2 – C Closures|§4.2]]).

Functions that do not need a specific mutable position, but only a value (e.g., query functions), can be called with acceptable indices. An _acceptable index_ can be any valid index, but it also can be any positive index after the stack top within the space allocated for the stack, that is, indices up to the stack size. (Note that 0 is never an acceptable index.) Indices to upvalues (see [[#4.2 – C Closures|§4.2]]) greater than the real number of upvalues in the current C function are also acceptable (but invalid). Except when noted otherwise, functions in the API work with acceptable indices.

Acceptable indices serve to avoid extra tests against the stack top when querying the stack. For instance, a C function can query its third argument without the need to check whether there is a third argument, that is, without the need to check whether 3 is a valid index.

For functions that can be called with acceptable indices, any non-valid index is treated as if it contains a value of a virtual type `LUA_TNONE`, which behaves like a nil value. ^264146

### 4.1.3 – Pointers to Strings

Several functions in the API return pointers (`const char*`) to Lua strings in the stack. (See <code>[[#`lua_pushfstring`|lua_pushfstring]]</code>, <code>[[#`lua_pushlstring`|lua_pushlstring]]</code>, <code>[[#`lua_pushstring`|lua_pushstring]]</code>, and <code>[[#`lua_tolstring`|lua_tolstring]]</code>. See also <code>[[5-The Auxiliary Library#`luaL_checklstring`|luaL_checklstring]]</code>, <code>[[5-The Auxiliary Library#`luaL_checkstring`|luaL_checkstring]]</code>, and <code>[[5-The Auxiliary Library#`luaL_tolstring`|luaL_tolstring]]</code> in the auxiliary library.)

In general, Lua's garbage collection can free or move memory and then invalidate pointers to strings handled by a Lua state. To allow a safe use of these pointers, the API guarantees that any pointer to a string in a stack index is valid while the string value at that index is not removed from the stack. (It can be moved to another index, though.) When the index is a pseudo-index (referring to an upvalue), the pointer is valid while the corresponding call is active and the corresponding upvalue is not modified.

Some functions in the debug interface also return pointers to strings, namely <code>[[#`lua_getlocal`|lua_getlocal]]</code>, <code>[[#`lua_getupvalue`|lua_getupvalue]]</code>, <code>[[#`lua_setlocal`|lua_setlocal]]</code>, and <code>[[#`lua_setupvalue`|lua_setupvalue]]</code>. For these functions, the pointer is guaranteed to be valid while the caller function is active and the given closure (if one was given) is in the stack.

Except for these guarantees, the garbage collector is free to invalidate any pointer to internal strings.

## 4.2 – C Closures

When a C function is created, it is possible to associate some values with it, thus creating a _C closure_ (see <code>[[#`lua_pushcclosure`|lua_pushcclosure]]</code>); these values are called _upvalues_ and are accessible to the function whenever it is called.

Whenever a C function is called, its upvalues are located at specific pseudo-indices. These pseudo-indices are produced by the macro <code>[[#`lua_upvalueindex`|lua_upvalueindex]]</code>. The first upvalue associated with a function is at index `lua_upvalueindex(1)`, and so on. Any access to `lua_upvalueindex(_n_)`, where _n_ is greater than the number of upvalues of the current function (but not greater than 256, which is one plus the maximum number of upvalues in a closure), produces an acceptable but invalid index.

A C closure can also change the values of its corresponding upvalues.

## 4.3 – Registry

Lua provides a _registry_, a predefined table that can be used by any C code to store whatever Lua values it needs to store. The registry table is always accessible at pseudo-index `LUA_REGISTRYINDEX`. Any C library can store data into this table, but it must take care to choose keys that are different from those used by other libraries, to avoid collisions. Typically, you should use as key a string containing your library name, or a light userdata with the address of a C object in your code, or any Lua object created by your code. As with variable names, string keys starting with an underscore followed by uppercase letters are reserved for Lua.

The integer keys in the registry are used by the reference mechanism (see <code>[[5-The Auxiliary Library#`luaL_ref`|luaL_ref]]</code>), with some predefined values. Therefore, integer keys in the registry must not be used for other purposes.

When you create a new Lua state, its registry comes with some predefined values. These predefined values are indexed with integer keys defined as constants in `lua.h`. The following constants are defined:

- **`LUA_RIDX_MAINTHREAD`**: At this index the registry has the main thread of the state. (The main thread is the one created together with the state.)
- **`LUA_RIDX_GLOBALS`**: At this index the registry has the global environment.

## 4.4 – Error Handling in C

Internally, Lua uses the C `longjmp` facility to handle errors. (Lua will use exceptions if you compile it as C++; search for `LUAI_THROW` in the source code for details.) When Lua faces any error, such as a memory allocation error or a type error, it _raises_ an error; that is, it does a long jump. A _protected environment_ uses `setjmp` to set a recovery point; any error jumps to the most recent active recovery point.

Inside a C function you can raise an error explicitly by calling <code>[[#`lua_error`|lua_error]]</code>.

Most functions in the API can raise an error, for instance due to a memory allocation error. The documentation for each function indicates whether it can raise errors.

If an error happens outside any protected environment, Lua calls a _panic function_ (see <code>[[#`lua_atpanic`|lua_atpanic]]</code>) and then calls `abort`, thus exiting the host application. Your panic function can avoid this exit by never returning (e.g., doing a long jump to your own recovery point outside Lua).

The panic function, as its name implies, is a mechanism of last resort. Programs should avoid it. As a general rule, when a C function is called by Lua with a Lua state, it can do whatever it wants on that Lua state, as it should be already protected. However, when C code operates on other Lua states (e.g., a Lua-state argument to the function, a Lua state stored in the registry, or the result of <code>[[#`lua_newthread`|lua_newthread]]</code>), it should use them only in API calls that cannot raise errors.

The panic function runs as if it were a message handler (see [[2-Basic Concepts#2.3 – Error Handling|§2.3]]); in particular, the error object is on the top of the stack. However, there is no guarantee about stack space. To push anything on the stack, the panic function must first check the available space (see [[#4.1.1 – Stack Size|§4.1.1]]).

### 4.4.1 – Status Codes

Several functions that report errors in the API use the following status codes to indicate different kinds of errors or other conditions:

- **`LUA_OK` (0)**: no errors. ^b348bd
- **`LUA_ERRRUN`**: a runtime error. ^25e8fc
- **`LUA_ERRMEM`**: memory allocation error. For such errors, Lua does not call the message handler. ^0149b8
- **`LUA_ERRERR`**: stack overflow while running the message handler due to another stack overflow. More often than not, this error is the result of some other error while running a message handler. An error in a message handler will call the handler again, which will generate the error again, and so on, until this loop exhausts the stack and cause this error. ^62ab88
- **`LUA_ERRSYNTAX`**: syntax error during precompilation or format error in a binary chunk. ^6ebc3f
- **`LUA_YIELD`**: the thread (coroutine) yields. ^fdddd5
- **`LUA_ERRFILE`**: a file-related error; e.g., it cannot open or read the file. ^ae7670

These constants are defined in the header file `lua.h`.

## 4.5 – Handling Yields in C

Internally, Lua uses the C `longjmp` facility to yield a coroutine. Therefore, if a C function `foo` calls an API function and this API function yields (directly or indirectly by calling another function that yields), Lua cannot return to `foo` any more, because the `longjmp` removes its frame from the C stack.

To avoid this kind of problem, Lua raises an error whenever it tries to yield across an API call, except for three functions: <code>[[#`lua_yieldk`|lua_yieldk]]</code>, <code>[[#`lua_callk`|lua_callk]]</code>, and <code>[[#`lua_pcallk`|lua_pcallk]]</code>. All those functions receive a _continuation function_ (as a parameter named `k`) to continue execution after a yield.

We need to set some terminology to explain continuations. We have a C function called from Lua which we will call the _original function_. This original function then calls one of those three functions in the C API, which we will call the _callee function_, that then yields the current thread. This can happen when the callee function is <code>[[#`lua_yieldk`|lua_yieldk]]</code>, or when the callee function is either <code>[[#`lua_callk`|lua_callk]]</code> or <code>[[#`lua_pcallk`|lua_pcallk]]</code> and the function called by them yields.

Suppose the running thread yields while executing the callee function. After the thread resumes, it eventually will finish running the callee function. However, the callee function cannot return to the original function, because its frame in the C stack was destroyed by the yield. Instead, Lua calls a _continuation function_, which was given as an argument to the callee function. As the name implies, the continuation function should continue the task of the original function.

As an illustration, consider the following function:

     int original_function (lua_State *L) {
       ...     /* code 1 */
       status = lua_pcall(L, n, m, h);  /* calls Lua */
       ...     /* code 2 */
     }

Now we want to allow the Lua code being run by <code>[[#`lua_pcall`|lua_pcall]]</code> to yield. First, we can rewrite our function like here:

     int k (lua_State *L, int status, lua_KContext ctx) {
       ...  /* code 2 */
     }
     
     int original_function (lua_State *L) {
       ...     /* code 1 */
       return k(L, lua_pcall(L, n, m, h), ctx);
     }

In the above code, the new function `k` is a _continuation function_ (with type <code>[[#`lua_KFunction`|lua_KFunction]]</code>), which should do all the work that the original function was doing after calling <code>[[#`lua_pcall`|lua_pcall]]</code>. Now, we must inform Lua that it must call `k` if the Lua code being executed by <code>[[#`lua_pcall`|lua_pcall]]</code> gets interrupted in some way (errors or yielding), so we rewrite the code as here, replacing <code>[[#`lua_pcall`|lua_pcall]]</code> by <code>[[#`lua_pcallk`|lua_pcallk]]</code>:

     int original_function (lua_State *L) {
       ...     /* code 1 */
       return k(L, lua_pcallk(L, n, m, h, ctx2, k), ctx1);
     }

Note the external, explicit call to the continuation: Lua will call the continuation only if needed, that is, in case of errors or resuming after a yield. If the called function returns normally without ever yielding, <code>[[#`lua_pcallk`|lua_pcallk]]</code> (and <code>[[#`lua_callk`|lua_callk]]</code>) will also return normally. (Of course, instead of calling the continuation in that case, you can do the equivalent work directly inside the original function.)

Besides the Lua state, the continuation function has two other parameters: the final status of the call and the context value (`ctx`) that was passed originally to <code>[[#`lua_pcallk`|lua_pcallk]]</code>. Lua does not use this context value; it only passes this value from the original function to the continuation function. For <code>[[#`lua_pcallk`|lua_pcallk]]</code>, the status is the same value that would be returned by <code>[[#`lua_pcallk`|lua_pcallk]]</code>, except that it is <code>[[#^fdddd5|LUA_YIELD]]</code> when being executed after a yield (instead of <code>[[#^b348bd|LUA_OK]]</code>). For <code>[[#`lua_yieldk`|lua_yieldk]]</code> and <code>[[#`lua_callk`|lua_callk]]</code>, the status is always <code>[[#^fdddd5|LUA_YIELD]]</code> when Lua calls the continuation. (For these two functions, Lua will not call the continuation in case of errors, because they do not handle errors.) Similarly, when using <code>[[#`lua_callk`|lua_callk]]</code>, you should call the continuation function with <code>[[#^b348bd|LUA_OK]]</code> as the status. (For <code>[[#`lua_yieldk`|lua_yieldk]]</code>, there is not much point in calling directly the continuation function, because <code>[[#`lua_yieldk`|lua_yieldk]]</code> usually does not return.)

Lua treats the continuation function as if it were the original function. The continuation function receives the same Lua stack from the original function, in the same state it would be if the callee function had returned. (For instance, after a <code>[[#`lua_callk`|lua_callk]]</code> the function and its arguments are removed from the stack and replaced by the results from the call.) It also has the same upvalues. Whatever it returns is handled by Lua as if it were the return of the original function.

## 4.6 – Functions and Types

Here we list all functions and types from the C API in alphabetical order. Each function has an indicator like this: [-o, +p, _x_]

The first field, `o`, is how many elements the function pops from the stack. The second field, `p`, is how many elements the function pushes onto the stack. (Any function always pushes its results after popping its arguments.) A field in the form `x|y` means the function can push (or pop) `x` or `y` elements, depending on the situation; an interrogation mark '`?`' means that we cannot know how many elements the function pops/pushes by looking only at its arguments. (For instance, they may depend on what is in the stack.) The third field, `x`, tells whether the function may raise errors: '`-`' means the function never raises any error; '`m`' means the function may raise only out-of-memory errors; '`v`' means the function may raise the errors explained in the text; '`e`' means the function can run arbitrary Lua code, either directly or through metamethods, and therefore may raise any errors.

### `lua_absindex`

[-0, +0, –]

int lua_absindex (lua_State *L, int idx);

Converts the acceptable index `idx` into an equivalent absolute index (that is, one that does not depend on the stack size).

### `lua_Alloc`

typedef void * (*lua_Alloc) (void *ud,
                             void *ptr,
                             size_t osize,
                             size_t nsize);

The type of the memory-allocator function used by Lua states. The allocator function must provide a functionality similar to `realloc`, but not exactly the same. Its arguments are `ud`, an opaque pointer passed to <code>[[#`lua_newstate`|lua_newstate]]</code>; `ptr`, a pointer to the block being allocated/reallocated/freed; `osize`, the original size of the block or some code about what is being allocated; and `nsize`, the new size of the block.

When `ptr` is not `NULL`, `osize` is the size of the block pointed by `ptr`, that is, the size given when it was allocated or reallocated.

When `ptr` is `NULL`, `osize` encodes the kind of object that Lua is allocating. `osize` is any of <code>[[#^e5175e|LUA_TSTRING]]</code>, <code>[[#^e5175e|LUA_TTABLE]]</code>, <code>[[#^e5175e|LUA_TFUNCTION]]</code>, <code>[[#^e5175e|LUA_TUSERDATA]]</code>, or <code>[[#^e5175e|LUA_TTHREAD]]</code> when (and only when) Lua is creating a new object of that type. When `osize` is some other value, Lua is allocating memory for something else.

Lua assumes the following behavior from the allocator function:

When `nsize` is zero, the allocator must behave like `free` and then return `NULL`.

When `nsize` is not zero, the allocator must behave like `realloc`. In particular, the allocator returns `NULL` if and only if it cannot fulfill the request.

Here is a simple implementation for the allocator function, corresponding to the function <code>[[5-The Auxiliary Library#`luaL_alloc`|luaL_alloc]]</code> from the auxiliary library.

     void *luaL_alloc (void *ud, void *ptr, size_t osize,
                                            size_t nsize) {
       (void)ud;  (void)osize;  /* not used */
       if (nsize == 0) {
         free(ptr);
         return NULL;
       }
       else
         return realloc(ptr, nsize);
     }

Note that ISO C ensures that `free(NULL)` has no effect and that `realloc(NULL,size)` is equivalent to `malloc(size)`.

### `lua_arith`

[-(2|1), +1, _e_]

void lua_arith (lua_State *L, int op);

Performs an arithmetic or bitwise operation over the two values (or one, in the case of negations) at the top of the stack, with the value on the top being the second operand, pops these values, and pushes the result of the operation. The function follows the semantics of the corresponding Lua operator (that is, it may call metamethods).

The value of `op` must be one of the following constants:

- **`LUA_OPADD`**: performs addition (`+`)
- **`LUA_OPSUB`**: performs subtraction (`-`)
- **`LUA_OPMUL`**: performs multiplication (`*`)
- **`LUA_OPDIV`**: performs float division (`/`)
- **`LUA_OPIDIV`**: performs floor division (`//`)
- **`LUA_OPMOD`**: performs modulo (`%`)
- **`LUA_OPPOW`**: performs exponentiation (`^`)
- **`LUA_OPUNM`**: performs mathematical negation (unary `-`)
- **`LUA_OPBNOT`**: performs bitwise NOT (`~`)
- **`LUA_OPBAND`**: performs bitwise AND (`&`)
- **`LUA_OPBOR`**: performs bitwise OR (`|`)
- **`LUA_OPBXOR`**: performs bitwise exclusive OR (`~`)
- **`LUA_OPSHL`**: performs left shift (`<<`)
- **`LUA_OPSHR`**: performs right shift (`>>`)

### `lua_atpanic`

[-0, +0, –]

lua_CFunction lua_atpanic (lua_State *L, lua_CFunction panicf);

Sets a new panic function and returns the old one (see [[#4.4 – Error Handling in C|§4.4]]).

### `lua_call`

[-(nargs+1), +nresults, _e_]

void lua_call (lua_State *L, int nargs, int nresults);

Calls a function. Like regular Lua calls, `lua_call` respects the `__call` metamethod. So, here the word "function" means any callable value.

To do a call you must use the following protocol: first, the function to be called is pushed onto the stack; then, the arguments to the call are pushed in direct order; that is, the first argument is pushed first. Finally you call <code>[[#`lua_call`|lua_call]]</code>; `nargs` is the number of arguments that you pushed onto the stack. When the function returns, all arguments and the function value are popped and the call results are pushed onto the stack. The number of results is adjusted to `nresults`, unless `nresults` is `LUA_MULTRET`, which makes all results from the function to be pushed. In the first case, an explicit number of results, the caller must ensure that the stack has space for the returned values. In the second case, all results, Lua takes care that the returned values fit into the stack space, but it does not ensure any extra space in the stack. The function results are pushed onto the stack in direct order (the first result is pushed first), so that after the call the last result is on the top of the stack. ^05c2a8

The maximum value for `nresults` is 250.

Any error while calling and running the function is propagated upwards (with a `longjmp`).

The following example shows how the host program can do the equivalent to this Lua code:

     a = f("how", t.x, 14)

Here it is in C:

     lua_getglobal(L, "f");                  /* function to be called */
     lua_pushliteral(L, "how");                       /* 1st argument */
     lua_getglobal(L, "t");                    /* table to be indexed */
     lua_getfield(L, -1, "x");        /* push result of t.x (2nd arg) */
     lua_remove(L, -2);                  /* remove 't' from the stack */
     lua_pushinteger(L, 14);                          /* 3rd argument */
     lua_call(L, 3, 1);     /* call 'f' with 3 arguments and 1 result */
     lua_setglobal(L, "a");                         /* set global 'a' */

Note that the code above is _balanced_: at its end, the stack is back to its original configuration. This is considered good programming practice.

### `lua_callk`

[-(nargs + 1), +nresults, _e_]

void lua_callk (lua_State *L,
                int nargs,
                int nresults,
                lua_KContext ctx,
                lua_KFunction k);

This function behaves exactly like <code>[[#`lua_call`|lua_call]]</code>, but allows the called function to yield (see [[#4.5 – Handling Yields in C|§4.5]]).

### `lua_CFunction`

typedef int (*lua_CFunction) (lua_State *L);

Type for C functions.

In order to communicate properly with Lua, a C function must use the following protocol, which defines the way parameters and results are passed: a C function receives its arguments from Lua in its stack in direct order (the first argument is pushed first). So, when the function starts, `lua_gettop(L)` returns the number of arguments received by the function. The first argument (if any) is at index 1 and its last argument is at index `lua_gettop(L)`. To return values to Lua, a C function just pushes them onto the stack, in direct order (the first result is pushed first), and returns in C the number of results. Any other value in the stack below the results will be properly discarded by Lua. Like a Lua function, a C function called by Lua can also return many results.

As an example, the following function receives a variable number of numeric arguments and returns their average and their sum:

     static int foo (lua_State *L) {
       int n = lua_gettop(L);    /* number of arguments */
       lua_Number sum = 0.0;
       int i;
       for (i = 1; i <= n; i++) {
         if (!lua_isnumber(L, i)) {
           lua_pushliteral(L, "incorrect argument");
           lua_error(L);
         }
         sum += lua_tonumber(L, i);
       }
       lua_pushnumber(L, sum/n);        /* first result */
       lua_pushnumber(L, sum);         /* second result */
       return 2;                   /* number of results */
     }

### `lua_checkstack`

[-0, +0, –]

int lua_checkstack (lua_State *L, int n);

Ensures that the stack has space for at least `n` extra elements, that is, that you can safely push up to `n` values into it. It returns false if it cannot fulfill the request, either because it would cause the stack to be greater than a fixed maximum size (typically at least several thousand elements) or because it cannot allocate memory for the extra space. This function never shrinks the stack; if the stack already has space for the extra elements, it is left unchanged.

### `lua_close`

[-0, +0, –]

void lua_close (lua_State *L);

Close all active to-be-closed variables in the main thread, release all objects in the given Lua state (calling the corresponding garbage-collection metamethods, if any), and frees all dynamic memory used by this state.

On several platforms, you may not need to call this function, because all resources are naturally released when the host program ends. On the other hand, long-running programs that create multiple states, such as daemons or web servers, will probably need to close states as soon as they are not needed.

### `lua_closeslot`

[-0, +0, _e_]

void lua_closeslot (lua_State *L, int index);

Close the to-be-closed slot at the given index and set its value to **nil**. The index must be the last index previously marked to be closed (see <code>[[#`lua_toclose`|lua_toclose]]</code>) that is still active (that is, not closed yet).

A `__close` metamethod cannot yield when called through this function.

### `lua_closethread`

[-0, +?, –]

int lua_closethread (lua_State *L, lua_State *from);

Resets a thread, cleaning its call stack and closing all pending to-be-closed variables. The parameter `from` represents the coroutine that is resetting `L`. If there is no such coroutine, this parameter can be `NULL`.

Unless `L` is equal to `from`, the call returns a status code: <code>[[#^b348bd|LUA_OK]]</code> for no errors in the thread (either the original error that stopped the thread or errors in closing methods), or an error status otherwise. In case of error, the error object is put on the top of the stack.

If `L` is equal to `from`, it corresponds to a thread closing itself. In that case, the call does not return; instead, the resume that (re)started the thread returns. The thread must be running inside a resume.

### `lua_compare`

[-0, +0, _e_]

int lua_compare (lua_State *L, int index1, int index2, int op);

Compares two Lua values. Returns 1 if the value at index `index1` satisfies `op` when compared with the value at index `index2`, following the semantics of the corresponding Lua operator (that is, it may call metamethods). Otherwise returns 0. Also returns 0 if any of the indices is not valid.

The value of `op` must be one of the following constants:

- **`LUA_OPEQ`**: compares for equality (`==`)
- **`LUA_OPLT`**: compares for less than (`<`)
- **`LUA_OPLE`**: compares for less or equal (`<=`)

### `lua_concat`

[-n, +1, _e_]

void lua_concat (lua_State *L, int n);

Concatenates the `n` values at the top of the stack, pops them, and leaves the result on the top. If `n` is 1, the result is the single value on the stack (that is, the function does nothing); if `n` is 0, the result is the empty string. Concatenation is performed following the usual semantics of Lua (see [[3-The Language#3.4.6 – Concatenation|§3.4.6]]).

### `lua_copy`

[-0, +0, –]

void lua_copy (lua_State *L, int fromidx, int toidx);

Copies the element at index `fromidx` into the valid index `toidx`, replacing the value at that position. Values at other positions are not affected.

### `lua_createtable`

[-0, +1, _m_]

void lua_createtable (lua_State *L, int nseq, int nrec);

Creates a new empty table and pushes it onto the stack. Parameter `nseq` is a hint for how many elements the table will have as a sequence; parameter `nrec` is a hint for how many other elements the table will have. Lua may use these hints to preallocate memory for the new table. This preallocation may help performance when you know in advance how many elements the table will have. Otherwise you should use the function <code>[[#`lua_newtable`|lua_newtable]]</code>.

### `lua_dump`

[-0, +0, –]

int lua_dump (lua_State *L,
                        lua_Writer writer,
                        void *data,
                        int strip);

Dumps a function as a binary chunk. Receives a Lua function on the top of the stack and produces a binary chunk that, if loaded again, results in a function equivalent to the one dumped. As it produces parts of the chunk, <code>[[#`lua_dump`|lua_dump]]</code> calls function `writer` (see <code>[[#`lua_Writer`|lua_Writer]]</code>) with the given `data` to write them.

The function <code>[[#`lua_dump`|lua_dump]]</code> fully preserves the Lua stack through the calls to the writer function, except that it may push some values for internal use before the first call, and it restores the stack size to its original size after the last call.

If `strip` is true, the binary representation may not include all debug information about the function, to save space.

The value returned is the error code returned by the last call to the writer; 0 means no errors.

### `lua_error`

[-1, +0, _v_]

int lua_error (lua_State *L);

Raises a Lua error, using the value on the top of the stack as the error object. This function does a long jump, and therefore never returns (see <code>[[5-The Auxiliary Library#`luaL_error`|luaL_error]]</code>).

### `lua_gc`

[-0, +0, –]

int lua_gc (lua_State *L, int what, ...);

Controls the garbage collector.

This function performs several tasks, according to the value of the parameter `what`. For options that need extra arguments, they are listed after the option.

- **`LUA_GCCOLLECT`**: Performs a full garbage-collection cycle.
- **`LUA_GCSTOP`**: Stops the garbage collector.
- **`LUA_GCRESTART`**: Restarts the garbage collector.
- **`LUA_GCCOUNT`**: Returns the current amount of memory (in Kbytes) in use by Lua.
- **`LUA_GCCOUNTB`**: Returns the remainder of dividing the current amount of bytes of memory in use by Lua by 1024.
- **`LUA_GCSTEP` (size_t n)**: Performs a step of garbage collection.
- **`LUA_GCISRUNNING`**: Returns a boolean that tells whether the collector is running (i.e., not stopped).
- **`LUA_GCINC`**: Changes the collector to incremental mode. Returns the previous mode (`LUA_GCGEN` or `LUA_GCINC`). ^fac4f6
- **`LUA_GCGEN`**: Changes the collector to generational mode. Returns the previous mode (`LUA_GCGEN` or `LUA_GCINC`). ^3e50e4
- **`LUA_GCPARAM` (int param, int val)**: Changes and/or returns the value of a parameter of the collector. If `val` is -1, the call only returns the current value. The argument `param` must have one of the following values: ^a80d98
    
    - **`LUA_GCPMINORMUL`**: The minor multiplier.
    - **`LUA_GCPMAJORMINOR`**: The major-minor multiplier.
    - **`LUA_GCPMINORMAJOR`**: The minor-major multiplier.
    - **`LUA_GCPPAUSE`**: The garbage-collector pause.
    - **`LUA_GCPSTEPMUL`**: The step multiplier.
    - **`LUA_GCPSTEPSIZE`**: The step size.
    

For more details about these options, see <code>[[6-The Standard Libraries#`collectgarbage ([opt [, arg )`|collectgarbage]]</code>.

This function should not be called by a finalizer.

### `lua_getallocf`

[-0, +0, –]

lua_Alloc lua_getallocf (lua_State *L, void **ud);

Returns the memory-allocator function of a given state. If `ud` is not `NULL`, Lua stores in `*ud` the opaque pointer given when the memory-allocator function was set.

### `lua_getfield`

[-0, +1, _e_]

int lua_getfield (lua_State *L, int index, const char *k);

Pushes onto the stack the value `t[k]`, where `t` is the value at the given index. As in Lua, this function may trigger a metamethod for the "index" event (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

Returns the type of the pushed value.

### `lua_getextraspace`

[-0, +0, –]

void *lua_getextraspace (lua_State *L);

Returns a pointer to a raw memory area associated with the given Lua state. The application can use this area for any purpose; Lua does not use it for anything.

Each new thread has this area initialized with a copy of the area of the main thread.

By default, this area has the size of a pointer to void, but you can recompile Lua with a different size for this area. (See `LUA_EXTRASPACE` in `luaconf.h`.)

### `lua_getglobal`

[-0, +1, _e_]

int lua_getglobal (lua_State *L, const char *name);

Pushes onto the stack the value of the global `name`. Returns the type of that value.

### `lua_geti`

[-0, +1, _e_]

int lua_geti (lua_State *L, int index, lua_Integer i);

Pushes onto the stack the value `t[i]`, where `t` is the value at the given index. As in Lua, this function may trigger a metamethod for the "index" event (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

Returns the type of the pushed value.

### `lua_getmetatable`

[-0, +(0|1), –]

int lua_getmetatable (lua_State *L, int index);

If the value at the given index has a metatable, the function pushes that metatable onto the stack and returns 1. Otherwise, the function returns 0 and pushes nothing on the stack.

### `lua_gettable`

[-1, +1, _e_]

int lua_gettable (lua_State *L, int index);

Pushes onto the stack the value `t[k]`, where `t` is the value at the given index and `k` is the value on the top of the stack.

This function pops the key from the stack, pushing the resulting value in its place. As in Lua, this function may trigger a metamethod for the "index" event (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

Returns the type of the pushed value.

### `lua_gettop`

[-0, +0, –]

int lua_gettop (lua_State *L);

Returns the index of the top element in the stack. Because indices start at 1, this result is equal to the number of elements in the stack; in particular, 0 means an empty stack.

### `lua_getiuservalue`

[-0, +1, –]

int lua_getiuservalue (lua_State *L, int index, int n);

Pushes onto the stack the `n`-th user value associated with the full userdata at the given index and returns the type of the pushed value.

If the userdata does not have that value, pushes **nil** and returns <code>[[#^264146|LUA_TNONE]]</code>.

### `lua_insert`

[-1, +1, –]

void lua_insert (lua_State *L, int index);

Moves the top element into the given valid index, shifting up the elements above this index to open space. This function cannot be called with a pseudo-index, because a pseudo-index is not an actual stack position.

### `lua_Integer`

typedef ... lua_Integer;

The type of integers in Lua.

By default this type is `long long`, (usually a 64-bit two's complement integer), but that can be changed to `long` or `int` (usually a 32-bit two's complement integer). (See `LUA_INT_TYPE` in `luaconf.h`.)

Lua also defines the constants `LUA_MININTEGER` and `LUA_MAXINTEGER`, with the minimum and the maximum values that fit in this type.

### `lua_isboolean`

[-0, +0, –]

int lua_isboolean (lua_State *L, int index);

Returns 1 if the value at the given index is a boolean, and 0 otherwise.

### `lua_iscfunction`

[-0, +0, –]

int lua_iscfunction (lua_State *L, int index);

Returns 1 if the value at the given index is a C function, and 0 otherwise.

### `lua_isfunction`

[-0, +0, –]

int lua_isfunction (lua_State *L, int index);

Returns 1 if the value at the given index is a function (either C or Lua), and 0 otherwise.

### `lua_isinteger`

[-0, +0, –]

int lua_isinteger (lua_State *L, int index);

Returns 1 if the value at the given index is an integer (that is, the value is a number and is represented as an integer), and 0 otherwise.

### `lua_islightuserdata`

[-0, +0, –]

int lua_islightuserdata (lua_State *L, int index);

Returns 1 if the value at the given index is a light userdata, and 0 otherwise.

### `lua_isnil`

[-0, +0, –]

int lua_isnil (lua_State *L, int index);

Returns 1 if the value at the given index is **nil**, and 0 otherwise.

### `lua_isnone`

[-0, +0, –]

int lua_isnone (lua_State *L, int index);

Returns 1 if the given index is not valid, and 0 otherwise.

### `lua_isnoneornil`

[-0, +0, –]

int lua_isnoneornil (lua_State *L, int index);

Returns 1 if the given index is not valid or if the value at this index is **nil**, and 0 otherwise.

### `lua_isnumber`

[-0, +0, –]

int lua_isnumber (lua_State *L, int index);

Returns 1 if the value at the given index is a number or a string convertible to a number, and 0 otherwise.

### `lua_isstring`

[-0, +0, –]

int lua_isstring (lua_State *L, int index);

Returns 1 if the value at the given index is a string or a number (which is always convertible to a string), and 0 otherwise.

### `lua_istable`

[-0, +0, –]

int lua_istable (lua_State *L, int index);

Returns 1 if the value at the given index is a table, and 0 otherwise.

### `lua_isthread`

[-0, +0, –]

int lua_isthread (lua_State *L, int index);

Returns 1 if the value at the given index is a thread, and 0 otherwise.

### `lua_isuserdata`

[-0, +0, –]

int lua_isuserdata (lua_State *L, int index);

Returns 1 if the value at the given index is a userdata (either full or light), and 0 otherwise.

### `lua_isyieldable`

[-0, +0, –]

int lua_isyieldable (lua_State *L);

Returns 1 if the given coroutine can yield, and 0 otherwise.

### `lua_KContext`

typedef ... lua_KContext;

The type for continuation-function contexts. It must be a numeric type. This type is defined as `intptr_t` when `intptr_t` is available, so that it can store pointers too. Otherwise, it is defined as `ptrdiff_t`.

### `lua_KFunction`

typedef int (*lua_KFunction) (lua_State *L, int status, lua_KContext ctx);

Type for continuation functions (see [[#4.5 – Handling Yields in C|§4.5]]).

### `lua_len`

[-0, +1, _e_]

void lua_len (lua_State *L, int index);

Returns the length of the value at the given index. It is equivalent to the '`#`' operator in Lua (see [[3-The Language#3.4.7 – The Length Operator|§3.4.7]]) and may trigger a metamethod for the "length" event (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]). The result is pushed on the stack.

### `lua_load`

[-0, +1, –]

int lua_load (lua_State *L,
              lua_Reader reader,
              void *data,
              const char *chunkname,
              const char *mode);

Loads a Lua chunk without running it. If there are no errors, `lua_load` pushes the compiled chunk as a Lua function on top of the stack. Otherwise, it pushes an error message.

The `lua_load` function uses a user-supplied `reader` function to read the chunk (see <code>[[#`lua_Reader`|lua_Reader]]</code>). The `data` argument is an opaque value passed to the reader function.

The `chunkname` argument gives a name to the chunk, which is used for error messages and in debug information (see [[#4.7 – The Debug Interface|§4.7]]).

`lua_load` automatically detects whether the chunk is text or binary and loads it accordingly (see program `luac`). The string `mode` works as in function <code>[[6-The Standard Libraries#`load (chunk [, chunkname [, mode [, env ])`|load]]</code>, with the addition that a `NULL` value is equivalent to the string "`bt`". Moreover, it may have a '`B`' instead of a '`b`', meaning a _fixed buffer_ with the binary dump.

A fixed buffer means that the address returned by the reader function will contain the chunk until everything created by the chunk has been collected; therefore, Lua can avoid copying to internal structures some parts of the chunk. (In general, a fixed buffer would keep its contents until the end of the program, for instance with the chunk in ROM.) Moreover, for a fixed buffer, the reader function should return the entire chunk in the first read. (As an example, <code>[[5-The Auxiliary Library#`luaL_loadbufferx`|luaL_loadbufferx]]</code> does that, which means that you can use it to load fixed buffers.)

The function <code>[[#`lua_load`|lua_load]]</code> fully preserves the Lua stack through the calls to the reader function, except that it may push some values for internal use before the first call, and it restores the stack size to its original size plus one (for the pushed result) after the last call.

`lua_load` can return <code>[[#^b348bd|LUA_OK]]</code>, <code>[[#^6ebc3f|LUA_ERRSYNTAX]]</code>, or <code>[[#^0149b8|LUA_ERRMEM]]</code>. The function may also return other values corresponding to errors raised by the read function (see [[#4.4.1 – Status Codes|§4.4.1]]).

If the resulting function has upvalues, its first upvalue is set to the value of the global environment stored at index `LUA_RIDX_GLOBALS` in the registry (see [[#4.3 – Registry|§4.3]]). When loading main chunks, this upvalue will be the `_ENV` variable (see [[2-Basic Concepts#2.2 – Scopes, Variables, and Environments|§2.2]]). Other upvalues are initialized with **nil**.

### `lua_newstate`

[-0, +0, –]

lua_State *lua_newstate (lua_Alloc f, void *ud,
                                   unsigned int seed);

Creates a new independent state and returns its main thread. Returns `NULL` if it cannot create the state (due to lack of memory). The argument `f` is the allocator function; Lua will do all memory allocation for this state through this function (see <code>[[#lua_Alloc|lua_Alloc]]</code>). The second argument, `ud`, is an opaque pointer that Lua passes to the allocator in every call. The third argument, `seed`, is a seed for the hashing of strings.

### `lua_newtable`

[-0, +1, _m_]

void lua_newtable (lua_State *L);

Creates a new empty table and pushes it onto the stack. It is equivalent to `lua_createtable(L,0,0)`.

### `lua_newthread`

[-0, +1, _m_]

lua_State *lua_newthread (lua_State *L);

Creates a new thread, pushes it on the stack, and returns a pointer to a <code>[[#`lua_State`|lua_State]]</code> that represents this new thread. The new thread returned by this function shares with the original thread its global environment, but has an independent execution stack.

Threads are subject to garbage collection, like any Lua object.

### `lua_newuserdatauv`

[-0, +1, _m_]

void *lua_newuserdatauv (lua_State *L, size_t size, int nuvalue);

This function creates and pushes on the stack a new full userdata, with `nuvalue` associated Lua values, called `user values`, plus an associated block of raw memory with `size` bytes. (The user values can be set and read with the functions <code>[[#`lua_setiuservalue`|lua_setiuservalue]]</code> and <code>[[#`lua_getiuservalue`|lua_getiuservalue]]</code>.)

The function returns the address of the block of memory. Lua ensures that this address is valid as long as the corresponding userdata is alive (see [[2-Basic Concepts#2.5 – Garbage Collection|§2.5]]). Moreover, if the userdata is marked for finalization (see [[2-Basic Concepts#2.5.3 – Garbage-Collection Metamethods|§2.5.3]]), its address is valid at least until the call to its finalizer.

### `lua_next`

[-1, +(2|0), _v_]

int lua_next (lua_State *L, int index);

Pops a key from the stack, and pushes a key–value pair from the table at the given index, the "next" pair after the given key. If there are no more elements in the table, then <code>[[#`lua_next`|lua_next]]</code> returns 0 and pushes nothing.

A typical table traversal looks like this:

     /* table is in the stack at index 't' */
     lua_pushnil(L);  /* first key */
     while (lua_next(L, t) != 0) {
       /* uses 'key' (at index -2) and 'value' (at index -1) */
       printf("%s - %s\n",
              lua_typename(L, lua_type(L, -2)),
              lua_typename(L, lua_type(L, -1)));
       /* removes 'value'; keeps 'key' for next iteration */
       lua_pop(L, 1);
     }

While traversing a table, avoid calling <code>[[#`lua_tolstring`|lua_tolstring]]</code> directly on a key, unless you know that the key is actually a string. Recall that <code>[[#`lua_tolstring`|lua_tolstring]]</code> may change the value at the given index; this confuses the next call to <code>[[#`lua_next`|lua_next]]</code>.

This function may raise an error if the given key is neither **nil** nor present in the table. See function <code>[[6-The Standard Libraries#`next (table [, index])`|next]]</code> for the caveats of modifying the table during its traversal.

### `lua_Number`

typedef ... lua_Number;

The type of floats in Lua.

By default this type is double, but that can be changed to a single float or a long double. (See `LUA_FLOAT_TYPE` in `luaconf.h`.)

### `lua_numbertointeger`

int lua_numbertointeger (lua_Number n, lua_Integer *p);

Tries to convert a Lua float to a Lua integer; the float `n` must have an integral value. If that value is within the range of Lua integers, it is converted to an integer and assigned to `*p`. The macro results in a boolean indicating whether the conversion was successful. (Note that this range test can be tricky to do correctly without this macro, due to rounding.)

This macro may evaluate its arguments more than once.

### `lua_numbertocstring`

[-0, +0, –]

unsigned lua_numbertocstring (lua_State *L, int idx,
                                        char *buff);

Converts the number at acceptable index `idx` to a string and puts the result in `buff`. The buffer must have a size of at least `LUA_N2SBUFFSZ` bytes. The conversion follows a non-specified format (see [[3-The Language#3.4.3 – Coercions and Conversions|§3.4.3]]). The function returns the number of bytes written to the buffer (including the final zero), or zero if the value at `idx` is not a number.

### `lua_pcall`

[-(nargs + 1), +(nresults|1), –]

int lua_pcall (lua_State *L, int nargs, int nresults, int msgh);

Calls a function (or a callable object) in protected mode.

Both `nargs` and `nresults` have the same meaning as in <code>[[#`lua_call`|lua_call]]</code>. If there are no errors during the call, <code>[[#`lua_pcall`|lua_pcall]]</code> behaves exactly like <code>[[#`lua_call`|lua_call]]</code>. However, if there is any error, <code>[[#`lua_pcall`|lua_pcall]]</code> catches it, pushes a single value on the stack (the error object), and returns an error code. Like <code>[[#`lua_call`|lua_call]]</code>, <code>[[#`lua_pcall`|lua_pcall]]</code> always removes the function and its arguments from the stack.

If `msgh` is 0, then the error object returned on the stack is exactly the original error object. Otherwise, `msgh` is the stack index of a _message handler_. (This index cannot be a pseudo-index.) In case of runtime errors, this handler will be called with the error object and its return value will be the object returned on the stack by <code>[[#`lua_pcall`|lua_pcall]]</code>.

Typically, the message handler is used to add more debug information to the error object, such as a stack traceback. Such information cannot be gathered after the return of <code>[[#`lua_pcall`|lua_pcall]]</code>, since by then the stack has unwound.

The <code>[[#`lua_pcall`|lua_pcall]]</code> function returns one of the following status codes: <code>[[#^b348bd|LUA_OK]]</code>, <code>[[#^25e8fc|LUA_ERRRUN]]</code>, <code>[[#^0149b8|LUA_ERRMEM]]</code>, or <code>[[#^62ab88|LUA_ERRERR]]</code>.

### `lua_pcallk`

[-(nargs + 1), +(nresults|1), –]

int lua_pcallk (lua_State *L,
                int nargs,
                int nresults,
                int msgh,
                lua_KContext ctx,
                lua_KFunction k);

This function behaves exactly like <code>[[#`lua_pcall`|lua_pcall]]</code>, except that it allows the called function to yield (see [[#4.5 – Handling Yields in C|§4.5]]).

### `lua_pop`

[-n, +0, _e_]

void lua_pop (lua_State *L, int n);

Pops `n` elements from the stack. It is implemented as a macro over <code>[[#`lua_settop`|lua_settop]]</code>.

### `lua_pushboolean`

[-0, +1, –]

void lua_pushboolean (lua_State *L, int b);

Pushes a boolean value with value `b` onto the stack.

### `lua_pushcclosure`

[-n, +1, _m_]

void lua_pushcclosure (lua_State *L, lua_CFunction fn, int n);

Pushes a new C closure onto the stack. This function receives a pointer to a C function and pushes onto the stack a Lua value of type `function` that, when called, invokes the corresponding C function. The parameter `n` tells how many upvalues this function will have (see [[#4.2 – C Closures|§4.2]]).

Any function to be callable by Lua must follow the correct protocol to receive its parameters and return its results (see <code>[[#`lua_CFunction`|lua_CFunction]]</code>).

When a C function is created, it is possible to associate some values with it, the so called upvalues; these upvalues are then accessible to the function whenever it is called. This association is called a C closure (see [[#4.2 – C Closures|§4.2]]). To create a C closure, first the initial values for its upvalues must be pushed onto the stack. (When there are multiple upvalues, the first value is pushed first.) Then <code>[[#`lua_pushcclosure`|lua_pushcclosure]]</code> is called to create and push the C function onto the stack, with the argument `n` telling how many values will be associated with the function. <code>[[#`lua_pushcclosure`|lua_pushcclosure]]</code> also pops these values from the stack.

The maximum value for `n` is 255.

When `n` is zero, this function creates a _light C function_, which is just a pointer to the C function. In that case, it never raises a memory error.

### `lua_pushcfunction`

[-0, +1, –]

void lua_pushcfunction (lua_State *L, lua_CFunction f);

Pushes a C function onto the stack. This function is equivalent to <code>[[#`lua_pushcclosure`|lua_pushcclosure]]</code> with no upvalues.

### `lua_pushexternalstring`

[-0, +1, _m_]

const char *lua_pushexternalstring (lua_State *L,
                const char *s, size_t len, lua_Alloc falloc, void *ud);

Creates an _external string_, that is, a string that uses memory not managed by Lua. The pointer `s` points to the external buffer holding the string content, and `len` is the length of the string. The string should have a zero at its end, that is, the condition `s[len] == '\0'` should hold. As with any string in Lua, the length must fit in a Lua integer.

If `falloc` is different from `NULL`, that function will be called by Lua when the external buffer is no longer needed. The contents of the buffer should not change before this call. The function will be called with the given `ud`, the string `s` as the block, the length plus one (to account for the ending zero) as the old size, and 0 as the new size.

Even when using an external buffer, Lua still has to allocate a header for the string. In case of a memory-allocation error, Lua will call `falloc` before raising the error.

The function returns a pointer to the string (that is, `s`).

### `lua_pushfstring`

[-0, +1, _v_]

const char *lua_pushfstring (lua_State *L, const char *fmt, ...);

Pushes onto the stack a formatted string and returns a pointer to this string (see [[#4.1.3 – Pointers to Strings|§4.1.3]]). The result is a copy of `fmt` with each _conversion specifier_ replaced by a string representation of its respective extra argument. A conversion specifier (and its corresponding extra argument) can be '`%%`' (inserts the character '`%`'), '`%s`' (inserts a zero-terminated string, with no size restrictions), '`%f`' (inserts a <code>[[#`lua_Number`|lua_Number]]</code>), '`%I`' (inserts a <code>[[#`lua_Integer`|lua_Integer]]</code>), '`%p`' (inserts a void pointer), '`%d`' (inserts an `int`), '`%c`' (inserts an `int` as a one-byte character), and '`%U`' (inserts an `unsigned long` as a UTF-8 byte sequence).

Every occurrence of '`%`' in the string `fmt` must form a valid conversion specifier.

Besides memory allocation errors, this function may raise an error if the resulting string is too large.

### `lua_pushglobaltable`

[-0, +1, –]

void lua_pushglobaltable (lua_State *L);

Pushes the global environment onto the stack.

### `lua_pushinteger`

[-0, +1, –]

void lua_pushinteger (lua_State *L, lua_Integer n);

Pushes an integer with value `n` onto the stack.

### `lua_pushlightuserdata`

[-0, +1, –]

void lua_pushlightuserdata (lua_State *L, void *p);

Pushes a light userdata onto the stack.

Userdata represent C values in Lua. A _light userdata_ represents a pointer, a `void*`. It is a value (like a number): you do not create it, it has no individual metatable, and it is not collected (as it was never created). A light userdata is equal to "any" light userdata with the same C address.

### `lua_pushliteral`

[-0, +1, _v_]

const char *lua_pushliteral (lua_State *L, const char *s);

This macro is equivalent to <code>[[#`lua_pushstring`|lua_pushstring]]</code>, but should be used only when `s` is a literal string. (Lua may optimize this case.)

### `lua_pushlstring`

[-0, +1, _v_]

const char *lua_pushlstring (lua_State *L, const char *s, size_t len);

Pushes the string pointed to by `s` with size `len` onto the stack. Lua will make or reuse an internal copy of the given string, so the memory at `s` can be freed or reused immediately after the function returns. The string can contain any binary data, including embedded zeros.

Returns a pointer to the internal copy of the string (see [[#4.1.3 – Pointers to Strings|§4.1.3]]).

Besides memory allocation errors, this function may raise an error if the string is too large.

### `lua_pushnil`

[-0, +1, –]

void lua_pushnil (lua_State *L);

Pushes a nil value onto the stack.

### `lua_pushnumber`

[-0, +1, –]

void lua_pushnumber (lua_State *L, lua_Number n);

Pushes a float with value `n` onto the stack.

### `lua_pushstring`

[-0, +1, _m_]

const char *lua_pushstring (lua_State *L, const char *s);

Pushes the zero-terminated string pointed to by `s` onto the stack. Lua will make or reuse an internal copy of the given string, so the memory at `s` can be freed or reused immediately after the function returns.

Returns a pointer to the internal copy of the string (see [[#4.1.3 – Pointers to Strings|§4.1.3]]).

If `s` is `NULL`, pushes **nil** and returns `NULL`.

### `lua_pushthread`

[-0, +1, –]

int lua_pushthread (lua_State *L);

Pushes the thread represented by `L` onto the stack. Returns 1 if this thread is the main thread of its state.

### `lua_pushvalue`

[-0, +1, –]

void lua_pushvalue (lua_State *L, int index);

Pushes a copy of the element at the given index onto the stack.

### `lua_pushvfstring`

[-0, +1, –]

const char *lua_pushvfstring (lua_State *L,
                              const char *fmt,
                              va_list argp);

Equivalent to <code>[[#`lua_pushfstring`|lua_pushfstring]]</code>, except that it receives a `va_list` instead of a variable number of arguments, and it does not raise errors. Instead, in case of errors it pushes the error message and returns `NULL`.

### `lua_rawequal`

[-0, +0, –]

int lua_rawequal (lua_State *L, int index1, int index2);

Returns 1 if the two values in indices `index1` and `index2` are primitively equal (that is, equal without calling the `__eq` metamethod). Otherwise returns 0. Also returns 0 if any of the indices are not valid.

### `lua_rawget`

[-1, +1, –]

int lua_rawget (lua_State *L, int index);

Similar to <code>[[#`lua_gettable`|lua_gettable]]</code>, but does a raw access (i.e., without metamethods). The value at `index` must be a table.

### `lua_rawgeti`

[-0, +1, –]

int lua_rawgeti (lua_State *L, int index, lua_Integer n);

Pushes onto the stack the value `t[n]`, where `t` is the table at the given index. The access is raw, that is, it does not use the `__index` metavalue.

Returns the type of the pushed value.

### `lua_rawgetp`

[-0, +1, –]

int lua_rawgetp (lua_State *L, int index, const void *p);

Pushes onto the stack the value `t[k]`, where `t` is the table at the given index and `k` is the pointer `p` represented as a light userdata. The access is raw; that is, it does not use the `__index` metavalue.

Returns the type of the pushed value.

### `lua_rawlen`

[-0, +0, –]

lua_Unsigned lua_rawlen (lua_State *L, int index);

Returns the raw "length" of the value at the given index: for strings, this is the string length; for tables, this is the result of the length operator ('`#`') with no metamethods; for userdata, this is the size of the block of memory allocated for the userdata. For other values, this call returns 0.

### `lua_rawset`

[-2, +0, _m_]

void lua_rawset (lua_State *L, int index);

Similar to <code>[[#`lua_settable`|lua_settable]]</code>, but does a raw assignment (i.e., without metamethods). The value at `index` must be a table.

### `lua_rawseti`

[-1, +0, _m_]

void lua_rawseti (lua_State *L, int index, lua_Integer i);

Does the equivalent of `t[i] = v`, where `t` is the table at the given index and `v` is the value on the top of the stack.

This function pops the value from the stack. The assignment is raw, that is, it does not use the `__newindex` metavalue.

### `lua_rawsetp`

[-1, +0, _m_]

void lua_rawsetp (lua_State *L, int index, const void *p);

Does the equivalent of `t[p] = v`, where `t` is the table at the given index, `p` is encoded as a light userdata, and `v` is the value on the top of the stack.

This function pops the value from the stack. The assignment is raw, that is, it does not use the `__newindex` metavalue.

### `lua_Reader`

typedef const char * (*lua_Reader) (lua_State *L,
                                    void *data,
                                    size_t *size);

The reader function used by <code>[[#`lua_load`|lua_load]]</code>. Every time <code>[[#`lua_load`|lua_load]]</code> needs another piece of the chunk, it calls the reader, passing along its `data` parameter. The reader must return a pointer to a block of memory with a new piece of the chunk and set `size` to the block size. The block must exist until the reader function is called again. To signal the end of the chunk, the reader must return `NULL` or set `size` to zero. The reader function may return pieces of any size greater than zero.

### `lua_register`

[-0, +0, _e_]

void lua_register (lua_State *L, const char *name, lua_CFunction f);

Sets the C function `f` as the new value of global `name`. It is defined as a macro:

     #define lua_register(L,n,f) \
            (lua_pushcfunction(L, f), lua_setglobal(L, n))

### `lua_remove`

[-1, +0, –]

void lua_remove (lua_State *L, int index);

Removes the element at the given valid index, shifting down the elements above this index to fill the gap. This function cannot be called with a pseudo-index, because a pseudo-index is not an actual stack position.

### `lua_replace`

[-1, +0, –]

void lua_replace (lua_State *L, int index);

Moves the top element into the given valid index without shifting any element (therefore replacing the value at that given index), and then pops the top element.

### `lua_resume`

[-?, +?, –]

int lua_resume (lua_State *L, lua_State *from, int nargs,
                          int *nresults);

Starts and resumes a coroutine in the given thread `L`.

To start a coroutine, you push the main function plus any arguments onto the empty stack of the thread. then you call <code>[[#`lua_resume`|lua_resume]]</code>, with `nargs` being the number of arguments. The function returns when the coroutine suspends, finishes its execution, or raises an unprotected error. When it returns without errors, `*nresults` is updated and the top of the stack contains the `*nresults` values passed to <code>[[#`lua_yield`|lua_yield]]</code> or returned by the body function. <code>[[#`lua_resume`|lua_resume]]</code> returns <code>[[#^fdddd5|LUA_YIELD]]</code> if the coroutine yields, <code>[[#^b348bd|LUA_OK]]</code> if the coroutine finishes its execution without errors, or an error code in case of errors (see [[#4.4.1 – Status Codes|§4.4.1]]). In case of errors, the error object is pushed on the top of the stack. (In that case, `nresults` is not updated, as its value would have to be 1 for the sole error object.)

To resume a suspended coroutine, you remove the `*nresults` yielded values from its stack, push the values to be passed as results from `yield`, and then call <code>[[#`lua_resume`|lua_resume]]</code>.

The parameter `from` represents the coroutine that is resuming `L`. If there is no such coroutine, this parameter can be `NULL`.

### `lua_rotate`

[-0, +0, –]

void lua_rotate (lua_State *L, int idx, int n);

Rotates the stack elements between the valid index `idx` and the top of the stack. The elements are rotated `n` positions in the direction of the top, for a positive `n`, or `-n` positions in the direction of the bottom, for a negative `n`. The absolute value of `n` must not be greater than the size of the slice being rotated. This function cannot be called with a pseudo-index, because a pseudo-index is not an actual stack position.

### `lua_setallocf`

[-0, +0, –]

void lua_setallocf (lua_State *L, lua_Alloc f, void *ud);

Changes the allocator function of a given state to `f` with user data `ud`.

### `lua_setfield`

[-1, +0, _e_]

void lua_setfield (lua_State *L, int index, const char *k);

Does the equivalent to `t[k] = v`, where `t` is the value at the given index and `v` is the value on the top of the stack.

This function pops the value from the stack. As in Lua, this function may trigger a metamethod for the "newindex" event (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

### `lua_setglobal`

[-1, +0, _e_]

void lua_setglobal (lua_State *L, const char *name);

Pops a value from the stack and sets it as the new value of global `name`.

### `lua_seti`

[-1, +0, _e_]

void lua_seti (lua_State *L, int index, lua_Integer n);

Does the equivalent to `t[n] = v`, where `t` is the value at the given index and `v` is the value on the top of the stack.

This function pops the value from the stack. As in Lua, this function may trigger a metamethod for the "newindex" event (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

### `lua_setiuservalue`

[-1, +0, –]

int lua_setiuservalue (lua_State *L, int index, int n);

Pops a value from the stack and sets it as the new `n`-th user value associated to the full userdata at the given index. Returns 0 if the userdata does not have that value.

### `lua_setmetatable`

[-1, +0, –]

int lua_setmetatable (lua_State *L, int index);

Pops a table or **nil** from the stack and sets that value as the new metatable for the value at the given index. (**nil** means no metatable.)

(For historical reasons, this function returns an `int`, which now is always 1.)

### `lua_settable`

[-2, +0, _e_]

void lua_settable (lua_State *L, int index);

Does the equivalent to `t[k] = v`, where `t` is the value at the given index, `v` is the value on the top of the stack, and `k` is the value just below the top.

This function pops both the key and the value from the stack. As in Lua, this function may trigger a metamethod for the "newindex" event (see [[2-Basic Concepts#2.4 – Metatables and Metamethods|§2.4]]).

### `lua_settop`

[-?, +?, _e_]

void lua_settop (lua_State *L, int index);

Receives any acceptable stack index, or 0, and sets the stack top to this index. If the new top is greater than the old one, then the new elements are filled with **nil**. If `index` is 0, then all stack elements are removed.

This function can run arbitrary code when removing an index marked as to-be-closed from the stack.

### `lua_setwarnf`

[-0, +0, –]

void lua_setwarnf (lua_State *L, lua_WarnFunction f, void *ud);

Sets the warning function to be used by Lua to emit warnings (see <code>[[#`lua_WarnFunction`|lua_WarnFunction]]</code>). The `ud` parameter sets the value `ud` passed to the warning function.

### `lua_State`

typedef struct lua_State lua_State;

An opaque structure that points to a thread and indirectly (through the thread) to the whole state of a Lua interpreter. The Lua library is fully reentrant: it has no global variables. All information about a state is accessible through this structure.

A pointer to this structure must be passed as the first argument to every function in the library, except to <code>[[#`lua_newstate`|lua_newstate]]</code>, which creates a Lua state from scratch.

### `lua_status`

[-0, +0, –]

int lua_status (lua_State *L);

Returns the status of the thread `L`.

The status can be <code>[[#^b348bd|LUA_OK]]</code> for a normal thread, an error code if the thread finished the execution of a <code>[[#`lua_resume`|lua_resume]]</code> with an error, or <code>[[#^fdddd5|LUA_YIELD]]</code> if the thread is suspended.

You can call functions only in threads with status <code>[[#^b348bd|LUA_OK]]</code>. You can resume threads with status <code>[[#^b348bd|LUA_OK]]</code> (to start a new coroutine) or <code>[[#^fdddd5|LUA_YIELD]]</code> (to resume a coroutine).

### `lua_stringtonumber`

[-0, +1, –]

size_t lua_stringtonumber (lua_State *L, const char *s);

Converts the zero-terminated string `s` to a number, pushes that number into the stack, and returns the total size of the string, that is, its length plus one. The conversion can result in an integer or a float, according to the lexical conventions of Lua (see [[3-The Language#3.1 – Lexical Conventions|§3.1]]). The string may have leading and trailing whitespaces and a sign. If the string is not a valid numeral, returns 0 and pushes nothing. (Note that the result can be used as a boolean, true if the conversion succeeds.)

### `lua_toboolean`

[-0, +0, –]

int lua_toboolean (lua_State *L, int index);

Converts the Lua value at the given index to a C boolean value (0 or 1). Like all tests in Lua, <code>[[#`lua_toboolean`|lua_toboolean]]</code> returns true for any Lua value different from **false** and **nil**; otherwise it returns false. (If you want to accept only actual boolean values, use <code>[[#`lua_isboolean`|lua_isboolean]]</code> to test the value's type.)

### `lua_tocfunction`

[-0, +0, –]

lua_CFunction lua_tocfunction (lua_State *L, int index);

Converts a value at the given index to a C function. That value must be a C function; otherwise, returns `NULL`.

### `lua_toclose`

[-0, +0, _v_]

void lua_toclose (lua_State *L, int index);

Marks the given index in the stack as a to-be-closed slot (see [[3-The Language#3.3.8 – To-be-closed Variables|§3.3.8]]). Like a to-be-closed variable in Lua, the value at that slot in the stack will be closed when it goes out of scope. Here, in the context of a C function, to go out of scope means that the running function returns to Lua, or there is an error, or the slot is removed from the stack through <code>[[#`lua_settop`|lua_settop]]</code> or <code>[[#`lua_pop`|lua_pop]]</code>, or there is a call to <code>[[#`lua_closeslot`|lua_closeslot]]</code>. A slot marked as to-be-closed should not be removed from the stack by any other function in the API except <code>[[#`lua_settop`|lua_settop]]</code> or <code>[[#`lua_pop`|lua_pop]]</code>, unless previously deactivated by <code>[[#`lua_closeslot`|lua_closeslot]]</code>.

This function raises an error if the value at the given slot neither has a `__close` metamethod nor is a false value.

This function should not be called for an index that is equal to or below an active to-be-closed slot.

Note that, both in case of errors and of a regular return, by the time the `__close` metamethod runs, the C stack was already unwound, so that any automatic C variable declared in the calling function (e.g., a buffer) will be out of scope.

### `lua_tointeger`

[-0, +0, –]

lua_Integer lua_tointeger (lua_State *L, int index);

Equivalent to <code>[[#`lua_tointegerx`|lua_tointegerx]]</code> with `isnum` equal to `NULL`.

### `lua_tointegerx`

[-0, +0, –]

lua_Integer lua_tointegerx (lua_State *L, int index, int *isnum);

Converts the Lua value at the given index to the signed integral type <code>[[#`lua_Integer`|lua_Integer]]</code>. The Lua value must be an integer, or a number or string convertible to an integer (see [[3-The Language#3.4.3 – Coercions and Conversions|§3.4.3]]); otherwise, `lua_tointegerx` returns 0.

If `isnum` is not `NULL`, its referent is assigned a boolean value that indicates whether the operation succeeded.

### `lua_tolstring`

[-0, +0, _m_]

const char *lua_tolstring (lua_State *L, int index, size_t *len);

Converts the Lua value at the given index to a C string. The Lua value must be a string or a number; otherwise, the function returns `NULL`. If the value is a number, then `lua_tolstring` also _changes the actual value in the stack to a string_. (This change confuses <code>[[#`lua_next`|lua_next]]</code> when `lua_tolstring` is applied to keys during a table traversal.)

If `len` is not `NULL`, the function sets `*len` with the string length. The returned C string always has a zero ('`\0`') after its last character, but can contain other zeros in its body.

The pointer returned by `lua_tolstring` may be invalidated by the garbage collector if the corresponding Lua value is removed from the stack (see [[#4.1.3 – Pointers to Strings|§4.1.3]]).

This function can raise memory errors only when converting a number to a string (as then it may create a new string).

### `lua_tonumber`

[-0, +0, –]

lua_Number lua_tonumber (lua_State *L, int index);

Equivalent to <code>[[#`lua_tonumberx`|lua_tonumberx]]</code> with `isnum` equal to `NULL`.

### `lua_tonumberx`

[-0, +0, –]

lua_Number lua_tonumberx (lua_State *L, int index, int *isnum);

Converts the Lua value at the given index to the C type <code>[[#`lua_Number`|lua_Number]]</code> (see <code>[[#`lua_Number`|lua_Number]]</code>). The Lua value must be a number or a string convertible to a number (see [[3-The Language#3.4.3 – Coercions and Conversions|§3.4.3]]); otherwise, <code>[[#`lua_tonumberx`|lua_tonumberx]]</code> returns 0.

If `isnum` is not `NULL`, its referent is assigned a boolean value that indicates whether the operation succeeded.

### `lua_topointer`

[-0, +0, –]

const void *lua_topointer (lua_State *L, int index);

Converts the value at the given index to a generic C pointer (`void*`). The value can be a userdata, a table, a thread, a string, or a function; otherwise, `lua_topointer` returns `NULL`. Different objects will give different pointers. There is no way to convert the pointer back to its original value.

Typically this function is used only for hashing and debug information.

### `lua_tostring`

[-0, +0, _m_]

const char *lua_tostring (lua_State *L, int index);

Equivalent to <code>[[#`lua_tolstring`|lua_tolstring]]</code> with `len` equal to `NULL`.

### `lua_tothread`

[-0, +0, –]

lua_State *lua_tothread (lua_State *L, int index);

Converts the value at the given index to a Lua thread (represented as `lua_State*`). This value must be a thread; otherwise, the function returns `NULL`.

### `lua_touserdata`

[-0, +0, –]

void *lua_touserdata (lua_State *L, int index);

If the value at the given index is a full userdata, returns its memory-block address. If the value is a light userdata, returns its value (a pointer). Otherwise, returns `NULL`.

### `lua_type`

[-0, +0, –]

int lua_type (lua_State *L, int index);

Returns the type of the value in the given valid index, or `LUA_TNONE` for a non-valid but acceptable index. The types returned by <code>[[#`lua_type`|lua_type]]</code> are coded by the following constants defined in `lua.h`: `LUA_TNIL`, `LUA_TNUMBER`, `LUA_TBOOLEAN`, `LUA_TSTRING`, `LUA_TTABLE`, `LUA_TFUNCTION`, `LUA_TUSERDATA`, `LUA_TTHREAD`, and `LUA_TLIGHTUSERDATA`. ^e5175e

### `lua_typename`

[-0, +0, –]

const char *lua_typename (lua_State *L, int tp);

Returns the name of the type encoded by the value `tp`, which must be one the values returned by <code>[[#`lua_type`|lua_type]]</code>.

### `lua_Unsigned`

typedef ... lua_Unsigned;

The unsigned version of <code>[[#`lua_Integer`|lua_Integer]]</code>.

### `lua_upvalueindex`

[-0, +0, –]

int lua_upvalueindex (int i);

Returns the pseudo-index that represents the `i`-th upvalue of the running function (see [[#4.2 – C Closures|§4.2]]). `i` must be in the range _[1,256]_.

### `lua_version`

[-0, +0, –]

lua_Number lua_version (lua_State *L);

Returns the version number of this core.

### `lua_WarnFunction`

typedef void (*lua_WarnFunction) (void *ud, const char *msg, int tocont);

The type of warning functions, called by Lua to emit warnings. The first parameter is an opaque pointer set by <code>[[#`lua_setwarnf`|lua_setwarnf]]</code>. The second parameter is the warning message. The third parameter is a boolean that indicates whether the message is to be continued by the message in the next call.

See <code>[[6-The Standard Libraries#`warn (msg1, ···)`|warn]]</code> for more details about warnings.

### `lua_warning`

[-0, +0, –]

void lua_warning (lua_State *L, const char *msg, int tocont);

Emits a warning with the given message. A message in a call with `tocont` true should be continued in another call to this function.

See <code>[[6-The Standard Libraries#`warn (msg1, ···)`|warn]]</code> for more details about warnings.

### `lua_Writer`

typedef int (*lua_Writer) (lua_State *L,
                           const void* p,
                           size_t sz,
                           void* ud);

The type of the writer function used by <code>[[#`lua_dump`|lua_dump]]</code>. Every time <code>[[#`lua_dump`|lua_dump]]</code> produces another piece of chunk, it calls the writer, passing along the buffer to be written (`p`), its size (`sz`), and the `ud` parameter supplied to <code>[[#`lua_dump`|lua_dump]]</code>.

After <code>[[#`lua_dump`|lua_dump]]</code> writes its last piece, it will signal that by calling the writer function one more time, with a `NULL` buffer (and size 0).

The writer returns an error code: 0 means no errors; any other value means an error and stops <code>[[#`lua_dump`|lua_dump]]</code> from calling the writer again.

### `lua_xmove`

[-?, +?, –]

void lua_xmove (lua_State *from, lua_State *to, int n);

Exchange values between different threads of the same state.

This function pops `n` values from the stack `from`, and pushes them onto the stack `to`.

### `lua_yield`

[-?, +?, _v_]

int lua_yield (lua_State *L, int nresults);

This function is equivalent to <code>[[#`lua_yieldk`|lua_yieldk]]</code>, but it has no continuation (see [[#4.5 – Handling Yields in C|§4.5]]). Therefore, when the thread resumes, it continues the function that called the function calling `lua_yield`. To avoid surprises, this function should be called only in a tail call.

### `lua_yieldk`

[-?, +?, _v_]

int lua_yieldk (lua_State *L,
                int nresults,
                lua_KContext ctx,
                lua_KFunction k);

Yields a coroutine (thread).

When a C function calls <code>[[#`lua_yieldk`|lua_yieldk]]</code>, the running coroutine suspends its execution, and the call to <code>[[#`lua_resume`|lua_resume]]</code> that started this coroutine returns. The parameter `nresults` is the number of values from the stack that will be passed as results to <code>[[#`lua_resume`|lua_resume]]</code>.

When the coroutine is resumed again, Lua calls the given continuation function `k` to continue the execution of the C function that yielded (see [[#4.5 – Handling Yields in C|§4.5]]). This continuation function receives the same stack from the previous function, with the `n` results removed and replaced by the arguments passed to <code>[[#`lua_resume`|lua_resume]]</code>. Moreover, the continuation function receives the value `ctx` that was passed to <code>[[#`lua_yieldk`|lua_yieldk]]</code>.

Usually, this function does not return; when the coroutine eventually resumes, it continues executing the continuation function. However, there is one special case, which is when this function is called from inside a line or a count hook (see [[#4.7 – The Debug Interface|§4.7]]). In that case, `lua_yieldk` should be called with no continuation (probably in the form of <code>[[#`lua_yield`|lua_yield]]</code>) and no results, and the hook should return immediately after the call. Lua will yield and, when the coroutine resumes again, it will continue the normal execution of the (Lua) function that triggered the hook.

This function can raise an error if it is called from a thread with a pending C call with no continuation function (what is called a _C-call boundary_), or it is called from a thread that is not running inside a resume (typically the main thread).

## 4.7 – The Debug Interface

Lua has no built-in debugging facilities. Instead, it offers a special interface by means of functions and _hooks_. This interface allows the construction of different kinds of debuggers, profilers, and other tools that need "inside information" from the interpreter.

### `lua_Debug`

typedef struct lua_Debug {
  int event;
  const char *name;           /* (n) */
  const char *namewhat;       /* (n) */
  const char *what;           /* (S) */
  const char *source;         /* (S) */
  size_t srclen;              /* (S) */
  int currentline;            /* (l) */
  int linedefined;            /* (S) */
  int lastlinedefined;        /* (S) */
  unsigned char nups;         /* (u) number of upvalues */
  unsigned char nparams;      /* (u) number of parameters */
  char isvararg;              /* (u) */
  unsigned char extraargs;    /* (t) number of extra arguments */
  char istailcall;            /* (t) */
  int ftransfer;              /* (r) index of first value transferred */
  int ntransfer;              /* (r) number of transferred values */
  char short_src[LUA_IDSIZE]; /* (S) */
  /* private part */
  _other fields_
} lua_Debug;

A structure used to carry different pieces of information about a function or an activation record. <code>[[#`lua_getstack`|lua_getstack]]</code> fills only the private part of this structure, for later use. To fill the other fields of <code>[[#`lua_Debug`|lua_Debug]]</code> with useful information, you must call <code>[[#`lua_getinfo`|lua_getinfo]]</code> with an appropriate parameter. (Specifically, to get a field, you must add the letter between parentheses in the field's comment to the parameter `what` of <code>[[#`lua_getinfo`|lua_getinfo]]</code>.)

The fields of <code>[[#`lua_Debug`|lua_Debug]]</code> have the following meaning:

- **`source`**: the source of the chunk that created the function. If `source` starts with a '`@`', it means that the function was defined in a file where the file name follows the '`@`'. If `source` starts with a '`=`', the remainder of its contents describes the source in a user-dependent manner. Otherwise, the function was defined in a string where `source` is that string.
- **`srclen`**: The length of the string `source`.
- **`short_src`**: a "printable" version of `source`, to be used in error messages.
- **`linedefined`**: the line number where the definition of the function starts.
- **`lastlinedefined`**: the line number where the definition of the function ends.
- **`what`**: the string `"Lua"` if the function is a Lua function, `"C"` if it is a C function, `"main"` if it is the main part of a chunk.
- **`currentline`**: the current line where the given function is executing. When no line information is available, `currentline` is set to -1.
- **`name`**: a reasonable name for the given function. Because functions in Lua are first-class values, they do not have a fixed name: some functions can be the value of multiple global variables, while others can be stored only in a table field. The `lua_getinfo` function checks how the function was called to find a suitable name. If it cannot find a name, then `name` is set to `NULL`.
- **`namewhat`**: explains the `name` field. The value of `namewhat` can be `"global"`, `"local"`, `"upvalue"`, `"field"`, `""` (the empty string), plus some other options, according to how the function was called. (Lua uses the empty string when no other option seems to apply.)
- **`istailcall`**: true if this function invocation was called by a tail call. In this case, the caller of this level is not in the stack.
- **`extraargs`**: The number of extra arguments added by the call to functions called through `__call` metamethods. (Each `__call` metavalue adds a single extra argument, the object being called, but there may be a chain of `__call` metavalues.)
- **`nups`**: the number of upvalues of the function.
- **`nparams`**: the number of parameters of the function (always 0 for C functions).
- **`isvararg`**: true if the function is a variadic function (always true for C functions).
- **`ftransfer`**: the index in the stack of the first value being "transferred", that is, parameters in a call or return values in a return. (The other values are in consecutive indices.) Using this index, you can access and modify these values through <code>[[#`lua_getlocal`|lua_getlocal]]</code> and <code>[[#`lua_setlocal`|lua_setlocal]]</code>. This field is only meaningful during a call hook, denoting the first parameter, or a return hook, denoting the first value being returned. (For call hooks, this value is always 1.)
- **`ntransfer`**: The number of values being transferred (see previous item). (For calls of Lua functions, this value is always equal to `nparams`.)

### `lua_gethook`

[-0, +0, –]

lua_Hook lua_gethook (lua_State *L);

Returns the current hook function.

### `lua_gethookcount`

[-0, +0, –]

int lua_gethookcount (lua_State *L);

Returns the current hook count.

### `lua_gethookmask`

[-0, +0, –]

int lua_gethookmask (lua_State *L);

Returns the current hook mask.

### `lua_getinfo`

[-(0|1), +(0|1|2), _m_]

int lua_getinfo (lua_State *L, const char *what, lua_Debug *ar);

Gets information about a specific function or function invocation.

To get information about a function invocation, the parameter `ar` must be a valid activation record that was filled by a previous call to <code>[[#`lua_getstack`|lua_getstack]]</code> or given as argument to a hook (see <code>[[#`lua_Hook`|lua_Hook]]</code>).

To get information about a function, you push it onto the stack and start the `what` string with the character '`>`'. (In that case, `lua_getinfo` pops the function from the top of the stack.) For instance, to know in which line a function `f` was defined, you can write the following code:

     lua_Debug ar;
     lua_getglobal(L, "f");  /* get global 'f' */
     lua_getinfo(L, ">S", &ar);
     printf("%d\n", ar.linedefined);

Each character in the string `what` selects some fields of the structure `ar` to be filled or a value to be pushed on the stack. (These characters are also documented in the declaration of the structure <code>[[#`lua_Debug`|lua_Debug]]</code>, between parentheses in the comments following each field.)

- **'`f`'**: pushes onto the stack the function that is running at the given level;
- **'`l`'**: fills in the field `currentline`;
- **'`n`'**: fills in the fields `name` and `namewhat`;
- **'`r`'**: fills in the fields `ftransfer` and `ntransfer`;
- **'`S`'**: fills in the fields `source`, `short_src`, `linedefined`, `lastlinedefined`, and `what`;
- **'`t`'**: fills in the fields `istailcall` and `extraargs`;
- **'`u`'**: fills in the fields `nups`, `nparams`, and `isvararg`;
- **'`L`'**: pushes onto the stack a table whose indices are the lines on the function with some associated code, that is, the lines where you can put a break point. (Lines with no code include empty lines and comments.) If this option is given together with option '`f`', its table is pushed after the function. This is the only option that can raise a memory error.

This function returns 0 to signal an invalid option in `what`; even then the valid options are handled correctly.

### `lua_getlocal`

[-0, +(0|1), –]

const char *lua_getlocal (lua_State *L, const lua_Debug *ar, int n);

Gets information about a local variable or a temporary value of a given activation record or a given function.

In the first case, the parameter `ar` must be a valid activation record that was filled by a previous call to <code>[[#`lua_getstack`|lua_getstack]]</code> or given as argument to a hook (see <code>[[#`lua_Hook`|lua_Hook]]</code>). The index `n` selects which local variable to inspect; see <code>[[6-The Standard Libraries#`debug.getlocal ([thread,] f, local)`|debug.getlocal]]</code> for details about variable indices and names.

<code>[[#`lua_getlocal`|lua_getlocal]]</code> pushes the variable's value onto the stack and returns its name.

In the second case, `ar` must be `NULL` and the function to be inspected must be on the top of the stack. In this case, only parameters of Lua functions are visible (as there is no information about what variables are active) and no values are pushed onto the stack.

Returns `NULL` (and pushes nothing) when the index is greater than the number of active local variables.

### `lua_getstack`

[-0, +0, –]

int lua_getstack (lua_State *L, int level, lua_Debug *ar);

Gets information about the interpreter runtime stack.

This function fills parts of a <code>[[#`lua_Debug`|lua_Debug]]</code> structure with an identification of the _activation record_ of the function executing at a given level. Level 0 is the current running function, whereas level _n+1_ is the function that has called level _n_ (except for tail calls, which do not count in the stack). When called with a level greater than the stack depth, <code>[[#`lua_getstack`|lua_getstack]]</code> returns 0; otherwise it returns 1.

### `lua_getupvalue`

[-0, +(0|1), –]

const char *lua_getupvalue (lua_State *L, int funcindex, int n);

Gets information about the `n`-th upvalue of the closure at index `funcindex`. It pushes the upvalue's value onto the stack and returns its name. Returns `NULL` (and pushes nothing) when the index `n` is greater than the number of upvalues.

See <code>[[6-The Standard Libraries#`debug.getupvalue (f, up)`|debug.getupvalue]]</code> for more information about upvalues.

### `lua_Hook`

typedef void (*lua_Hook) (lua_State *L, lua_Debug *ar);

Type for debugging hook functions.

Whenever a hook is called, its `ar` argument has its field `event` set to the specific event that triggered the hook. Lua identifies these events with the following constants: `LUA_HOOKCALL`, `LUA_HOOKRET`, `LUA_HOOKTAILCALL`, `LUA_HOOKLINE`, and `LUA_HOOKCOUNT`. Moreover, for line events, the field `currentline` is also set. To get the value of any other field in `ar`, the hook must call <code>[[#`lua_getinfo`|lua_getinfo]]</code>.

For call events, `event` can be `LUA_HOOKCALL`, the normal value, or `LUA_HOOKTAILCALL`, for a tail call; in this case, there will be no corresponding return event.

While Lua is running a hook, it disables other calls to hooks. Therefore, if a hook calls back Lua to execute a function or a chunk, this execution occurs without any calls to hooks.

Hook functions cannot have continuations, that is, they cannot call <code>[[#`lua_yieldk`|lua_yieldk]]</code>, <code>[[#`lua_pcallk`|lua_pcallk]]</code>, or <code>[[#`lua_callk`|lua_callk]]</code> with a non-null `k`.

Hook functions can yield under the following conditions: Only count and line events can yield; to yield, a hook function must finish its execution calling <code>[[#`lua_yield`|lua_yield]]</code> with `nresults` equal to zero (that is, with no values).

### `lua_sethook`

[-0, +0, –]

void lua_sethook (lua_State *L, lua_Hook f, int mask, int count);

Sets the debugging hook function.

Argument `f` is the hook function. `mask` specifies on which events the hook will be called: it is formed by a bitwise OR of the constants `LUA_MASKCALL`, `LUA_MASKRET`, `LUA_MASKLINE`, and `LUA_MASKCOUNT`. The `count` argument is only meaningful when the mask includes `LUA_MASKCOUNT`. For each event, the hook is called as explained below:

- **The call hook**: is called when the interpreter calls a function. The hook is called just after Lua enters the new function.
- **The return hook**: is called when the interpreter returns from a function. The hook is called just before Lua leaves the function.
- **The line hook**: is called when the interpreter is about to start the execution of a new line of code, or when it jumps back in the code (even to the same line). This event only happens while Lua is executing a Lua function.
- **The count hook**: is called after the interpreter executes every `count` instructions. This event only happens while Lua is executing a Lua function.

Hooks are disabled by setting `mask` to zero.

### `lua_setlocal`

[-(0|1), +0, –]

const char *lua_setlocal (lua_State *L, const lua_Debug *ar, int n);

Sets the value of a local variable of a given activation record. It assigns the value on the top of the stack to the variable and returns its name. It also pops the value from the stack.

Returns `NULL` (and pops nothing) when the index is greater than the number of active local variables.

Parameters `ar` and `n` are as in the function <code>[[#`lua_getlocal`|lua_getlocal]]</code>.

### `lua_setupvalue`

[-(0|1), +0, –]

const char *lua_setupvalue (lua_State *L, int funcindex, int n);

Sets the value of a closure's upvalue. It assigns the value on the top of the stack to the upvalue and returns its name. It also pops the value from the stack.

Returns `NULL` (and pops nothing) when the index `n` is greater than the number of upvalues.

Parameters `funcindex` and `n` are as in the function <code>[[#`lua_getupvalue`|lua_getupvalue]]</code>.

### `lua_upvalueid`

[-0, +0, –]

void *lua_upvalueid (lua_State *L, int funcindex, int n);

Returns a unique identifier for the upvalue numbered `n` from the closure at index `funcindex`.

These unique identifiers allow a program to check whether different closures share upvalues. Lua closures that share an upvalue (that is, that access a same external local variable) will return identical ids for those upvalue indices.

Parameters `funcindex` and `n` are as in the function <code>[[#`lua_getupvalue`|lua_getupvalue]]</code>, but `n` cannot be greater than the number of upvalues.

### `lua_upvaluejoin`

[-0, +0, –]

void lua_upvaluejoin (lua_State *L, int funcindex1, int n1,
                                    int funcindex2, int n2);

Make the `n1`-th upvalue of the Lua closure at index `funcindex1` refer to the `n2`-th upvalue of the Lua closure at index `funcindex2`.
