## 1.Introduction

Functional programming (FP) is based on a simple premise with far reaching implications: we construct our programs using
only pure functions —in other words, functions that have no side effects. What are side effects? A function has a side
effect if it does something other than simply return a result, for example:

- Modifying a variable
- Modifying a data structure in place
- Setting a field on an object
- Throwing an exception or halting with an error
- Printing to the console or reading user input

### 1.1. A program with side effects

Suppose we’re implementing a program to handle purchases at a coffee shop. We’ll begin with a Scala program that uses
side effects in its implementation (also called an impure program).

```class cafe {
def buyCoffee(cc: Creditcard): Coffee = {
 val cup = new Coffee()
 // Side effect as the charge can return 
 cc.charge(cup.price) 
 cup
}
} 
```

The line cc.charge(cup.price) is an example of a side effect. Charging a credit card involves some interaction with the
outside world—suppose it requires contacting the credit card company via some web service, authorizing the transaction,
charging the card, and (if successful) persisting some record of the transaction for later reference. But our function
merely returns a Coffee and these other actions are happening on the side, hence the term “side effect.”

### 1.2 A functional solution: removing the side effects

The functional solution is to eliminate side effects and have buyCoffee return the charge as a value in addition to
returning the Coffee. The concerns of processing the charge by sending it off to the credit card company, persisting a
record of it, and so on, will be handled elsewhere.

```
class Cafe {
    def buyCoffee(cc: CreditCard): (Coffee, Charge) = {
        val cup = new Coffee()
        (cup, Charge(cc, cup.price))
    }
}
```
### 1.3 Pure functions
- The function’s output depends only on its input variables
- It doesn’t mutate any hidden state
- It doesn’t have any “back doors”: It doesn’t read data from the outside world (including the console, web services, databases, files, etc.), or write data to the outside world

Given that definition of pure functions, as you might imagine, methods like these in the scala.math._ package are pure functions:
- abs
- ceil
- max
- min

These Scala String methods are also pure functions:
- isEmpty
- length
- substring

```
Here’s a pure function that calculates the sum of a list of integers (List[Int]):
def sum(list: List[Int]): Int = list match {
    case Nil => 0
    case head :: tail => head + sum(tail)
}
```

## 2.Monoid

### What is a monoid?
Let’s consider the algebra of string concatenation. We can add "foo" +
"bar" to get "foobar", and the empty string is an identity element for that
operation. That is, if we say (s + "") or ("" + s), the result is always s.
Furthermore, if we combine three strings by saying (r + s + t), the
operation is associative—it doesn’t matter whether we parenthesize it: ((r
+ s) + t) or (r + (s + t)).
  The exact same rules govern integer addition. It’s associative, since (x +
  y) + z is always equal to x + (y + z), and it has an identity element, 0,
  which “does nothing” when added to another integer. Ditto for
  multiplication, whose identity element is 1.
  The Boolean operators && and || are likewise associative, and they have
  identity elements true and false, respectively.
  These are just a few simple examples, but algebras like this are virtually
  everywhere. The term for this kind of algebra is monoid. The laws of
  associativity and identity are collectively called the monoid laws. A monoid
  consists of the following:
  Some type A
  An associative binary operation, op, that takes two values of type A and
  combines them into one: op(op(x,y), z) == op(x, op(y,z)) for
  any choice of x: A, y: A, z: A
  A value, zero: A, that is an identity for that operation: op(x, zero)
  == x and op(zero, x) == x for any x: A
  We can express this with a Scala trait:
```
trait Monoid[A] {
def op(a1: A, a2: A): A
def zero: A
}
```
An example instance of this trait is the String monoid:
```
val stringMonoid = new Monoid[String] {
   def op(a1: String, a2: String) = a1 + a2
   val zero = ""
}
```
List concatenation also forms a monoid:
```
def listMonoid[A] = new Monoid[List[A]] {
 def op(a1: List[A], a2: List[A]) = a1 ++ a2
 val zero = Nil
}
```

### Folding lists with monoids
```
def foldRight[B](z: B)(f: (A, B) => B): B
def foldLeft[B](z: B)(f: (B, A) => B): B
```
The components of a monoid fit these argument types like a glove. So if we had a list of Strings, we could simply pass the op and zero of the stringMonoid in order to reduce the list with the monoid and concatenate all the strings:
```
scala> val words = List("Hic", "Est", "Index")
words: List[String] = List(Hic, Est, Index)

scala> val s = words.foldRight(stringMonoid.zero)(stringMonoid.op)
s: String = "HicEstIndex"

scala> val t = words.foldLeft(stringMonoid.zero)(stringMonoid.op)
t: String = "HicEstIndex"
```

Note that it doesn’t matter if we choose foldLeft or foldRight when folding with a monoid;[3] we should get the same result. This is precisely because the laws of associativity and identity hold. A left fold associates operations to the left, whereas a right fold associates to the right, with the identity element on the left and right respectively:

Given that both foldLeft and foldRight have tail-recursive implementations.

```
words.foldLeft("")(_ + _) == (("" + "Hic") + "Est") + "Index"
words.foldRight("")(_ + _) == "Hic" + ("Est" + ("Index" + ""))
```
We can write a general function concatenate that folds a list with a monoid:
```
def concatenate[A](as: List[A], m: Monoid[A]): A =
  as.foldLeft(m.zero)(m.op
```

But what if our list has an element type that doesn’t have a Monoid instance? Well, we can always map over the list to turn it into a type that does:
```
def foldMap[A,B](as: List[A], m: Monoid[B])(f: A => B): B
```
### Associativity and parallelism
The fact that a monoid’s operation is associative means we can choose how we fold a data structure like a list. We’ve already seen that operations can be associated to the left or right to reduce a list sequentially with foldLeft or foldRight. But if we have a monoid, we can reduce a list using a balanced fold, which can be more efficient for some operations and also allows for parallelism.

As an example, suppose we have a sequence a, b, c, d that we’d like to reduce using some monoid. Folding to the right, the combination of a, b, c, and d would look like this:
```
op(a, op(b, op(c, d)))
Folding to the left would look like this:
op(op(op(a, b), c), d)
But a balanced fold looks like this:
op(op(a, b), op(c, d))
Note that the balanced fold allows for parallelism, because the two inner op calls are independent and can be run simultaneously. But beyond that, the more balanced tree structure can be more efficient in cases where the cost of each op is proportional to the size of its arguments. For instance, consider the runtime performance of this expression:
List("lorem", "ipsum", "dolor", "sit").foldLeft("")(_ + _)
At every step of the fold, we’re allocating the full intermediate String only to discard it and allocate a larger string in the next step. Recall that String values are immutable, and that evaluating a + b for strings a and b requires allocating a fresh character array and copying both a and b into this new array. It takes time proportional to a.length + b.length.

Here’s a trace of the preceding expression being evaluated:
List("lorem", ipsum", "dolor", "sit").foldLeft("")(_ + _)
List("ipsum", "dolor", "sit").foldLeft("lorem")(_ + _)
List("dolor", "sit").foldLeft("loremipsum")(_ + _)
List("sit").foldLeft("loremipsumdolor")(_ + _)
List().foldLeft("loremipsumdolorsit")(_ + _)
"loremipsumdolorsit"
```
### Example: Parallel parsing
As a nontrivial use case, let’s say that we wanted to count the number of words in a String. This is a fairly simple parsing problem. We could scan the string character by character, looking for whitespace and counting up the number of runs of consecutive nonwhitespace characters. Parsing sequentially like that, the parser state could be as simple as tracking whether the last character seen was a whitespace.

But imagine doing this not for just a short string, but an enormous text file, possibly too big to fit in memory on a single machine. It would be nice if we could work with chunks of the file in parallel. The strategy would be to split the file into manageable chunks, process several chunks in parallel, and then combine the results. In that case, the parser state needs to be slightly more complicated, and we need to be able to combine intermediate results regardless of whether the section we’re looking at is at the beginning, middle, or end of the file. In other words, we want the combining operation to be associative.

To keep things simple and concrete, let’s consider a short string and pretend it’s a large file:
```
"lorem ipsum dolor sit amet, "
```
If we split this string roughly in half, we might split it in the middle of a word. In the case of our string, that would yield "lorem ipsum do" and "lor sit amet, ". When we add up the results of counting the words in these strings, we want to avoid double-counting the word dolor. Clearly, just counting the words as an Int isn’t sufficient. We need to find a data structure that can handle partial results like the half words do and lor, and can track the complete words seen so far, like ipsum, sit, and amet

The partial result of the word count could be represented by an algebraic data type:
```
sealed trait WC
case class Stub(chars: String) extends WC
case class Part(lStub: String, words: Int, rStub: String) extends WC
```

A Stub is the simplest case, where we haven’t seen any complete words yet. But a Part keeps the number of complete words we’ve seen so far, in words. The value lStub holds any partial word we’ve seen to the left of those words, and rStub holds any partial word on the right.

For example, counting over the string "lorem ipsum do" would result in Part ("lorem", 1, "do") since there’s one certainly complete word, "ipsum". And since there’s no whitespace to the left of lorem or right of do, we can’t be sure if they’re complete words, so we don’t count them yet. Counting over "lor sit amet, " would result in Part("lor", 2, "").

#### MONOID HOMOMORPHISMS
If you have your law-discovering cap on while reading this chapter, you may notice that there’s a law that holds for some functions between monoids. Take the String concatenation monoid and the integer addition monoid. If you take the lengths of two strings and add them up, it’s the same as taking the length of the concatenation of those two strings:

```
"foo".length + "bar".length == ("foo" + "bar").length
```
Here, length is a function from String to Int that preserves the monoid structure. Such a function is called a monoid homomorphism.[6] A monoid homomorphism f between monoids M and N obeys the following general law for all values x and y:

```
M.op(f(x), f(y)) == f(N.op(x, y))
```

### Foldable data structures
When we’re writing code that needs to process data contained in one of these structures, we often don’t care about the shape of the structure (whether it’s a tree or a list), or whether it’s lazy or not, or provides efficient random access, and so forth.

For example, if we have a structure full of integers and want to calculate their sum, we can use foldRight:

```
ints.foldRight(0)(_ + _)
```
Looking at just this code snippet, we shouldn’t have to care about the type of ints. It could be a Vector, a Stream, or a List, or anything at all with a foldRight method. We can capture this commonality in a trait:

```
trait Foldable[F[_]] {
  def foldRight[A,B](as: F[A])(z: B)(f: (A,B) => B): B
  def foldLeft[A,B](as: F[A])(z: B)(f: (B,A) => B): B
  def foldMap[A,B](as: F[A])(f: A => B)(mb: Monoid[B]): B
  def concatenate[A](as: F[A])(m: Monoid[A]): A =
    foldLeft(as)(m.zero)(m.op)
}
```

Here we’re abstracting over a type constructor F, much like we did with the Parser type in the previous chapter. We write it as F[_], where the underscore indicates that F is not a type but a type constructor that takes one type argument. Just like functions that take other functions as arguments are called higher-order functions, something like Foldable is a higher-order type constructor or a higher-kinded type.

### Composing Monoids
The Monoid abstraction in itself is not all that compelling, and with the generalized foldMap it’s only slightly more interesting. The real power of monoids comes from the fact that they compose.

This means, for example, that if types A and B are monoids, then the tuple type (A, B) is also a monoid (called their product).

### Assembling more complex monoids
Some data structures form interesting monoids as long as the types of the elements they contain also form monoids. For instance, there’s a monoid for merging key-value Maps, as long as the value type is a monoid.

```
Merging key-value Maps
def mapMergeMonoid[K,V](V: Monoid[V]): Monoid[Map[K, V]] =
  new Monoid[Map[K, V]] {
    def zero = Map[K,V]()
    def op(a: Map[K, V], b: Map[K, V]) =
      (a.keySet ++ b.keySet).foldLeft(zero) { (acc,k) =>
        acc.updated(k, V.op(a.getOrElse(k, V.zero),
                            b.getOrElse(k, V.zero)))
      }
  }
```

Using this simple combinator, we can assemble more complex monoids fairly easily:

```
scala> val M: Monoid[Map[String, Map[String, Int]]] =
     | mapMergeMonoid(mapMergeMonoid(intAddition))
M: Monoid[Map[String, Map[String, Int]]] = $anon$1@21dfac82
```
This allows us to combine nested expressions using the monoid, with no additional programming:
```
scala> val m1 = Map("o1" -> Map("i1" -> 1, "i2" -> 2))
m1: Map[String,Map[String,Int]] = Map(o1 -> Map(i1 -> 1, i2 -> 2))

scala> val m2 = Map("o1" -> Map("i2" -> 3))
m2: Map[String,Map[String,Int]] = Map(o1 -> Map(i2 -> 3))

scala> val m3 = M.op(m1, m2)
m3: Map[String,Map[String,Int]] = Map(o1 -> Map(i1 -> 1, i2 -> 5))
```

### Using composed monoids to fuse traversals
The fact that multiple monoids can be composed into one means that we can perform multiple calculations simultaneously when folding a data structure. For example, we can take the length and sum of a list at the same time in order to calculate the mean:
```
scala> val m = productMonoid(intAddition, intAddition)
m: Monoid[(Int, Int)] = $anon$1@8ff557a

scala> val p = listFoldable.foldMap(List(1,2,3,4))(a => (1, a))(m)
p: (Int, Int) = (4, 10)

scala> val mean = p._1 / p._2.toDouble
mean: Double = 2.5
```

## 2.Monads
### Functors: generalizing the map function
In parts 1 and 2, we implemented several different combinator libraries. In each case, we proceeded by writing a small set of primitives and then a number of combinators defined purely in terms of those primitives. We noted some similarities between derived combinators across the libraries we wrote. For instance, we implemented a map function for each data type, to lift a function taking one argument “into the context of” some data type. For Gen, Parser, and Option, the type signatures were as follows:

```
def map[A,B](ga: Gen[A])(f: A => B): Gen[B]

def map[A,B](pa: Parser[A])(f: A => B): Parser[B]

def map[A,B](oa: Option[A])(f: A => B): Option[A]
```
These type signatures differ only in the concrete data type (Gen, Parser, or Option). We can capture as a Scala trait the idea of “a data type that implements map”:
```
trait Functor[F[_]] {
def map[A,B](fa: F[A])(f: A => B): F[B]
}
```

Here we’ve parameterized map on the type constructor, F[_], much like we did with Foldable in the previous chapter.[2] Instead of picking a particular F[_], like Gen or Parser, the Functor trait is parametric in the choice of F. Here’s an instance for List:

Recall that a type constructor is applied to a type to produce a type. For example, List is a type constructor, not a type. There are no values of type List, but we can apply it to the type Int to produce the type List[Int]. Likewise, Parser can be applied to String to yield Parser[String].

```
val listFunctor = new Functor[List] {
def map[A,B](as: List[A])(f: A => B): List[B] = as map f
}
```

We say that a type constructor like List (or Option, or F) is a functor, and the Functor[F] instance constitutes proof that F is in fact a functor.

What can we do with this abstraction? As we did in several places throughout this book, we can discover useful functions just by playing with the operations of the interface, in a purely algebraic way. You may want to pause here to see what (if any) useful operations you can define only in terms of map.

Let’s look at one example. If we have F[(A, B)] where F is a functor, we can “distribute” the F over the pair to get (F[A], F[B]):

```
trait Functor[F[_]] {
  ...
  def distribute[A,B](fab: F[(A, B)]): (F[A], F[B]) =
    (map(fab)(_._1), map(fab)(_._2))
}
```

Here we’ve parameterized map on the type constructor
Instead of picking a particular
F[_], like Gen or Parser, the Functor trait is parametric in the choice of F.
Here’s an instance for List:
2
Recall that a type constructor is applied to a type to produce a type. For example, List is a type constructor, not a type. There are no values of type List, but we can
apply it to the type Int to produce the type List[Int]. Likewise, Parser can be applied to String to yield Parser[String].
```
val listFunctor = new Functor[List] {
def map[A,B](as: List[A])(f: A => B): List[B] = as map f
}
```
We say that a type constructor like List (or Option, or F) is a functor, and
the Functor[F] instance constitutes proof that F is in fact a functor.
What can we do with this abstraction? As we did in several places
throughout this book, we can discover useful functions just by playing with
the operations of the interface, in a purely algebraic way. You may want to
pause here to see what (if any) useful operations you can define only in
terms of map.
Let’s look at one example. If we have F[(A, B)] where F is a functor, we
can “distribute” the F over the pair to get (F[A], F[B]):
```
trait Functor[F[_]] {
...
def distribute[A,B](fab: F[(A, B)]): (F[A], F[B]) =
(map(fab)(_._1), map(fab)(_._2))
}
```
We wrote this just by following the types, but let’s think about what it
means for concrete data types like List, Gen, Option, and so on. For
example, if we distribute a List[(A, B)], we get two lists of the same
length, one with all the As and the other with all the Bs. That operation is
sometimes called unzip. So we just wrote a generic unzip function that
works not just for lists, but for any functor!
And when we have an operation on a product like this, we should see if we
can construct the opposite operation over a sum or coproduct:
```
def codistribute[A,B](e: Either[F[A], F[B]]): F[Either[A, B]] =
e match {
case Left(fa) => map(fa)(Left(_))
case Right(fb) => map(fb)(Right(_))
}
```
What does codistribute mean for Gen? If we have either a generator for A
or a generator for B, we can construct a generator that produces either A or B
depending on which generator we actually have.
We just came up with two really general and potentially useful combinators
based purely on the abstract interface of Functor, and we can reuse them
for any type that allows an implementation of map.
### Functor laws
Whenever we create an abstraction like Functor, we should consider not
only what abstract methods it should have, but which laws we expect to
hold for the implementations. The laws you stipulate for an abstraction are
entirely up to you,[3] and of course Scala won’t enforce any of these laws.
But laws are important for two reasons:
- Laws help an interface form a new semantic level whose algebra may
be reasoned about independently of the instances. For example, when
we take the product of a Monoid[A] and a Monoid[B] to form a
Monoid[(A,B)], the monoid laws let us conclude that the “fused”
monoid operation is also associative. We don’t need to know anything
about A and B to conclude this.
- More concretely, we often rely on laws when writing various
combinators derived from the functions of some abstract interface like
Functor. We’ll see examples of this later.
For Functor, we’ll stipulate the familiar law we first introduced in chapter
7 for our Par data type:[4]
In other words, mapping over a structure x with the identity function should
itself be an identity. This law is quite natural, and we noticed later in part 2
that this law was satisfied by the map functions of other types besides Par.
This law (and its corollaries given by parametricity) capture the
requirement that map(x) “preserves the structure” of x. Implementations
satisfying this law are restricted from doing strange things like throwing
exceptions, removing the first element of a List, converting a Some to None,
and so on. Only the elements of the structure are modified by map; the shape
or structure itself is left intact. Note that this law holds for List, Option,
Par, Gen, and most other data types that define map!
To give a concrete example of this preservation of structure, we can
consider distribute and codistribute, defined earlier. Here are their
signatures again:
```
def distribute[A,B](fab: F[(A, B)]): (F[A], F[B])
def codistribute[A,B](e: Either[F[A], F[B]]): F[Either[A, B]]
```
Since we know nothing about F other than that it’s a functor, the law assures
us that the returned values will have the same shape as the arguments. If the
input to distribute is a list of pairs, the returned pair of lists will be of the
same length as the input, and corresponding elements will appear in the
same order. This kind of algebraic reasoning can potentially save us a lot of
work, since we don’t have to write separate tests for these properties.

### Monads: generalizing the flatMap and unit functions
Functor is just one of many abstractions that we can factor out of our
libraries. But Functor isn’t too compelling, as there aren’t many useful
operations that can be defined purely in terms of map. Next, we’ll look at a
more interesting interface, Monad. Using this interface, we can implement a
number of useful operations, once and for all, factoring out what would
otherwise be duplicated code. And it comes with laws with which we can
reason that our libraries work the way we expect.
Recall that for several of the data types in this book so far, we implemented
map2 to “lift” a function taking two arguments. For Gen, Parser, and
Option, the map2 function could be implemented as follows.
```
Implementing map2 for Gen, Parser, and Option
def map2[A,B,C](
fa: Gen[A], fb: Gen[B])(f: (A,B) => C): Gen[C] =
fa flatMap (a => fb map (b => f(a,b)))



def map2[A,B,C](
fa: Parser[A], fb: Parser[B])(f: (A,B) => C): Parser[C] =
fa flatMap (a => fb map (b => f(a,b)))



def map2[A,B,C](
fa: Option[A], fb: Option[B])(f: (A,B) => C): Option[C] =
fa flatMap (a => fb map (b => f(a,b)))
```

These functions have more in common than just the name. In spite of
operating on data types that seemingly have nothing to do with one another,
the implementations are identical! The only thing that differs is the
particular data type being operated on. This confirms what we’ve suspected
all along—that these are particular instances of some more general pattern.
We should be able to exploit that fact to avoid repeating ourselves. For
example, we should be able to write map2 once and for all in such a way
that it can be reused for all of these data types.
We’ve made the code duplication particularly obvious here by choosing
uniform names for our functions, taking the arguments in the same order,
and so on. It may be more difficult to spot in your everyday work. But the
more libraries you write, the better you’ll get at identifying patterns that you
can factor out into common abstractions.
### The Monad trait
What unites Parser, Gen, Par, Option, and many of the other data types
we’ve looked at is that they’re monads. Much like we did with Functor and
Foldable, we can come up with a Scala trait for Monad that defines map2
and numerous other functions once and for all, rather than having to
duplicate their definitions for every concrete data type.
In part 2 of this book, we concerned ourselves with individual data types,
finding a minimal set of primitive operations from which we could derive a
large number of useful combinators. We’ll do the same kind of thing here to
refine an abstract interface to a small set of primitives.
Let’s start by introducing a new trait, called Mon for now. Since we know we
want to eventually define map2, let’s go ahead and add that.
```
Creating a Mon trait for map2
 trait Mon[F[_]] {
  def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C] =
    fa flatMap (a => fb map (b => f(a,b)))	
}
```
Here we’ve just taken the implementation of map2 and changed Parser,
Gen, and Option to the polymorphic F of the Mon[F] interface in the
signature.[5] But in this polymorphic context, this won’t compile! We don’t
know anything about F here, so we certainly don’t know how to flatMap or
map over an F[A].
What we can do is simply add map and flatMap to the Mon interface and
keep them abstract. The syntax for calling these functions changes a bit (we
can’t use infix syntax anymore), but the structure is otherwise the same.
```
Adding map and flatMap to our trait

trait Mon[F[_]] {
  def map[A,B](fa: F[A])(f: A => B): F[B]
  def flatMap[A,B](fa: F[A])(f: A => F[B]): F[B]

  def map2[A,B,C](
      fa: F[A], fb: F[B])(f: (A,B) => C): F[C] =
    flatMap(fa)(a => map(fb)(b => f(a,b)))	
}
```
This translation was rather mechanical. We just inspected the
implementation of map2, and added all the functions it called, map and
flatMap, as suitably abstract methods on our interface. This trait will now
compile, but before we declare victory and move on to defining instances of
Mon[List], Mon[Parser], Mon[Option], and so on, let’s see if we can refine
our set of primitives. Our current set of primitives is map and flatMap, from
which we can derive map2. Is flatMap and map a minimal set of primitives?
Well, the data types that implemented map2 all had a unit, and we know
that map can be implemented in terms of flatMap and unit. For example,
on Gen:
```
def map[A,B](f: A => B): Gen[B] =
flatMap(a => unit(f(a)))
```
So let’s pick unit and flatMap as our minimal set. We’ll unify under a
single concept all data types that have these functions defined. The trait is
called Monad, it has flat-Map and unit abstract, and it provides default
implementations for map and map2.
### Creating our Monad trait
The name monad
We could have called Monad anything at all, like FlatMappable, Unicorn, or
Bicycle. But monad is already a perfectly good name in common use. The
name comes from category theory, a branch of mathematics that has
inspired a lot of functional programming concepts. The name monad is
intentionally similar to monoid, and the two concepts are related in a deep
way. See the chapter notes for more information.
To tie this back to a concrete data type, we can implement the Monad
instance for Gen.
```
Implementing Monad for Gen
object Monad {
val genMonad = new Monad[Gen] {
def unit[A](a: => A): Gen[A] = Gen.unit(a)
def flatMap[A,B](ma: Gen[A])(f: A => Gen[B]): Gen[B] =
ma flatMap f
}
}
```
We only need to implement unit and flatMap, and we get map and map2 at
no additional cost. We’ve implemented them once and for all, for any data
type for which it’s possible to supply an instance of Monad! But we’re just
getting started. There are many more functions that we can implement once
and for all in this manner.

### The associative law
For example, if we wanted to combine three monadic values into one,
which two should we combine first? Should it matter? To answer this
question, let’s for a moment take a step down from the abstract level and
look at a simple concrete example using the Gen monad.
Say we’re testing a product order system and we need to mock up some
orders. We might have an Order case class and a generator for that class.
Listing 11.6. Defining our Order class
Here we’re generating the Item inline (from name and price), but there
might be places where we want to generate an Item separately. So we could
pull that into its own generator:
```
val genItem: Gen[Item] = for {
name <- Gen.stringN(3)
price <- Gen.uniform.map(_ * 10)
} yield Item(name, price)
Then we can use that in genOrder:
val genOrder: Gen[Order] = for {
item <- genItem
quantity <- Gen.choose(1,100)
} yield Order(item, quantity)
```
And that should do exactly the same thing, right? It seems safe to assume
that. But not so fast. How can we be sure? It’s not exactly the same code.
Let’s expand both implementations of genOrder into calls to map and
flatMap to better see what’s going on. In the former case, the translation is
straightforward:
```
Gen.nextString.flatMap(name =>
Gen.nextDouble.flatMap(price =>
Gen.nextInt.map(quantity =>
Order(Item(name, price), quantity))))
But the second case looks like this (inlining the call to genItem):
Gen.nextString.flatMap(name =>
Gen.nextInt.map(price =>
Item(name, price))).flatMap(item =>
Gen.nextInt.map(quantity =>
Order(item, quantity)
```
Once we expand them, it’s clear that those two implementations aren’t
identical. And yet when we look at the for-comprehension, it seems
perfectly reasonable to assume that the two implementations do exactly the
same thing. In fact, it would be surprising and weird if they didn’t. It’s
because we’re assuming that flatMap obeys an associative law:
x.flatMap(f).flatMap(g) == x.flatMap(a => f(a).flatMap(g))
And this law should hold for all values x, f, and g of the appropriate types
—not just for Gen but for Parser, Option, and any other monad.
### Proving the associative law for a specific monad
Let’s prove that this law holds for Option. All we have to do is substitute
None or Some(v) for x in the preceding equation and expand both sides of it.
We start with the case where x is None, and then both sides of the equal sign
are
```
None:
None.flatMap(f).flatMap(g) == None.flatMap(a => f(a).flatMap(g))
Since None.flatMap(f) is None for all f, this simplifies to
None == None
Thus, the law holds if x is None. What about if x is Some(v) for an arbitrary
choice of v? In that case, we have
```

### The identity laws
The other monad law is now pretty easy to see. Just like zero was an
identity element for append in a monoid, there’s an identity element for
compose in a monad. Indeed, that’s exactly what unit is, and that’s why we
chose this name for this operation:
```def unit[A](a: => A): F[A]```
This function has the right type to be passed as an argument to compose.
The effect should be that anything composed with unit is that same thing.
This usually takes the form of two laws, left identity and right identity:
```
compose(f, unit) == f
compose(unit, f) == f
```
We can also state these laws in terms of flatMap, but they’re less clear that
way:
```
flatMap(x)(unit) == x
flatMap(unit(y))(f) == f(y)
```

### What is a monad?
Let’s now take a wider perspective. There’s something unusual about the
Monad interface. The data types for which we’ve given monad instances
don’t seem to have much to do with each other. Yes, Monad factors out code
duplication among them, but what is a monad exactly? What does “monad”
mean?
You may be used to thinking of interfaces as providing a relatively
complete API for an abstract data type, merely abstracting over the specific
representation. After all, a singly linked list and an array-based list may be
implemented differently behind the scenes, but they’ll share a common
interface in terms of which a lot of useful and concrete application code can
be written. Monad, like Monoid, is a more abstract, purely algebraic
interface. The Monad combinators are often just a small fragment of the full
API for a given data type that happens to be a monad. So Monad doesn’t
generalize one type or another; rather, many vastly different data types can
satisfy the Monad interface and laws.
We’ve seen three minimal sets of primitive Monad combinators, and
instances of Monad will have to provide implementations of one of these
sets:
- unit and flatMap
- unit and compose
- unit, map, and join
And we know that there are two monad laws to be satisfied, associativity
and identity, that can be formulated in various ways. So we can state plainly
what a monad is:
A monad is an implementation of one of the minimal sets of monadic
combinators, satisfying the laws of associativity and identity.

### The identity monad
To distill monads to their essentials, let’s look at the simplest interesting
specimen, the identity monad, given by the following type:
```case class Id[A](value: A)```
Implement map and flatMap as methods on this class, and give an
implementation for Monad[Id].
Now, Id is just a simple wrapper. It doesn’t really add anything. Applying
Id to A is an identity since the wrapped type and the unwrapped type are
totally isomorphic (we can go from one to the other and back again without
any loss of information). But what is the meaning of the identity monad?
Let’s try using it in the REPL:
```
scala> Id("Hello, ") flatMap (a =>
 | Id("monad!") flatMap (b =>
 | Id(a + b)))
res0: Id[java.lang.String] = Id(Hello, monad!)
```
When we write the exact same thing with a for-comprehension, it might be
clearer:
```
scala> for {
 | a <- Id("Hello, ")
 | b <- Id("monad!")
 | } yield a + b
res1: Id[java.lang.String] = Id(Hello, monad!)
```

So what is the action of flatMap for the identity monad? It’s simply
variable substitution. The variables a and b get bound to "Hello, " and
"monad!", respectively, and then substituted into the expression a + b. We
could have written the same thing without the Id wrapper, using just Scala’s
own variables:

```
scala> val a = "Hello, "
a: java.lang.String = "Hello, "
scala> val b = "monad!"
b: java.lang.String = monad!
scala> a + b
res2: java.lang.String = Hello, monad!

```
Besides the Id wrapper, there’s no difference. So now we have at least a
partial answer to the question of what monads mean. We could say that
monads provide a context for introducing and binding variables, and
performing variable substitution.

### The State monad and partial type application
```
Revisiting our State data type
case class State[S, A](run: S => (A, S)) {
 def map[B](f: A => B): State[S, B] =
 State(s => {
 val (a, s1) = run(s)
 (f(a), s1)
 })
 def flatMap[B](f: A => State[S, B]): State[S, B] =
 State(s => {
 val (a, s1) = run(s)
 f(a).run(s1)
 })
}
```
It looks like State definitely fits the profile for being a monad. But its type
constructor takes two type arguments, and Monad requires a type constructor
of one argument, so we can’t just say Monad[State]. But if we choose some
particular S, then we have something like State[S, _], which is the kind of
thing expected by Monad. So State doesn’t just have one monad instance
but a whole family of them, one for each choice of S. We’d like to be able to
partially apply State to where the S type argument is fixed to be some
concrete type.
This is much like how we might partially apply a function, except at the
type level. For example, we can create an IntState type constructor, which
is an alias for State with its first type argument fixed to be Int:
type IntState[A] = State[Int, A]
And IntState is exactly the kind of thing that we can build a Monad for:
object IntStateMonad extends Monad[IntState] {
def unit[A](a: => A): IntState[A] = State(s => (a, s))
def flatMap[A,B](st: IntState[A])(f: A => IntState[B]): IntState[B] =
st flatMap f
}
Of course, it would be really repetitive if we had to manually write a
separate Monad instance for each specific state type. Unfortunately, Scala
doesn’t allow us to use underscore syntax to simply say State[Int, _] to
create an anonymous type constructor like we create anonymous functions.
But instead we can use something similar to lambda syntax at the type
level. For example, we could have declared IntState directly inline like
this:
object IntStateMonad extends
Monad[({type IntState[A] = State[Int, A]})#IntState] {
...
}
This syntax can be jarring when you first see it. But all we’re doing is
declaring an anonymous type within parentheses. This anonymous type has,
as one of its members, the type alias IntState, which looks just like before.
Outside the parentheses we’re then accessing its IntState member with the
syntax. Just like we can use a dot (.) to access a member of an object at
the value level, we can use the # symbol to access a type member (see the
“Type Projection” section of the Scala Language Specification:
http://mng.bz/u70U).
A type constructor declared inline like this is often called a type lambda in
Scala. We can use this trick to partially apply the State type constructor
and declare a State-Monad trait. An instance of StateMonad[S] is then a
monad instance for the given state type S:
```
def stateMonad[S] = new Monad[({type f[x] = State[S,x]})#f] {	
  def unit[A](a: => A): State[S,A] = State(s => (a, s))
  def flatMap[A,B](st: State[S,A])(f: A => State[S,B]): State[S,B] =
    st flatMap f
}
```
Again, just by giving implementations of unit and flatMap, we get
implementations of all the other monadic combinators for free.

### Getting and setting state with a for-comprehension
```
val F = stateMonad[Int]
def zipWithIndex[A](as: List[A]): List[(Int,A)] =
as.foldLeft(F.unit(List[(Int, A)]()))((acc,a) => for {
xs <- acc
n <- getState
_ <- setState(n + 1)
} yield (n, a) :: xs).run(0)._1.reverse
```

This function numbers all the elements in a list using a State action. It
keeps a state that’s an Int, which is incremented at each step. We run the
whole composite state action starting from 0. We then reverse the result
since we constructed it in reverse order.
This function numbers all the elements in a list using a State action. It
keeps a state that’s an Int, which is incremented at each step. We run the
whole composite state action starting from 0. We then reverse the result
since we constructed it in reverse order.

