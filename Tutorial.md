
## Installation

If you have Cabal-Install installed, the following two commands should install the latest SCC package on your system:

    cabal update
    cabal install scc

If everything goes well, there should be executable named `shsh`. On Unix it gets installed in your `$HOME/.cabal/bin/` directory by default.

## Command-line Shell

To see the options supported by _shsh_, type `shsh --help` and you'll get:

    Usage: shsh (-c <command> | -f <file> | -i | -s) 
      -c       --command      Execute a single command
      -h       --help         Show help
      -f file  --file=file    Execute commands from a script file
      -i       --interactive  Execute commands interactively
      -s       --stdin        Execute commands from the standard input

Here are a few simple command examples:

<table>
<tbody>
<tr><th> Bash + GNU tools</th><th>shsh</th></tr>
<tr><td>`echo "Hello, World!"`</td><td>`echo "Hello, World!\n"`</td></tr>
<tr><td>`wc -c`</td><td>`count | show | concatenate`</td></tr>
<tr><td>`wc -l`</td><td>`foreach line then substitute x else suppress end | count | show | concatenate`</td></tr>
<tr><td>`grep "foo"`</td><td>`foreach line having substring "foo" then append "\n" else suppress end`</td></tr>
<tr><td>`sed "s:foo:bar:"`</td><td>`foreach substring "foo" then substitute "bar" end`</td></tr>
<tr><td>`sed "s:foo:[\\&]:"`</td><td>`foreach substring "foo" then prepend "[" | append "]" end`</td></tr>
<tr><td>`sed "s:foo:[\\&, \\&]:"`</td><td>`foreach substring "foo" then id; echo ", "; id end`</td></tr>
</tbody>
</table>

## Using the framework from Haskell

The shell interface is basically only syntax on top of the underlying EDSL (embedded domain-specific language) in Haskell. If you require anything more than stringing together of existing components using existing combinators, you'll need to write Haskell code.

