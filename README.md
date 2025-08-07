# TOY Language Compiler

A simple functional programming language compiler implemented in Racket as part of a Programming Languages course (PL CS5400).

## Overview

TOY is a minimalist functional language that supports:
- First-class functions with lexical scoping
- Local variable bindings
- Conditional expressions
- Primitive arithmetic and comparison operations
- Function application and closures

## Language Syntax

The TOY language uses a Lisp-like S-expression syntax:

```racket
<TOY> ::= <num>                              ; Numbers
        | <id>                               ; Identifiers  
        | { bind {{ <id> <TOY> } ... } <TOY> } ; Local bindings
        | { fun { <id> ... } <TOY> }         ; Function definitions
        | { if <TOY> <TOY> <TOY> }           ; Conditionals
        | { <TOY> <TOY> ... }                ; Function calls
```

## Examples

### Basic arithmetic
```racket
{+ 2 3}                    ; => 5
{* {+ 1 2} 4}             ; => 12
```

### Function definitions and calls
```racket
{{fun {x} {+ x 1}} 4}     ; => 5 (anonymous function)
```

### Local bindings
```racket
{bind {{x 5}} {+ x 3}}    ; => 8

{bind {{add3 {fun {x} {+ x 3}}}} 
  {add3 10}}              ; => 13
```

### Closures and lexical scoping
```racket
{bind {{x 3}}
  {bind {{f {fun {y} {+ x y}}}}
    {bind {{x 5}}
      {f 4}}}}            ; => 7 (uses x=3, not x=5)
```

### Higher-order functions
```racket
{bind {{identity {fun {x} x}}
       {add1 {fun {x} {+ x 1}}}}
  {{identity add1} 5}}    ; => 6
```

### Conditionals
```racket
{if {< 4 5} 10 20}        ; => 10
{if false 10 20}          ; => 20
```

## Built-in Operations

### Arithmetic
- `+`, `-`, `*`, `/` - Basic arithmetic operations
- `<`, `>`, `=` - Comparison operations

### Boolean values
- `true` - Boolean true
- `false` - Boolean false

## Architecture

The interpreter consists of several key components:

### 1. **Parser** (`parse-sexpr`, `parse`)
Converts S-expressions into an Abstract Syntax Tree (AST) using pattern matching.

### 2. **AST Definition** (`TOY` datatype)
```racket
(define-type TOY
  [Num  Number]           ; Numeric literals
  [Id   Symbol]           ; Variable references
  [Bind (Listof Symbol) (Listof TOY) TOY]  ; Local bindings
  [Fun  (Listof Symbol) TOY]               ; Function definitions
  [Call TOY (Listof TOY)]                  ; Function calls
  [If   TOY TOY TOY])                      ; Conditionals
```

### 3. **Values** (`VAL` datatype)
```racket
(define-type VAL
  [RktV  Any]                    ; Racket values (numbers, booleans)
  [FunV  (Listof Symbol) TOY ENV] ; User-defined functions (closures)
  [PrimV ((Listof VAL) -> VAL)])  ; Built-in primitive functions
```

### 4. **Environment** (`ENV` datatype)
Implements lexical scoping using a frame-based environment structure:
```racket
(define-type ENV
  [EmptyEnv]              ; Empty environment
  [FrameEnv FRAME ENV])   ; Frame with parent environment
```

### 5. **Evaluator** (`eval`)
Recursively evaluates AST nodes in the given environment, handling:
- Variable lookup with lexical scoping
- Function closures that capture their definition environment
- Proper argument binding and scoping rules

## Key Features

### Lexical Scoping
Functions capture their definition environment, enabling proper closure behavior:

```racket
{bind {{make-adder {fun {n} {fun {x} {+ x n}}}}}
  {bind {{add5 {make-adder 5}}}
    {add5 10}}}           ; => 15
```

### First-Class Functions
Functions can be passed as arguments and returned as values:

```racket
{bind {{apply-twice {fun {f x} {f {f x}}}}}
  {apply-twice {fun {x} {+ x 1}} 5}}  ; => 7
```

### Error Handling
The interpreter provides meaningful error messages for:
- Syntax errors in parsing
- Unbound variable references  
- Type errors in primitive operations
- Arity mismatches in function calls

## Usage

```racket
(run "{+ 2 3}")                    ; Basic evaluation
(run "{bind {{x 5}} {+ x 3}}")     ; With bindings
```

## Testing

The implementation includes comprehensive tests covering:
- Basic arithmetic and function calls
- Lexical scoping and closures
- Error conditions and edge cases
- Complex nested expressions

## Implementation Notes

- **Type Safety**: Uses Racket's type system with explicit type annotations
- **Immutable Data**: All values and environments are immutable
- **Functional Style**: Pure functional implementation without side effects
- **Extensible Design**: Easy to add new primitive operations or language constructs

## Educational Purpose

This interpreter demonstrates fundamental concepts in programming language design:
- Recursive descent parsing
- Environment-based evaluation
- Closure implementation
- Type system design
- Error handling strategies

Perfect for understanding how functional programming languages work under the hood!