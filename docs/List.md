# List Operations
| Name  | Example | Description |
| ------|------- | ------------- |
| ::  | 1 :: 2 :: Nil  |Appends Individual elements to this list. A right-associative operator.|
| :::  | List(1, 2) ::: List(2, 3)  |Prepends another list to this one.A right-associative operator.|
| ++  | List(1, 2) ++ Set(3, 4, 3)  |Appends another collection to this list.|
| ==  |List(1, 2) == List(1, 2)  |Returns true if the collection types and contents are equal.|
| distinct  | List(3, 5, 4, 3, 4).distinct  |Returns a version of the list without duplicate elements.|
| drop  | List('a', 'b', 'c', 'd') drop 2 |Subtracts the first n elements from the list.|
| filter  | List(23, 8, 14, 21) filter (_ > 18)  |Returns elements from the list that pass a true/false function.|
| flatten  | List(List(1, 2), List(3, 4)).flatten  |Converts a list of lists into a single list of elements.|
| partition | List(1, 2, 3, 4, 5) partition (_ < 3)  |Groups elements into a tuple of two lists based on the result of a true/false function.|
| reverse  | List(1, 2, 3).reverse  |Reverses the list.|
| slice  | List(2, 3, 5, 7) slice (1, 3)  |Returns a segment of the list from the first index up to but not including the second index.|
| sortBy  | List("apple", "to") sortBy (_.size)  | Orders the list by the value returned from the given function.|
| sorted | List("apple", "to").sorted  |Orders a list of core Scala types by their natural value..|
| splitAt  | List(2, 3, 5, 7) splitAt 2  |Groups elements into a tuple of two lists based on if they fall before or after the given index.|
| take | List(2, 3, 5, 7, 11, 13) take 3 |Extracts the first n elements from the list.|
| zip  | List(1, 2) zip List("a", "b")  |Combines two lists into a list of tuples of elements at each index.|


Did you spot the higher-order functions in the table? Here are the three examples of higher-order operations, filter, partition, and sortBy, executed in the Scala REPL:

scala> val f = List(23, 8, 14, 21) filter (_ > 18)
f: List[Int] = List(23, 21)

scala> val p = List(1, 2, 3, 4, 5) partition (_ < 3)
p: (List[Int], List[Int]) = (List(1, 2),List(3, 4, 5))

scala> val s = List("apple", "to") sortBy (_.size)
s: List[String] = List(to, apple)
The sortBy method takes a function that returns a value for use in ordering the elements of the list, while the filter and partition methods each take a predicate function. A predicate function takes an input value and returns either true or false. In the case of the partition function, the predicate uses placeholder syntax (see Placeholder Syntax) to return true if the input value is less than three and false otherwise.

Collection methods that are also higher-order functions, such as filter, map, and partition, are excellent candidates for using placeholder syntax. The function parameter they take as input acts on a single element in their list. Thus the underscore (_) in an anonymous function sent to one of these methods represents each item in the list.

For example, the anonymous function we passed to the partition method, _ < 3, indicates that each element in the list will be checked to see if it is less than 3. The values 1 and 2 were less than 3 and thus partitioned into a separate list.

An important point to make about these arithmetic methods is that ::, drop, and take act on the front of the list and thus do not have performance penalties. Recall that List is a linked list, so adding items to or removing items from its front does not require a full traversal. A list traversal is a trivial operation for short lists, but when you start getting into lists of thousands or millions of items, an operation that requires a list traversal can be a big deal.

That said, these operations have corollary operations that act on the end of the list and thus do require a full list traversal. Additionally, because adding items to the end of a list would mutate it, they require copying the entire list and returning the copy. Again, not an important memory consideration unless you are working with large lists, but in general it is best to operate on the front of a list, not its end.

The corollary operations to ::, drop, and take are +: (a left-associative operator), dropRight, and takeRight. The arguments to these operators are the same as to their corollary operations.

Here are examples of these list-appending operations:

scala> val appended = List(1, 2, 3, 4) :+ 5
appended: List[Int] = List(1, 2, 3, 4, 5)

scala> val suffix = appended takeRight 3
suffix: List[Int] = List(3, 4, 5)

scala> val middle = suffix dropRight 2
middle: List[Int] = List(3)
With the extremely small list sizes used in these examples, the additional time and memory space required to traverse the list and copy its contents to a new array is miniscule. However, it’s still considered good form to prefer operations on the start of a list over those that work on the end.

Mapping Lists
Map methods are those that take a function and apply it to every member of a list, collecting the results into a new list. In set theory, and the field of mathematics in general, to map is to create an assocation between each element in one set to each element in another set. In a sense, both definitions describe what the map methods in List are doing: mapping each item from one list to another list, so that the other list has the same size as the first but with different data or element types. Table 6-2 shows a selection of these map methods available on Scala’s lists.

Table 6-2. List mapping operations
Name	Example	Description
collect

List(0, 1, 0) collect {case 1 => "ok"}

Transforms each element using a partial function, retaining applicable elements.

flatMap

List("milk,tea") flatMap (_.split(','))

Transforms each element using the given function and “flattens” the list of results into this list.

map

List("milk","tea") map (_.toUpperCase)

Transforms each element using the given function.

Let’s see how these list-mapping operators work in the REPL:

scala> List(0, 1, 0) collect {case 1 => "ok"}
res0: List[String] = List(ok)

scala> List("milk,tea") flatMap (_.split(','))
res1: List[String] = List(milk, tea)

scala> List("milk","tea") map (_.toUpperCase)
res2: List[String] = List(MILK, TEA)
The flatMap example uses the String.split() method to convert pipe-delimited text into a list of strings. Specifically this is the java.lang.String.split() method, and it returns a Java array, not a list. Fortunately, Scala converts Java arrays to its own type, Array, which extends Iterable. Because List is also a subtype of Iterable, and the flatMap method is defined at the Iterable level, a list of string arrays can be safely flattened into a list of strings.

Reducing Lists
We have studied ways to change the size and structure of lists, and ways to convert lists into completely different values and types. Now let’s look at ways to shrink down all that work to a single value, an action known as reducing a list.

List reduction is a common operation for working with collections. Need to sum up a list of grades, or calculate the average duration of several benchmarks? How about if you want to check if a collection contains a specific element, or to see if a predicate function will return “true” for every element in the list? These are all list reductions, because they use logic to reduce a list down to a single value.

Scala’s collections support mathematical reduction operations (e.g., finding the sum of a list) and Boolean reduction operations (e.g., determining if a list contains a given element). They also support generic higher-order operations known as folds that you can use to create any other type of list reduction algorithm.

We’ll start by looking at the built-in mathematical reduction operations. See Table 6-3 for a selection of these methods.

Table 6-3. Math reduction operations
Name	Example	Description
max

List(41, 59, 26).max

Finds the maximum value in the list.

min

List(10.9, 32.5, 4.23, 5.67).min

Finds the minimum value in the list.

product

List(5, 6, 7).product

Multiplies the numbers in the list.

sum

List(11.3, 23.5, 7.2).sum

Sums up the numbers in the list.

The next set of operations we’ll look at reduces lists down to a single Boolean value. See Table 6-4 for a selection of these methods.

Table 6-4. Boolean reduction operations
Name	Example	Description
contains

List(34, 29, 18) contains 29

Checks if the list contains this element.

endsWith

List(0, 4, 3) endsWith List(4, 3)

Checks if the list ends with a given list.

exists

List(24, 17, 32) exists (_ < 18)

Checks if a predicate holds true for at least one element in the list.

forall

List(24, 17, 32) forall (_ < 18)

Checks if a predicate holds true for every element in the list.

startsWith

List(0, 4, 3) startsWith List(0)

Tests whether the list starts with a given list.

These Boolean list reduction operations work great with infix operator notation, not only because they take a single argument but also due to their names being verbs. The phrase “list contains x” reads more like a written description about an operation, even though it is actually a valid, statically typed function invocation.

In addition to being highly readable, they are also rather similar. Choosing the right operation for a task may be more of a question of readability than suitability. As an example of their similar nature, let’s search through a list of validation results for a “false” entry using three different operations:

scala> val validations = List(true, true, false, true, true, true)
validations: List[Boolean] = List(true, true, false, true, true, true)

scala> val valid1 = !(validations contains false)
valid1: Boolean = false

scala> val valid2 = validations forall (_ == true)
valid2: Boolean = false

scala> val valid3 = validations.exists(_ == false) == false
valid3: Boolean = false
Logically, checking that a list of validations does not contain “false” is the same as ensuring that the list only contains “true.”

These operations are useful enough to have been included with Scala collections, but are not so complex that we couldn’t implement them ourselves. Let’s create our own list reduction operation to demonstrate how it is done. Doing so simply requires iterating over a collection with an accumulator variable, which contains the current result so far, and logic that updates the accumulator based on the current element:

scala> def contains(x: Int, l: List[Int]): Boolean = {
     |   var a: Boolean = false
     |   for (i <- l) { if (!a) a = (i == x) }
     |   a
     | }
contains: (x: Int, l: List[Int])Boolean

scala> val included = contains(19, List(46, 19, 92))
included: Boolean = true
This works perfectly well, but could also stand to be improved. How about if we separate the “contains” logic from the work of maintaining an accumulator value and iterating through the list? By moving the “contains” logic to a function parameter, we could create a reusable function to support additional list reduction operations.

Here’s the same logic as in the previous example except with the core “contains” logic moved to a function parameter. We’ll name this common function boolReduce to indicate that it is a Boolean list reduction operation:

scala> def boolReduce(l: List[Int], start: Boolean)(f: (Boolean, Int) =>
     |   Boolean): Boolean = {
     |
     |   var a = start
     |   for (i <- l) a = f(a, i)
     |   a
     | }
boolReduce: (l: List[Int], start: Boolean)(f: (Boolean, Int) => Boolean)Boolean

scala> val included = boolReduce(List(46, 19, 92), false) { (a, i) =>
     |   if (a) a else (i == 19)
     | }
included: Boolean = true
Our generic-sounding boolReduce function is no longer tied to determining if a list contains an element, and could be reused for any of the other Boolean reduction operations. We could theoretically implement exists, forall, startsWith, and the rest of the Boolean operations.

Let’s take this example one step further and make it even more generally applicable. The boolReduce function is fine for Boolean operations on integer lists, but we could “genericize” it to make it applicable to lists and reduction operations of any type. Once this function takes type parameters for its list elements and the accumulator value and result (which necessarily need to match), we could use it to implement max, sum, and other mathematical operations.

Here is the boolReduce operation rewritten as reduceOp, renamed because it is no longer Boolean-specific, with the Int and Boolean types replaced with the type parameters A and B, respectively. What’s really nice is that our sample invocation doesn’t require any changes from working with boolReduce thanks to Scala’s inference of type parameters. To verify that this new operation isn’t limited to an integer list and a Boolean result, I have added an implementation of the sum example:

scala> def reduceOp[A,B](l: List[A], start: B)(f: (B, A) => B): B = { 1
     |   var a = start
     |   for (i <- l) a = f(a, i)
     |   a
     | }
reduceOp: [A, B](l: List[A], start: B)(f: (B, A) => B)B

scala> val included = reduceOp(List(46, 19, 92), false) { (a, i) => 2
     |   if (a) a else (i == 19)
     | }
included: Boolean = true

scala> val answer = reduceOp(List(11.3, 23.5, 7.2), 0.0)(_ + _) 3
answer: Double = 42.0
1
Replacing real types with type parameters can make the code less readable. If it isn’t clear what the A and B parameters are referring to, have a look at the boolReduce function definition and compare the parameters in both functions.

2
I’ve chosen “a” as the name of the accumulator value and “i” as the name of the current element in the list. Writing function literals gives you the option to define your own names for input parameters!

3
In this case I chose placeholder syntax because the parameters are each accessed only once in the function body.

Our reduceOp method is now a generic, left-to-right (or, start-to-finish) list reduction operation. It could be used to implement a math reduction operation such as max or a Boolean reduction operation such as contains. In fact, it could be used to create any other list reduction operation, at least one that supports its use of scanning the list from left to right (i.e., from the first element to the last).

Fortunately, you won’t need to write down or remember the reduceOp function in order to take advantage of its functionality. Scala’s collections provide built-in operations similar to reduceOp that are flexible enough to provide left-to-right, right-to-left, and order-agnostic versions, as well as offering different ways to work with the accumulator and accumulated values. These higher-order functions to reduce a list based on the input function are popularly known as list-folding operations, because the function of reducing a list is better known as a fold.

Table 6-5 displays a selection of the list-folding operations in Scala’s collections. To simplify the process of comparing the functions, each operation’s example reuses the “sum” function implemented in the previous example.

Table 6-5. Generic list reduction operations
Name	Example	Description
fold

List(4, 5, 6).fold(0)(_ + _)

Reduces the list given a starting value and a reduction function.reduction function.

foldLeft

List(4, 5, 6).foldLeft(0)(_ + _)

Reduces the list from left to right given a starting value and a reduction function.

foldRight

List(4, 5, 6).foldRight(0)(_ + _)

Reduces the list from right to left given a starting value and a reduction function.

reduce

List(4, 5, 6).reduce(_ + _)

Reduces the list given a reduction function, starting with the first element in the list.

reduceLeft

List(4, 5, 6).reduceLeft(_ + _)

Reduces the list from left to right given a reduction function, starting with the first element in the list.

reduceRight

List(4, 5, 6).reduceRight(_ + _)

Reduces the list from right to left given a reduction function, starting with the first element in the list.

scan

List(4, 5, 6).scan(0)(_ + _)

Takes a starting value and a reduction function and returns a list of each accumulated value.

scanLeft

List(4, 5, 6).scanLeft(0)(_ + _)

Takes a starting value and a reduction function and returns a list of each accumulated value from left to right.

scanRight

List(4, 5, 6).scanRight(0)(_ + _)

Takes a starting value and a reduction function and returns a list of each accumulated value from right to left.

The three folding operations, fold, reduce, and scan, are really not very different from each other. Can you figure out how you might implement reduce as a specific case of fold, or implement fold if you were given the scan function?

Interestingly, the differences between the left/right directional varieties of each operation, e.g., foldLeft, and the nondirectional variety, e.g., fold, may be more significant than the differences between the three folding operations. For one thing, fold, reduce, and scan are all limited to returning a value of the same type as the list elements, while the left/right varities of each operation support unique return types. Thus you could implement the forall Boolean operation on a list of integers with foldLeft but would not be able to do so with fold.

Another major difference is in the ordering. Whereas foldLeft and foldRight, as an example, specify the direction in which they will iterate through the list, the non-directional operations specify no order to their iteration. This often puzzles developers, because it doesn’t make clear which direction will be used.

For example, what if your collection is not sequential but is distributed among a dozen different computers? Or what if it is all on the same computer, but your fold operation is so expensive that you want it to run in parallel? In such cases, it makes sense to distinguish between a fold that iterates through the list sequentially versus a fold that may, based on the collection that implements it, run in any order it needs to.

Unless you are specifically using distributed or parallel collections, or you are developing a library that may be reused with such collections, it is safe to simply choose the left/right directional varieties. I will also recommend that, unless you require right-to-left iteration, it is better to select the “left” operations because they require fewer traversals through the list in their implementation.

So, before studying the three list-folding operations we implemented the contains and sum operations the hard way. Now let’s reimplement them using the new folding operations we just covered:

scala> val included = List(46, 19, 92).foldLeft(false) { (a, i) => 1
     |   if (a) a else (i == 19)
     | }
included: Boolean = true

scala> val answer = List(11.3, 23.5, 7.2).reduceLeft(_ + _)        2
answer: Double = 42.0
1
Not much has changed here other than that we’re calling foldLeft, a list operation. Would reduceLeft work here?

2
This operation is now even shorter thanks to reduceLeft, which uses the first element in the list for a starting value instead of taking it as a parameter.

In this section we covered list reduction/folding operations, both specific and generic. The numeric and Boolean list reduction operations are widely useful, but in case you need additional operations, you now know how to create your own.

Converting Collections
Lists are ubiquitous, especially in the examples in this chapter, but the other collections are certainly also important for their own uses. I find myself reaching for lists by default when I need a collection, but sometimes you do need a map, set, or other type. Fortunately, it is easy to convert between these types, so you can create a collection with one type and end up with the other.

Table 6-6 contains a selection of these methods. Because a List.toList() operation would be silly (but possible), the examples demonstrate converting from one type to a completely different type.

Table 6-6. Operations to convert collections
Name	Example	Description
mkString

List(24, 99, 104).mkString(", ")

Renders a collection to a Set using the given delimiters.

toBuffer

List('f', 't').toBuffer

Converts an immutable collection to a mutable one.

toList

Map("a" -> 1, "b" -> 2).toList

Converts a collection to a List.

toMap

Set(1 -> true, 3 -> true).toMap

Converts a collection of 2-arity (length) tuples to a Map.

toSet

List(2, 5, 5, 3, 2).toSet

Converts a collection to a Set.

toString

List(2, 5, 5, 3, 2).toString

Renders a collection to a String, including the collection’s type.

Consider these operations when you have a map but only want a list of its keys, or are given a list and want to generate a lookup map with it. As immutable collections, List, Map, and Set cannot be built from empty collections and so are better suited to being mapped from existing collections. With these operations you can map data in one type to another type, even if you’re going from a sequence to a key-value store (or back).

Java and Scala Collection Compatibility
There is another important angle to converting collections that we need to cover. Because Scala compiles to and runs on the JVM, interacting with the JDK as well as any Java libraries you may add is a common requirement. Part of this task of interacting is to convert between Java and Scala collections, because the two collection types are incompatible by default.

You can add the following command to enable manual conversions between Java and Scala collections. Although this command is a bit esoteric now, it’ll make more sense when we study it in the context of object-oriented Scala later in the book:

scala> import collection.JavaConverters._
import collection.JavaConverters._
This import command adds JavaConverters and its methods to the current namespace. In the REPL this means the current session, while in source files this means the rest of the file or local scope, wherever the import command is added. Table 6-7 displays the operations added to Java and Scala collections when JavaConverters has been imported.

Table 6-7. Java and Scala collection conversions
Name	Example	Description
asJava

List(12, 29).asJava

Converts this Scala collection to a corresponding Java collection.

asScala

new java.util.ArrayList(5).asScala

Converts this Java collection to a corresponding Scala collection.

By exercising this import of JavaConverters, a greater selection of Java libraries and JVM functions is made available without significantly changing how you use Scala collections.

Pattern Matching with Collections
The final operation that we’ll review in this chapter isn’t a named collection method, but the use of match expressions (see Match Expressions) with collections. If you recall, we have used match expressions to match single value patterns:

scala> val statuses = List(500, 404)
statuses: List[Int] = List(500, 404)

scala> val msg = statuses.head match {
     |   case x if x < 500 => "okay"
     |   case _ => "whoah, an error"
     | }
msg: String = whoah, an error
With a pattern guard (see Matching with Pattern Guards), you could also match a single value inside a collection:

scala> val msg = statuses match {
     |   case x if x contains(500) => "has error"
     |   case _ => "okay"
     | }
msg: String = has error
Because collections support the equals operator (==) it shouldn’t be a surprise that they also support pattern matching. To match the entire collection, use a new collection as your pattern:

scala> val msg = statuses match {
     |   case List(404, 500) => "not found & error"
     |   case List(500, 404) => "error & not found"
     |   case List(200, 200) => "okay"
     |   case _ => "not sure what happened"
     | }
msg: String = error & not found
You can use value binding (see Matching with Wildcard Patterns) to bind values to some or all elements of a collection in your pattern guard:

scala> val msg = statuses match {
     |   case List(500, x) => s"Error followed by $x"
     |   case List(e, x) => s"$e was followed by $x"
     | }
msg: String = Error followed by 404
Lists are decomposable into their head element and their tail. In the same way, as patterns they can be matched on their head and tail elements:

scala> val head = List('r','g','b') match {
     |   case x :: xs => x
     |   case Nil => ' '
     | }
head: Char = r
Tuples, while not officially collections, also support pattern matching and value binding. Because a single tuple can support values of different types, their pattern-matching capability is at times even more useful than that of collections:

scala> val code = ('h', 204, true) match {
     |   case (_, _, false) => 501
     |   case ('c', _, true) => 302
     |   case ('h', x, true) => x
     |   case (c, x, true) => {
     |     println(s"Did not expect code $c")
     |     x
     |   }
     | }
code: Int = 204
Pattern matching is a core feature of the Scala language, not simply another operation in its standard collection library. It is broadly applicable to Scala’s data structures, and when used wisely can shorten and simplify logic that would require expansive work in other languages.

Summary
Working with collections, whether creating, mapping, filtering, or performing other operations, is a major component of software development. And lists, maps, and sets, some of the main building blocks for scalable data structures, are included as part of the default libraries for Java, Ruby, Python, PHP, and C++. What sets Scala’s collections library apart from the others is its core support for immutable data structures and higher-order operations.

The core data structures in Scala, List, Map, and Set, are immutable. They cannot be resized, nor can their contents be swapped out. As a way of giving precedence over mutable collections, their package (collection.immutable) is automatically imported into Scala namespaces by default. This precedence aims to steer developers toward the immutable collections and immutable data in general, a “best practice” in functional programming circles. This is not to say that mutable collections are less powerful or less capable than immutable ones. Scala’s mutable collections have all the same features as the immutable ones and also support a range of modification operations. We’ll learn about mutable collections and how to convert mutable to immutable ones (and vice versa) in the next chapter.

The ability to take a collection and iterate or map it with an anonymous function is common to many languages, including Ruby and Python. However, the ability to do so while ensuring the type requirements of both the collection and the input and return types of the anonymous functions is relatively uncommon. Collections with type-safe higher-order functions support a declarative programming style, the ability to create expressive code, and very few runtime type conversion errors. This powerful combination of features helps to set the Scala collections library apart from those available in other languages and frameworks and provides a fairly large productivity boost to its users. In addition, Scala collections are monadic, supporting the ability to chain operations together in a high-level, type-safe manner. We’ll learn about monadic collections as well in the next chapter.

Exercises
Do you recall the suggestion I previously made (see Exercises) to switch your development environment from inside-the-REPL to an external Scala source file? If you haven’t made the switch yet, you’ll find working on these exercises in the REPL to be downright impractical given their size and complexity.

I also recommend working on these exercises using a professional IDE such as IntelliJ IDEA CE or the Eclipse-based Scala IDE. You’ll gain instant feedback about whether code is compilable and get code completion and documentation for Scala library functions. There are also plug-ins for simpler editing environments like Sublime Text, VIM, and Emacs that enable this functionality, but if you’re getting started with Scala a full-fledged IDE will probably be easier and quicker to use.

The exercises in this section will help you become familiar with the core collections and operations we have studied in this chapter. I recommend spending time to not only write the most basic solution, but to find alternate methods for each implementation. This will help you become familiar with the subtle differences between similar functions such as fold and reduce, or head and slice, in addition to giving you the tools to bypass these functions and develop your own solutions.

Create a list of the first 20 odd Long numbers. Can you create this with a for-loop, with the filter operation, and with the map operation? What’s the most efficient and expressive way to write this?
Write a function titled “factors” that takes a number and returns a list of its factors, other than 1 and the number itself. For example, factors(15) should return List(3, 5).

Then write a new function that applies “factors” to a list of numbers. Try using the list of Long numbers you generated in exercise 1. For example, executing this function with List(9, 11, 13, 15) should return List(3, 3, 5), because the factor of 9 is 3 while the factors of 15 are 3 again and 5. Is this a good place to use map and flatten? Or would a for-loop be a better fit?

Write a function, first[A](items: List[A], count: Int): List[A], that returns the first x number of items in a given list. For example, first(List('a','t','o'), 2) should return List('a','t'). You could make this a one-liner by invoking one of the built-in list operations that already performs this task, or (preferably) implement your own solution. Can you do so with a for-loop? With foldLeft? With a recursive function that only accesses head and tail?
Write a function that takes a list of strings and returns the longest string in the list. Can you avoid using mutable variables here? This is an excellent candidate for the list-folding operations (Table 6-5) we studied. Can you implement this with both fold and reduce? Would your function be more useful if it took a function parameter that compared two strings and returned the preferred one? How about if this function was applicable to generic lists, i.e., lists of any type?
Write a function that reverses a list. Can you write this as a recursive function? This may be a good place for a match expression.
Write a function that takes a List[String] and returns a (List[String],List[String]), a tuple of string lists. The first list should be items in the original list that are palindromes (written the same forward and backward, like “racecar”). The second list in the tuple should be all of the remaining items from the original list. You can implement this easily with partition, but are there other operations you could use instead?
The last exercise in this chapter is a multipart problem. We’ll be reading and processing a forecast from the excellent and free OpenWeatherMap API.

To read content from the URL we’ll use the Scala library operation io.Source.+fromURL(url: String), which returns an +io.Source instance. Then we’ll reduce the source to a collection of individual lines using the getLines.toList operation. Here is an example of using io.Source to read content from a URL, separate it into lines, and return the result as a list of strings:

scala> val l: List[String] = io.Source.fromURL(url).getLines.toList
Here is the URL we will use to retrieve the weather forecast, in XML format:

scala> val url =
  "http://api.openweathermap.org/data/2.5/forecast?mode=xml&lat=55&lon=0"
Go ahead and read this URL into a list of strings. Once you have it, print out the first line to verify you’ve captured an XML file. The result should look pretty much like this:

scala> println( l(0) )
<?xml version="1.0" encoding="utf-8"?>
If you don’t see an XML header, make sure that your URL is correct and your Internet connection is up.

Let’s begin working with this List[String] containing the XML document.

To make doubly sure we have the right content, print out the top 10 lines of the file. This should be a one-liner.
The forecast’s city’s name is there in the first 10 lines. Grab it from the correct line and print out its XML element. Then extract the city name and country code from their XML elements and print them out together (e.g., “Paris, FR”). This is a good place to use regular expressions to extract the text from XML tags (see Regular expressions).

If you don’t want to use regular expression capturing groups, you could instead use the replaceAll() operation on strings to remove the text surrounding the city name and country name.

How many forecast segments are there? What is the shortest expression you can write to count the segments?
The “symbol” XML element in each forecast segment includes a description of the weather forecast. Extract this element in the same way you extracted the city name and country code. Try iterating through the forecasts, printing out the description.

Then create an informal weather report by printing out the weather descriptions over the next 12 hours (not including the XML elements).

Let’s find out what descriptions are used in this forecast. Print a sorted listing of all of these descriptions in the forecast, with duplicate entries removed.
These descriptions may be useful later. Included in the “symbol” XML element is an attribute containing the symbol number. Create a Map from the symbol number to the description. Verify this is accurate by manually accessing symbol values from the forecast and checking that the description matches the XML document.
What are the high and low temperatures over the next 24 hours?
What is the average temperature in this weather forecast? You can use the “value” attribute in the temperature element to calculate this value.
Now that you have solved the exercises, are there simpler or shorter solutions than the ones you chose? Did you prefer infix dot notation or infix operator notation? Was using for..yield easier than higher-order operations like map and filter?

This is a good place to rework some of your solutions to really find your favored coding style, which is often the intersection between ease of writing, ease of reading, and expressiveness.
