# Notes about C programming language

# Table of contents
1. [Gcc options](#Gcc)
2. [Makefile](#Makefile)
3. [Pointers](#Pointers)
4. [Bit Flags](#BitFlags)
5. [Lock Up Tables](#LockUpTables)
6. [Restrict keyword](#Restrict)
7. [Generics](#Generics)
8. [Structs](#Structs)


## Gcc

> [!NOTE]
> Gcc is the GNU compiler for C, g++ is for C++.

### Optimization

- `-O1`: Optimize
- `-O2`: Optimize more
- `-O3`: Optimize too much
- `-Os`: Optimize for size

### Output

- `-g`: Turn on debug info
- `-c`: Compile or assemble but dont link. (Generates .o)
- `-S`: Stop after compilation. (Generates .s, assembly)
- `-o`: Output name

### C dialect

- `-std=standard`: Determine the language standard. Possible values are c89, gnu89, ..., c23.

### Warnings

- `-Wall`: Warn about almost all warnings.


## Makefile

> [!NOTE]
> Makefile and makefile are both valid names.

### Little observation

Under a header you can place any command that the shell
can execute, for example

``` makefile
git: compile
    git add *.c *.h
    git commit -m "msg"
    git push
```

This rule would execute compile and then run the git commands.

### wildcard

This keyword allow select files in current folder

``` makefile
SRC = $(wildcard *.c)
```

### Select files with different extensions

It is possible to select files with same name but different extensions

``` makefile
OBJ = $(SRC:.c=.o)
```

Selects all .o with the same name as all .c in SRC

## Pointers

### What I learn about how memory works

As a newbie I used to think about variables as a box where
you can store values inside of it, where each variable have
a memory address assigned to it. It is not incorrect, but
addresses look a magical thing that `&` returns and is
useless.

Nowadays I change my point of view. I see memory as a huge
array, that you can access each byte of it. At variable
creation, OS is filling `n` bytes of this array with some
data, where `n` is the type len (in bytes). As you can see
variables are not a box but an offset into the array, given
the position where data is placed.

### Function pointers

#### Boilerplate

``` c
return_type (* pointer_name)(params);
```

#### Example

``` c
void *realloc(void *, size_t);
void *(*realloc_ptr)(void *, size_t);
```

#### How to get function addresses

``` c
realloc_ptr = &realloc;
realloc_ptr = realloc;
```

Both `&realloc` and `realloc` give the function address.
Function pointers can be a field into structs.

### Mindchanging example

``` c
void free(void *ptr){

    header_t *hptr;
    ...
    hptr = (header_t*) ptr - 1;
```

To give a little more context, when allocating a block
into the heap, a header with some info is stored above the n bytes
block that user ask for. As this header is just above returning pointer,
the first byte of the struct is exacly the size of this struct before the
returning pointer.

## BitFlags

Bit flags are a memory-efficient options storege. Assume
that you want to pass to a function a lot of boolean info,
asking for each value would be inefficient.

Bit flags use each bit into a variable to store boolean values.

### How to declare Bit Flags

Easiest and cleanest way is to use enums

``` c
typedef enum {
    NONE = 1 << 0,
    OPT1 = 1 << 1,
    OPT2 = 1 << 2,
    OPT3 = 1 << 3,
    OPT4 = 1 << 4,
} Options;
```

To declare an option param you can do something like
`foo(int a, int b, Options options);`.

### Set options

To set options you have to unary-or option values:
`foo(1, 2, OPT1 | OPT2 | OPT3);` this way you are
passing this 3 options to options param.

### Check for options

#### Using if

`if(options & OPT1){...}` Check if options have OPT1.

#### Using swith

``` c
switch (options){
    case (options == NONE):
    ...
    case (options & OPT1):
    case (options & OPT2):
    case (options & OPT3):
}
```

Note that if check for NONE is needed you have to use `==`
instead of `&`. Think about what happened when doing
`something & 0` (as NONE is 0).

### Unset values

`options &= ~(OPT1);` would unset OPT1 if set from variable
options.

### Quick note for wtfisthis-people

- `1<<n`: move 1 `n` bits to the left.
- `~ a`: bit not.
- `a | b`: bit or.
- `a & b`: bit and.
- `a &= b`: a = a & b.
- `a <<= b`, `a |= b` also valid.


## LockUpTables

Is 0(1) search tech, but sometimes space inefficient.

```c
static const int lookup[] = {
    [0] = 10,
    [1] = 20,
    [2] = 40,
    [3] = 50,
    [10] = 120,
}
```

By doing this you are assigning a constant value to each
array index. You can use an enum to assign values to enum
keys. As you can see accessing time is constant.


## Restrict

Restrict keyword tell the compiler that the pointer marked
as restrict would not be modified by any other pointer, so
value remains constant until modified using restrict pointer.
This allow compiler to generate more optimized code.

Example: `void *memcpy(restrict void dest, restrict void src, size_t n);`

## Changing enum type

This requires c23 standard

``` c
enum myenum: char {
    A = 'a',
    B = 'b',
}
```

## Generics

``` c
#define add(a, b) _Generic(a,
    int: add_int(a,b),
    float: add_float(a,b),
    default: add_unknown(a,b)
    )
```

I think this requires c23 standard

## Structs

### Specify field size

``` c
struct file_header
{
    uint8_t transmission_system : 8;
    uint32_t identifier : 24;
    uint16_t line_ending : 16;
    uint8_t eof_character : 8;
    uint8_t eol_character : 8;
};
```

By doing so you are giving `n` bits to the field instead of
sizeof-type bytes.

### Removing padding

``` c
struct IHDR
{
    uint32_t width : 32;
    uint32_t height : 32;
    uint8_t bit_depth : 8;
    uint8_t color_type : 8;
    uint8_t compression_method : 8;
    uint8_t filter_method : 8;
    uint8_t interlace_method : 8;
} __attribute__((packed));
```

Removing padding is usefull when you use your struct to split
some data into fields, despite you afford some bits it
would do accessing to the struct fields slower. Do this only
if needed.

### Read data with structs

``` c
bool read_header(int fd, struct file_header* header)
{
    read(fd, header, 8);
}
```

By passing the address of an struct to read, the data read
would be stored into the struct, so you can access it by
field name instead of raw bytes. (8 is the struct length in bytes).

#### Playing with types

``` c
struct ieee754_float
{
    unsigned sign : 1;
    unsigned exponent : 8;
    unsigned mantissa : 23;
} __attribute__((packed));

float                f     = 3.14;
struct ieee754_float ieeef = *(struct ieee754_float *) &f;
```

Just try to meassure what this code should do. Is important to typecast
to pointers because otherwise errors about arithmetic stuff raise.
