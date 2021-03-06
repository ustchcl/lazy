# DEPRECATED

It turned out that many folks were unclear on the difference between laziness
and delayed computation:

  - **Delayed Computation** is when you have a value like `answer : () -> Int`
  that you can evaluate later. You only do all the computation when you say
  `answer ()` at some later time. If you call `answer ()` four times, you do
  the computation four times.

  - **Laziness** is an optimization on top of delayed computation. It is just
  like having `answer : () -> Int` but when this function is evaluated for the
  first time, the results are saved. So if you call `answer ()` four times, you
  do the computation *one* time.

In all the cases in Elm that I have heard of that use this library, folks only
really needed *delayed computation* and ended up with simpler code when they
went that way.

So in the end, there are two major reasons to stop supporting laziness:

  1. It is overkill for all the scenarios I have seen in Elm.
  2. It allows the creation of cyclic data, significantly complicating GC.

With laziness you can create a list like `ones = 1 :: ones` that refers to
itself. Without laziness, there is no way to create cyclic data in Elm. That
means we can use a naive reference counting approach to collect garbage if we
wanted. So although people have dreamed up data structures that use laziness
in interesting ways, I do not feel these cases are compelling enough to commit
to the collateral complications.

<br>

* * *

What follows is some of content from the old README.

* * *

<br>


# Pitfalls

**Laziness + Time** &mdash;
Over time, laziness can become a bad strategy. As a very simple example, think
of a timer that counts down from 10 minutes, decrementing every second. Each
step is very cheap to compute. You subtract one from the current time and store
the new time in memory, so each step has a constant cost and memory usage is
constant. Great! If you are lazy, you say &ldquo;here is how you would subtract
one&rdquo; and store that *entire computation* in memory. This means our memory
usage grows linearly as each second passes. When we finally need the result, we
might have 10 minutes of computation to run all at once. In the best case, this
introduces a delay that no one *really* notices. In the worst case, this
computation is actually too big to run all at once and crashes. Just like with
dishes or homework, being lazy over time can be quite destructive.

**Laziness + Concurrency** &mdash;
When you add concurrency into the mix, you need to be even more careful with
laziness. As an example, say we are running expensive computations on three
worker threads, and the results are sent to a fourth thread just for rendering.
If our three worker threads are doing their work lazily, they
&ldquo;finish&rdquo; super quick and pass the entire workload onto the render
thread. All the work we put into designing this concurrent system is wasted,
everything is run sequentially on the render thread! It is just like working on
a team with lazy people. You have to pay the cost of coordinating with them,
but you end up doing all the work anyway. You are better off making things
single threaded!


## Learn More

One of the most delightful uses of laziness is to create infinite streams of
values. Hopefully we can get a set of interesting challenges together so
you can run through them and get comfortable.

For a deeper dive, Chris Okasaki's book *Purely Functional Data Structures*
and [thesis](http://www.cs.cmu.edu/~rwh/theses/okasaki.pdf)
have interesting examples of data structures that get great
benefits from laziness, and hopefully it will provide some inspiration for the
problems you face in practice.

