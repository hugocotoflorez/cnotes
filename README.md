# Notes about C programming language

# Table of contents
1. [Gcc options](#Gcc)
2. [Makefile](#Makefile)


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

Boilerplate

``` c
return_type (* pointer_name)(params);
```

Example

``` c
void *realloc(void *, size_t);
void *(*realloc_ptr)(void *, size_t);
```

How to get function addresses

``` c
realloc_ptr = &realloc;
realloc_ptr = realloc;
```

Both `&realloc` and `realloc` give the function address.

