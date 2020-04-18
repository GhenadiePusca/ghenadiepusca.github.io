---
date: 2020-04-18 16:13
description: The begining of Structure and Interpretation of Computer Programs book. We will investigate what are expressions and how those differ from statements. After that we'll built a custom functional component to provide the functionality of if expression, by this we'll stress the capability of swift to handle functional concepts.
---

# If expression in Swift [SICP P1]


This blogpost is a part of going through the Structure and Interpretation of Computer Programs book, you can get a free copy from:  

[https://web.mit.edu/alexmv/6.037/sicp.pdf](https://web.mit.edu/alexmv/6.037/sicp.pdf)  

A while ago, I’ve heard about functional programming. 
Being in the pursuit of improving my skills as a programmer, all of the functional methodologies and rules, seem to be of a great value. 
Thus, I embark in a journey of learning functional programming principles, by trying to implement all of them in Swift programming language.
Myself, being an iOS developer, I think it will be of a great value to try and implement the FP principles in Swift. 
This will allow to get more knowledge about the language itself, and how suitable it is for applying functional programming.
Also, I most probably will retain and use most of FP principles, if implemented in Swift rather in Haskell or LISP(I suggest for you to do the same in your favorite/working language). 
In my experience I found that even though I do learn new things in some other languages, those will be forgotten with time if not exercised in Swift, even though those things do have a great value.  

## Today we will discuss

- The beginning of the SICP book
- If statement in swift
- Ternary operator in swift
- Expressions vs statements
- If statement vs ternary operator
- Wrapping ternary operator in a custom functional component


## The beginning of the SICP book

As you start reading the first pages of the SICP book, you’ll see that the authors takes you through the simplest means of a programming language. 
This first part is simple and pretty straightforward, besides having a lot of parentheses(but if you wrote at least some code in Objective C, no big deal :)). 
The interesting part is the mention of expressions, like almost everything is an expression, even Conditional logic, like “if” and “cond”(aka switch).
Today I want to focus on “if” expression, and through it get a better knowledge of what is an expression.   

By the book, “if” expression is defined as:  

(if ⟨predicate⟩ ⟨consequent⟩ ⟨alternative⟩)  

To evaluate an if expression, the interpreter starts by evaluating the ⟨predicate⟩ part of the expression. If the ⟨predicate⟩ evaluates to a true value, the interpreter then evaluates the ⟨consequent⟩ and returns its value. Otherwise it evaluates the ⟨alternative⟩ and returns its value.
And the most important part is that “if” - returns a value. We’ll start exploring what options do we have in Swift language, what is the difference between an “if” returning a value and the one that does not, and what are the applications.  

## If statement in swift  

If statement in swift is no different than in other languages, XCode docs says:

 Changes which path your code takes

And this already implies that if statement in swift does not return a value, we can say that the if statement in Swift always performs some side effect. Thus we can define it as:

```swift
if <predicate> {
 <then side effect>
} else {
 <else side effect>
}
```
Where side effect can be anything - from changing the value of a local variable, to performing IO operations. What if we do want to return a value instead of performing a side effect? And actually most of the time we want to return a value from the if statement, rather than performing a side effect. Well, Swift(and many other languages) defines the ternary operator as shorthand for the if-else statement, but it has one more important property - it does return a value.
Ternary operator

As in many other languages ternary operator in swift is defined as:

```swift
<predicate> ? <consequent> : <alternative>
```

Note that it is very similar to the `if` expression defined in the SICP book:

```swift
(if ⟨predicate⟩ ⟨consequent⟩ ⟨alternative⟩)
```

Ternary operator has two important properties:
It always returns a value.
<consequent> and <alternative> should have same type.
Ternary operator still can perform side effects, in this case the return type will be Void. You may wonder then - Why do we have if statement and also ternary operator? Can we have just one? In many, if not all, functional languages `if` can do both.  

## If statement vs ternary operator  

### If statement:
- Does not return a value, always performs side effects
- Else branch is optional
- Can add multiple branches else-if branches

### Ternary operator:
- Always returns a value, in case of side effects returns Void
- Both, true and false branches should return the same type
- False branch(alternative) cannot be omitted.
- Multiple operators can be nested, right associativity is applied:  

```swift
    a > b ? 1 : a < b ? -1 : 1
    Equivalent of:
    a > b ? 1 : (a < b ? -1 : 1)
```  

As we see, if statement and ternary operator cannot easily substitute one another,
as with ternary operator the else branch cannot be omitted and nesting is quite clumsy. What if we could have an `if` which will have the benefits of both?
Expressions vs statements

`if` defined in swift is a statement, while `if` defined in the SICP book, and in many other functional programming, is an expression. Almost all of the Functional Languages have `if` as an expression, thus there is no need ternary operator defined.

It appears that most of the functional languages are expression-oriented, and it is somewhat correct to say that most of the imperative languages are statement-oriented. Everything has to do with side effects, almost in any definition of FP you will find that functional programming avoids as hell side effects, while in imperative programming side effects are everywhere(sure you can avoid this, but the language itself does not constrain you). 

Expressions always return a value, but statements do not return a value and are used to perform side effects.  

## Creating a swift if expression  

Let’s take a simple example, and start creating our own if expression.   

```swift
func example(a: Int, b: Int) -> Int {
 let multiplier: Int
 if a > b {
  multiplier = a - b
 } else {
  multiplier = b - a
 }
 
 return a * multiplier
}
```  

This is a pretty standard example of using an if statement - when in function body we need to perform some additional calculations. As you can see if statement is having side effects on multiplier. We can refactor this to use ternary operator as such:  

```swift
func example(a: Int, b: Int) -> Int {
 let multiplier = a > b ? a - b : b - a
 return a * multiplier
}
```  

And even more:  

```swift
func example(a: Int, b: Int) -> Int {
 return a * (a > b ? a - b : b - a)
}
```  

Since ternary operator enforces same type for true/false branches, swift can safely infer the type.  
Now from here on, we do want to create our own if expression to be used:  

```swift
func If(_ predicate: Bool,
        then consequent: Int,
        else alternative: Int) -> Int {
 predicate ? consequent : alternative
}
```  

And we can use it:  

```swift
func example(a: Int, b: Int) -> Int {
 return a * If(a > b, then: a - b, else: b - a)
}
```  

I’d say it looks better than ternary operator. But this implementation does not mirror completely the functionality of the ternary operator, ternary operator has short circuit evaluation - false branch will not be evaluated if the condition is true and vice versa. Our If expression currently does not have this behaviour, we can achieve this by delaying the execution using closures:  

```swift
func If(_ predicate: Bool,
        then consequent: () -> Int, 
        else alternative: () -> Int) -> Int {
 predicate ? consequent() : alternative()
}

func example(a: Int, b: Int) -> Int {
 return a * If(a > b, then: { a - b }, else: { b - a })
}
```  

Better, but having to specify the curly braces again and again can be tedious. We can use autoclosure feature to let the compiler generate the closure for us:  

```swift
func If(_ predicate: Bool,
        then consequent: @autoclosure () -> Int,
        else alternative: @autoclosure () -> Int) -> Int {
 predicate ? consequent() : alternative()
}

func example(a: Int, b: Int) -> Int {
 return a * If(a > b, then: a - b, else: b - a)
}
```  

Next step is to make it work not only for Ints, after all we will not program only calculators :):  

```swift
func If<T>(_ predicate: Bool,
           then consequent: @autoclosure () -> T,
           else alternative: @autoclosure () -> T) -> T {
 predicate ? consequent() : alternative()
}
```  

All good, we have defined the ternary operator in terms of a function. As mentioned earlier, the else statement was mandatory in ternary operator, we can define If function overload that will not require an else branch:  


```swift
func If<T>(_ predicate: Bool,
           then consequent: @autoclosure () -> T) -> T? {
 If(predicate, then: consequent(), else: .none)
}
```  

You can still use the If expression to perform only side effects, let’s improve our expression to have the result as discardable if not needed, which is the case when doing side effects:  

```swift
@discardableResult
func If<T>(_ predicate: Bool,
           then consequent: @autoclosure () -> T,
           else alternative: @autoclosure () -> T) -> T {
 predicate ? consequent() : alternative()
}

@discardableResult
func If<T>(_ predicate: Bool,
           then consequent: @autoclosure () -> T) -> T? {
 If(predicate, then: consequent(), else: .none)
}
```  

That is, we have defined an if expression that has the best from the if statement and ternary operator.  




</br>
</br>

## Conclusion  
</br>
</br>

This was the start of exploring functional programming through SICP book. The main idea was to play and explore how well functional principles do apply in Swift. The first 'alien' encounter is that much of everything in FP(at least for the language in SICP) is an expression. We saw that Swift provides an expression like `if` under the ternary operator, however it has its shortcomings - chaining multiple ternary operators look ugly and inability to have only the true branch executed. By comparing if statement to the ternary operator we have concluded what are the main differences between an expression and a statement. In the last part a straightforward functional implementation for the if expression was done. With this implementation we have explored how easy it is in swift to create a general purpose functional component, and as far as I can tell it looks promising.
Moving forward with the knowledge acquired, we will have to build other general purpose functional component such as cond(aka switch), this will be explored in the next post.
Having those components in place, we will be able to conquer the examples and exercises from the book in a functional manner.
Till next time!
