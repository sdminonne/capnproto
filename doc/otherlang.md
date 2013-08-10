---
layout: page
title: Other Languages
---

# Other Languages

Cap'n Proto's reference implementation is in C++.  Implementations in other languages are
maintained by respective authors and have not been reviewed by me
([@kentonv](https://github.com/kentonv)).  Here are the implementations I'm aware of:

##### Ready To Use

* [C++](cxx.html) by [@kentonv](https://github.com/kentonv) -- Official reference implementation

##### Works In Progress

* [C and Go](https://github.com/jmckaskill/go-capnproto) by
  [@jmckaskill](https://github.com/jmckaskill)
* [Python (via C extensions)](https://github.com/jparyani/capnpc-python-cpp) by
  [@jparyani](https://github.com/jparyani)
* [Rust](https://github.com/dwrensha/capnproto-rust) by [@dwrensha](https://github.com/dwrensha)

## Contribute Your Own!

We'd like to support many more languages in the future!

If you'd like to own the implementation of Cap'n Proto in some particular language,
[let us know](https://groups.google.com/group/capnproto)!

**You should e-mail the list _before_ you start hacking.**  We don't bite, and we'll probably have
useful tips that will save you time.  :)

**Do not implement your own schema parser.**  The schema language is more complicated than it
looks, and the algorithm to determine offsets of fields is subtle.  If you reuse the official
parser, you won't risk getting these wrong, and you won't have to spend time keeping your parser
up-to-date.  In fact, you can still write your code generator in any language you want, using
compiler plugins!

### How to Write Compiler Plugins

The Cap'n Proto tool, `capnp`, does not actually know how to generate code.  It only parses schemas,
then hands the parse tree off to another binary -- known as a "plugin" -- which generates the code.
Plugins are independent executables (written in any language) which read a description of the
schema from standard input and then generate the necessary code.  The description is itself a
Cap'n Proto message, defined by
[schema.capnp](https://github.com/kentonv/capnproto/blob/master/c%2B%2B/src/capnp/schema.capnp).
Specifically, the plugin receives a `CodeGeneratorRequest`, using
[standard serialization](http://kentonv.github.io/capnproto/encoding.html#serialization_over_a_stream)
(not packed).  (Note that installing the C++ runtime causes schema.capnp to be placed in
`$PREFIX/include/capnp` -- `/usr/local/include/capnp` by default).

Of course, because the input to a plugin is itself in Cap'n Proto format, if you write your
plugin directly in the language you wish to support, you may have a bootstrapping problem:  you
somehow need to generate code for `schema.capnp` before you write your code generator.  Luckily,
because of the simplicity of the Cap'n Proto format, it is generally not too hard to do this by
hand.  Remember that you can use `capnp compile -ocapnp schema.capnp` to get a dump of the sizes
and offsets of all structs and fields defined in the file.

`capnp compile` normally looks for plugins in `$PATH` with the name `capnpc-[language]`, e.g.
`capnpc-c++` or `capnpc-capnp`.  However, if the language name given on the command line contains
a slash character, `capnp` assumes that it is an exact path to the plugin executable, and does not
search `$PATH`.  Examples:

    # Searches $PATH for executable "capnpc-mylang".
    capnp compile -o mylang addressbook.capnp

    # Uses plugin executable "myplugin" from the current directory.
    capnp compile -o ./myplugin addressbook.capnp

If the user specifies an output directory, the compiler will run the plugin with that directory
as the working directory, so you do not need to worry about this.

For examples of plugins, take a look at
[capnpc-capnp](https://github.com/kentonv/capnproto/blob/master/c%2B%2B/src/capnp/compiler/capnpc-capnp.c%2B%2B)
or [capnpc-c++](https://github.com/kentonv/capnproto/blob/master/c%2B%2B/src/capnp/compiler/capnpc-c%2B%2B.c%2B%2B).

### Supporting Dynamic Languages

Dynamic languages have no compile step.  This makes it difficult to work `capnp compile` into the
workflow for such languages.  Additionally, dynamic languages are often scripting languages that do
not support pointer arithmetic or any reasonably-performant alternative.

Fortunately, dynamic languages usually have facilities for calling native code.  The best way to
support Cap'n Proto in a dynamic language, then, is to wrap the C++ library, in particular the
[C++ dynamic API](cxx.html#dynamic_reflection).  This way you get reasonable performance while
still avoiding the need to generate any code specific to each schema.

Of course, you still need to parse the schema.  Version 0.3 of Cap'n Proto will introduce a public
C++ API to the schema parser which your bindings can invoke.  By the time you read this, the API
is probably already available at git head, or will be within a few days;
[send us a note](https://groups.google.com/group/capnproto) if you want to try it out.