# Replace ?: Ternary Operator with “if-then-else“ expression 

* Proposal: [SE-NNNN](https://github.com/apple/swift-evolution/blob/master/proposals/NNNN-name.md)
* Author(s): J. Cheyo Jimenez (https://github.com/masters3d)
* Status: **Review**
* Review manager: TBD

## Introduction

The ternary operator ?: serves as a control flow expression in Swift (as well as other c-based languages).  It’s use leads to very compact, hard to read  code that can be confused with Swift’s Optionals. 

Swift-evolution thread: [ternary operator ?: suggestion ](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151207/000810.html)

## Motivation

The main purpose is to replace the ternary operator with and stand alone if-then-else expression that will be distinct from the if-else statements. With this approach type inference can figure out what to initially conditionally assign a variable, with statements this is not possible. Statements can also consist of multiple statements, which is not possible with an expression.

##Reason for replacement:

* Eliminate the use of ”?”s that are not related to Swift Optionals
* Eliminate the use of the colon as an assignment operator
* Hard to read in code with lost of optionals
* Confusing to beginners ?:
* Bizarre C magic


## Proposed solution

If-then-else as a drop in replacement of the ternary operator (salve for the use keywords). 

#### Background:

Expressions and statements are inherently different. For example, in a statement it is perfectly fine to have no "else" clause but in an expression it must always have "else" to provide a result for the variable being assigned. Statements can consist of multiple lines of statements where expressions can not. Trying to make the existing statements into expressions will add complexity, clutter, harder to implement and will be confusing to users. Please see **Alternatives Considered** section for details. 

Other functional languages, such as Haskell, use the if...then...else construct and require the else part in the expression. Likewise the current ternary operator requires the "else" part after the colon.

#### Overview 

First the ternary "?:" operators are removed from the language, replaced with the if...then...else expression, (the language conversion tools would automate this process). For example:

``` haskell
  result = if condition then expression1 else expression2 
```  

where the if...then...else clause requires all three keywords. This will help to differentiate it from a statement. 

Note that any of these expressions can be surrounded by parenthesis to make clear that the expression returns a value but it is not required. 

``` haskell
  let color = (if stop then Color.red else Color.green)  
```

Nesting of expressions is supported providing "else if" like capabilities. 

``` haskell
  let color = if stop then (if !broken then Color.red else Color.blinkinRed) else 
              (if changing then Color.yellow else Color.green)  
```

### Compatibility with existing if statement

One objection is that now it become harder when teaching the language to tell people when to use "then". This is a valid concern but I would argue that the concept of the if statement in Swift has more than one meaning that happy coexist already when it comes to binding ``if let``, ``if case`` and ``if #available`` etc. 


## Detailed design

### Grammar:

#### If...then...else expression

``` haskell
   if condition then expressionUsedIfTrue else expressionUsedIfFalse 
```  

"else" is required in the if-then-else expression. 
```
 if-expression → if condition then expression else expression 
```

"if" expression results must be a compatible type of the result. 

for if statements an optional "then" keyword would be added to the grammar. 

## Impact on existing code

All places where the ternary operator is used would need to be replaced with if...then...else. The Convert To Latest Swift Syntax in Xcode could ease this transition in the mac platform. 


## Alternatives considered

### Why not extend existing statements to be usable as expressions?

The biggest question was why does Swift not already support the result of the "if" statement or "switch" as part of the existing statement? You could simply allow:

``` swift
let result = if conditional { X } else { Y }. 
```

The problem with this is that in expressions all cases must be handled. In a statement it is common to not have else. So you could make it an expression if the return value is used. This makes it more complex from both the user's perspective and the compiler perspective. That means that if you were to change a "if" conditional into an expression suddenly you would get an an error if the "else" part was not provided. 

Another problem is the proliferation of braces, expressions should be light, it looks cumbersome to have braces all through your expressions. It does not look as clean having braces in an expression and you would never use braces in a ternary expression. Further, it will help express the difference between an expression and a statement to only use braces for statement blocks. 

There is a clear distinction made between these two concepts in the existing language. *If making statements into expressions is the answer that everyone wants then this proposal should be declined*; however that is a much deeper change than what is being proposed here. 

Expressions are mathematical concepts where there are inputs and one return value. Statement blocks are lists of commands without any such guaranteed output or side effects. Functional programming uses expressions to avoid much of the state and side effects that are common with imperative programming. It is very likely it would encourage a bunch of imperative programming side effect code based code, if statements become expressions.  It will muddy the concept of expressions in people’s minds, trying to explain this to a student would be harder if the concepts are combined. I will say, I am not a huge fan of this idea, even if it is possible (although will try to keep an open mind).

Swift straddles the worlds of functional and imperative programming and a lot of functional programmers are drawn to the language because of it, keeping these concepts separate by having expressions and statements would help to keep the approaches separate in people’s minds. If you want to do the imperative approach use the if-else statement, if you want to do the functional approach use the if-then-else expressions. If they are combined then code that was written functionally might start getting imperative by the changes made by other developers on the team.

### Is it really better? Why not just keep ternary ?: expressions?

This is a valid question, there is an advantage in compactness to ternary expressions; however, I believe that the inconsistencies in the use of the question mark and the colon are alone grounds for its replacement, in addition to the lack readability. If Swift already had if-else expression then there would be no need to introduce an if-then-else expression. I believe that by adding a functional/ no side effects if-then-else expression to stand-in as a replacement for the ?: ternary operator, will strike a balance between a **functional** if expression and the current if statement. 

### What do other languages do?

Other languages such as F#, Haskell, OCaml, SML, Lua and Ruby have long ternary like operations that use the keyword “then”.

``` haskell
  result = if cond then "ABC" else "CDF"
```

Many of these approaches were explored in the thread, the "then" approach seemed superior to alternatives likes python’s middle conditional . 

``` python
result = "ABC" if cond else “CDF"
```

### Maybe keep both ternary and keywords?

This is a possibility as a transition point but I should be flagged for removal as a way to help people move their code to the new way. Keeping both will fracture the language. 

### Why add a keyword? 

A new keyword is required for this proposal: ”then” or a equivalent keyword / character 

We considered ways of not adding additional keywords. As it may be unclear when to use "then" or not, this is addressed in two ways. It is invalid to have "then" without "else" and  statements must be surrounded in braces as they can be multiple lines long. 

``` swift
  color = if conditional "string1" else "string2"  
```

This leads to syntactic ambiguity, by adding "then" that goes away. Other keywords could work but they are unlikely to be an improvement over "then". Another possibility to avoid taking another keyword: 

``` swift
  color = if conditional : "string1" else if conditional2 : “string2” else “string3”
```

The colon fixes the ambiguity but the colon is drifting into the ternary operator territory. This does not read as clearly as "then". 

``` haskell
  color = where conditional then "string1" else where conditional2 then “string2” else “string3” 
```

The use of where was also discussed to replaced the leading if but we found this to be more verbose ad confusing with he already establish notion of where in the language. 

``` swift
  color = if conditional do "string1" else if conditional2 do “string2” else “string3” 
```
If really avoiding another keyword is a problem, it was  suggested to use "do" but this does not work read as well, the notion of do is already formulated in the language as a block and there is no good history of using do as control flow (salve for the BCPL Language)

### Since "then" used a lot in libraries, we should be careful about taking it. 

This is a valid point. Alternatives are [ : ] [ -> ] [ do ]


### Feedback from the community

“ FWIW, I have no love for the ternary operator (it is totally "bizarre C magic”), but it does solve a problem that Swift currently otherwise has no solution for…” - Chris Lattner

“One thing that comes to mind is that if "if" is an expression, every if needs an else branch. This makes it harder to use if to conditionally perform side effects. ”  - Ole Begemann

“Rust has this feature (all statements are expressions), and it requires if statements to have an else branch with the same type unless the type is `()`. It's solution to the issue of the branches returning unwanted values is that Rust uses semicolons, and the semicolon acts sort of like an operator that consumes any value and returns `()`, so if you terminate the last statement of the branch with a semicolon, the whole branch returns `()`, and if you leave it off, the branch returns a value. It's actually very elegant and straightforward.That said, proposing that Swift introduce this same rule for semicolons is probably not a good idea…” -Kevin Ballard


“Once we have control flow expressions I would like to see the ternary operator removed from the language as it would no longer server a purpose.  Removing the ternary operator seems to fit nicely with the direction to remove some features that are carried over from C-based languages but don’t necessarily fit with the direction Swift is heading” - Matthew Johnson




“ … if A and B are intended to be *expressions* instead of an arbitrary sequence of statements|decls|exprs, then a more consistent syntax would be:

	let v = if condition (A) else (b) 

The immediate problem with that is that juxtaposition of two expressions (condition, and A [with or without parens]) will lead to immediate syntactic ambiguity.” - Chris Lattner




“FWIW, I (and many other people) would like to consider turning many statement-y things in swift into expressions.  I’d love to see the weird ?: ternary operator get nuked and replaced with an if/else expression of some sort.  This is an area that the apple team hasn’t had bandwidth to consider carefully.

That said, there are challenges here in the details.  How will the grammar work? Exactly which statements should be included (certainly if and switch, any others)?

Further, it is important to consider whether the code written using this will actually be *better* than the code written with these things as statements.  For example, the “switch” blocks tend to be very large, and turning them into expressions encourages additional indentation.  Swift already allows ‘let’ values to be initialized on multiple paths, so is the win actually that great?

Given that statements-as-expressions would provide another way to do things (they are a purely syntax extension) the barrier should high to add them.  They will add complexity and surface area to the language, so they need to pay that complexity.”- Chris Lattner



_____

