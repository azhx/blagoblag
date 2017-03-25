A survey of error handling techniques
=====================================

Different programming languages have different approaches to handling errors
that arise in code. I'm not ascribing any particular meaning to "error" here,
this post will attempt to cover everything from SIGSEGV to AttributeError, and
lots of stuff in between.

The languages, and constructs, this post will be looking at are:

* C

  * Segfaults
  * Returning / out pointer error codes
* Erlang

  * Pattern matching
  * Exceptions
  * Crashing
* Python

  * Exceptions
* Java

  * Exceptions
* Rust

  * ``Option`` and ``Result`` types
  * Task failures
* Go

  * panic/defer
  * error

I want to look at these on a few different axes:

* Propagation
* Scope of destruction
* Handling requirements

This post is meant to be entirely a survey, I'm going to try to avoid any
normative judgements.

That said: let's dive into this:

C's segfaults automatically propagate, and cannot be stopped by an intermediary
frame. Returning an error code in C doesn't automatically propagate up through
the call stack, since it's just a normal return value. A segfault kills the
entire UNIX process, regardless of which thread it occurs in. Segfaults cannot
be recovered from.

In Erlang, a pattern matching failure gets turned into an exception, and
exceptions which aren't explicitly handled propagate up through the call
stack, until they reach the top, at which point they kill the current
process (n.b. this means an Erlang process, not a UNIX process). If an
Erlang process is linked, it's possible for this to kill another process.

Python's exceptions propagate up through call frames, and any intermediary can
handle them. If an exception reaches the topmost stack frame it kills the
current thread, or if that thread is the main thread, the entire process.

Java's exceptions also automatically propagate up through call frames,
however, these must be marked in a function declaration (this is called
"checked exceptions"). A function must declare which exception types it can
raise, with a few exceptions ("unchecked exceptions") such as
``NullPointerException``.

Rust's ``Option`` and ``Result`` types do not automatically propagate,
however Rust includes a ``try!`` macro, which allows one to propagate a
result without writing out the flow control::

.. code-block:: rust

    // ``x.read()`` returns a ``Result``, if it returns an ``Err``, that will
    // be propagated up the stack, if it returns an ``Ok``, ``data`` will be
    // assigned to that value.
    let data = try!(x.read())

Rust also has task failures, which are similar to Erlang's. They kill only the
currently executing task.

Finally, Go's panics do propagate up the stack automatically, but returning
errors does not. Go's panics can be caught using the ``defer`` feature, if the
reach the topmost frame they kill the entire UNIX process, regardless of which
goroutine they came from.
