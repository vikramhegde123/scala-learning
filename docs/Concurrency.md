## Futures
A Future represents a value which may or may not currently be available,
but will be available at some point, or an exception if that value could not
be made available.

To help demonstrate this, in single-threaded programming you bind the result of a
function call to a variable like this:

```
def aShortRunningTask(): Int = 42
val x = aShortRunningTask
```
With code like that, the value 42 is bound to the variable x immediately.
When you’re working with a Future, the assignment process looks similar:

```
def aLongRunningTask(): Future[Int] = ???
val x = aLongRunningTask
```

### Executor Context
Before we create any Future, we need to provide an implicit ExecutionContext. This specifies how and on which thread pool the Future code will execute. We can create it from Executor or ExecutorService:

```
val forkJoinPool: ExecutorService = new ForkJoinPool(4)
implicit val forkJoinExecutionContext: ExecutionContext =
ExecutionContext.fromExecutorService(forkJoinPool)

val singleThread: Executor = Executors.newSingleThreadExecutor()
implicit val singleThreadExecutionContext: ExecutionContext =
ExecutionContext.fromExecutor(singleThread)
```
There is also a global built-in ExecutionContext, which uses ForkJoinPool with its parallelism level set to the number of available processors:
```
implicit val globalExecutionContext: ExecutionContext = ExecutionContext.global
```
In the following sections, we’ll use ExecutionContext.global, making it available using a single import:
```
import scala.concurrent.ExecutionContext.Implicits.global
```

### Await for Future
We’ve already created a Future, so we need a way to wait for its result:

```
val maxWaitTime: FiniteDuration = Duration(5, TimeUnit.SECONDS)
val magicNumber: Int = Await.result(generatedMagicNumberF, maxWaitTime)
```
Await.result blocks the main thread and waits a defined duration for the result of the given Future. If it’s not ready after that time or complete with a failure, Await.result throws an exception.
In this example, we’re waiting a maximum of 5 seconds for the generatedMagicNumberF result.

If we wanted to wait forever, we should use Duration.Inf.

Because Await.result blocks the main thread, we should only use it when we really need to wait. If we want to transform the Future result or combine it with others, then we can do that in non-blocking ways, which we’ll look at later on.

### Callbacks - OnComplete
Instead of waiting for the Future result and blocking the main thread, we can register a callback using the onComplete method:
```
def printResult[A](result: Try[A]): Unit = result match {
  case Failure(exception) => println("Failed with: " + exception.getMessage)
  case Success(number)    => println("Succeed with: " + number)
}
magicNumberF.onComplete(printResult)
```
In this case, the Future executes the printResult method when the magicNumberF completes, either successfully or with a failure.
### Callbacks - Foreach
If we want to call a callback only when the Future completes successfully, we should use the foreach method:
```
def printSucceedResult[A](result: A): Unit = println("Succeed with: " + result)
magicNumberF.foreach(printSucceedResult)
Unlike the onComplete method, this won’t execute the given callback function when magicNumberF completes with failure.
```
It has the same meaning as the foreach method on Try or Option.

### Error Handling - failed
If we have a use-case that depends on knowing if a given Future failed with the appropriate exception, we want to treat failure as success. For this purpose, we have the failed method:
```
val failedF: Future[Int] = Future.failed(new IllegalArgumentException("Boom!"))
val failureF: Future[Throwable] = failedF.failed
```
It tries to transform a failed Future into a successfully completed one with Throwable as a result. If Future completes successfully, then the resulting Future will fail with NoSuchElementException.

### Error Handling - fallbackTo
Let’s imagine that we have a DatabaseRepository that is able to read a magic number from the database and FileBackup that reads a magic number from the latest backup file (created once per day):

```
trait DatabaseRepository {
def readMagicNumber(): Future[Int]
def updateMagicNumber(number: Int): Future[Boolean]
}
trait FileBackup {
def readMagicNumberFromLatestBackup(): Future[Int]
}
```

Because this number is crucial for our business, we want to read it from the latest backup in case of any problem with the database. In this situation, we should use the fallbackTo method:
```
trait MagicNumberService {
val repository: DatabaseRepository
val backup: FileBackup

val magicNumberF: Future[Int] =
repository.readMagicNumber()
.fallbackTo(backup.readMagicNumberFromLatestBackup())
}
```
It takes an alternative Future in case of a failure of the current one and evaluates them simultaneously. If both fail, the resulting Future will fail with the Throwable taken from the current one.

### Error Handling - recover
If we want to handle a particular exception by providing an alternative value, we should use the recover method:

```
val recoveredF: Future[Int] = Future(3 / 0).recover {
  case _: ArithmeticException => 0
}
```
It takes a partial function that turns any matching exception into a successful result. Otherwise, it will keep the original exception.

### Error Handling - recoverWith
If we want to handle a particular exception with another Future, we should use recoverWith instead of recover:

```
val recoveredWithF: Future[Int] = Future(3 / 0).recoverWith {
  case _: ArithmeticException => magicNumberF
}
```

### Transform Future - map
When we have a Future instance, we can use the map method to transform its successful result without blocking the main thread:

```
def increment(number: Int): Int = number + 1
val nextMagicNumberF: Future[Int] = magicNumberF.map(increment)
```
It creates a new Future[Int] by applying the increment method to the successful result of magicNumberF. Otherwise, the new one will contain the same exception as magicNumberF.

Evaluation of the increment method happens on another thread taken from the implicit ExecutionContext.

### Transform Future - flatMap

If we want to transform a Future using a function that returns Future, then we should use the flatMap method:
```
val updatedMagicNumberF: Future[Boolean] =
nextMagicNumberF.flatMap(repository.updateMagicNumber)
```
It behaves in the same way as the map method but keeps the resulting Future flat, returning Future[Boolean] instead of Future[Future[Boolean]].

Having the flatMap and map methods gives us the ability to write code that’s easier to understand.

### Transform Future - transform
As opposed to the map() function, we can map both successful and failed cases with the transform(f: Try[T] => Try[S]) function:
```
val value = Future.successful(42)
val transformed = value.transform {
case Success(value) => Success(s"Successfully computed the $value")
case Failure(cause) => Failure(new IllegalStateException(cause))
}
```
As shown above, the transform() method accepts a function that takes the Future result as the input and returns a Try instance as the output. In this case, we’re converting the given Future[Int] to a Future[String].

This method has an overloaded version that takes two functions as the input — one for a successful case, and the other for failure scenarios:
```
val overloaded = value.transform(
value => s"Successfully computed $value",
cause => new IllegalStateException(cause)
)
```
The first function maps the successful result to something else, and the second one maps the thrown exception.

### Transform Future - transformWith
Similar to the transform() function, the transformWith(f: Try[T] => Future[S]) accepts a function as the input. This function, however, converts the given Try instance directly to a Future instance:
```
value.transformWith {
case Success(value) => Future.successful(s"Successfully computed the $value")
case Failure(cause) => Future.failed(new IllegalStateException(cause))
}
```
As shown above, the return type of the input function is a Future instead of a Try.

### Transform Future - andThen

The andThen() function applies a side-effecting function to the given Future and returns the same Future:

```
Future.successful(42).andThen {
case Success(v) => println(s"The answer is $v")
}
```
As shown above, the andThen() consumes the successful result here. Since the andThen() function returns the same Future after applying the given function, we can chain multiple andThen() invocations together:

```
val f = Future.successful(42).andThen {
case Success(v) => println(s"The answer is $v")
} andThen {
case Success(_) => // send HTTP request to signal success
case Failure(_) =>  // send HTTP request to signal failure
}

f.onComplete { v =>
println(s"The original future has returned: ${v}")
}
```

As shown above, the variable f still contains the original Future after applying two side-effecting functions.