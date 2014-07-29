##Diving into D
This is a short tutorial on D. This tutorial assumes you are
familiar with basic programming concepts as well as programming
language concepts.

#Installing D
We are going to be using dmd(The digital mars compiler) for
this tutorial. Please download it from dlang.org. It is
available for Mac, Windows and Linux. BSD people can get the
LLVM D compiler(ldc), it should work just fine for this 
tutorial.

If you're more comfortable using an IDE then I recommend
Code::Blocks which has really good D integration.
Installation instructions are here:
http://wiki.dlang.org/CodeBlocks

#Hello World

```
#!d
import std.stdio;

void main() {
    writeln("Hello World");
}
```

you can use writefln for formatted output.
```
#!d
writefln("%d %d %s", 5, 3, "I refuse to use foo");
```
#Types
http://dlang.org/type

D has several primitive types. It has all the basic ones we
know(int, long, float, etc.).

Unsigned types are prefixed with u(uint, ulong, etc.)
Complex types are prefixed with i(ireal, idouble, etc.)

D also has type inference.
```
#!d
auto a = 4;// int a = 4;
auto b = 10l; //long a = 10;
```
#Functions
http://dlang.org/function

D functions are similar to C functions:

```
#!d
int func(int a, double b, real c) {
    //Contents
}
```

Function and parameter types can be prefixed as well:

```
#!d
ref int func(in int a, out double b, ref real c) {
    //Contents
}
```
Ref implies the type is a reference type, i.e. changes inside
the functions will change the variable outside the function.
in means immutable
out is an alias for ref(preferably used for parameters)

#Structs
http://dlang.org/struct

The D reference summarizes them well enough already. If you are 
familiar with C structs then you can skim this since it's pretty 
much the same minus some minor semantic differences and some
syntactic sugar.

#Classes
http://dlang.org/class

The link above summarizes classes pretty well, I just have a
small thing to add to the above.

Access control can be done like this:

```
#!d
class A {
    public int x;
    private int y;
}
```
or

```
#!d
class B {
    public:
        int x;
    private:
        int y;
}
```

Whichever is your preference.

#Operator Overloading

There are two types of operators in D, unary which take one
operand such as ```a++```, and binary operators which take
two operands such as ```a + b```. There are also three types of
unary operators: normal(```++a```), index(```a[2]```) and 
slice(```a[5..10]```). As for binary operators, there are
mathematical operators(```a+b```, ```a-b```), comparison
operators (```a <= b```, ```a > b```) and equality operators
(```a == b```, ```a != b```).

In order to overload a normal unitary operator, override the
opUnary template in the class. For example:

```
#!d
struct A {
    int m;

    int opUnary(string operator)() if(operator == "++") {
        //Overloads the ++ operator
        // A a; ++a; calls this function for object a.
    }
}

struct B {
    int m;

    int opUnary(string operator)() if(operator == "-") {
        //Overloads the - operator
        //A a; -a; calls this function for object a.
    }
}
```

Note: You cannot directly overload the ```a++``` operator,
instead the compiler translates it to an expression in terms
of ```++a```. For example:
```
#!d
int b = a++;
```
translates into
```
#!d
int b = a;
++a;
```

In order to overload index operators, you need to overload the
opIndex template in the class you want.

For example:
```
#!d
struct A {
    int m;

    int opIndex(int i) if(operator == "++") {
        //Overloads the [] operator
        // A a; a[3]; calls this function for object a with
        // i as 3.
        // Notice the return type can be different than A.
    }
}

struct B {
    int m;

    int opIndex(int i, int j, int k) if(operator == "++") {
        //Overloads the [] operator
        // A a; a[3, 4, 5]; calls this function for object a with
        // i as 3, j as 4 and k as 5.
        // Notice the return type can be different than A.
    }
}
```

You can also override unary operators on index operator. I
don't know when you would ever need to do something like that,
but you can do it in any case.

```
#!d
struct A {
    int m;

    int opIndexUnary(string operator)(int i) if(operator == "-") {
        //Overloads the -a[] operator.
        // A a; -a[i] calls this operator.
    }
}
```

In order to override slice operator, you need to override the
opSlice function.

```
#!d
struct A {
    int m;

    int opSlice(int i, int j) {
        //Overloads the a[i..j] operator.
        //A a; a[1..4] calls this function for object A.
    }
}

```

Use the same pattern as opIndexUnary to overload opSliceUnary
if you want to overload a unary operator on a slice such as
```++a[4..10]```

Binary operators, unlike normal unary operators take a parameter
which is the right hand side, so ```a+b``` translates to 
```a.opBinary!("+")(b)```.

If you want to overload a mathematical binary operator, you need 
to overload the opBinary template. If you want to overload
the equality operator, you need to overload opEquals and for
comparison operators you need to overload opCmp

```
#!d
Struct A {
    A opBinary(string operator)(const A rhs) if(operator == "+"){
        //Called when an object of type A is added to another 
        //object of type A
        //rhs can have multiple types, just overload opBinary
    }

    bool opEquals(A lhs, A rhs) {
        //Example of call:
        //A a;
        //A b;
        // writeln(a == b);
        // writeln(a != b);
    }

    int opCmp(A lhs, A rhs) {
        // Example of call
        // A a;
        // A b;
        // writeln(a < b);
    }
}
```

TODO: Add opAssign documentation.

#Templates
Templates are D's response to Java generics and C++ templates.
Basically templates are metaclasses that take types as
parameters. Templates can contain classes, methods or variables.

For example let's take a List template.

```
#!d
template List(T) {
    class ArrayList {
        T[] l;
    }

    class LinkedList {
        T l;
        T* next;
    }
}
```

Now suppose I want to create an instance of an ArrayList
containing integers;

```
#!d
alias List!int.ArrayList IntList;
IntList list = new IntList();
```

#Template Mixins

There is only one difference between mixins and regular 
templates, everything inside a mixin exists in the parent
scope, while regular templates have their own scope.

For example:

```
#!d
mixin template Temp(){
    int x;
}

struct A {
    mixin Temp;
}

int main() {
    A a;
    a.x = 5;
}
```

#Arrays
Arrays in D are pretty much like C++ or Java arrays.
```
#!d
int[] a = new int[4000]; //Dynamic array
int[20] b; //static array
```

You can slice an array as well:
```
#!d
a[x..y] 
```
is an array containing a[x] all the way to a[y]
The array is not copied, it is a reference so you can write
functions that work on whole arrays without having to worry
about left and right. For example in the partition step of
quicksort you need to give it the lower bound and upper bound.

```
#!d
void partition(int[] arr, int lower, int upper) {
    //Partition the array between lower and upper
}
void main() {
    int[] a = new int[5000];
    partition(a, 10, 40);
}
```
which forces you to ensure that you are only working on the
part of the array defined by lower and upper.

With slicing you can do the following:
```
#!d
void partition(int[] arr) {
    //Partition the entire array
    assert(arr.length == 30);
}
void main() {
    int[] a = new int[5000];
    partition(a[10..40]);
}
```
Which allows D to perform the bounds checking and leads to much
more robust code.

#Functional Programming
http://dlang.org/phobos/std\_functional.html
Now for one of the more amazing things about D.
D has functional features like currying and piping, though
not part of the language itself. They are built entirely from
templates which are part of the standard library.

```
#!d
import std.functional;

int func(int a, int b){return a+b;}
alias curry!(func, 5) func5;

int main() {
    writeln(func5(4)); //9
}
```

A particular template of interest is memoize, which memorizes
the return values given a set of parameters, which is awesome
for dynamic programming.

As an example: trivially, factorial is written as follows 
recursively:
```
#!d
int factorial(int x) {
    return (x < 2) ? 1:x*factorial(x-1);
}

void main() {
    factorial(5) // 5 steps
    factorial(10) // 10 steps
}
```
While with memoize:
```
#!d
import std.functional;


int fact(int x) {
    alias memoize!(fact) factorial;
    return (x < 2) ? 1:x*factorial(x-1);
}


void main() {
    factorial(5) // 5 steps
    factorial(10) // 5 steps
}
```
