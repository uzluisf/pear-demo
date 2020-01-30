---
title: "To be called as a subroutine or as a method? That's the question"
author: "Luis F. Uceta"
date: 01-07-2020
template: page
draft: true
---

In Raku, the general term for a code object with its own lexical scope and
return handling is [routine](https://docs.raku.org/type/Routine). While not all
routines are subs and/or methods, all subs and/or methods are routines. 
A routine can be both a subroutine and a method in which case you call it using
the subroutine call syntax and method call syntax respectively. For instance:

```raku
say "Hello"; # subroutine call syntay
"Hello".say; # method call syntax
```

Nonetheless, even when a routine is only a subroutine (or only a method) you can
still use it as a method (or a subroutine). All is needed is an extra bit of
syntax.

# From subroutine to method

Given a subroutine `f(arg1, arg2, arg3)`, you can invoke it as method on its
invocant with the methodop `.&` as in `arg1.f(arg2, arg3)`.  Here the invocant will be
bound to the first positional argument, namely `arg1` and the remaining
arguments are passed to the subroutine. For example:

```raku
'/home'.&dir; # dir '/home';
```

Note that `'/home'.dir` would've failed with the error `No such method 'dir' for
invocant of type 'Str'` but using `.&` allowed us to a syntax much akin to 
the regular method call syntax.

This works the same with user-defined subroutines. For instance:

```raku
sub multiply {
    [*] @_
}

2.&multiply;       # multiply 2;
2.&multiply(3);    # multiply 2, 3;
2.&multiply(3, 4); # multiply 2, 3, 4;
```

# From method to subroutine

We already know how to call a method on its object (i.e., `obj.method(args)`). 
To call a method as a subroutine, we use the syntax `method(obj: args)` (Notice
the colon after the invocant). Here the object becomes an explicit invocant
to the method. For instance:

```raku
base(255: 16); # 255.base(16); 
base 255: 16;  # same
```

## A detour

The syntax `method(obj: args)` is reminiscent of the fact that in Raku you can
name the invocant in a method something different instead of solely using `self`
which is what's passed implicitly as the first positional argument to each
method by default. For instance, let's consider the followung class:

```
class Point2D {
    has  $.x;
    has  $.y;
}
```

A method that finds the `distance` between two `Point2D` points could be
implemented as follows:

```
method distance( Point2D $B ) {
    sqrt ($!x - $B.x) ** 2 + ($!y - $B.y) ** 2
}
```

Here we're accesing the invocant's attributes (`$!x` and `$!y`) directly while
accesing the provided argument through its accessors (e.g., `$B.x`). However,
if we were to rely on both points's accesor methods, we could also declare
the `distance` method like this:

```
method distance( Point2D $B ) {
    # self is available inside any method and bound to the invocant.
    sqrt (self.x - $B.x) ** 2 + (self.y - $B.y) ** 2
}
```

Nonetheless, we could make this method more *explicit* by providing an explicit
invocant. This is done by specifying the invocant as a positional parameter
followed by a colon (`:`), then followed by the method's usual parameters if
any:

```
# get distance from point $A to point $B.
method distance( $A: Point2D $B ) {
    sqrt ($A.x - $B.x) ** 2 + ($A.y - $B.y) ** 2
}
```

Granted, either using `self` or an explicit invocant isn't that much different
but the latter option adds more clarity to the method when it's needed.

We could go a step further and constrain the type of invocant on which the method
ought to be called on. Thus, the method could be called on a type object (in
which case we have a class method) or on an object instance (in which case we
have an object method). For this, we must use the `::?CLASS` variable, placed in
front of the colon, which is combined with either `:U` (for class methods) or
`:D` (for object methods). It's worth noting this work with both implicit and
explicit invocants as shown below:

```
method distance( ::?CLASS:D $A: Point $B ) {
    sqrt ($A.x - $B.x) ** 2 + ($A.y - $B.y) ** 2
}

method dimensions( ::?CLASS:U : ) {
    2
}
```

**NOTE:** The whitespace around `::?CLASS`, an explicit invocant and `:` isn't
strictly necessary but it aids the readability of the method's signature.

Thus:

```
my Point2D $x .= new(:x(4), :y(6));
my Point2D $y .= new(:x(7), :y(10));

say $x.distance($y);      #=> 5
say Point2D.distance($y); # Error: X::Parameter::InvalidConcretenessInvocant

say $x.dimensions;      # Error: X::Parameter::InvalidConcretenessInvocant
say Point2D.dimensions; #=> 2
```

However, declaring a method as a class method or an object method isn't a
either-or situation in Raku. If you want a method to be both, just use the 
[`multi`](https://docs.raku.org/syntax/multi) declarator.

# Parentheses 

Unless required for disambiguation of nested calls, parentheses in subroutine
calls can be omitted:

```raku
#| Return a list of its arguments squared.
sub list-of-squares {
    slip @_.map(* ** 2)
}

say sum 1, list-of-squares 2, 3, 4, 5;  # 55 = 1 + 4 + 9 + 16 + 25
say sum 1, list-of-squares(2, 3), 4, 5; # 23 = 1 + 4 + 9 + 4 + 5
```

For the method call syntax, you can use the colon to replace the parentheses. 
For instance:

```
say 255.base(1 + 3 ** 2 + 6); 
say 255.base: 1 + 3 ** 2 + 6; 
```

In this context, the colon is used as precedence dropper and results in cleaner
and easier to read code, especially with methods that handle subroutines or
blocks as arguments such as `map`, `grep`, `sort`, etc. For instance:

```
my 
```

It's worth noting that the colon is an alternative for the method call syntax, not
an alternative for the subroutine call syntax. Thus, in

```
say: "Hello", "World"
```

the colon is used in a different context; it creates a [label]() `say`,
calling the function `say` with arguments. However, the `say` subroutine is
also defined as method so we could still the colon as a precedence dropper:

```
"Hello".say: "World", "!"
```

# Summary

* Use `.&` to call a subroutine much like a method as in `arg1.&sub(arg2, arg3, ...)`.
* Use the colon `:` to call a method as a subroutine as in `method(obj: arg1, arg2, ...)`.
* Parentheses can be omitted in subroutine calls unless to disambiguate nested calls. 
* In the method call syntax, parentheses can be replaced with a colon. This is
  particularly useful when passing a subroutine or a block to a method.
