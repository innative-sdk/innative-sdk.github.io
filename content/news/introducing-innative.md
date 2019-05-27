---
title: "Introducing inNative"
subtitle: "Run WebAssembly Outside The Sandbox at 95% Native Speed"
user: "Erik McClure"
userpage: "https://github.com/blackhole12"
avatar: "/img/avatar-erikm.png"
date: 2019-05-17T05:09:04-07:00
draft: false
categories: ["news"]
---

Many people are excited about [WebAssembly](https://webassembly.org/), which lets you use languages other than JavaScript on the web. With WebAssembly, you can run C, C++, Rust, Zig, Go, or pretty much anything that compiles to LLVM in a sandbox running inside your web browser. This has performance costs, but many people are working on highly optimized JIT compilers designed to minimize these costs as much as possible.

I'm not interested in that. I'm interested in a completely different question: **How fast can WebAssembly go *outside* the sandbox?**

If you think about it, most of your programs run outside a sandbox. This is why we have anti-virus software and "Run As Administrator", since the kernel doesn't put many restrictions on what programs can do. We just trust that the programs we install will be well behaved. What if we gave WebAssembly this same level of trust? How fast can it go? 

The answer is: *really fast*. [**inNative**](https://github.com/innative-sdk/innative) is an AOT (Ahead-Of-Time) compiler for WebAssembly using LLVM with a customizable level of sandboxing. You can grab [a precompiled SDK from GitHub](https://github.com/innative-sdk/innative/releases), or build from source. If you turn off all the isolation and let the LLVM optimizer go crazy, you can almost reach native speeds and nearly recreate the same optimized assembly that a fully optimized C++ compiler would give you, while leveraging all the features of the host CPU. Let's look at some benchmarks, adapted from [these C++ benchmarks](https://benchmarksgame-team.pages.debian.net/benchmarksgame/faster/cpp.html):

```
inNative v0.1.0 Test Utility

Benchmark                C/C++       Debug       Strict      Sandbox     Native
---------                -----       -----       ------      -------     ------
fac                      241499      475463      224926      212410      234685
                         1.00        0.51        1.07        1.14        1.03
nbody                    1876882     5433721     2458428     2459722     2020208
                         1.00        0.35        0.76        0.76        0.93
fannkuch_redux           1895252     5641177     2451521     2436746     1987114
                         1.00        0.34        0.77        0.78        0.95
```

This is an average benchmark, with times in microseconds, compiled using GCC `-O3 --march=native` on WSL. We usually see 75% native speed with sandboxing and 95% without. The C++ benchmark is actually run twice - we use the second run, after the cache has had time to warm up. Turning on `fastmath` for both inNative and GCC makes both go faster, but the relative speed stays the same.

The only reason we haven't already gotten to 99% native speed is because WebAssembly's 32-bit integer indexes break LLVM's vectorization due to pointer aliasing. Once [fixed-width SIMD instructions](https://github.com/webassembly/simd/blob/master/proposals/simd/SIMD.md) are added, native WebAssembly will close the gap entirely, because this vectorization analysis will have happened before the WebAssembly compilation step.

What a lot of people don't realize is that these benchmarks give the C++ compiler a ridiculous advantage it usually doesn't have: we're using `--march=native`. Most projects stick to SSE4 instructions, lest they turn out like [No Man's Sky](https://gearnuke.com/no-mans-sky-pc-fix-crashes-sse4-1-support/). inNative has the same advantage that JIT compilers have, which is that it can _always_ take full advantage of the native processor architecture. At the same time, it can perform expensive brute force optimizations like a traditional AOT compiler, by caching it's compilation result. By compiling on the target machine _once_, we get the best of _both_ Just-In-Time and Ahead-Of-Time!

Of course, being fast isn't the only thing inNative does, it also allows webassembly modules to **interface directly with the operating system**. Here is a webassembly module that uses the win32 API to create a GUI:

{{<img src="/img/wasm-win32.png" alt="inNative win32 example" width="413">}}

Unfortunately, this kind of C interop is definitely not supported by the standard yet (but there is [a proposal for it](https://github.com/WebAssembly/wasm-c-api)). As a result, inNative uses it's [own unofficial extension](https://github.com/innative-sdk/innative/wiki/inNative-cref-Extension) to allow it to pass WebAssembly pointers into C functions. inNative also lets you write C libraries that expose themselves as WebAssembly modules, which would make it possible to build an interop library in C++. Once [WebIDL bindings are standardized](https://github.com/WebAssembly/webidl-bindings/blob/master/proposals/webidl-bindings/Explainer.md), it will be a lot easier to compile WebAssembly that binds to C APIs. This opens up a world of tightly integrated WebAssembly plugins for any language that supports calling standard C interfaces, integrated directly into the program.

inNative lays the groundwork needed for us to question the basic tenets of modern software deployment: **What if we could ship WebAssembly instead?** What if shipping WebAssembly could make our software *faster* than compiling a version for every architecture? It doesn't need to be platform-independent, only *architecture-independent*. We could break the stranglehold of i386 on the software industry and free developers to experiment with novel CPU architectures without having to worry about whether our favorite language compiles to it. A WebAssembly application built against POSIX could run on any CPU architecture that implements a POSIX compatible kernel!

WebAssembly isn't just a way to run C++ in a web browser, it's a chance to reinvent how we write programs, and build a radical new foundation for software development. inNative is just a first step towards a new frontier. Check out the many ways you can [embed inNative into your program](https://github.com/innative-sdk/innative/wiki/Quick-Start-Guide), and join the conversation in our [discord chat!](https://discord.gg/teQ9Uz5)