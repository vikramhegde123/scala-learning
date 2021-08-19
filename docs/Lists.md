#List
The List class is a linear, immutable sequence. All this means is that it’s a linked-list
that you can’t modify. Any time you want to add or remove List elements, you create
a new List from an existing List.

### a) Creating Lists
This is how you create an initial List:
```
val ints = List(1, 2, 3)
val names = List("Joel", "Chris", "Ed")

You can also declare the List’s type, if you prefer, though it generally isn’t necessary:

val ints: List[Int] = List(1, 2, 3)
val names: List[String] = List("Joel", "Chris", "Ed")
```
### b) Adding elements to a List
Because List is immutable, you can’t add new elements to it. Instead you create a new
list by prepending or appending elements to an existing List. For instance, given this
```
List:
val a = List(1,2,3)

You prepend elements to a List like this:
val b = 0 +: a
and this:
val b = List(-1, 0) ++: a
105
106

The REPL shows how this works:
scala> val b = 0 +: a
b: List[Int] = List(0, 1, 2, 3)
scala> val b = List(-1, 0) ++: a
b: List[Int] = List(-1, 0, 1, 2, 3)
```
### c) HOW TO LOOP OVER LISTS

```
val names = List("Joel", "Chris", "Ed")
you can print each string like this:
for (name <- names) println(name)
This is what it looks like in the REPL:
scala> for (name <- names) println(name)
Joel
Chris
Ed
```

### d) Pattern Matching on Lists
  ```
    scala> val statuses = List(500, 404)
    statuses: List[Int] = List(500, 404)

    scala> val msg = statuses.head match {
     |   case x if x < 500 => "okay"
     |   case _ => "whoah, an error"
     | }
    msg: String = whoah, an error
   ```

Because collections support the equals operator (==) it shouldn’t be a surprise that they also support pattern matching. To match the entire collection, use a new      
collection as your pattern:
  ```
  scala> val msg = statuses match {
     |   case List(404, 500) => "not found & error"
     |   case List(500, 404) => "error & not found"
     |   case List(200, 200) => "okay"
     |   case _ => "not sure what happened"
     | }
  msg: String = error & not found
  ```

You can use value binding (see Matching with Wildcard Patterns) to bind values to some or all elements of a collection in your pattern guard:
  ```
  scala> val msg = statuses match {
     |   case List(500, x) => s"Error followed by $x"
     |   case List(e, x) => s"$e was followed by $x"
     | }
  msg: String = Error followed by 404
  ```

### e) Concatenating two lists
Here are some examples:

```
scala> List(1, 2) ::: List(3, 4, 5)
res0: List[Int] = List(1, 2, 3, 4, 5)

scala> List() ::: List(1, 2, 3)
res1: List[Int] = List(1, 2, 3)

scala> List(1, 2, 3) ::: List(4)
res2: List[Int] = List(1, 2, 3, 4)
```

### f) Accessing the end of a list: init and last

```
 scala> val abcde = List('a', 'b', 'c', 'd', 'e')
 abcde: List[Char] = List(a, b, c, d, e)
 
 scala> abcde.last
 res4: Char = e
 
 scala> abcde.init
 res5: List[Char] = List(a, b, c, d)
```
Like head and tail, these methods throw an exception when applied to an empty list:
```
scala> List().init
java.lang.UnsupportedOperationException: Nil.init
at scala.List.init(List.scala:544)
at ...

scala> List().last
java.util.NoSuchElementException: Nil.last
at scala.List.last(List.scala:563)
```

### g) Reversing lists: reverse

```
scala> abcde.reverse
 res6: List[Char] = List(e, d, c, b, a)
```

### h) Prefixes and suffixes: drop, take, and splitAt

```
scala> abcde take 2
res8: List[Char] = List(a, b)

scala> abcde drop 2
res9: List[Char] = List(c, d, e)

scala> abcde splitAt 2
res10: (List[Char], List[Char]) = (List(a, b),List(c, d, e))
```

### i) Flattening a list of lists: flatten

The flatten method takes a list of lists and flattens it out to a single list:
```
scala> List(List(1, 2), List(3), List(), List(4, 5)).flatten
res14: List[Int] = List(1, 2, 3, 4, 5)
scala> fruit.map(_.toCharArray).flatten
res15: List[Char] = List(a, p, p, l, e, s, o, r, a, n, g, e,
s, p, e, a, r, s)
```

It can only be applied to lists whose elements are all lists. Trying to flatten any other list will give a
compilation error:
```
scala> List(1, 2, 3).flatten
<console>:8: error: No implicit view available from Int =>
scala.collection.GenTraversableOnce[B].
List(1, 2, 3).flatten
```

### j) Zipping lists: zip and unzip
The zip operation takes two lists and forms a list of pairs:
```
scala> abcde.indices zip abcde
res17: scala.collection.immutable.IndexedSeq[(Int, Char)] =
Vector((0,a), (1,b), (2,c), (3,d), (4,e))
If the two lists are of different length, any unmatched elements are dropped:
scala> val zipped = abcde zip List(1, 2, 3)
zipped: List[(Char, Int)] = List((a,1), (b,2), (c,3))
```

A useful special case is to zip a list with its index. This is done most efficiently with
thezipWithIndex method, which pairs every element of a list with the position where it appears in the
list.
```
scala> abcde.zipWithIndex
res18: List[(Char, Int)] = List((a,0), (b,1), (c,2), (d,3),
(e,4))
Any list of tuples can also be changed back to a tuple of lists by using the unzip method:
scala> zipped.unzip
res19: (List[Char], List[Int])
= (List(a, b, c),List(1, 2, 3))
```
The zip and unzip methods provide one way to operate on multiple lists together. See Section 16.9 for a
more concise way to do this

### HIGHER-ORDER METHODS ON CLASS LIST
### k) Mapping over lists: map, flatMap and foreach
The operation xs map f takes as operands a list xs of type List[T] and a function f of type T => U. It
returns the list that results from applying the function f to each list element in xs. For instance:
```
scala> List(1, 2, 3) map (_ + 1)
res32: List[Int] = List(2, 3, 4)

scala> val words = List("the", "quick", "brown", "fox")
words: List[String] = List(the, quick, brown, fox)

scala> words map (_.length)
res33: List[Int] = List(3, 5, 5, 3)

scala> words map (_.toList.reverse.mkString)
res34: List[String] = List(eht, kciuq, nworb, xof
```

The flatMap operator is similar to map, but it takes a function returning a list of elements as its right
operand. It applies the function to each list element and returns the concatenation of all function results.
The difference between map and flatMap is illustrated in the following example:
```
scala> words map (_.toList)
res35: List[List[Char]] = List(List(t, h, e), List(q, u, i,
c, k), List(b, r, o, w, n), List(f, o, x))

scala> words flatMap (_.toList)
res36: List[Char] = List(t, h, e, q, u, i, c, k, b, r, o, w,
n, f, o, x)
You see that where map returns a list of lists, flatMap returns a single list in which all element lists are
concatenated
```
### l)Filtering lists: filter, partition, find, takeWhile, dropWhile, and span

The operation "xs filter p" takes as operands a list xs of type List[T] and a predicate function pof
type T => Boolean. It yields the list of all elements x in xs for which p(x) is true. For instance:
```
scala> List(1, 2, 3, 4, 5) filter (_ % 2 == 0)
res40: List[Int] = List(2, 4)

scala> words filter (_.length == 3)
res41: List[String] = List(the, fox)
```

The partition method is like filter but returns a pair of lists. One list contains all elements for which the
predicate is true, while the other contains all elements for which the predicate is false. It is defined by
the equality:
xs partition p equals (xs filter p, xs filter (!p(_)))
Here's an example:
```
scala> List(1, 2, 3, 4, 5) partition (_ % 2 == 0)
res42: (List[Int], List[Int]) = (List(2, 4),List(1, 3, 5))
```

The takeWhile and dropWhile operators also take a predicate as their right operand. The
operation xs takeWhile p takes the longest prefix of list xs such that every element in the prefix
satisfies p. Analogously, the operation xs dropWhile p removes the longest prefix from list xssuch that
every element in the prefix satisfies p. Here are some examples:

```
scala> List(1, 2, 3, -4, 5) takeWhile (_ > 0)
res45: List[Int] = List(1, 2, 3)

scala> words dropWhile (_ startsWith "t")
res46: List[String] = List(quick, brown, fox)
```

The span method combines takeWhile and dropWhile in one operation, just
like splitAt combinestake and drop. It returns a pair of two lists, defined by the equality:
xs span p equals (xs takeWhile p, xs dropWhile p)
Like splitAt, span avoids traversing the list xs twice:
```
scala> List(1, 2, 3, -4, 5) span (_ > 0)
res47: (List[Int], List[Int]) = (List(1, 2, 3),List(-4, 5))
```

### m) Predicates over lists: forall and exists

The operation xs forall p takes as arguments a list xs and a predicate p. Its result is true if all elements
in the list satisfy p. Conversely, the operation xs exists p returns true if there is an element in xs that
satisfies the predicate p. For instance, to find out whether a matrix represented as a list of lists has a
row with only zeroes as elements:
scala> def hasZeroRow(m: List[List[Int]]) =
m exists (row => row forall (_ == 0))
hasZeroRow: (m: List[List[Int]])Boolean

scala> hasZeroRow(diag3)
res48: Boolean = false

### n) Folding lists: /: and :\
Another common kind of operation combines the elements of a list with some operator. For instance:

```
sum(List(a, b, c)) equals 0 + a + b + c
```
This is a special instance of a fold operation:
```
scala> def sum(xs: List[Int]): Int = (0 /: xs) (_ + _)
sum: (xs: List[Int])Int
```

Similarly:
product(List(a, b, c)) equals 1 * a * b * c
is a special instance of this fold operation:
```
scala> def product(xs: List[Int]): Int = (1 /: xs) (_ * _)
product: (xs: List[Int])Int
```
A fold left operation "(z /: xs) (op)" involves three objects: a start value z, a list xs, and a binary
operation op. The result of the fold is op applied between successive elements of the list prefixed by z.
For instance:
(z /: List(a, b, c)) (op) equals op(op(op(z, a), b), c)

Here's another example that illustrates how /: is used. To concatenate all words in a list of strings with
spaces between them and in front, you can write this:
```
scala> ("" /: words) (_ + " " + _)
res49: String = " the quick brown fox"
```
This gives an extra space at the beginning. To remove the space, you can use this slight variation:
```
scala> (words.head /: words.tail) (_ + " " + _)
res50: String = the quick brown fox
```
The /: operator produces left-leaning operation trees (its syntax with the slash rising forward is intended
to be a reflection of that). The operator has :\ as an analog that produces right-leaning trees. For
instance:
(List(a, b, c) :\ z) (op) equals op(a, op(b, op(c, z)))


### o) List reversal using fold
Fold left operation based on the following scheme:
```
def reverseLeft[T](xs: List[T]) = (startvalue /: xs)(operation)
```
What remains is to fill in the startvalue and operation parts. In fact, you can try to deduce these parts
from some simple examples. To deduce the correct value of startvalue, you can start with the smallest
possible list, List(), and calculate as follows:

```
List()
equals (by the properties of reverseLeft)

reverseLeft(List())
equals (by the template for reverseLeft)

(startvalue /: List())(operation)
equals (by the definition of /:)

startvalue
```
Hence, startvalue must be List(). To deduce the second operand, you can pick the next smallest list as
an example case. You know already that startvalue is List(), so you can calculate as follows:

```
List(x)
equals (by the properties of reverseLeft)

reverseLeft(List(x))
equals (by the template for reverseLeft, with startvalue = List())

(List() /: List(x)) (operation)
equals (by the definition of /:)

operation(List(), x)
```

Hence, operation(List(), x) equals List(x), which can also be written as x :: List(). This suggests taking
as operation the :: operator with its operands exchanged. (This operation is sometimes called "snoc," in
reference to ::, which is called cons.) We arrive then at the following implementation for reverseLeft:

```
def reverseLeft[T](xs: List[T]) =
(List[T]() /: xs) {(ys, y) => y :: ys}
```

### p) Creating a range of numbers: List.range
```
scala> List.range(1, 5)
 res54: List[Int] = List(1, 2, 3, 4)
 
 scala> List.range(1, 9, 2)
 res55: List[Int] = List(1, 3, 5, 7)
 
 scala> List.range(9, 1, -3)
 res56: List[Int] = List(9, 6, 3)

```

### q) Creating uniform lists: List.fill
The fill method creates a list consisting of zero or more copies of the same element. It takes two
parameters: the length of the list to be created, and the element to be repeated. Each parameter is given
in a separate list:
```
scala> List.fill(5)('a')
res57: List[Char] = List(a, a, a, a, a)

scala> List.fill(3)("hello")
res58: List[String] = List(hello, hello, hello)
```
If fill is given more than two arguments, then it will make multi-dimensional lists. That is, it will make
lists of lists, lists of lists of lists, etc. The additional arguments go in the first argument list.
```
scala> List.fill(2, 3)('b')
res59: List[List[Char]] = List(List(b, b, b), List(b, b, b))
```

### r) Tabulating a function: List.tabulate

```
scala> val squares = List.tabulate(5)(n => n * n)
squares: List[Int] = List(0, 1, 4, 9, 16)
scala> val multiplication = List.tabulate(5,5)(_ * _)
multiplication: List[List[Int]] = List(List(0, 0, 0, 0, 0),
List(0, 1, 2, 3, 4), List(0, 2, 4, 6, 8),
List(0, 3, 6, 9, 12), List(0, 4, 8, 12, 16))
```

### s) PROCESSING MULTIPLE LISTS TOGETHER
The zipped method on tuples generalizes several common operations to work on multiple lists instead
of just one. One such operation is map. The map method for two zipped lists maps pairs of elements
rather than individual elements. One pair is for the first element of each list, another pair is for the
second element of each list, and so on—as many pairs as the lists are long. Here is an example of its
use:

```
scala> (List(10, 20), List(3, 4, 5)).zipped.map(_ * _)
res63: List[Int] = List(30, 80)
```
Notice that the third element of the second list is discarded. The zipped method zips up only as many
elements as appear in all the lists together. Any extra elements on the end are discarded.
There are also zipped analogs to exists and forall. They are just like the single-list versions of those
methods except they operate on elements from multiple lists instead of just one:
```
scala> (List("abc", "de"), List(3, 2)).zipped.
forall(_.length == _)
res64: Boolean = true
scala> (List("abc", "de"), List(3, 2)).zipped.
exists(_.length != _)
res65: Boolean = false
```




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

# Mapping Lists
| Name  | Example | Description |
| ------|------- | ------------- |
| collect  | List(0, 1, 0) collect {case 1 => "ok"}  |Transforms each element using a partial function, retaining applicable elements.|
| flatMap  | List("milk,tea") flatMap (_.split(','))  |Transforms each element using the given function and “flattens” the list of results into this list.|
| map |List("milk","tea") map (_.toUpperCase) |Transforms each element using the given function.|

# Reducing Lists
| Name  | Example | Description |
| ------|------- | ------------- |
| max  | List(41, 59, 26).max  |Finds the maximum value in the list.|
| min  | List(10.9, 32.5, 4.23, 5.67).min |Finds the minimum value in the list.|
| product |List(5, 6, 7).product|Multiplies the numbers in the list.|
| sum |List(11.3, 23.5, 7.2).sum|Sums up the numbers in the list.|

# Boolean reduction operations
| Name  | Example | Description |
| ------|------- | ------------- |
| contains  | List(34, 29, 18) contains 29  |Checks if the list contains this element.|
| endsWith  | List(0, 4, 3) endsWith List(4, 3) |Checks if the list ends with a given list.|
| exists |List(24, 17, 32) exists (_ < 18)|Checks if a predicate holds true for at least one element in the list.|
| forall |List(24, 17, 32) forall (_ < 18)|Checks if a predicate holds true for every element in the list.|
| startsWith |List(0, 4, 3) startsWith List(0)|Tests whether the list starts with a given list.|

# Generic list reduction operations
| Name  | Example | Description |
| ------|------- | ------------- |
| fold  | List(4, 5, 6).fold(0)(_ + _)  |Reduces the list given a starting value and a reduction function.reduction function.|
| foldLeft  | List(4, 5, 6).foldLeft(0)(_ + _)|Reduces the list from left to right given a starting value and a reduction function.|
| foldRight |List(4, 5, 6).foldRight(0)(_ + _)|Reduces the list from right to left given a starting value and a reduction function.|
| reduce |List(4, 5, 6).reduce(_ + _)|Reduces the list given a reduction function, starting with the first element in the list.|
| reduceLeft |List(4, 5, 6).reduceLeft(_ + _)|Reduces the list from left to right given a reduction function, starting with the first element in the list.|
| reduceRight |List(4, 5, 6).reduceRight(_ + _)|Reduces the list from right to left given a reduction function, starting with the first element in the list.|
| scan |List(4, 5, 6).scan(0)(_ + _)|Takes a starting value and a reduction function and returns a list of each accumulated value.|
| scanLeft |List(4, 5, 6).scanLeft(0)(_ + _)|Takes a starting value and a reduction function and returns a list of each accumulated value from left to right.|
| scanRight |List(4, 5, 6).scanRight(0)(_ + _)|Takes a starting value and a reduction function and returns a list of each accumulated value from right to left.|

# Converting Collections
| Name  | Example | Description |
| ------|------- | ------------- |
| mkString  | List(24, 99, 104).mkString(", ")  |Renders a collection to a Set using the given delimiters.|
| toBuffer  | List('f', 't').toBuffer|Converts an immutable collection to a mutable one.|
| toList |Map("a" -> 1, "b" -> 2).toList|Converts a collection to a List.|
| toMap |Set(1 -> true, 3 -> true).toMap|Converts a collection of 2-arity (length) tuples to a Map.|
| toSet |List(2, 5, 5, 3, 2).toSet|Converts a collection to a Set.|
| toString |List(2, 5, 5, 3, 2).toString|Renders a collection to a String, including the collection’s type.|