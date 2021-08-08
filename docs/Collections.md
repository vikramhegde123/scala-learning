# Collections Hierachy
[](images/collections-hierachy.png)

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

#List
The List class is a linear, immutable sequence. All this means is that it’s a linked-list
that you can’t modify. Any time you want to add or remove List elements, you create
a new List from an existing List.

a) Creating Lists
This is how you create an initial List:
```
val ints = List(1, 2, 3)
val names = List("Joel", "Chris", "Ed")

You can also declare the List’s type, if you prefer, though it generally isn’t necessary:

val ints: List[Int] = List(1, 2, 3)
val names: List[String] = List("Joel", "Chris", "Ed")
```
b) Adding elements to a List
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
c) HOW TO LOOP OVER LISTS

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

# Pattern Matching with Collections
  ```
    scala> val statuses = List(500, 404)
    statuses: List[Int] = List(500, 404)

    scala> val msg = statuses.head match {
     |   case x if x < 500 => "okay"
     |   case _ => "whoah, an error"
     | }
    msg: String = whoah, an error
   ```
    
   With a pattern guard (see Matching with Pattern Guards), you could also match a single value inside a collection:
   ```
   scala> val msg = statuses match {
     |   case x if x contains(500) => "has error"
     |   case _ => "okay"
     | }
    msg: String = has error
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
  


# Exercises
1. Create a list of the first 20 odd Long numbers. Can you create this with a for-loop, with the filter operation, and with the map operation? What’s the most efficient and      expressive way to write this?

2. Write a function titled “factors” that takes a number and returns a list of its factors, other than 1 and the number itself. For example, factors(15) should return 
   List(3, 5) .Then write a new function that applies “factors” to a list of numbers. Try using the list of Long numbers you generated in exercise 1. For example, executing      this function with List(9, 11, 13, 15) should return List(3, 3, 5), because the factor of 9 is 3 while the factors of 15 are 3 again and 5. Is this a good place to use   
   map and flatten? Or would a for-loop be a better fit?
    
3. Write a function, first[A](items: List[A], count: Int): List[A], that returns the first x number of items in a given list. For example, first(List('a','t','o'), 2) should
   return List('a','t'). You could make this a one-liner by invoking one of the built-in list operations that already performs this task, or (preferably) implement your own 
   solution. Can you do so with a for-loop? With foldLeft? With a recursive function that only accesses head and tail?
   
4. Write a function that takes a list of strings and returns the longest string in the list. Can you avoid using mutable variables here? This is an excellent candidate for      the list-folding operations (Table 6-5) we studied. Can you implement this with both fold and reduce? Would your function be more useful if it took a function parameter      that compared two strings and returned the preferred one? How about if this function was applicable to generic lists, i.e., lists of any type?

5. Write a function that reverses a list. Can you write this as a recursive function? This may be a good place for a match expression.

6. Write a function that takes a List[String] and returns a (List[String],List[String]), a tuple of string lists. The first list should be items in the original list that are
   palindromes (written the same forward and backward, like “racecar”). The second list in the tuple should be all of the remaining items from the original list. You can 
   implement this easily with partition, but are there other operations you could use instead?
