The mysterious case of (deny dynamic-code-generation)
=====================================================

My day job is working on sandboxing for Firefox. In the context of a browser,
sandboxing refers to the processes that run web pages, generally called
"content" or "renderer" processes. These are in contrast to the "parent" or
"browser" process, which coordinates the content processes and is not
sandboxed, so it can do things like write files anywhere on disk to save
downloaded files or access the camera.

A related computer security technique is exploit mitigation. In the context of
a browser specifically memory corruption exploit mitigation. Exploit mitigation
is a family of approaches with the goal of making it harder for an attacker to
take a memory corruption vulnerability (heap or stack buffer overflow, use
after free, type confusion, etc.) and turn that vulnerability into full
arbitrary code execution. Examples of exploit mitigation are ASLR and stack
cookies. Justin Schuh, of the Chrome Security Team, has an excellent blog post
`comparing Chrome and Edge's sandboxing vs. exploit mitigation approaches`_.

A few years ago, Microsoft added a new exploit mitigation called Arbitrary Code
Guard (ACG). In short, ACG makes ``mmap`` and ``mprotect`` (I'm going to use
the POSIX names throughout this post, instead of mixing the Windows and POSIX
names) refuse to allow new pages to be created with ``PROT_EXEC``, or existing
``PROT_EXEC`` pages to be marked ``PROT_WRITE``. In unsandboxed processes, it's
common to achieve remote code execution by hijacking control flow to call
``system`` or a similar function which gives the attacker a fresh process of
their choosing. However, inside a browser sandbox an attacker can't create new
processes; instead they need to run their payload inside the process's address
space. One technique for doing this is to hijack control flow to create a new
page of executable memory, place shellcode inside that memory, and then hijack
control flow to jump to the shellcode. ACG mitigates this technique, by
preventing the attacker from creating a page of executable memory with their
shellcode; instead they are forced to use other techniques such as ROP to
encode their payload.

One of my recent projects for Firefox was investigating whether enabling ACG
made sense for us. Enabling ACG in a browser is a complex task, because
JavaScript JIT compilers rely on allocating new executable memory pages at
runtime; Edge is the only browser which currently leverages ACG. However,
that's for another blog post. Instead the rest of this post is about my
investigation into whether it was possible for us to have a mitigation like ACG
on macOS.

Folks familiar with the iOS security model probably recognize that on iOS *all*
processes have an ACG like mechanism -- besides Safari, no process is allowed
to allocate new executable pages at runtime, because of the requirement that
all code on iOS has to be signed -- however, it wasn't clear if macOS has a
similar mechanism. As a bit of a digression, macOS has two ways to sandbox your
process. One is to use macOS's `App Sandbox API`_, which is the official,
supported, and documented way to sandbox your application. At present, no
browser (Chrome, Firefox, or Safari) uses this API. Instead they all use the
private macOS "Seatbelt" API. The Seatbelt API is really a subset of Scheme
that provides a declarative syntax for expressing what permissions a process
should have. It is by far my favorite sandboxing API of any I have worked
with (primarily Windows and Linux's), the one downside is that there's no
documentation and anytime you file a bug Apple tells you that you should use
App Sandboxing instead. Oh well.

Given there's no documentation, there's two tactics we use to figure out what
functionality is available: 1) disassembling the binaries that provide the
functionality, 2) looking at the sandbox policies that macOS ships for system
binaries. So when I went to see if macOS had anything like ACG, I started
grepping around with the system profiles in
``/System/Library/Sandbox/Profiles/``. Eventually I stumbled upon ``(deny
dynamic-code-generation)`` in a few of the profiles. Off the cuff this sounded
like what I was looking for, and seemed like the sort of name that might
describe iOS's functionality. In this process I also discovered the
``file-map-executable`` policy rule, which I'll discuss later.

So I put together a pretty quick proof-of-concept to test whether this works.
All it did was apply the sandbox policy, ``mmap`` some memory, stick a simple
assembly function into it, ``mprotect`` the memory as ``PROT_EXEC``, and
attempt to call it:

.. code-block:: c++

    #include <assert.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    #include <sys/mman.h>

    #include <sandbox.h>

    static const char kSandboxPolicy[] = R"(
    (version 1)
    (deny default)
    (deny dynamic-code-generation)
    )";

    void enable_sandbox() {
      char *err;
      int rv = sandbox_init(kSandboxPolicy, 0, &err);
      if (rv) {
        fprintf(stderr, "sandbox_init error: %s\n", err);
        exit(1);
      }
    }

    static const uint8_t kShellcode[] = {
        // movq %rdi, %rax
        0x48,
        0x89,
        0xf8,
        // addq $4, %rax
        0x48,
        0x83,
        0xc0,
        0x04,
        // ret
        0xc3,
    };

    typedef uint64_t (*Function)(uint64_t);

    Function create_function(const uint8_t *shellcode, size_t length) {
      void *codebuf = mmap(nullptr, length, PROT_READ | PROT_WRITE,
                           MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
      assert(codebuf != nullptr);
      memcpy(codebuf, kShellcode, length);
      int rv = mprotect(codebuf, length, PROT_READ | PROT_EXEC);
      assert(rv == 0);

      return (Function)(codebuf);
    }

    int main() {
      enable_sandbox();
      printf("Sandbox enabled!\n");

      Function f = create_function(kShellcode, sizeof(kShellcode));
      printf("Function created!\n");

      uint64_t res = f(12);
      printf("Function called! 12 + 4 = %llu\n", res);
    }

Compile it, run it, and... it works... The program runs without error, rather
than failing to ``mprotect`` the memory ``PROT_EXEC`` or crashing as I would
have expected. I spent some time seeing if various changes would give me the
behaviour I expected: mapping the memory ``PROT_READ | PROT_WRITE | PROT_EXEC``
instead of just ``PROT_READ | PROT_EXEC``, including ``PROT_EXEC`` in the
``mmap`` rather than ``mprotect``, ``mmaping`` multiple pages instead of a
single one. But I struck out, none of these got me what I needed: an exploit
mitigation that protected against an attacker creating new executable pages.

At this point I decided that I should report this to Apple as a potential
security issue. I was a bit on the fence, since I couldn't know for certain
what the intended behaviour of ``dynamic-code-generation`` was without
documentation, and maybe it was never expected to work at all on macOS!
Nonetheless, there were a few macOS sandbox profiles that were using it which
was evidence that someone expected it to work on macOS. Plus I had a clear
reproducer, so if this was expected behaviour it should be easy enough for them
to recognize it as such.

Unfortunately this story has an unhappy ending. Apple declared that my PoC:
"did not demonstrate any behavior from dynamic-code-generation that was
unexpected." I'm still not sure what the expected behaviour is! Perhaps someone
who is better at reverse-engineering than I am will read this and figured it
out.

If ``(deny dynamic-code-generation)`` had done what I'd expected, it'd have
been the missing piece in building an ACG-alike mitigation for macOS. The other
piece, which did exist, was limiting what sorts of dynamic libraries can be
loaded. On Windows, this is achieved with Code Integrity Guard, which
requires that any DLLs which are loaded be signed. On macOS we achieve this
with the ``file-map-executable`` permission. By default, macOS's sandbox policy
allows loading a dynamic library from anywhere you can read files from. With
``file-map-executable`` you can add a deny-all rule and then whitelist
particular places on disk to load libraries from. We've now landed a patch for
Firefox which limits us to loading libraries only from system directories and
from the Firefox.app directory -- content processes can't write to any of those
directories, meaning that they can't load any attacker controlled dynamic
libraries.

I'm hopeful Apple will consider providing an ACG-like mitigation for macOS, as
they do on iOS. In the meantime, hopefully this blog post serves as a useful
resource for other folks exploring sandboxing and exploit mitigation on macOS.

.. _`comparing Chrome and Edge's sandboxing vs. exploit mitigation approaches`: https://medium.com/@justin.schuh/securing-browsers-through-isolation-versus-mitigation-15f0baced2c2
.. _`App Sandbox API`: https://developer.apple.com/library/content/documentation/Security/Conceptual/AppSandboxDesignGuide/AboutAppSandbox/AboutAppSandbox.html#//apple_ref/doc/uid/TP40011183-CH1-SW1
