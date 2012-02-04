---
layout: post
title: "Fork/Join"
date: 2011-06-12 19:56
comments: true
categories: 
---
As our computing environment and needs change, certain programming styles and patterns which are less effective recede and others which are more applicable rise in popularity.  As these previously implicit patterns are identified and discussed they are ascribed proper names.

"Fork/Join" is one of these patterns which is gaining prominence.  It refers to both a simple and ubiquitous concept (intuitive enough that you probably hadn't thought to name it) and a very specific and narrow implementation of that concept.

Small-f "fork/join" is a general approach to parallelizing a decomposable task.  The main task is decomposed into many sub-tasks that can be run independently, these tasks are then run in parallel on multiple threads (forks), and the main task waits (joins) for their completion before proceeding.  This can be easily implemented with existing concurrency frameworks such as Java's Executors facility:

``` java
List<Callable<Output>> callables = 
  new ArrayList<Callable<Output>>(inputs.size());

for (Input i: inputs) {
  callables.add(new Callable() {
    public Output call() {
      // do some work here
      return output;
    }
  });
}

// following call performs the fork and join
List<Future<Output>> futures 
  = EXECUTOR_SERVICE.invokeAll(callables);
```

On the other hand, Fork/Join also refers to a specific application of the "fork/join" pattern which has been proposed by Doug Lea for inclusion in Java 7: <http://gee.cs.oswego.edu/dl/papers/fj.pdf>

Going back to our previous fork/join example, note that in a generic implementation tasks will be run on independent threads which will typically block on a single task queue.  When the number of threads scales up, and the size of tasks scales down, lock contention on this queue becomes problematic.

The Fork/Join implementation that Doug Lea proposes is aimed at a very specific (I would say narrow), but possibly very common case: parallel algorithms which do not block.  When a purely cpu-bound algorithm is parallelized, there will be high contention on any shared queue.  The Fork/Join implementation deals with this by using an alternate queuing arrangement call "work stealing".  Instead of all threads pulling from a shared (and contended) queue, each thread has its own dedicated (mostly uncontended) task queue (a deque).  Only when a given thread runs out of tasks will contention occur - it will attempt to "steal" a task of the end of another thread's queue.

The hierarchical (and possibly cyclical) decomposition of tasks, and execution of tasks via a "work stealing" arrangement is what is referred to by "Fork/Join" proper.

There is at least one critique of Fork/Join's inclusion in Java 7 which seems well reasoned to me: <http://www.coopsoft.com/ar/CalamityArticle.html>. I for one remain skeptical that this implementation will produce significantly better results than the simpler "fork/join" pattern with a standard thread pool and shared queue, to outweigh the cost in implementation and conceptual complexity.  However Doug contributed the Executors framework to start with, so he certainly knows his concurrency.

If you're like me then hopefully the above explanation has cleared up the term "Fork/Join" and distinguished the-pattern-i-was-using-all-along from the very specific framework/library proposal for Java 7.
