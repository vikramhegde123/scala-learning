### Introduction
Because modern computers have multiple cores per CPU, and often
multiple CPUs, it’s more important than ever to design programs in such a
way that they can take advantage of this parallel processing power. But the
interaction of programs that run with parallelism is complex, and the
traditional mechanism for communication among execution threads—
shared mutable memory—is notoriously difficult to reason about. This can
all too easily result in programs that have race conditions and deadlocks,
aren’t readily testable, and don’t scale well.
In this chapter, we’ll build a purely functional library for creating parallel
and asynchronous computations. We’ll rein in the complexity inherent in
parallel programs by describing them using only pure functions. This will
let us use the substitution model to simplify our reasoning and hopefully
make working with concurrent computations both easy and enjoyable.
What you should take away from this chapter is not only how to write a
library for purely functional parallelism, but how to approach the problem
of designing a purely functional library. Our main concern will be to make
our library highly composable and modular. To this end, we’ll keep with our
theme of separating the concern of describing a computation from actually
running it. We want to allow users of our library to write programs at a very
high level, insulating them from the nitty-gritty of how their programs will
be executed. For example, towards the end of the chapter we’ll develop a
combinator, parMap, that will let us easily apply a function f to every
element in a collection simultaneously:

```
val outputList = parMap(inputList)(f)
```
To get there, we’ll work iteratively. We’ll begin with a simple use case that
we’d like our library to handle, and then develop an interface that facilitates
this use case. Only then will we consider what our implementation of this
interface should be. As we keep refining our design, we’ll oscillate between
the interface and implementation as we gain a better understanding of the
domain and the design space through progressively more complex use
cases. We’ll emphasize algebraic reasoning and introduce the idea that an
API can be described by an algebra that obeys specific laws.
Why design our own library? Why not just use the concurrency primitives
that come with Scala’s standard library in the scala.concurrent package?
This is partially for pedagogical purposes—we want to show you how easy
it is to design your own practical libraries. But there’s another reason: we
want to encourage the view that no existing library is authoritative or
beyond reexamination, even if designed by experts and labeled “standard.”
There’s a certain safety in doing what everybody else does, but what’s
conventional isn’t necessarily the most practical. Most libraries contain a lot
of arbitrary design choices, many made unintentionally. When you start
from scratch, you get to revisit all the fundamental assumptions that went
into designing the library, take a different path, and discover things about
the problem space that others may not have even considered. As a result,
you might arrive at your own design that suits your purposes better. In this
particular case, our fundamental assumption will be that our library admits
absolutely no side effects.
We’ll write a lot of code in this chapter, largely posed as exercises for you,
the reader. As always, you can find the answers in the downloadable
content that goes along with the book.
7.1. Choosing data types and functions
When you begin designing a functional library, you usually have some
general ideas about what you want to be able to do, and the difficulty in the
design process is in refining these ideas and finding a data type that enables
the functionality you want. In our case, we’d like to be able to “create
parallel computations,” but what does that mean exactly? Let’s try to refine
this into something we can implement by examining a simple, parallelizable
computation—summing a list of integers. The usual left fold for this would
be as follows:
```
def sum(ints: Seq[Int]): Int =
 ints.foldLeft(0)((a,b) => a + b)
 ```

Here Seq is a superclass of lists and other sequences in the standard library.
Importantly, it has a foldLeft method.
Instead of folding sequentially, we could use a divide-and-conquer
algorithm; see the following listing.

## Summing a list using a divide-and-conquer algorithm
We divide the sequence in half using the splitAt function, recursively sum
both halves, and then combine their results. And unlike the foldLeft-based
implementation, this implementation can be parallelized—the two halves
can be summed in parallel.
The importance of simple examples
Summing integers is in practice probably so fast that parallelization
imposes more overhead than it saves. But simple examples like this are
exactly the kind that are most helpful to consider when designing a
functional library. Complicated examples include all sorts of incidental
structure and extraneous detail that can confuse the initial design process.
We’re trying to explain the essence of the problem domain, and a good way
to do this is to start with trivial examples, factor out common concerns
across these examples, and gradually add complexity. In functional design,
our goal is to achieve expressiveness not with mountains of special cases,
but by building a simple and composable set of core data types and
functions.
As we think about what sort of data types and functions could enable
parallelizing this computation, we can shift our perspective. Rather than
focusing on how this parallelism will ultimately be implemented and
forcing ourselves to work with the implementation APIs directly (likely
related to java.lang.Thread and the java.util.concurrent library), we’ll
instead design our own ideal API as illuminated by our examples and work
backward from there to an implementation.

## A data type for parallel computations
Look at the line sum(l) + sum(r), which invokes sum on the two halves
recursively. Just from looking at this single line, we can see that any data
type we might choose to represent our parallel computations needs to be
able to contain a result. That result will have some meaningful type (in this
case Int), and we require some way of extracting this result. Let’s apply
this newfound knowledge to our design. For now, we can just invent a
container type for our result, Par[A] (for parallel), and legislate the
existence of the functions we need:
def unit[A](a: => A): Par[A], for taking an unevaluated A and
returning a computation that might evaluate it in a separate thread. We
call it unit because in a sense it creates a unit of parallelism that just
wraps a single value.
def get[A](a: Par[A]): A, for extracting the resulting value from a
parallel computation.
Can we really do this? Yes, of course! For now, we don’t need to worry
about what other functions we require, what the internal representation of
Par might be, or how these functions are implemented. We’re simply
reading off the needed data types and functions by inspecting our simple
example. Let’s update this example now.

## Updating sum with our custom data type
We’ve wrapped the two recursive calls to sum in calls to unit, and we’re
calling get to extract the two results from the two subcomputations.
The problem with using concurrency primitives directly
What of java.lang.Thread and Runnable? Let’s take a look at these
classes. Here’s a partial excerpt of their API, transcribed into Scala:
Already, we can see a problem with both of these types—none of the
methods return a meaningful value. Therefore, if we want to get any
information out of a Runnable, it has to have some side effect, like mutating
some state that we can inspect. This is bad for compositionality—we can’t
manipulate Runnable objects generically since we always need to know
something about their internal behavior. Thread also has the disadvantage
that it maps directly onto operating system threads, which are a scarce
resource. It would be preferable to create as many “logical threads” as is
natural for our problem, and later deal with mapping these onto actual OS
threads.
This kind of thing can be handled by something like
java.util.concurrent .Future, ExecutorService, and friends. Why
don’t we use them directly? Here’s a portion of their API:
```
class ExecutorService {
 def submit[A](a: Callable[A]): Future[A]
}
trait Future[A] {
 def get: A
}
```

Though these are a tremendous help in abstracting over physical threads,
these primitives are still at a much lower level of abstraction than the library
we want to create in this chapter. A call to Future.get, for example, blocks
the calling thread until the ExecutorService has finished executing it, and
its API provides no means of composing futures. Of course, we can build
the implementation of our library on top of these tools (and this is in fact
what we end up doing later in the chapter), but they don’t present a modular
and compositional API that we’d want to use directly from functional
programs.
We now have a choice about the meaning of unit and get—unit could
begin evaluating its argument immediately in a separate (logical) thread,[1]
or it could simply hold onto its argument until get is called and begin
evaluation then. But note that in this example, if we want to obtain any
degree of parallelism, we require that unit begin evaluating its argument
concurrently and return immediately. Can you see why?[2]
1 We’ll use the term logical thread somewhat informally throughout this chapter to mean a computation that runs concurrently with the main execution thread of our
program. There need not be a one-to-one correspondence between logical threads and OS threads. We may have a large number of logical threads mapped onto a smaller
number of OS threads via thread pooling, for instance.
2
Function arguments in Scala are strictly evaluated from left to right, so if unit delays execution until get is called, we will both spawn the parallel computation and
wait for it to finish before spawning the second parallel computation. This means the computation is effectively sequential!
But if unit begins evaluating its argument concurrently, then calling get
arguably breaks referential transparency. We can see this by replacing sumL
and sumR with their definitions—if we do so, we still get the same result,
but our program is no longer parallel:
Par.get(Par.unit(sum(l))) + Par.get(Par.unit(sum(r)))
If unit starts evaluating its argument right away, the next thing to happen is
that get will wait for that evaluation to complete. So the two sides of the +
sign won’t run in parallel if we simply inline the sumL and sumR variables.
We can see that unit has a definite side effect, but only with regard to get.
That is, unit simply returns a Par[Int] in this case, representing an
asynchronous computation. But as soon as we pass that Par to get, we
explicitly wait for it, exposing the side effect. So it seems that we want to
avoid calling get, or at least delay calling it until the very end. We want to
be able to combine asynchronous computations without waiting for them to
finish.
Before we continue, note what we’ve done. First, we conjured up a simple,
almost trivial example. We next explored this example a bit to uncover a
design choice. Then, via some experimentation, we discovered an
interesting consequence of one option and in the process learned something
fundamental about the nature of our problem domain! The overall design
process is a series of these little adventures. You don’t need any special
license to do this sort of exploration, and you don’t need to be an expert in
functional programming either. Just dive in and see what you find.

## Combining parallel computations
Let’s see if we can avoid the aforementioned pitfall of combining unit and
get. If we don’t call get, that implies that our sum function must return a
Par[Int]. What consequences does this change reveal? Again, let’s just
invent functions with the required signatures:
def sum(ints: IndexedSeq[Int]): Par[Int] =
 if (ints.size <= 1)
 Par.unit(ints.headOption getOrElse 0)
 else {
 val (l,r) = ints.splitAt(ints.length/2)
 Par.map2(sum(l), sum(r))(_ + _)
 }

## Exercise
Par.map2 is a new higher-order function for combining the result of two
parallel computations. What is its signature? Give the most general
signature possible (don’t assume it works only for Int).
Observe that we’re no longer calling unit in the recursive case, and it isn’t
clear whether unit should accept its argument lazily anymore. In this
example, accepting the argument lazily doesn’t seem to provide any benefit,
but perhaps this isn’t always the case. Let’s come back to this question later.
What about map2—should it take its arguments lazily? It would make sense
for map2 to run both sides of the computation in parallel, giving each side
equal opportunity to run (it would seem arbitrary for the order of the map2
arguments to matter—we simply want map2 to indicate that the two
computations being combined are independent, and can be run in parallel).
What choice lets us implement this meaning? As a simple test case,
consider what happens if map2 is strict in both arguments, and we’re
evaluating sum(IndexedSeq(1,2,3,4)). Take a minute to work through and
understand the following (somewhat stylized) program trace.

## Program trace for sum
```
sum(IndexedSeq(1,2,3,4))
map2(
 sum(IndexedSeq(1,2)),
 sum(IndexedSeq(3,4)))(_ + _)
map2(
 map2(
 sum(IndexedSeq(1)),
 sum(IndexedSeq(2)))(_ + _),
 sum(IndexedSeq(3,4)))(_ + _)
map2(
 map2(
 unit(1),
 unit(2))(_ + _),
 sum(IndexedSeq(3,4)))(_ + _)
map2(
 map2(
 unit(1),
 unit(2))(_ + _),
 map2(
 sum(IndexedSeq(3)),
 sum(IndexedSeq(4)))(_ + _))(_ + _)
...
```
In this trace, to evaluate sum(x), we substitute x into the definition of sum,
as we’ve done in previous chapters. Because map2 is strict, and Scala
evaluates arguments left to right, whenever we encounter
map2(sum(x),sum(y))(_ + _), we have to then evaluate sum(x) and so on
recursively. This has the rather unfortunate consequence that we’ll strictly
construct the entire left half of the tree of summations first before moving
on to (strictly) constructing the right half. Here sum(IndexedSeq(1,2)) gets
fully expanded before we consider sum(IndexedSeq(3,4)). And if map2
evaluates its arguments in parallel (using whatever resource is being used to
implement the parallelism, like a thread pool), that implies the left half of
our computation will start executing before we even begin constructing the
right half of our computation.
What if we keep map2 strict, but don’t have it begin execution immediately?
Does this help? If map2 doesn’t begin evaluation immediately, this implies a
Par value is merely constructing a description of what needs to be
computed in parallel. Nothing actually occurs until we evaluate this
description, perhaps using a get-like function. The problem is that if we
construct our descriptions strictly, they’ll be rather heavyweight objects.
Looking back at our trace, our description will have to contain the full tree
of operations to be performed:
```
map2(
 map2(
 unit(1),
 unit(2))(_ + _),
 map2(
 unit(3),
 unit(4))(_ + _))(_ + _)
 ```
Whatever data structure we use to store this description, it’ll likely occupy
more space than the original list itself! It would be nice if our descriptions
were more lightweight.
It seems we should make map2 lazy and have it begin immediate execution
of both sides in parallel. This also addresses the problem of giving neither
side priority over the other.

## Explicit forking
Something still doesn’t feel right about our latest choice. Is it always the
case that we want to evaluate the two arguments to map2 in parallel?
Probably not. Consider this simple hypothetical example:
```
Par.map2(Par.unit(1), Par.unit(1))(_ + _)
```
In this case, we happen to know that the two computations we’re combining
will execute so quickly that there isn’t much point in spawning off a
separate logical thread to evaluate them. But our API doesn’t give us any
way of providing this sort of information. That is, our current API is very
inexplicit about when computations get forked off the main thread—the
programmer doesn’t get to specify where this forking should occur. What if
we make the forking more explicit? We can do that by inventing another
function, ``` def fork[A](a: => Par[A]): Par[A],``` which we can take to
mean that the given Par should be run in a separate logical thread:
```
def sum(ints: IndexedSeq[Int]): Par[Int] =
 if (ints.length <= 1)
 Par.unit(ints.headOption getOrElse 0)
 else {
 val (l,r) = ints.splitAt(ints.length/2)
 Par.map2(Par.fork(sum(l)), Par.fork(sum(r)))(_ + _)
 }
 ```
With fork, we can now make map2 strict, leaving it up to the programmer to
wrap arguments if they wish. A function like fork solves the problem of
instantiating our parallel computations too strictly, but more fundamentally
it puts the parallelism explicitly under programmer control. We’re
addressing two concerns here. The first is that we need some way to
indicate that the results of the two parallel tasks should be combined.
Separate from this, we have the choice of whether a particular task should
be performed asynchronously. By keeping these concerns separate, we
avoid having any sort of global policy for parallelism attached to map2 and
other combinators we write, which would mean making tough (and
ultimately arbitrary) choices about what global policy is best.
Let’s now return to the question of whether unit should be strict or lazy.
With fork, we can now make unit strict without any loss of
expressiveness. A non-strict version of it, let’s call it lazyUnit, can be
implemented using unit and fork:\
```
def unit[A](a: A): Par[A]
def lazyUnit[A](a: => A): Par[A] = fork(unit(a))
```
The function lazyUnit is a simple example of a derived combinator, as
opposed to a primitive combinator like unit. We were able to define
lazyUnit just in terms of other operations. Later, when we pick a
representation for Par, lazyUnit won’t need to know anything about this
representation—its only knowledge of Par is through the operations fork
and unit that are defined on Par.
[3]
This sort of indifference to representation is a hint that the operations are actually more general, and can be abstracted to work for types other than just Par. We’ll
explore this topic in detail in part 3.
We know we want fork to signal that its argument gets evaluated in a
separate logical thread. But we still have the question of whether it should
begin doing so immediately upon being called, or hold on to its argument, to
be evaluated in a logical thread later, when the computation is forced using
something like get. In other words, should evaluation be the responsibility
of fork or of get? Should evaluation be eager or lazy? When you’re unsure
about a meaning to assign to some function in your API, you can always
continue with the design process—at some point later the trade-offs of
different choices of meaning may become clear. Here we make use of a
helpful trick—we’ll think about what sort of information is required to
implement fork and get with various meanings.
If fork begins evaluating its argument immediately in parallel, the
implementation must clearly know something, either directly or indirectly,
about how to create threads or submit tasks to some sort of thread pool.
Moreover, this implies that the thread pool (or whatever resource we use to
implement the parallelism) must be (globally) accessible and properly
initialized wherever we want to call fork.
[4] This means we lose the ability
to control the parallelism strategy used for different parts of our program.
And though there’s nothing inherently wrong with having a global resource
for executing parallel tasks, we can imagine how it would be useful to have
more fine-grained control over what implementations are used where (we
might like for each subsystem of a large application to get its own thread
pool with different parameters, for example). It seems much more
appropriate to give get the responsibility of creating threads and submitting
execution tasks.
4 Much like the credit card processing system was accessible to the buyCoffee method in our Cafe example in chapter 1.
Note that coming to these conclusions didn’t require knowing exactly how
fork and get will be implemented, or even what the representation of Par
will be. We just reasoned informally about the sort of information required
to actually spawn a parallel task, and examined the consequences of having
Par values know about this information.
In contrast, if fork simply holds on to its unevaluated argument until later,
it requires no access to the mechanism for implementing parallelism. It just
takes an unevaluated Par and “marks” it for concurrent evaluation. Let’s
now assume this meaning for fork. With this model, Par itself doesn’t need
to know how to actually implement the parallelism. It’s more a description
of a parallel computation that gets interpreted at a later time by something
like the get function. This is a shift from before, where we were
considering Par to be a container of a value that we could simply get when
it becomes available. Now it’s more of a first-class program that we can
run. So let’s rename our get function to run, and dictate that this is where
the parallelism actually gets implemented:
```
def run[A](a: Par[A]): A
```
Because Par is now just a pure data structure, run has to have some means
of implementing the parallelism, whether it spawns new threads, delegates
tasks to a thread pool, or uses some other mechanism.

## Picking a representation
Just by exploring this simple example and thinking through the
consequences of different choices, we’ve sketched out the following API.

## Basic sketch for an API for Par
We’ve also loosely assigned meaning to these various functions:
unit promotes a constant value to a parallel computation.
map2 combines the results of two parallel computations with a binary
function.
fork marks a computation for concurrent evaluation. The evaluation
won’t actually occur until forced by run.
lazyUnit wraps its unevaluated argument in a Par and marks it for
concurrent evaluation.
run extracts a value from a Par by actually performing the
computation.
At any point while sketching out an API, you can start thinking about
possible representations for the abstract types that appear.

## Exercise
Before continuing, try to come up with representations for Par that make it
possible to implement the functions of our API.
Let’s see if we can come up with a representation. We know run needs to
execute asynchronous tasks somehow. We could write our own low-level
API, but there’s already a class that we can use in the Java Standard Library,
java.util.concurrent .ExecutorService. Here is its API, excerpted and
transcribed to Scala:
So ExecutorService lets us submit a Callable value (in Scala we’d
probably just use a lazy argument to submit) and get back a corresponding
Future that’s a handle to a computation that’s potentially running in a
separate thread. We can obtain a value from a Future with its get method
(which blocks the current thread until the value is available), and it has
some extra features for cancellation (throwing an exception after blocking
for a certain amount of time, and so on).
Let’s try assuming that our run function has access to an ExecutorService
and see if that suggests anything about the representation for Par:
```
def run[A](s: ExecutorService)(a: Par[A]): A
The simplest possible model for Par[A] might be ExecutorService => A.
This would obviously make run trivial to implement. But it might be nice to
defer the decision of how long to wait for a computation, or whether to
cancel it, to the caller of run. So Par[A] becomes ExecutorService =>
Future[A], and run simply returns the Future:
type Par[A] = ExecutorService => Future[A]
def run[A](s: ExecutorService)(a: Par[A]): Future[A] = a(s)
```

Note that since Par is represented by a function that needs an
ExecutorService, the creation of the Future doesn’t actually happen until
this ExectorService is provided.
Is it really that simple? Let’s assume it is for now, and revise our model if
we find it doesn’t allow some functionality we’d like.

## Refining the API
The way we’ve worked so far is a bit artificial. In practice, there aren’t such
clear boundaries between designing the API and choosing a representation,
and one doesn’t necessarily precede the other. Ideas for a representation can
inform the API, the API can inform the choice of representation, and it’s
natural to shift fluidly between these two perspectives, run experiments as
questions arise, build prototypes, and so on.
We’ll devote this section to exploring our API. Though we got a lot of
mileage out of considering a simple example, before we add any new
primitive operations, let’s try to learn more about what’s expressible using
those we already have. With our primitives and choices of meaning for
them, we’ve carved out a little universe for ourselves. We now get to
discover what ideas are expressible in this universe. This can and should be
a fluid process—we can change the rules of our universe at any time, make
a fundamental change to our representation or introduce a new primitive,
and explore how our creation then behaves.
Let’s begin by implementing the functions of the API that we’ve developed
so far. Now that we have a representation for Par, a first crack at it should
be straightforward. What follows is a simplistic implementation using the
representation of Par that we’ve chosen.
Listing 7.5. Basic implementation for Par
We should note that Future doesn’t have a purely functional interface. This
is part of the reason why we don’t want users of our library to deal with
Future directly. But importantly, even though methods on Future rely on
side effects, our entire Par API remains pure. It’s only after the user calls
run and the implementation receives an ExecutorService that we expose
the Future machinery. Our users therefore program to a pure interface
whose implementation nevertheless relies on effects at the end of the day.
But since our API remains pure, these effects aren’t side effects. In part 4
we’ll discuss this distinction in detail.

## Exercise
Hard: Fix the implementation of map2 so that it respects the contract of
timeouts on Future.
Exercise 7.4
This API already enables a rich set of operations. Here’s a simple example:
using lazyUnit, write a function to convert any function A => B to one that
evaluates its result asynchronously.
```
def asyncF[A,B](f: A => B): A => Par[B]
```
Adding infix syntax using implicit conversions
If Par were an actual data type, functions like map2 could be placed in the
class body and then called with infix syntax like x.map2(y)(f) (much like
we did for Stream and Option). But since Par is just a type alias, we can’t
do this directly. There’s a trick to add infix syntax to any type using implicit
conversions. We won’t discuss that here since it isn’t that relevant to what
we’re trying to cover, but if you’re interested, check out the answer code
associated with this chapter.
What else can we express with our existing combinators? Let’s look at a
more concrete example.
```
Suppose we have a Par[List[Int]] representing a parallel computation
that produces a List[Int], and we’d like to convert this to a
Par[List[Int]] whose result is sorted:
def sortPar(parList: Par[List[Int]]): Par[List[Int]]
```
We could of course run the Par, sort the resulting list, and repackage it in a
Par with unit. But we want to avoid calling run. The only other
combinator we have that allows us to manipulate the value of a Par in any
way is map2. So if we passed parList to one side of map2, we’d be able to
gain access to the List inside and sort it. And we can pass whatever we
want to the other side of map2, so let’s just pass a no-op:
```
def sortPar(parList: Par[List[Int]]): Par[List[Int]] =
 map2(parList, unit(()))((a, _) => a.sorted)
 ```
That was easy. We can now tell a Par[List[Int]] that we’d like that list
sorted. But we might as well generalize this further. We can “lift” any
function of type A => B to become a function that takes Par[A] and returns
Par[B]; we can map any function over a Par:
```
def map[A,B](pa: Par[A])(f: A => B): Par[B] =
 map2(pa, unit(()))((a,_) => f(a))
 
For instance, sortPar is now simply this:
def sortPar(parList: Par[List[Int]]) = map(parList)(_.sorted)
```
That’s terse and clear. We just combined the operations to make the types
line up. And yet, if you look at the implementations of map2 and unit, it
should be clear this implementation of map means something sensible.
Was it cheating to pass a bogus value, unit(()), as an argument to map2,
only to ignore its value? Not at all! The fact that we can implement map in
terms of map2, but not the other way around, just shows that map2 is strictly
more powerful than map. This sort of thing happens a lot when we’re
designing libraries—often, a function that seems to be primitive will turn
out to be expressible using some more powerful primitive.
What else can we implement using our API? Could we map over a list in
parallel? Unlike map2, which combines two parallel computations, parMap
(let’s call it) needs to combine N parallel computations. It seems like this
should somehow be expressible:
def parMap[A,B](ps: List[A])(f: A => B): Par[List[B]]
We could always just write parMap as a new primitive. Remember that
Par[A] is simply an alias for ExecutorService => Future[A].
There’s nothing wrong with implementing operations as new primitives. In
some cases, we can even implement the operations more efficiently by
assuming something about the underlying representation of the data types
we’re working with. But right now we’re interested in exploring what
operations are expressible using our existing API, and grasping the
relationships between the various operations we’ve defined. Understanding
what combinators are truly primitive will become more important in part 3,
when we show how to abstract over common patterns across libraries.[5]
5
In this case, there’s another good reason not to implement parMap as a new primitive—it’s challenging to do correctly, particularly if we want to properly respect
timeouts. It’s frequently the case that primitive combinators encapsulate some rather tricky logic, and reusing them means we don’t have to duplicate this logic.
Let’s see how far we can get implementing parMap in terms of existing
combinators:
def parMap[A,B](ps: List[A])(f: A => B): Par[List[B]] = {
 val fbs: List[Par[B]] = ps.map(asyncF(f))
 ...
}
Remember, asyncF converts an A => B to an A => Par[B] by forking a
parallel computation to produce the result. So we can fork off our N parallel
computations pretty easily, but we need some way of collecting their
results. Are we stuck? Well, just from inspecting the types, we can see that
we need some way of converting our List[Par[B]] to the Par[List[B]]
required by the return type of parMap.
Exercise 7.5
Hard: Write this function, called sequence. No additional primitives are
required. Do not call run.
def sequence[A](ps: List[Par[A]]): Par[List[A]]
Once we have sequence, we can complete our implementation of parMap:
def parMap[A,B](ps: List[A])(f: A => B): Par[List[B]] = fork {
 val fbs: List[Par[B]] = ps.map(asyncF(f))
 sequence(fbs)
}
Note that we’ve wrapped our implementation in a call to fork. With this
implementation, parMap will return immediately, even for a huge input list.
When we later call run, it will fork a single asynchronous computation
which itself spawns N parallel computations, and then waits for these
computations to finish, collecting their results into a list.
Exercise 7.6
Implement parFilter, which filters elements of a list in parallel.
def parFilter[A](as: List[A])(f: A => Boolean): Par[List[A]]
Can you think of any other useful functions to write? Experiment with
writing a few parallel computations of your own to see which ones can be
expressed without additional primitives. Here are some ideas to try:
Is there a more general version of the parallel summation function we
wrote at the beginning of this chapter? Try using it to find the
maximum value of an IndexedSeq in parallel.
Write a function that takes a list of paragraphs (a List[String]) and
returns the total number of words across all paragraphs, in parallel.
Generalize this function as much as possible.
Implement map3, map4, and map5, in terms of map2.
