# Functional Programming: an Introduction

Welcome to the first of what could (if there's enough interest) be a few posts on functional programming and its application to our day-to-day work as software engineers. Whether you're working in mobile, web, or backend services - it is important to understand the current landscape of development languages, programming paradigms and how many platforms are slowly migrating to a multi-paradigm language architecture.

This post is designed for entry-to-intermediate level programmers who have heard bits and pieces about functional programming, but were most likely scared off due to the confusing and misleading terminology used. The truth of the matter is that contrary to first experiences FP is not a difficult concept, so long as you understand the underpinning fundamentals and look at how they can be applied. We'll complicate things in later posts, but I want us to start off on the right foot: wine and dine you a little bit, maybe see a nice movie, before heading back to your place to talk about _Monads_ and _Functors_.

Since the inception of languages like C++, Java and Objective-C, the majority of developers slowly migrated from procedural programming to a programming paradigm known in modern terms as Object-Oriented Programming (OOP). Developers of old moved away from the concept of mere structs and began creating classes of objects, and defining _methods_ for those objects. These methods could not only change the state of that object, but could be called on the object itself!

OOP allowed us to go from:

```C
struct Object {
    // ...
};

void manipulateObjectWithData(Object *object, Data *data) { ... }

struct Object object1;
manipulateObjectWithData(&object1, &someData);
```

to

```java
public class SomeObject {
    // ...
    public void manipulateWithData(Data data) { ... }
}

SomeObject object = new SomeObject();
object.manipulateWithData(new Data());
```

Doing this not only let us clean up our code, but it also created a bunch of handy techniques and concepts we regularly use today such as abstraction, encapsulation, inheritance, interfaces and information hiding, just to name a few.

It may not be obvious to the untrained eye, but having everything chained to the Object has a long list of weird drawbacks. Not to mention OOP begins to seriously deviate from the foundations programming was built on: mathematics, to be specific!

Functional programming, by definition, challenges some of these concepts in interesting ways, and takes advantage of mechanisms that didn't really exist in Java and C++'s glory days (unless you count Erlang which was decades ahead of its time). To really get into this, without popping heads, we need to take a step back and visit some core concepts.

Typically a lot of introduction articles around FP immediately jump into concepts like function composition - which is incredibly powerful, don't get me wrong. I feel like without an understanding of how to appreciate concepts like pure functions before learning operations on functions some of the charm is lost, like learning to add before knowing what numbers are. Forgive me if the concepts expressed here are "too basic" but I want to provide some groundwork to get people to really understand the concepts at play here.

## What is a function

The short answer: _"A function is a relation or expression mapping one set of input variables to an output."_

In high school mathematics we are introduced to algebra: a once-thought contrived and confusing system where the numbers we were so used to calculating with were replaced with letters and our collective heads exploded onto the desk.

Or maybe that was just me - I was a pretty dumb kid.

Eventually we were introduced to a familiar syntax: `f(x) = x` or even `f(x) = ax^2 + bx + c`. Hopefully most of you by now are understanding what I'm getting at.

The first statement's syntax roughly translates to the following: "A function, f, with some input, x, will output the evaluation of the expression, f." So `f(2)` will output 2.

### Why am I bringing up ~old nightmares~ high school maths

Something your high school math teacher probably didn't explicitly teach you, but will almost certainly be common sense at this point, are something called the _Axioms of Algebra_. These axioms in essence define the bounds of what defines something called a Pure Function.

The Axioms of Algebra are roughly as follows:

The first two axioms are going to make some immediate sense in the realm of programming:

+ Identical expressions are reflexive: `2 = 2`, `f(2) = f(2)`. No matter how many times I evaluate those expressions, that statement cannot be invalidated.

+ Expressions should be symmetric on evaluation: if `a = b` then `b = a`, `f(2) = 4` then `4 = f(2)`.

The next three may be a bit weird when defined in programmatic terms, but we will revisit this later:

+ Expressions should be transitive: if `a = b` and `b = c`, then `a = c` must be true. If `A(x)` equals `B(x)`, and `B(x)` equals `C(x)`, then `A(x)` must equal `C(x)`.

+ Symmetric variables should be additively equivalent: When `a = b` and `c = d` then `a + c = b + d` must be true. If `A(x)` and `B(x)` produce the same output, and `C(x)` and `D(x)` produce the same output, then the composition of A and C, and the composition of B and D should produce the same output given the same input: `A >>> C ~ B >>> D`

+ Equivalent variables should be multiplicatively equivalent: `a = b, c = d, then ac = bd`.

This seems like a bit of a tangent to programming, but it's really important to understand these rules because they most likely define the evidence that the thing you wrote earlier today and called a function probably isn't a function.

## Methods vs Functions

I gave a bit of information away earlier regarding this but to really concrete the difference I want to give an example of a basic method.

```javascript
class Person {

    constructor(name) {
        this.name = name;
    }

    greeting() {
        console.log(`Hello there, ${this.name}!`);
    }
}
```

`greeting` is a method that takes no parameters, and returns a string. If I create multiple Person objects with different names, the output string will be different between objects of type Person. In the scope of the method called alone, without any context of the object it's being called on, it seems a little weird that when I supply no input to this method that I get a different output. In essence, all the Axioms mentioned earlier have been broken.

Let's corrupt this example further:

```javascript
class Person {

    constructor(name) {
        this.name = name;
        this.timesRead = 0;
    }

    greeting() {
        this.timesRead++;
        console.log(`Hello there, ${this.name}! You have been greeted ${this.timesRead} times now.`);
    }

}
```

Now if I call `greeting` with the same Person object, I will get a different output every time. Just looking at the generated interface it isn't clear that it should do this! What happened?

## Death, Taxes and Side Effects

A side effect is basically what the name suggests, something unexpected running in the execution of a function or method.

In the previous example:

```javascript
greeting() {
    this.timesRead++;
    console.log(`Hello there, ${this.name}! You have been greeted ${this.timesRead} times now.`);
}
```

There are three side effects here. Two of which are probably (without prior knowledge) more obvious than the other.

The first two side effects are the same: the updating and/or fetching of an external (class-bound) property to affect the execution of the method. To some extent this is an inherent property of methods, but it's also made worse that one of the properties is being mutated.

The third side-effect is the "output". My method specifies no output, so why do I see stuff in the console? I asked for no return! Help!

Functional programming is a paradigm that, avoiding the use of class-bound methods (and classes entirely for that matter), seeks to eliminate things like side-effects to produce understandable, side-effect free, reusable code.

## Pure Functions

A Pure Function, obnoxiously named, is a function that returns some output based on its inputs _without observable side-effects_. This inherently means that it isn't object-bound, and doesn't do things like:

+ Modifying external state
+ Reference external state (eg: no calls to `this` or `self`)
+ Produces no notifications
+ Calls no delegates
+ Doesn't call `println` or `console.log`

So how would we rewrite the above code, while avoiding all of these pre-conditions?

For the initial greeting:

```swift
func greeting(for name: String) -> String {
    return "Hello there, \(name)!"
}

print(greeting(for: "Jesse")) // Note how we compose functions together!
```

Simple enough, but what about the `timesRead` part? I still need that!

```swift
struct Person {
    let name: String
    let timesRead: Int = 0
} // Note the immutability here! I can't change anything without returning a new data type!

typealias Greeting = String

func createGreeting(for name: String) -> Greeting {
    return "Hello there, \(name)!"
}

func greet(_ person: Person, with greeting: (String) -> Greeting) -> (Person, Greeting) {
    return (
        Person(name: person.name, timesRead: person.timesRead + 1),
        greeting(for: person.name)
    )
}

let (person, greeting) = greet(Person(name: "Jesse"), with: createGreeting)
print(greeting)
```

Note the following about that second code sample:

+ No side-effects! Everything executed in the context of `greet` is defined from other, smaller components that we've composed.
+ We never mutate our Person object. Instead of using a class and modifying an object's properties by reference, we return another struct that uses copied members.
+ We used a function as a parameter. What's up with that? (more on this shortly)
+ This code is very modular: if we wanted to, we could created a different type of greeting function that took any String and return a new greeting.

```swift
func createADifferentGreeting(for name: String) -> Greeting {
    return "Hey \(name), what's good?"
}
```

Bonus for those who are super on-point (_pointfree_, rather wink wink nudge nudge): we can make this even fancier if we can provide consistent formatting!

```swift
func greeter(using greeting: String) -> (String) -> (Greeting) {
    return { name in
        [greeting, name].joined(separator: ", ")
    }
}

print(greeter("Hello there")("Jesse"))
```

(Having a format string would also work here, as long as you specify that in some docs.)

## First-Class Functions (Lambdas & Closures)

Functional and (usually) multi-paradigm programming languages have a unique feature that stands them out from other traditional procedural or Object-Oriented languages in that they support the functionality to store and use functions as variables. This is best illustrated in Javascript, where types are not explicit.

```javascript
let something = "This is a string";
something = 0.5; // Now it's a Float?
something = x => { return `Now it's a method that returns some input: ${x}`; };

console.log(something("whoa!"));
```

In Javascript we call this syntax an _Arrow Function_, named after the operator that separates the input with the implementation. This is not the only way to represent functions as objects in Javascript, partially due to the fact that functions have for the most part _always_ been "first-class".

Here's another function-as-stored-member representation in Javascript:

```javascript
const something = function(x) {
    return `Now it's a method that returns some input: ${x}`;
};
```

These anonymous (as yet technically unnamed) functions stored to properties are referred to as _lambdas_ or _closures_. The difference between the two (without breaking out the hat with the propeller on it) is that a closure can reference external members while a lambda cannot. If someone ever tests you on this they are not good people, since most people outside the world of Python don't care what you call it 99% of the time. A good consideration to easily tell them apart is that closures can be used as freeform blocks, whereas lambdas are more restrictive but more readily available in most programming languages.

C# uses its version lambda syntax for lots of things nowadays, most notably LINQ queries.

```c#
var names = ["Jesse", "Todisco"];
var nameStartingWithT = names
    .First(name => name.FirstOrDefault()?.ToUpper() == "T"); // Should be Nullable<string>
```

Swift's syntax for closures looks like this.

```swift
let something: (LosslessStringConvertible) -> String = { x in
    return "Yeah okay mate, another function that returns \(x) - very original."
}
```

Kotlin is basically the same.

```kotlin
val something: (Any) -> String = { x ->
    return "Something something value here $x"
}
```

Rust is on a similar train, but with weirder syntax.
(Value referencing syntax is probably not valid. Trained professioal at work here - always validate your code before posting, kids.)

```rs
let something = |i: &fmt::Display| -> String {
    return format!("Seriously, Jesse - your obsession with Rust has to stop. {}", i);
}
```

Note these languages typically need to specify input or return types if the compiler cannot infer them. Sometimes you can get away without supplying types at all.

## Higher-Order Functions

In languages where functions are treated as first-class, an opportunity opens up to let you supply functions as arguments to other functions, have functions return functions or have both!

In a lot of modern development in Javascript, Swift, Kotlin and Rust it is pretty common to see these in the context of asynchronous function callbacks.

```swift
func getAccounts(
    for user: User, 
    completion: @escaping ([Account]) -> Void
) { 
    // ...
    completion(accounts)
}
```

This roughly translates to "go get me some accounts, I'm going to go do other things, I'm not expecting a return immediately. When you _do_ finish getting accounts, call this function (completion) for me.

This is a super convenient syntax that replaces the delegate and observer classes from our old school OOP days, lets us define our implementation in place, or reference another function with the same signature!

```swift
getAccounts(for: user, completion: { accounts in
    print("I have \(accounts.count), neat!")
})

// or

func handleAccountsResponse(_ accounts: [Account]) {
    print("I have \(accounts.count), neat!")
}

getAccounts(for: user, completion: handleAccountsResponse)
```

I'm avoiding the nuances of last parameter closure syntax, escaping closures and probable memory leaks here for the sake of simplicity. Swift is a silly language.

Functions that return functions are a bit more complex. When utilised correctly, however, it can be an extremely powerful tool. Information hiding or switching implementations at runtime can be achieved easily.

```javascript
function sortMethodForType(type) {
    let sortLogic = (x, y) => { return true; };
    if (type === "ascending") {
        sortLogic = (x, y) => { return x < y; };
    } else if (type === "descending") {
        sortLogic = (x, y) => { return x > y; };
    }
    return sortLogic;
} // JSHint isn't screaming at me so I think I'm good?
```

This method could, hypothetically be passed to a function that takes a function parameter to compare two elements in an array for the purpose of sorting said elements. A function is used to produce a function that can be passed to a function: kind of a tongue twister, but if you can wrap your head around this you can use it to write some really nice and reusable code!

This concept can recurse as well! You can have a function that returns a function that returns a function that returns a function that (repeat ad nauseam). You can also have a function that takes functions as parameters that take functions as parameters that... (repeat ad nauseam as well)!

## Map, Filter, Reduce

This is the part where I discuss elements of functional programming you're almost definitely using without realising you are. Map, filter and reduce are three of the most powerful tools in a multi-paradigm developer's arsenal, and while sometimes these tools go by different names (LINQ uses weird ones) they are frequently used to cleanly handle collections.

### Map

Say you wanted to convert an integer to a string. Typically you would do something like

```swift
let number: Int = 1
let numberString = String(number)
```

What if you wanted to use that on an array of integers? One kinda gross way is

```swift
let numbers: [Int] = [1, 2, 3, 4, 5]
var numbersAsStrings: [String] = []

for number in numbers {
    numbersAsStrings.append(String(number))
}

// numbersAsStrings is now ["1", "2", "3", "4", "5"]. Neat... or is it?
```

I feel like the use of a mutable variable for storing this transformation is a bit clunky. As suggested before, we should try and reduce any mutability wherever possible - it typically leads to cleaner, less buggy code.

What we've basically done here is taken an input array and converted each element into a string and added it to the new array. You could say that we've _mapped_ the array of integers to an array of strings.

Let's pull up the Swift definition for the standard map function. I'm going to simplify the standard definiton signature in Swift's standard library for demonstration purposes. Throwing functions tends to pollute the waters a bit.

```swift

func map<T>(_ transform: (Element) -> T) -> [T]

```

Good enough!

This version of map is a class-bound higher-order method that applies a transformation to every element of a collection, then returns the new array. The method _does not_ mutate the existing array, `self`, but rather returns a new collection.

How does it work?

```swift
let numbers: [Int] = [1, 2, 3, 4, 5]

let numbersAsStrings = numbers.map { (number: Int) in
    return String(number)
}
```

That's a bit nicer, wouldn't you say? Swift infers that the Element in this case is of type Int, and so I don't need to discretely declare it. One-line closures also typically don't need a return statement, either. In fact, combining the previous two points with the face that String has an initialiser function specifically for integers we can crunch this whole thing down into two lines!

```swift
let numbers = [1, 2, 3, 4, 5]
let numbersAsStrings = numbers.map(String.init)

// or

let numberAsStrings = [1, 2, 3, 4, 5].map(String.init)
```

### Filter

Say we wanted to iterate through our collection and only keep even numbers. How would we do that? Well one way would be

```swift
let numbers = [1, 2, 3, 4, 5]
var filteredNumbers: [Int] = []

for number in numbers where number % 2 == 0 {
    filteredNumbers.append(number)
}
```

If it wasn't obvious based on the last section: yes, there is a filter method:

```swift

// simplified syntax for demonstration purposes
func filter(_ isIncluded: (Element) -> Bool) -> [Element]

```

(Fun trivia question for the Swift fans: map is a method of Collection, but filter is a method of Sequence. Do you know why this is the case?)

This method is applied to some sequence of elements. For each element in the sequence, execute the input function. If the function returns true, append a copy of the element to the output array.

(Fun trivia question for budding Swift enthusiasts: in Swift, is the copy appended to the new output array a reference copy or a value copy? Say `Element` was an object of type `Any`, if I modified a property on an element of the input array that wasn't filtered out after filtering does the output copy get affected too?)

Using our newfound method, we can simplify our code:

```swift
let numbers = [1, 2, 3, 4, 5]
let filteredNumbers = numbers.filter { number in number % 2 == 0 }

// or

let filteredNumbers = numbers.filter { $0.isMultiple(of: 2) }

// or, if you're feeling fancy

let valueIsMultipleOfMultiplier: (Int) -> (Int) -> Bool { multiplier in
    return { value in
        return value.isMultiple(of: multiplier)
    }
}
let filteredNumbers.filter(valueIsMultipleOfMultiplier(2))
```

### Reduce (or Fold)

This one is tricky at first, since it's super versatile. Reduce collapses elements of a sequence into a derivative value. It can sum integers in an array, it can create a dictionary tabulating recurring values in an array. Reduce can also be used to flatten arrays - take an array of arrays and collapse their elements into a flat array of those elements.

When summing elements in an array, the rookie way would be

```swift
let numbers = [1, 2, 3, 4, 5]
var total = 0

for number in numbers {
    total = total + number
}
```

The signature (simplified) for reduce is as follows:

```swift

func reduce<Result>(
    _ initialResult: Result, 
    _ nextPartialResult: (Result, Self.Element) -> Result
) -> Result

```

Don't be too intimidated by this signature, it's pretty simple. The first parameter is the value you want to start with; in our previous case this would be 0. If you were flattening an array, you would just start with an empty array. The second parameter is a function that takes that result declared earlier, an element from the sequence and wants you to return the new result.

Still confused? Here's an example.

```swift
let numbers = [1, 2, 3, 4, 5]
let sumOfValues = numbers.reduce(0, { result, newValue in
    return result + newValue
}) // outputs 15
```

Start with 0, then for each element in the sequence update the result and return it. Simple!

Swift can do really cool things with operators, and has functions defined for some basic operators specifically for reduce usage. This probably looks weird if you haven't seen it in the wild, but this is 100% valid syntax and works in Swift.

```swift
let numbers = [1, 2, 3, 4, 5]
let sumOfValues = numbers.reduce(0, +)
// also outputs 15!
```

Another nice thing about these methods is that they can be chained together to create some powerful effects.

```swift

let scoresCollection = [[1, 3, 4],[7, 2, 4], [6, 0, 3]]
let scoreQualifies: (Int) -> Bool = { score in
    return score > 8
}
let totalScoresPerElement = scoresCollection
.map { playerScore in
    return playerScore.reduce(0, +)
}
.filter(scoreQualifies)

```

Different languages have additional flavours of these three methods, and some languages even rename them. `flatten`, `flatMap`, `compactMap`, `firstWhere` are versions of chained functions combining the above three methods with some syntactic sugar.

You may notice the examples I'm giving are methods and _not functions_. Unfortunately, most multi-paradigm languages don't give an unbound (pure function) version of these out of the box, but it's very easy to write our own!

For example:

```swift

public func map<A, B>(_ f: @escaping (A) -> B) -> ([A]) -> [B] {
  return { $0.map(f) }
}

```

Having these operators in this "unbound" pure format gives us even more power than through class-bound methods. I'll talk about this in a later post when we get into slightly more advanced use of our newfound love for functions.

(Haskell calls the pure function version of reduce `fold`)

## Application: Functional Declarative UI Programming in iOS

At WWDC this year Apple released the new Swift declarative UI framework for Apple platforms called SwiftUI. It added neat declarative syntax for the creation of views, abstracted away UIKit/AppKit platform specifics that didn't belong, added state-based updating and custom Domain-Specific Languages.

One thing SwiftUI didn't support, to the disappointment of few, was functional declarative equivalents to the mutator methods on `View` structs.

Based on previous topics here they did do a lot of things right though:

+ Mutators return a copy of the view they're mutating, instead of mutating self - in UIKit and AppKit this is always a bit of a hack, but it produces really clean and descriptive code
+ Eliminate arbitrary imperative object declaration for sub-views
+ Support state-based reactive updates out of the box, courtesy of Apple's new Combine framework

Depending on adoption over the next few years, as well as Apple's commitment to improving the framework over time, this is a massive game changer.

We can still do similarly powerful things with view manipulation using _functions_, no subclasses, no extensions. Yes, it is actually possible.

Let's say we wanted to create a button that has rounded edges. In typical code one would go with either creating a sub-class of `UIButton` called `RoundedButton`, or maybe more recently creating an extension on `UIButton` that could mutate `self` to inherit the desired properties.

```swift
class RoundedButton: UIButton {
  override init(frame: CGRect) {
    super.init(frame: frame)
    self.clipsToBounds = true
    self.layer.cornerRadius = 6
  }

  required init?(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }
}

let roundedButton = RoundedButton(frame: .zero)
```

```swift
extension UIButton {
  static var rounded: UIButton {
    let button = UIButton.init(frame: .zero)
    button.clipsToBounds = true
    button.layer.cornerRadius = 4
    return button
  }
}

let roundedButton = UIButton.rounded
```

These approaches totally work _in this scope_, but make extensibility a bit complicated.

Imagine we did a similar thing for a button that was filled in with a red background, we could create similar implementations.

```swift
class FilledButton: UIButton {
  override init(frame: CGRect) {
    super.init(frame: frame)
    backgroundColor = UIColor.red
  }

  required init?(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }
}

let filledButton = FilledButton(frame: .zero)
```

```swift
extension UIButton {
  static var filled: UIButton {
    let button = UIButton.init(frame: .zero)
    backgroundColor = UIColor.red
    return button
  }
}

let filledButton = UIButton.filled
```

Okay, lot of duplicate code here already, and two classes (or extensions) need to be created. This approach still has a massive accessibility issue: what if I want a filled, rounded button?

We've already written all the code to do this in a couple places now, but we cannot combine these in any meaningful way without either adding more extensions or subclasses, or creating some parent abstraction that can work with multiple customisations. The latter approach mentioned here requires a lot of redundant abstraction, and may not seem fit for purpose at first.

What if I could show you a method in which you can write minimal code, throw polymorphism out the window and still get reusable mutators for these classes of objects?

Here we go:

```swift
func fillStyler(with colour: UIColor) -> (UIView) -> Void {
    return { view in
        view.backgroundColor = colour
    }
}

func roundedButtonStyler(withCornerRadius radius: CGFloat) -> (UIView) -> Void {
    return { view in
        view.layer.cornerRadius = radius
    }
}

// The above methods can produce functions that return their input type to be chainable, but let's try something more advanced

func sequence<A>(
    _ f: @escaping (inout A) -> Void,
    _ g: @escaping (inout A) -> Void
) -> ((inout A) -> Void) {
    return { a in
        f(&a)
        g(&a)
    }
}

let button = UIButton(frame: .zero)
let stylesToApply = sequence(fillStyler(with: UIColor.red), roundedButtonStyler(4.0))
stylesToApply(button)

// Button is now filled _and_ rounded!
```

## Closing

Using functions in clever ways eliminates some bloat invoked by redundant abstraction in OOP while also providing clean, reusable code. It's definitely not an obvious step when writing your code, especially when you've been raised in an Object-oriented world, but once you start appreciating the power of first-class functions, closures, lambdas, and the way your language already uses them you can do crazy-neat stuff.

In future posts I'll get into some of the more intermediate to complex parts of FP, like argument operators, function injection and composition. Until then, I would love to hear your feedback!
