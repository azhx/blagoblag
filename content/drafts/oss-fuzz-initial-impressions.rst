OSS-Fuzz initial impressions
============================

In case you haven't heard, this week Google announced a project called
`OSS-Fuzz`_. The basic idea of fuzz testing is take random inputs, throw them
at a program, and see if it breaks. The basic idea of OSS-Fuzz is to use
buttloads of servers that Google has lying around to do fuzz testing for open
source. OSS-Fuzz already has an impressive trophy case of vulnerabilities
found, from running over 4 trillion test cases per week.

I like open source, and I like security, and I don't have my own personal
server farm with thousands of cores, so I figured I'd give OSS-Fuzz a spin.
Here's what I found.

The first step to using OSS-Fuzz is to send a pull request proposing your
project for inclusion (OSS-Fuzz is currently limited to projects with "a
significant user base and/or be critical to the global IT infrastructure"). I
decided to go with ``libyaml``, and sending a pull request is `easy enough`_.
When OSS-Fuzz finds a crash, it files it on a private bug tracker where only
maintainers can see the details until it's fixed, so you'll see Ian Cordasco
needed to give explicit permission for me to have access to the bug reports,
since I'm not a ``libyaml`` maintainer.

Once that's accepted, you need to write the fuzzer itself. There's a few pieces
you'll need:

* A ``Dockerfile`` which clones the source and gets everything in order.
* A ``build.sh`` which compiles the library you want to test, and your fuzzing
  program.
* And the fuzzer itself: a C++ function,
  ``LLVMFuzzerTestOneInput(const uint8_t *, size_t)``.

You can also optionally include a corpus of seed files and a dictionary. For
``libyaml`` I use the ``examples/`` directory from the upstream repository as a
seed corpus, these show off various YAML syntaxes. The dictionary is a simple
list of every token I could find YAML using. Both of these help the fuzzer to
generate more interesting inputs.

All in all the pull request was `less than 100 lines of code`_. Once it's
merged, the infrastructure will notice in short order, build your fuzzer, and
it's off to the races!

So that's all the work you have to do. What do you get in return?

* Buttloads of compute power. In less than a day, OSS-Fuzz ran over 17 billion
  testcases against ``libyaml``. Based on the reported executions-per-second,
  it looks like ``libyaml`` received about 30 days of CPU time, in less than
  one calendar day.
* Life cycle management. OSS-Fuzz will automatically file a private bug when it
  discovers a crash, it'll leave a comment on the bug when it thinks the crash
  has been fixed, and then make the bug public seven days after it's been
  fixed. It also handles automatically rebuilding when the upstream source
  changes.
* Crash aggregation. OSS-Fuzz will try to automatically aggregate crashes with
  the same root cause, so you can focus on fixing them, rather than
  disentangling them.
* Coverage reports. OSS-Fuzz will show you a report of what coverage looks like
  from running all the generated inputs against your library. This helps you
  make sure that the fuzzing is finding all the interesting paths in your
  programs.

Overall, using OSS-Fuzz was an extremely good experience. It was almost no work
to write a fuzzing function and get it running, and OSS-Fuzz handles tons of
the details around making fuzzing at scale practical; this makes the experience
far more pleasant than if I'd jerry–rigged something together myself.

I'm hopeful that OSS-Fuzz will be a huge leap forward for the security of
critical open source software. If you've got a popular open source library
written in C/C++ that processes untrusted user input, you owe it to yourself to
get it running on OSS-Fuzz.

.. _`OSS-Fuzz`: https://security.googleblog.com/2016/12/announcing-oss-fuzz-continuous-fuzzing.html
.. _`easy enough`: https://github.com/google/oss-fuzz/pull/107
.. _`less than 100 lines of code`: https://github.com/google/oss-fuzz/pull/115
