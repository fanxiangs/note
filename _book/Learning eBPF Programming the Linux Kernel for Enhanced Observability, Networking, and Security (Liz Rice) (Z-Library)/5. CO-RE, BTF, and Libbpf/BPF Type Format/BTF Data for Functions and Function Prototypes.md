# BTF Data for Functions and Function Prototypes

[[null|]][[null|]]So far the BTF data in this example output has related to data types, but the BTF data also contains information about functions and function prototypes. Here’s the information from the same BTF data blob that describes the `hello` function:

\[31\] FUNC\_PROTO '(anon)' ret\_type\_id=23 vlen=1
        'ctx' type\_id=10
\[32\] FUNC 'hello' type\_id=31 linkage=static

In type 32 you can see the function named `hello` is defined as having the type defined in the previous line. That’s a _function prototype_, which returns a value of type ID `23` and takes a single parameter (`vlen=1`) called `ctx` with type ID `10`. For completeness, here are the definitions of those types from earlier in the output:

\[10\] PTR '(anon)' type\_id=0
 
\[23\] INT 'int' size=4 bits\_offset=0 nr\_bits=32 encoding=SIGNED

Type 10 is an anonymous pointer with the default type of `0`, which isn’t explicitly included in the BTF output but is defined as a void pointer.[^4]

The return value with type 23 is a 4-byte integer, and `encoding=SIGNED` indicates that it’s a signed integer; that is, it can have either a positive or negative value. This corresponds to the function definition in the source code of _hello-buffer-config.py_, which looks like this:

    int

The example BTF information I’ve shown so far comes from listing the contents of a blob of BTF data. Let’s see how to obtain just the BTF information that relates to a particular map or program.