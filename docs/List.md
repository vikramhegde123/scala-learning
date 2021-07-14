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
