# Notes about C programming language

# About

This repo is created to share little tricks about C lang
that help me improving as a programmer. Feel free to open
an issue if something is wrong or can be better. Also is a good
place to ask for explanation about other repos (my other repos)
code that looks strange for you. Thanks for be here and I hope it
would help you.

# Table of contents
1. [Gcc options](#Gcc)
2. [Makefile](#Makefile)
3. [Pointers](#Pointers)
4. [Bit Flags](#BitFlags)
5. [Lock Up Tables](#LockUpTables)
6. [Restrict keyword](#Restrict)
7. [Generics](#Generics)
8. [Structs](#Structs)
9. [Fix inits](#Constructor)
10. [Static keyword](#Static)
11. [constexpr](#constexpr)
12. [gnu c](#gnu)


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

This rule would execute `compile` and then run the git commands.

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

### Silent output

By adding an `@` before any command, it is not printed
when executed.

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

To set options you have to binary-or option values:
`foo(1, 2, OPT1 | OPT2 | OPT3);` this way you are
passing this 3 options to options param.

Also you can add an option to a variable using 
`opt |= OPT1`.

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

## Constructor

All proyects require to initialize data at program start.
You can just create a init() function and call it from main.
But sometimes there are too much inits functions in too much files and
you dont want to have to ask the user to manually initialize every functionality.

By adding another attribute you can tell the compiler to call the function
before calling main().

``` c
static __attribute__((constructor)) void __init__()
{
}
```

### Static

You would have asked what tf is static as it appears more than once before
this line.

Static said the compiler that the function can only be accessed from this
file, allowing using the same name in other functions. This way
you can declare all init functions as `static __init__()` in all the files
without worry about function already defined stuff.

### constexpr

This keyword from c23 standard wants to fix constant defines from `#define`'s.
As you will notice `define` allow you substitue any name with any value,
without no security check as overflow or invalid values.

constexpr allow define a typed variable that can be evaluated at compile time.

#### Syntaxis

``` c
constexpr size_t len = 10;
```

This prevents define len with a value that size_t cant handle, like a negative
value or a huge num.


#gnu

## Intro
Notas sobre gnu C por Hugo Coto.

## Índice
1. [Expresiones condicionales](#Expresiones\ condicionales)
2. [Potenciación de números hexadecimales](#Potenciación\ de\ números\ hexadecimales)
3. [Puntero a entero](#Puntero\ a\ entero)
4. [Offset en una estructura](#Offset\ en\ una\ estructura)
5. [Estructuras y uniones sin nombre](#Estructuras\ y\ uniones\ sin\ nombre)
6. [Enums](#Enums)
7. [Switch-case](#Switch-case)
8. [Local labels](#Local\ labels)
9. [Acceder al valor de un label](#Acceder\ al\ valor\ de\ un\ label)
10. [auto_type](#auto_type)
11. [static](#static)
12. [extern](#extern)
13. [volatile](#volatile)
14. [Calificadores de tipo en arrays](#Calificadores\ de\ tipo\ en\ arrays)
15. [Restrict](#Restrict)
16. [Inline](#Inline)
17. [Search path](#Search\ path)
18. [Concatenación](#Concatenación)
19. [GNU Macros ya definidos](#GNU\ Macros\ ya\ definidos)
20. [Floating point](#Floating\ point)
21. [Bit flags](#Bit\ flags)

## Expresiones condicionales
En un condicional del tipo `cond? iftrue: else`, si la condicion y el valor `iftrue` coinciden se puede escribir de la siguiente manera: `iftrue? : else`.

## Potenciación de números hexadecimales
En un número hexadecimal de la forma `0x...`, añadiéndole `pN` como sufijo multiplica el numero original por `2^N`.
Al igual que funciona `e` en números decimales.
```c
0x100p-8 // 0x1
0x1p4 // 0x10
```

## Puntero a entero
En gnu C se permite hacer typecast a punteros y convertirlos en enteros, usando los tipos `uintptr_t` y `intptr_t`. Se recomienda no usarlo a menos que sea necesario.

## Offset en una estructura
Para obtener el offset de un campo dentro de una estructura en gnu existe el macro `offsetof(type, field)`.

## Estructuras y uniones sin nombre
Definamos la siguiente estructura con una unión sin nombre:
```c
struct _foo
{
    int a;
    union {
        float f;
        int   i;
        }
} foo;
```
Se puede acceder a los campos de la unión como si perteneciesen a la estructura principal. `foo.f` y `foo.i` son válidos.

## Enums
Los enums son ints por default. Se puede cambiar el tipo (en el estandard `C23`) de la siguiente forma:
```c
enum foo: long{
...
};
```
Los elementos de un enum definen su valor default sumando `1` al valor entero del campo anterior, siendo `0` el primer valor predeterminado. Se puede modificar el valor al definir el enum:
```c
enum foo:{
 a = 1;
 b; // 2
 c = 5;
}
```


## Switch-case
En un switch los case pueden referirse a un valor constante `case 1:` o a un rango de valores `case 1 ... 5`, **ambos incluidos**.

## Local labels
Se pueden declarar labels para usarlos com goto de manera local, de tal manera que solo se puedan usar desde un mismo bloque.
```c
{
__label__ label1;
...
}
```

## Acceder al valor de un label
Se puede obtener el valor de un label usando el operador unario `&&` y es de tipo puntero.
```c
void* labelptr = &&label1;
```
Se puede usar en un goto de la siguiente manera:
```c
goto *labelptr;
```

## auto_type
En un macro se puede copiar el tipo de un parametro utilizando \_\_auto_type.
```c
#define max(a, b)({   \
__auto_type _a = (a); \
__auto_type _b = (b); \
_a > _b ? _a : _b     \
})
```

## static
La utilización del keyword `static` antes de la declaración de una variable local hace que se almacene de forma global en ese archivo aunque fuese declarada dentro de una función.
```c
int count()
{
    static int counter = 0;
    return counter++;
}
```
En este ejemplo cada llamada a count devuelve un numero distinto. La declaración no se ejecuta en cada llamada a la función, sino una unica vez.

`static` antes de la declaración de una función hace que solo sea accesible desde el mismo archivo, y otras funciones estáticas pueden ser definidas en otros archivos con el mismo nombre.

## extern
El keyword `extern` declara una variable o funcion que ya fue o será definida en otro sitio. Al declarar usando extern no se reserva espacio por lo que se puede omitir información como la longitud de un array.

## volatile
El keyword `volatile` hace que se vuelva a obtener el valor de la variable cada vez que se necesita acceder a su valor. Es util si se va a acceder al valor de esa variable de manera asincrona.

## Calificadores de tipo en arrays
En una llamada a una funcion, la posición de los calificadores importa.
- `volatile int array[20]`: el array es equivalente a un puntero `volatile int *`.
- `int array[const 20]`: el array es equivalente a un puntero `const int *`.
- `const int array[20]`: el array es equivalente a un puntero de tipo `const int`.

## Restrict
El keyword `restrict` como argumento de una funcion asegura que el valor al que apunta un puntero solo puede ser modificado a traves de ese puntero. Permite al compilador optimizar el codigo reduciendo las lecturas de ese valor.

## Inline
El keyword inline permite al compilador optimizar llamadas a funciones de una linea.
```c
inline int add(int a, int b)
{
    return a + b;
}
```


## Search path
- *libdir/gcc/target/version/include*
- */usr/local/include*
- *libdir/gcc/target/version/include-fixed*
- *libdir/target/include*
- */usr/include/target*
- */usr/include*

## Concatenación
En macros, se pueden unir dos tokens usando `##`. Utilizando un único `#` se convierte en string.
```c
#define COM(c){#c, c ## _func}
```
Este macro crea un par que contiene el nombre del parametro como string y el resultado de añadirle \_func al parametro.

Cuando se utiliza `##` entre una `,` y un argumento, añade la coma y el argumento si el argumento existe, sino elimina ambos.
```c
#define eprintf(format, ...) \
    fprintf(std_err, format, ##__VA_ARGS__)
```

## GNU Macros ya definidos
- `__FILE__`
- `__LINE__`
- `__func__` (c standard)
- `__FUNCTION__`
- `__PRETTY_FUNCTION__`: Equivalente a `__FUNCTION__` en C.
- `__DATE__`: feb 12 1999
- `__TIME__`: 10:03:20
- `__TIME_STAMP__`: feb 12 10:03:20 1999
- `__COUNTER__`: Devuelve un valor unico empezando en 0.

## Floating point
Valores especiales:
- `+infinity`: más grande que cualquier valor.
- `-infinity`: más pequeño que cualquier valor.
- `QNaN`, `SNaN`: NaN silencioso y NaN de señal (necesita `-fsignaling-nans`). `SNaN` causa una señal de interrupción.

Se puede saber si un valor es NaN de las siguientes maneras:
```c
if (a != a)
    ...
```
```c
if (isnan(a))
    ...
```

Algunos macros importantes:
- `FLT_MIN`
- `DBL_MIN`
- `LDBL_MIN`: Defines the minimum normalized positive floating-point values that can be represented with the type.
- `FLT_HAS_SUBNORM`
- `DBL_HAS_SUBNORM`
- `LDBL_HAS_SUBNORM`: Defines if the floating-point type supports subnormal (or “denormalized”) numbers or not.
- `FLT_TRUE_MIN`
- `DBL_TRUE_MIN`
- `LDBL_TRUE_MIN`
- `FLT_MAX`
- `DBL_MAX`
- `LDBL_MAX`: Defines the largest values that can be represented with the type.
- `FLT_DECIMAL_DIG`
- `DBL_DECIMAL_DIG`
- `LDBL_DECIMAL_DIG`: Defines the number of decimal digits n such that any floating-point number that can be represented in the type can be rounded to a floating-point number with n decimal digits, and back again, without losing any precision of the value.


## Bit flags
Uniendo todas las opciones con valor booleano en una unica variable se ahorra espacio y se hace mas facil de utilizar. Cada opción utiliza un bit de la variable.

#### Declaración con enum
```c
enum OPTIONS {
    NONE = 1 << 0,
    A    = 1 << 1,
    B    = 1 << 2,
    C    = 1 << 3,
};
```

#### Añadir opciones
```c
enum OPTIONS opt;
opt |= A;
opt |= A | B;
```
Se pueden añadir opciones usando el operador binario `&`. Para añadir varias opciones a la vez se pueden combinar usando el operador `|`.

#### Eliminar opciones
```c
opt &= ~(A);
opt &= ~(A | B);
```
Se añade el inverso se la opcion que se quiere eliminar

#### Comprobar una opción
```c
if (opt & A)
    // A is set

if (opt & (A | B))
    // A and B are set
```
