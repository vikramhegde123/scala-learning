# FOR EXPRESSIONS
Scala's for expression is a Swiss army knife of iteration. It lets you combine a few simple ingredients in
different ways to express a wide variety of iterations. Simple uses enable common tasks such as
iterating through a sequence of integers. More advanced expressions can iterate over multiple
collections of different kinds, filter out elements based on arbitrary conditions, and produce new
collections.
Iteration through collections
The simplest thing you can do with for is to iterate through all the elements of a collection. For
example, This method returns an array of File objects, one per directory and file contained in
the current directory. We store the resulting array in the filesHere variable.

```
val filesHere = (new java.io.File(".")).listFiles

for (file <- filesHere)
println(file)
```

With the "file <- filesHere" syntax, which is called a generator, we iterate through the elements
of filesHere. In each iteration, a new val named file is initialized with an element value. The compiler
infers the type of file to be File, because filesHere is an Array[File]. For each iteration, the body of
the for expression, println(file), will be executed. Because File'stoString method yields the name of the
file or directory, the names of all the files and directories in the current directory will be printed.
The for expression syntax works for any kind of collection, not just arrays.[2] One convenient special
case is the Range type, which you briefly saw in Table 5.4 here. You can create Ranges using syntax
like "1 to 5" and can iterate through them with a for. Here is a simple example:

```
scala> for (i <- 1 to 4)
println("Iteration " + i)
Iteration 1
Iteration 2
Iteration 3
Iteration 4
```
If you don't want to include the upper bound of the range in the values that are iterated over,
use until instead of to:
```
scala> for (i <- 1 until 4)
println("Iteration " + i)
Iteration 1
Iteration 2
Iteration 3
```
Iterating through integers like this is common in Scala, but not nearly as much as in other languages. In
other languages, you might use this facility to iterate through an array, like this:
// Not common in Scala...
```
for (i <- 0 to filesHere.length - 1)
println(filesHere(i))
```
This for expression introduces a variable i, sets it in turn to each integer
between 0 andfilesHere.length - 1, and executes the body of the for expression for each setting of i. For
each setting of i, the i'th element of filesHere is extracted and processed.
The reason this kind of iteration is less common in Scala is that you can just iterate over the collection
directly. When you do, your code becomes shorter and you sidestep many of the off-by-one errors that
can arise when iterating through arrays. Should you start at 0 or 1? Should you add -1, +1, or nothing to
the final index? Such questions are easily answered, but also easily answered wrong. It is safer to avoid
such questions entirely.

## Filtering
Sometimes you don't want to iterate through a collection in its entirety; you want to filter it down to
some subset. You can do this with a for expression by adding a filter, an if clause inside the for's
parentheses. For example, the code shown in Listing 7.6 lists only those files in the current directory
whose names end with ".scala":

#### Finding .scala files using a for with a filter.
```
val filesHere = (new java.io.File(".")).listFiles

for (file <- filesHere if file.getName.endsWith(".scala"))
println(file)
```

You could alternatively accomplish the same goal with this code:
```
for (file <- filesHere)
if (file.getName.endsWith(".scala"))
println(file)
```

This code yields the same output as the previous code, and likely looks more familiar to programmers
with an imperative background. The imperative form, however, is only an option because this
particular for expression is executed for its printing side-effects and results in the unit value (). As
demonstrated later in this section, the for expression is called an "expression" because it can result in an
interesting value, a collection whose type is determined by the for expression's <- clauses.
#### Using multiple filters in a for expression.
```
for (
file <- filesHere
if file.isFile
if file.getName.endsWith(".scala")
) println(file)
```

## Nested iteration
If you add multiple <- clauses, you will get nested "loops."
For example, the for expression shown below has two nested loops. The outer loop iterates through filesHere, and the inner loop
iterates through fileLines(file) for any file that ends with .scala.
#### Using multiple generators in a for expression.
```
def grep(pattern: String) =
for (
file <- filesHere
if file.getName.endsWith(".scala");
line <- fileLines(file)
if line.trim.matches(pattern)
) println(file + ": " + line.trim)

grep(".*gcd.*")
```

## Mid-stream variable bindings
Note that the previous code repeats the expression line.trim. This is a non-trivial computation, so you
might want to only compute it once. You can do this by binding the result to a new variable using an
equals sign (=). The bound variable is introduced and used just like a val, only with the val keyword
left out.

#### Mid-stream assignment in a for expression.
```
def grep(pattern: String) =
for {
file <- filesHere
if file.getName.endsWith(".scala")
line <- fileLines(file)
trimmed = line.trim
if trimmed.matches(pattern)
} println(file + ": " + trimmed)

```

In above example, a variable named trimmed is introduced halfway through the for expression. That
variable is initialized to the result of line.trim. The rest of the for expression then uses the new variable
in two places, once in an if and once in println.
Producing a new collection
While all of the examples so far have operated on the iterated values and then forgotten them, you can
also generate a value to remember for each iteration. To do so, you prefix the body of the for expression
by the keyword yield. For example, here is a function that identifies the.scala files and stores them in
an array:
```
def scalaFiles =
for {
file <- filesHere
if file.getName.endsWith(".scala")
} yield file
```
Each time the body of the for expression executes, it produces one value, in this case simplyfile. When
the for expression completes, the result will include all of the yielded values contained in a single
collection. The type of the resulting collection is based on the kind of collections processed in the
iteration clauses. In this case the result is an Array[File], becausefilesHere is an array and the type of
the yielded expression is File.
Be careful, by the way, where you place the yield keyword. The syntax of a for-yield expression is like
this:
for clauses yield body
The yield goes before the entire body. Even if the body is a block surrounded by curly braces, put
the yield before the first curly brace, not before the last expression of the block. Avoid the temptation to
write things like this:
```
for (file <- filesHere if file.getName.endsWith(".scala")) {
yield file // Syntax error!
}
```
For example, the for expression shown above, first transforms the Array[File] namedfilesHere,
which contains all files in the current directory, to one that contains only .scala files. For each of these
it generates an Iterator[String], the result of the fileLines method, whose definition is shown in Listing
An Iterator offers methods next and hasNext that allow you to iterate over a collection of elements.
This initial iterator is transformed into anotherIterator[String] containing only trimmed lines that
include the substring "for". Finally, for each of these, an integer length is yielded. The result of
this for expression is an Array[Int]containing those lengths.
```
val forLineLengths =
for {
file <- filesHere
if file.getName.endsWith(".scala")
line <- fileLines(file)
trimmed = line.trim
if trimmed.matches(".*for.*")
} yield trimmed.length
```

## QUERYING WITH FOR EXPRESSIONS
The for notation is essentially equivalent to common operations of database query languages. For
instance, say you are given a database named books, represented as a list of books, whereBook is
defined as follows:
```
case class Book(title: String, authors: String*)
Here is a small example database represented as an in-memory list:
val books: List[Book] =
List(
Book(
"Structure and Interpretation of Computer Programs",
"Abelson, Harold", "Sussman, Gerald J."
),
Book(
"Principles of Compiler Design",
"Aho, Alfred", "Ullman, Jeffrey"
),
Book(
"Programming in Modula-2",
"Wirth, Niklaus"
),
Book(
"Elements of ML Programming",
"Ullman, Jeffrey"
),
Book(
"The Java Language Specification", "Gosling, James",
"Joy, Bill", "Steele, Guy", "Bracha, Gilad"
)
)
```
To find the titles of all books whose author's last name is "Gosling":
```
scala> for (b <- books; a <- b.authors
if a startsWith "Gosling")
yield b.title
res4: List[String] = List(The Java Language Specification)
Or to find the titles of all books that have the string "Program" in their title:
scala> for (b <- books if (b.title indexOf "Program") >= 0)
yield b.title
res5: List[String] = List(Structure and Interpretation of
Computer Programs, Programming in Modula-2, Elements of ML
Programming)
```
Or to find the names of all authors who have written at least two books in the database:
```
scala> for (b1 <- books; b2 <- books if b1 != b2;
a1 <- b1.authors; a2 <- b2.authors if a1 == a2)
yield a1
res6: List[String] = List(Ullman, Jeffrey, Ullman, Jeffrey)
```
The last solution is still not perfect because authors will appear several times in the list of results. You
still need to remove duplicate authors from result lists. This can be achieved with the following
function:
```
scala> def removeDuplicates[A](xs: List[A]): List[A] = {
if (xs.isEmpty) xs
else
xs.head :: removeDuplicates(
xs.tail filter (x => x != xs.head)
)
}
removeDuplicates: [A](xs: List[A])List[A]

scala> removeDuplicates(res6)
res7: List[String] = List(Ullman, Jeffrey)
```
It's worth noting that the last expression in method removeDuplicates can be equivalently expressed
using a for expression:
```
xs.head :: removeDuplicates(
for (x <- xs.tail if x != xs.head) yield x
)
```

## TRANSLATION OF FOR EXPRESSIONS
Every for expression can be expressed in terms of the three higher-order functions map,flatMap,
and withFilter. This section describes the translation scheme, which is also used by the Scala compiler.
Translating for expressions with one generator
First, assume you have a simple for expression:
```
for (null <- expr_1) yield expr_2
```
where x is a variable. Such an expression is translated to:
```
expr_1.map(null => expr_2)
```
Translating for expressions starting with a generator and a filter
Now, consider for expressions that combine a leading generator with some other elements.
A for expression of the form:
```
for (null <- expr_1 if expr_2) yield expr_3
```
is translated to:
```
for (null <- expr_1 withFilter (null => expr_2)) yield expr_3
```
This translation gives another for expression that is shorter by one element than the original, because
an if element is transformed into an application of withFilter on the first generator expression. The
translation then continues with this second expression, so in the end you obtain:
```
expr_1 withFilter (null => expr_2) map (null => expr_3)
```
The same translation scheme also applies if there are further elements following the filter. Ifseq is an
arbitrary sequence of generators, definitions, and filters, then:
```
for (null <- expr_1 if expr_2; seq) yield expr_3
```
is translated to:
```
for (null <- expr_1 withFilter expr_2; seq) yield expr_3
```
Then translation continues with the second expression, which is again shorter by one element than the
original one.
Translating for expressions starting with two generators
The next case handles for expressions that start with two generators, as in:
```
for (null <- expr_1; null <- expr_2; seq) yield expr_3
```
Again, assume that seq is an arbitrary sequence of generators, definitions, and filters. In fact,seq might
also be empty, and in that case there would not be a semicolon after expr_2. The translation scheme
stays the same in each case. The for expression above is translated to an application of flatMap:
```
expr_1.flatMap(null => for (null <- expr_2; seq) yield expr_3)
```
This time, there is another for expression in the function value passed to flatMap. That forexpression
(which is again simpler by one element than the original) is in turn translated with the same rules.
The three translation schemes given so far are sufficient to translate all for expressions that contain just
generators and filters, and where generators bind only simple variables. Take, for instance, the query,
"find all authors who have published at least two books,"
```
for (b1 <- books; b2 <- books if b1 != b2;
a1 <- b1.authors; a2 <- b2.authors if a1 == a2)
yield a1
```
This query translates to the following map/flatMap/filter combination:
```
books flatMap (b1 =>
books withFilter (b2 => b1 != b2) flatMap (b2 =>
b1.authors flatMap (a1 =>
b2.authors withFilter (a2 => a1 == a2) map (a2 =>
a1))))
```
The translation scheme presented so far does not yet handle generators that bind whole patterns instead
of simple variables. It also does not yet cover definitions. 

## Translating patterns in generators
The translation scheme becomes more complicated if the left hand side of generator is a pattern, pat,
other than a simple variable. The case where the for expression binds a tuple of variables is still
relatively easy to handle. In that case, almost the same scheme as for single variables applies.
A for expression of the form:
```
for ((null, ..., null) <- expr_1) yield expr_2
```
translates to:
```
expr_1.map { case (null, ..., null) => expr_2 }
```
Things become a bit more involved if the left hand side of the generator is an arbitrary
pattern pat instead of a single variable or a tuple.
In this case:
```
for (pat <- expr_1) yield expr_2
```
translates to:
```
expr_1 withFilter {
case pat => true
case _ => false
} map {
case pat => expr_2
}
```
That is, the generated items are first filtered and only those that match pat are mapped. Therefore, it's
guaranteed that a pattern-matching generator will never throw a MatchError.
The scheme here only treated the case where the for expression contains a single pattern-matching
generator. Analogous rules apply if the for expression contains other generators, filters or definitions.
Because these additional rules don't add much new insight, they are omitted from discussion here. If
you are interested, you can look them up in the Scala Language Specification [Ode11].
Translating definitions
The last missing situation is where a for expression contains embedded definitions. Here's a typical
```
case:
for (null <- expr_1; null = expr_2; seq) yield expr_3
```
Assume again that seq is a (possibly empty) sequence of generators, definitions, and filters. This
expression is translated to this one:
```
for ((null, null) <- for (null <- expr_1) yield (null, expr_2); seq)
yield expr_3
```
So you see that expr_2 is evaluated each time there is a new x value being generated. This re-evaluation
is necessary because expr_2 might refer to x and so needs to be re-evaluated for changing values of x.
For you as a programmer, the conclusion is that it's probably not a good idea to have definitions
embedded in for expressions that do not refer to variables bound by some preceding generator, because
re-evaluating such expressions would be wasteful. For instance, instead of:
```
for (x <- 1 to 1000; y = expensiveComputationNotInvolvingX)
yield x * y
```
it's usually better to write:
```
val y = expensiveComputationNotInvolvingX
for (x <- 1 to 1000) yield x * y
```

## Translating for loops
The previous subsections showed how for expressions that contain a yield are translated.What
about for loops that simply perform a side effect without returning anything? Their translation is
similar, but simpler than for expressions. In principle, wherever the previous translation scheme used
a map or a flatMap in the translation, the translation scheme for forloops uses just a foreach.
For instance, the expression:
```
for (null <- expr_1) body
```
translates to:
```
expr_1 foreach (null => body)
```
A larger example is the expression:
```
for (null <- expr_1; if expr_2; null <- expr_3) body
```
This expression translates to:
```
expr_1 withFilter (null => expr_2) foreach (null =>
expr_3 foreach (null => body))
```
For example, the following expression sums up all elements of a matrix represented as a list of lists:
```
var sum = 0
for (xs <- xss; x <- xs) sum += x
```
This loop is translated into two nested foreach applications:
```
var sum = 0
xss foreach (xs =>
xs foreach (x =>
sum += x))
```