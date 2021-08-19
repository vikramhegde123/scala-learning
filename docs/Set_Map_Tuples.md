# Using sets
The key characteristic of sets is that they will ensure that at most one of each object, as determined
by ==, will be contained in the set at any one time. As an example, we'll use a set to count the number
of different words in a string.
The split method on String can separate a string into words, if you specify spaces and punctuation as
word separators. The regular expression "[ !,.]+" will suffice: It indicates the string should be split at
each place that one or more space and/or punctuation characters exist.

```
scala> val text = "See Spot run. Run, Spot. Run!"
text: String = See Spot run. Run, Spot. Run!

scala> val wordsArray = text.split("[ !,.]+")
wordsArray: Array[String]
= Array(See, Spot, run, Run, Spot, Run)
```

To count the distinct words, you can convert them to the same case and then add them to a set. Because
sets exclude duplicates, each distinct word will appear exactly one time in the set.
First, you can create an empty set using the empty method provided on the Set companion objects:
```
scala> val words = mutable.Set.empty[String]
words: scala.collection.mutable.Set[String] = Set()
```
Then, just iterate through the words with a for expression, convert each word to lower case, and add it
to the mutable set with the += operator:
```
scala> for (word <- wordsArray)
words += word.toLowerCase

scala> words
res17: scala.collection.mutable.Set[String] =
Set(see, run, spot)
```
Thus, the text contained exactly three distinct words: spot, run, and see. The most commonly used
methods on both mutable and immutable sets are shown in Table 17.1.
## Common operations for sets
| What it is | Description |
| ------| ------------- |
| val nums = Set(1, 2, 3) | Creates an immutable set (nums.toString returnsSet(1, 2, 3))|
| nums + 5 | Adds an element (returns Set(1, 2, 3, 5))|
| nums - 3 | Removes an element (returns Set(1, 2))|
| nums ++ | List(5, 6) Adds multiple elements (returns Set(1, 2, 3, 5, 6))|
| nums -- | List(1, 2) Removes multiple elements (returns Set(3))|
| nums & Set(1, 3, 5, 7) | Takes the intersection of two sets (returns Set(1, 3))|
| nums.size | Returns the size of the set (returns 3)|
| nums.contains(3) | Checks for inclusion (returns true) |
| import scala.collection.mutable | Makes the mutable collections easy to access|
| val words = mutable.Set.empty[String] | Creates an empty, mutable set (words.toString returnsSet())|
| words += "the" | Adds an element (words.toString returns Set(the))|
| words -= "the" | Removes an element, if it exists (words.toStringreturns Set())|
| words ++= List("do", "re", "mi") | Adds multiple elements (words.toString returnsSet(do, re, mi))|
| words --= List("do", "re") | Removes multiple elements (words.toString returnsSet(mi))|
| words.clear | Removes all elements (words.toString returns Set())|


# Using maps
Maps let you associate a value with each element of a set. Using a map looks similar to using an array,
except instead of indexing with integers counting from 0, you can use any kind of key. If you import
the mutable package name, you can create an empty mutable map like this:
```
 scala> val map = mutable.Map.empty[String, Int]
 map: scala.collection.mutable.Map[String,Int] = Map()
```

Note that when you create a map, you must specify two types. The first type is for the keys of the map,
the second for the values. In this case, the keys are strings and the values are integers. Setting entries in
a map looks similar to setting entries in an array:
```
 scala> map("hello") = 1
 
 scala> map("there") = 2
 
 scala> map
 res20: scala.collection.mutable.Map[String,Int] =
 Map(hello -> 1, there -> 2)
 ```
Likewise, reading a map is similar to reading an array:
```
 scala> map("hello")
 res21: Int = 1
 ```
Putting it all together, here is a method that counts the number of times each word occurs in a string:
```
 scala> def countWords(text: String) = {
 val counts = mutable.Map.empty[String, Int]
 for (rawWord <- text.split("[ ,!.]+")) {
 val word = rawWord.toLowerCase
 val oldCount =
 if (counts.contains(word)) counts(word)
 else 0
 counts += (word -> (oldCount + 1))
 }
 counts
 }
 countWords: (text:
 String)scala.collection.mutable.Map[String,Int]
 
 scala> countWords("See Spot run! Run, Spot. Run!")
 res22: scala.collection.mutable.Map[String,Int] =
 Map(spot -> 2, see -> 1, run -> 3)
 ```
Given these counts, you can see that this text talks a lot about running, but not so much about seeing.
The way this code works is that a mutable map, named counts, maps each word to the number of times
it occurs in the text. For each word in the text, the word's old count is looked up, that count is
incremented by one, and the new count is saved back into counts. Note the use ofcontains to check
whether a word has been seen yet or not. If counts.contains(word) is not true, then the word has not yet
been seen and zero is used for the count.
Many of the most commonly used methods on both mutable and immutable maps are shown in Table

## Common operations for maps
| What it is | What it does |
| --------- | -------------------|
| val nums = Map("i" -> 1, "ii" -> 2) | Creates an immutable map (nums.toString returnsMap(i -> 1, ii -> 2)) |
| nums + ("vi" -> 6) | Adds an entry (returns Map(i -> 1, ii -> 2, vi -> 6)) |
| nums - "ii" |  Removes an entry (returns Map(i -> 1)) |
| nums ++ List("iii" -> 3, "v" -> 5) | Adds multiple entries (returns Map(i -> 1, ii -> 2, iii -> 3, v -> 5)) |
| nums -- List("i", "ii") | Removes multiple entries (returns Map()) |
| nums.size | Returns the size of the map (returns 2) |
| nums.contains("ii") | Checks for inclusion (returns true) |
| nums("ii") | Retrieves the value at a specified key (returns 2) |
| nums.keys | Returns the keys (returns an Iterable over the strings"i" and "ii") |
| nums.keySet | Returns the keys as a set (returns Set(i, ii)) |
| nums.values | Returns the values (returns an Iterable over the integers 1 and 2) |
| nums.isEmpty | Indicates whether the map is empty (returns false) |
| import scala.collection.mutable | Makes the mutable collections easy to access |
| val words = mutable.Map.empty[String, Int] | Creates an empty, mutable map words += ("one" -> 1) Adds a map entry from "one" to 1 (words.toStringreturns Map(one - > 1)) |
| words -= "one" | Removes a map entry, if it exists words.toStringreturns Map()) |
| words ++= List("one" -> 1, "two" -> 2, "three" -> 3) | Adds multiple map entries (words.toString returnsMap(one -> 1, two -> 2, three -> 3)) |
| words --= List("one", "two") | Removes multiple objects (words.toString returnsMap(three -> 3)) |

## Default sets and maps
For most uses, the implementations of mutable and immutable sets and maps provided
by the Set(), scala.collection.mutable.Map(), etc., factories will likely be sufficient. The
implementations provided by these factories use a fast lookup algorithm, usually involving a hash table,
so they can quickly decide whether or not an object is in the collection.
The scala.collection.mutable.Set() factory method, for example, returns
a scala.collection.mutable.HashSet, which uses a hash table internally. Similarly,
the scala.collection.mutable.Map() factory returns a scala.collection.mutable.HashMap.
The story for immutable sets and maps is a bit more involved. The class returned by
the scala.collection.immutable.Set() factory method, for example, depends on how many elements you
pass to it, as shown in Table 17.3. For sets with fewer than five elements, a special class devoted
exclusively to sets of each particular size is used to maximize performance. Once you request a set that
has five or more elements in it, however, the factory method will return an implementation that uses
hash tries.
Similarly, the scala.collection.immutable.Map() factory method will return a different class depending
on how many key-value pairs you pass to it, as shown in Table 17.4. As with sets, for immutable maps
with fewer than five elements, a special class devoted exclusively to maps of each particular size is
used to maximize performance. Once a map has five or more key-value pairs in it, however, an
immutable HashMap is used.
The default immutable implementation classes shown in Tables 17.3 and 17.4 work together to give
you maximum performance. For example, if you add an element to an EmptySet, it will return a Set1.
If you add an element to that Set1, it will return a Set2. If you then remove an element from the Set2,
you'll get another Set1.

### Default immutable set implementations

```
Number of elements Implementation
0 scala.collection.immutable.EmptySet
1 scala.collection.immutable.Set1
2 scala.collection.immutable.Set2
3 scala.collection.immutable.Set3
4 scala.collection.immutable.Set4
5 or more scala.collection.immutable.HashSet
Table 17.4 - Default immutable map implementations
Number of elements Implementation
0 scala.collection.immutable.EmptyMap
1 scala.collection.immutable.Map1
2 scala.collection.immutable.Map2
3 scala.collection.immutable.Map3
4 scala.collection.immutable.Map4
5 or more scala.collection.immutable.HashMap
```

## Sorted sets and maps
On occasion you may need a set or map whose iterator returns elements in a particular order. For this
purpose, the Scala collections library provides traits SortedSet and SortedMap. These traits are
implemented by classes TreeSet and TreeMap, which use a red-black tree to keep elements (in the case
of TreeSet) or keys (in the case of TreeMap) in order. The order is determined by the Ordered trait,
which the element type of the set, or key type of the map, must either mix in or be implicitly
convertible to. These classes only come in immutable variants. Here are some TreeSet examples:
```
 scala> import scala.collection.immutable.TreeSet
 import scala.collection.immutable.TreeSet
 
 scala> val ts = TreeSet(9, 3, 1, 8, 0, 2, 7, 4, 6, 5)
 ts: scala.collection.immutable.TreeSet[Int] =
 TreeSet(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
 
 scala> val cs = TreeSet('f', 'u', 'n')
 cs: scala.collection.immutable.TreeSet[Char] =
 TreeSet(f, n, u)
 ```
And here are a few TreeMap examples:
```
 scala> import scala.collection.immutable.TreeMap
 import scala.collection.immutable.TreeMap
 
 scala> var tm = TreeMap(3 -> 'x', 1 -> 'x', 4 -> 'x')
 tm: scala.collection.immutable.TreeMap[Int,Char] =
 Map(1 -> x, 3 -> x, 4 -> x)
 
 scala> tm += (2 -> 'x')
 
 scala> tm
 res30: scala.collection.immutable.TreeMap[Int,Char] =
 Map(1 -> x, 2 -> x, 3 -> x, 4 -> x)
```

# Tuples
A tuple combines a fixed number of items together so that they can
be passed around as a whole. Unlike an array or list, a tuple can hold objects with different types. Here
is an example of a tuple holding an integer, a string, and the console:
```
(1, "hello", Console)
```
A common application of tuples is returning multiple values from a method. For example, here is a
method that finds the longest word in a collection and also returns its index:
```
def longestWord(words: Array[String]) = {
var word = words(0)
var idx = 0
for (i <- 1 until words.length)
if (words(i).length > word.length) {
word = words(i)
idx = i
}
(word, idx)
}
```
Here is an example use of the method:
```
scala> val longest =
longestWord("The quick brown fox".split(" "))
longest: (String, Int) = (quick,1)
```
The longestWord function here computes two items: word, the longest word in the array, and idx, the
index of that word. To keep things simple, the function assumes there is at least one word in the list,
and it breaks ties by choosing the word that comes earlier in the list. Once the function has chosen
which word and index to return, it returns both of them together using the tuple syntax (word, idx).
To access elements of a tuple, you can use method _1 to access the first element, _2 to access the
second, and so on:
```
scala> longest._1
res53: String = quick

scala> longest._2
res54: Int = 1
```
Additionally, you can assign each element of the tuple to its own variable,[5] like this:
```
scala> val (word, idx) = longest
word: String = quick
idx: Int = 1

scala> word
res55: String = quick
```
By the way, if you leave off the parentheses you get a different result:
```
scala> val word, idx = longest
word: (String, Int) = (quick,1)
idx: (String, Int) = (quick,1)
```
This syntax gives multiple definitions of the same expression. Each variable is initialized with its own
evaluation of the expression on the right-hand side. That the expression evaluates to a tuple in this case
does not matter. Both variables are initialized to the tuple in its entirety. SeeChapter 18 for some
examples where multiple definitions are convenient.
As a note of warning, tuples are almost too easy to use. Tuples are great when you combine data that
has no meaning beyond "an A and a B." However, whenever the combination has some meaning, or
you want to add some methods to the combination, it is better to go ahead and create a class. For
example, do not use a 3-tuple for the combination of a month, a day, and a year. Make a Date class. It
makes your intentions explicit, which both clears up the code for human readers and gives the compiler
and language opportunities to help you catch mistakes

