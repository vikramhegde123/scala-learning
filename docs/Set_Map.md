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
Common operations for sets
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
```
