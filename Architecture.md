
The SCC framework is implemented in multiple layers. Lower layers are useful by themselves.

## Infrastructure

The framework relies on the following packages:

* [monad-parallel](http://hackage.haskell.org/package/monad-parallel/) to enable parallel execution of monadic computations, and
* [monad-coroutine](http://hackage.haskell.org/package/monad-coroutine/) for the ability to suspend and resume monadic computations.

## Streams
### Module `Control.Concurrent.SCC.Streams`

The lowest layer builds on the monad-coroutine foundation to provide streaming computations. The main idea here is to introduce sinks and sources:

    
    data Sink (m :: * -> *) a x = Sink {
       put :: forall d. (AncestorFunctor a d) => x -> Coroutine d m ()
       }

    newtype Source (m :: * -> *) a x = Source {
       get :: forall d. (AncestorFunctor a d) => Coroutine d m (Maybe x)
       }
    

The only way to obtain a new source to read from, or a sink to write to, is by launching a new nested coroutine using the function pipe:

    pipe :: forall m a a1 a2 x r1 r2. (Monad m, Functor a, a1 ~ SinkFunctor a x, a2 ~ SourceFunctor a x) =>
            (Sink m a1 x -> Coroutine a1 m r1) -> (Source m a2 x -> Coroutine a2 m r2) -> Coroutine a m (r1, r2)


This function takes two coroutines as arguments, ''producer'' and ''consumer''. The producer takes a sink argument, and the consumer a source argument. All data that the producer writes to the sink can be read from the source by the consumer. The arrangement couldn't be simpler. Here is a contrived example of a producer/consumer pair:

    import Control.Concurrent.Coroutine
    import Control.Concurrent.SCC.Streams

    main = runCoroutine (pipe (rangeProducer 1 10) (sumConsumer 0)) >>= print

    rangeProducer min max sink | min > max = return ()
                               | otherwise = put sink min
                                             >> rangeProducer (succ min) max sink
    sumConsumer sum source = get source
                             >>= maybe (return sum)
                                       (\n-> sumConsumer (sum + n) source)

The `rangeProducer` coroutine produces a range of integers while the `sumConsumer` consumes all the numbers and sums them up. The two coroutines are bound together by `pipe`. Note that either of them could be replaced by a different coroutine of the same type and the other coroutine would still perform its job. Furthermore, though in this example both coroutines run in the IO monad, they are completely generic and could run in any monad.

## Components and combinators
### Module `Control.Concurrent.SCC.Types`
### Module `Control.Concurrent.SCC.Coercions`
### Module `Control.Concurrent.SCC.Primitives`
### Module `Control.Concurrent.SCC.Combinators`
### Module `Control.Concurrent.SCC.XML`

What can one do with a number of sources and sinks? The next layer tries to organize the answers. First, in module `Types` it defines the types of various actors on sources and sinks:

    type OpenConsumer m a d x r = AncestorFunctor a d => Source m a x -> Coroutine d m r
    type OpenProducer m a d x r = AncestorFunctor a d => Sink m a x -> Coroutine d m r
    type OpenTransducer m a1 a2 d x y r = 
       (AncestorFunctor a1 d, AncestorFunctor a2 d) => Source m a1 x -> Sink m a2 y -> Coroutine d m r
    type OpenSplitter m a1 a2 a3 a4 d x b r =
       (AncestorFunctor a1 d, AncestorFunctor a2 d, AncestorFunctor a3 d, AncestorFunctor a4 d) =>
       Source m a1 x -> Sink m a2 x -> Sink m a3 x -> Sink m a4 b -> Coroutine d m r

    newtype Consumer m x r = Consumer {consume :: forall a d. OpenConsumer m a d x r}
    newtype Producer m x r = Producer {produce :: forall a d. OpenProducer m a d x r}
    newtype Transducer m x y = Transducer {transduce :: forall a1 a2 d. OpenTransducer m a1 a2 d x y ()}
    newtype Splitter m x b = Splitter {split :: forall a1 a2 a3 a4 d. OpenSplitter m a1 a2 a3 a4 d x b ()}

The `Primitives` module exports a number of functions that create ''primitive'' consumer, producer, transducer, and splitter components. They are primitive in the sense of not depending on any other components.

The `Combinators` module exports functions that combine primitive (or any other) components into more complex components.

Module `XML` groups together the primitive components and their combinators useful for parsing, manipulating, and writing XML.

## Dynamic component configuration
### Module `Control.Concurrent.Configuration`

This is not so much a layer as an overlay, because it's quite generic. Any value can be made into a configurable component, provided that we supply the following information about it:

 * a name,
 * the maximum number of threads it can use, and
 * the cost of using the component.

In exchange for this data, the component provides the ability to optimize itself for any given number of available threads.

## The entire library
### Module `Control.Concurrent.SCC.Sequential`
### Module `Control.Concurrent.SCC.Parallel`
### Module `Control.Concurrent.SCC.Configurable`

Each of these three modules collect and re-export all lower-level modules of SCC with different configuration. Components and combinators exported by module ''Sequential'' always run sequentally, using a single thread. Those exported from ''Parallel'' always fork extra threads whenever they can. Finally, the ''Configurable'' module exports components that can be dynamically configured to use a given number of threads in the optimal way.

## The highest layer: Command-line shell language
### Module `Shell`

This layer exposes the entire framework functionality in a command-line shell interface.
