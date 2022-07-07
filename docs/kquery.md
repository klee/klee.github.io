---
layout: default
title: KQuery
subtitle: The reference manual for the KQuery language
slug: documentation
---

{:.toc}
Contents
{:.toc__title .no_toc}
* Table of contents placeholder
{:.toc__list .list-anchor}
{:toc}

## Introduction

The KQuery language is the textual representation of constraint expressions and queries which is used as input to the Kleaver constraint solver. 

Currently the language is capable of representing quantifier free formulas over bitvectors and arrays, with direct support for all standard operations on bitvectors. The language has been designed to be compact and easy to read and write. 

The KQuery language is closely related to the C++ API for Exprs, see also the doxygen [Expr]({{site.baseurl}}/doxygen/html/classklee_1_1Expr.html) documentation.

## Notation

In this document, syntax is given in Extended Backus-Naur Form and appears as:

{% highlight ebnf %}
"(" "Eq" [ type ] LHS RHS ")"
{% endhighlight %}

Unless noted, the rules are described in terms of tokens not characters, and tokens can be separate by white space and comments.

In some case, a production like `child-expression` is used as an alias for the `expression` production, when subsequent text needs to differentiate the expression.

Examples are shown using:

```
(Eq w32 a b)
```

## Structure

A KQuery source file consists of a sequence of declarations.

**Syntax:**

{% highlight ebnf %}
kquery = { array-declaration | query-command }
{% endhighlight %}

Currently, the language supports two kinds of declarations:

  * _Array Declarations_: Use to declare an array of bitvectors for use in subsequent expressions.
  * _Query Commands_: Used to define queries which should be executed by the constraint solver. A query consists of a set of constraints (assumptions), a query expression, and optionally expressions and arrays to compute values for if the query expression is invalid.

Comments begin with "#" and continue until the end of line. For example:

```
(Add w32 1 1) # Two, hopefully
```

### Expression and Version Labels

Expressions are frequently shared among constraints and query expressions. In order to keep the output succinct and readable, expression labels can be used to introduce a lexical binding which can be used in subsequent expressions.  Expression labels are globally scoped through the entire source file, and a definition must preceed any use in the source file.

**Syntax:**

{% highlight ebnf %}
expression = identifier ":" expression  
{% endhighlight %}

Likewise, versions are frequently shared among reads and can be labelled in the same fashion.

**Examples:**

```
(Add w32 N0:(Add w32 1 1) N0) # Four  
```

```
array const_array[] : w32 -> w8 = [5,6]  
(Read w8 0 U0:[0=255] @ const_array) # U0 now refers to an array [255,6]  
(Read w8 1 U0) # Read from byte offset 1 of [255,6]  
```

## Literals

### Identifiers

Identifiers are used for specifying array names and for expression labels.

**Syntax:**

{% highlight ebnf %}
identifier = "[a-zA-Z_][a-zA-Z0-9._]*"  
{% endhighlight %}

**Examples:**

```
_foo  
arr10_20  
```

Note that in order to keep open the possibility to introduce explicit integral and floating-point types, the following identifiers are treated as reserved keywords:

```
floating-point-type = "fp[0-9]+([.].*)?"  
integer-type = "i[0-9]+"  
```

### Numbers

Numeric constants can be specified as follows.

**Syntax:**

{% highlight ebnf %}
number = "true" | "false" | signed-constant  
signed-constant = [ "+" | "-" ] ( dec-constant | bin-constant | oct-constant |
hex-constant )  
dec-constant = "[0-9_]+"  
bin-constant = "0b[01_]+"  
oct-constant = "0o[0-7_]+"  
hex-constant = "0x[0-9a-fA-F_]+"  
{% endhighlight %}

**Examples:**

```
false  
-10  
0b1000_0001 # 129  
```

Non-decimal constants can be signed. The "_" character is ignored when evaluating constants, but is available for use as a separator.

### Types

Types are explicit operands to most expressions, and indicate the bit-width of the type.

**Syntax:**

{% highlight ebnf %}
type = "w[0-9]+"  
{% endhighlight %}

**Example:**

```
w32  
```

The numeric portion of the token is taken to be a decimal integer specifying the bit-width of the type.

## Declarations

### Arrays

Arrays are the basic type for defining symbolic variables (the language does not currently support simple variables).

**Syntax:**

{% highlight ebnf %}
array-declaration = "array" name "[" [ size ] "]" ":" domain "->" range "="
array-initializer  
array-initializer = "symbolic" | "[" number-list "]"  
number-list = number | number "," number-list  
{% endhighlight %}

Arrays can be initialized to be either symbolic, or to have a given list of constant values. For constant arrays, the initializer list must exactly match the size of the array (if the size was unspecified, it will be the number of constant values).

**Examples:**

```
array foo[10] : w32 -> w8 = symbolic # A ten element symbolic array  
array foo[] : w8 -> w1 = [ true, false, false, true ] # A constant array of
four booleans  
```

### Query Commands

Query declarations describe the queries that the constraint solver should run, along with optional additional arguments to specify expressions and arrays for which counterexamples should be provided.

**Syntax:**

{% highlight ebnf %}
query-command = "(" "query" constraint-list query-expression [ eval-expr-list
[ eval-array-list ] ] ")"  
query-expression = expression  
constraint-list = "[" { expression } "]"  
eval-expr-list = "[" { expression } "]"  
eval-array-list = "[" { identifier } "]"  
{% endhighlight %}

**Examples:**

```
(query [] false)  
(query [(Eq w8 (Read w8 0 mem) 10)] false [] [ mem ])  
```

A query command consists a query, consisting of a constraint list and a query expression, and two optional lists for use when a counterexample is desired.

The `constraint-list` is a list of expressions (with boolean type) which are assumed to hold. Although not required in the language, many solvers require that this set of constraints be consistent. The `query-expression` is the expression to determine the validity of.

If a counterexample is desired for invalid queries, `eval-expr-list` is a list of expressions for which a possible value should be constructed, and `eval- array-list` is a list of arrays for which values for the entire array should be provided. All counterexamples results must be simultaneously feasible.

## Versions

Versions are used to refer to an array with an ordered sequence of writes to it.

**Syntax:**

{% highlight ebnf %}
version = identifier | "[" [ update-list ] "]" "@" version
update-list = lhs-expression "=" rhs-expression [ "," update-list ]
{% endhighlight %}

**Examples:**

```
array small_array[2] : w32 -> w8 = symbolic # The array we will read from  

(Read w8 0 small_array) # No Updates to small_array
(Read w8 1 [1=0xff] @ small_array) # Read from small_array at byte offset 1
with update where byte 1 set to decimal 255
```

A version can be specified either by an identifier, which can refer to an array or a labelled version, or by an explicit list of writes which are to be concatenated to another version (the most recent writes are first).

## Expressions

Expressions are strongly typed, and have the following general form:

{% highlight ebnf %}
"(" EXPR_NAME EXPR_TYPE ... arguments ... ")"
{% endhighlight %}

where `EXPR_NAME` is the expression name, `EXPR_TYPE` is the expression type (which may be optional), followed by any additional arguments.

### Primitive Expressions

#### Expression References

An expression reference can be used to refer to a previously labelled expression.

**Syntax:**

{% highlight ebnf %}
expression = identifier
{% endhighlight %}

Expression and version labels are in separate namespaces, it is the users responsibility to use separate labels to preserve readability.

#### Constants

Constants are specified by a numeric token or a type and numeric token.

**Syntax:**

{% highlight ebnf %}
expression = number | "(" type number ")"
{% endhighlight %}

When a constant is specified without a type, the resulting expression is only well-formed if its type can be inferred from the enclosing context. The `true` and `false` constants always have type `w1`.

**Examples:**

```
true
(w32 0)
(Add w32 10 20) # The type for 10 and 20 is inferred to be w32.
```

### Arithmetic Operations

#### Add, Sub, Mul, UDiv, SDiv, URem, SRem

**Syntax:**

{% highlight ebnf %}
arithmetic-expr-kind = ( "Add" | "Sub" | "Mul" | "UDiv" | "URem" | "SDiv" |
"SRem" )  
expression = "(" arithmetic-expr-kind type expression expression ")"  
{% endhighlight %}

Arithmetic operations are always binary and the types of the left- and right- hand side expressions must match the expression type.

#### UDiv

Truncated unsigned division. Undefined if divisor is 0.

#### URem

Unsigned remainder. Undefined if divisor is 0.

#### SDiv

Signed division. Undefined if divisor is 0.

#### SRem

Signed remainder. Undefined if divisor is 0. Sign of the remainder is the same
as that of the dividend.

### Bitwise Operations

#### Not

**Syntax:**

{% highlight ebnf %}
expression = "(" "Not" [ type ] expression ")"
{% endhighlight %}

Bitwise negation. The result is the bitwise negation (one's complement) of the input expression. If the type is specified, it must match the expression type.

#### And, Or, Xor, Shl, LShr, AShr

**Syntax:**

{% highlight ebnf %}
bitwise-expr-kind = ( "And" | "Or" | "Xor" | "Shl" | "LShr" | "AShr" )  
expression = "(" bitwise-expr-kind type expression expression ")"  
{% endhighlight %}

These bitwise operations are always binary and the types of the left- and right-hand side expressions must match the expression type.

#### Shl

{% highlight ebnf %}
expression = "(" "Shl" type X Y ")"
{% endhighlight %}

Logical shift left. Moves each bit of `X` to the left by `Y` positions. The `Y` right-most bits of `X` are replaced with zero, and the left-most bits discarded.

#### LShr

{% highlight ebnf %}
expression = "(" "LShr" type X Y ")"
{% endhighlight %}

Logical shift right. Moves each bit of `X` to the right by `Y` positions. The `Y` left-most bits of `X` are replaced with zero, and the right-most bits discarded.

#### AShr

{% highlight ebnf %}
expression = "(" "AShr" type X Y ")"
{% endhighlight %}

Arithmetic shift right. Behaves as `LShr` except that the left-most bits of `X` copy the initial left-most bit (the sign bit) of `X`.

### Comparisons

#### Eq, Ne, Ult, Ule, Ugt, Uge, Slt, Sle, Sgt, Sge

**Syntax:**

{% highlight ebnf %}
comparison-expr-kind = ( "Eq" | "Ne" | "Ult" | "Ule" | "Ugt" | "Uge" | "Slt" |
"Sle" | "Sgt" | "Sge" )  
expression = "(" comparison-expr-kind [ type ] expression expression ")"  
{% endhighlight %}

Comparison operations are always binary and the types of the left- and right- hand side expression must match. If the type is specified, it must be `w1`.

### Bitvector Manipulation

#### Concat

**Syntax:**

{% highlight ebnf %}
expression = "(" "Concat" [type] msb-expression lsb-expression ")"
{% endhighlight %}

_Concat_ evaluates to a `type` bits formed by concatenating `lsb-expression` to `msb-expression`.

#### Extract

**Syntax:**

{% highlight ebnf %}
expression = "(" "Extract" type offset-number child-expression ")"
{% endhighlight %}

_Extract_ evaluates to `type` bits from `child-expression` taken from `offset-number`, where `offset-number` is the index of the least-significant bit in `child-expression` which should be extracted. 

#### ZExt

**Syntax:**

{% highlight ebnf %}
expression = "(" "ZExt" type child-expression ")"
{% endhighlight %}

_ZExt_ evaluates to the lowest `type` bits of `child-expression`, with undefined bits set to zero.

#### SExt

**Syntax:**

{% highlight ebnf %}
expression = "(" "SExt" type input-expression ")"
{% endhighlight %}

_SExt_ evaluates to the lowest `type` bits of `child-expression`, with undefined bits set to the most-significant bit of `input-expression`.

### Special Expressions

#### Read

**Syntax:**

{% highlight ebnf %}
expression = "(" "Read" type index-expression version ")"  
{% endhighlight %}

The _Read_ expression evaluates to the first write in `version` for which `index-expression` is equivalent to the index in the write. The type of the expression must match the range of the root array in `version`, and the type of `index-expression` must match the domain.

#### Select

**Syntax:**

{% highlight ebnf %}
expression = "(" "Select" type cond-expression true-expression false-
expression ")"  
{% endhighlight %}

The _Select_ expression evalues to `true-expression` if the condition evaluates to true, and to `false-expression` if the condition evaluates to false. The `cond-expression` must have type `w1`.

Both the true and false expressions must be well-formed, regardless of the condition expression. In particular, it is not legal for one of the expressions to cause a division-by-zero during evaluation, even if the _Select_ expression will never evaluate to that expression.

### Macro Expressions

Several common expressions are not implemented directly in the Expr library, but can be expressed in terms of other operations. A number of these are implemented as "macros". The pretty printer recognizes and prints the appropriate Expr forms as the macro, and the parser recognizes them and turns them into the underlying representation.

#### Neg

**Syntax:**

{% highlight ebnf %}
expression = "(" "Neg" [ type ] expression ")"
{% endhighlight %}

This macro form can be used to generate a **Sub** from zero.

#### ReadLSB, ReadMSB

**Syntax:**

{% highlight ebnf %}
expression = "(" "ReadLSB" type index-expression version ")"  
expression = "(" "ReadMSB" type index-expression version ")"  
{% endhighlight %}

_ReadLSB_ and _ReadMSB_ can be used to simplify contiguous array accesses. The type of the expression must be a multiple `N` of the array range type. The expression expands to a concatenation of `N` read expressions, where each read is done at a subsequent offset from the `index-expression`. For _ReadLSB_ (_ReadMSB_), the concatenation is done such that the read at `index-expression` forms the least- (most-) significant bits.
