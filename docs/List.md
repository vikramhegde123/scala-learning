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



