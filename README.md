# Streaming Component Combinators


These Wiki pages are dedicated to the Streaming Component Combinators framework, or SCC for short.



The SCC framework is a collection of streaming components and component combinators. The components operate on data streams, and most of them start producing output before consuming their entire input. Multiple components can be grouped together into a compound component using one of the combinators. The resulting framework can be seen as a domain-specific language for stream processing.



## Features


* Data streams are homogeneous: each stream is a sequence of values of the same type.
* Components can have multiple inputs and multiple outputs, so they can form branching component networks. 
* Every component, primitive or compound, is assigned a type based on its interface.
* Components are synchronized through data flow alone; there is no global time notion.
* The framework is deterministic:
    + No stream can be consumed by more than one component.
    + Only one stream can feed a component input at a time.


## Documentation

- [Architecture](SCC.wiki/Architecture.md)
- [Tutorial](SCC.wiki/Tutorial.md)
- [Haddock reference](http://hackage.haskell.org/package/scc/)
- [Paper describing the original design](http://conferences.idealliance.org/extreme/html/2006/Blazevic01/EML2006Blazevic01.html)


## Related packages

- [incremental-parser](http://hackage.haskell.org/package/incremental-parser/)
- [monad-parallel](http://hackage.haskell.org/package/monad-parallel/)
- [monad-coroutine](http://hackage.haskell.org/package/monad-coroutine/)


## Links

- The latest release of SCC can be found at [Hackage](http://hackage.haskell.org/package/scc/).
- Development version is available in a Darcs repository at <https://hub.darcs.net/blamario/SCC/>.
