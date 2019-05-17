---
title: "Introducing Innative"
subtitle: "Running WebAssembly Outside The Sandbox at 95% Native Speed"
date: 2019-05-17T05:09:04-07:00
draft: false
categories: ["news"]
---

Many people are excited about [WebAssembly](https://webassembly.org/), which lets you run languages other than JavaScript inside the browser. With WebAssembly, you can run C, C++, Rust, Zig, Go, or pretty much anything that compiles to LLVM in a sandboxed environment running inside your web browser. This has performance costs, but many people are working on highly optimized JIT compilers designed to minimize these costs as much as possible.

I am not interested in this. I am interested in answering a completely different question: **How fast can WebAssembly go if you remove the sandbox?**

If you think about it, most of your programs run outside a sandbox. This is why we have anti-virus software and "Run As Administrator", since the kernel doesn't put many restrictions on what programs can do. We just trust that the programs we install will be well behaved. What if we gave WebAssembly this same level of trust? How fast can it go? 

The answer is: ***really fast***. inNative is an AOT (Ahead-Of-Time) compiler for WebAssembly with a customizable level of sandboxed isolation. If you turn off all the isolation and let the LLVM optimizer go crazy, you can achieve near native speeds and nearly recreate the same optimized assembly that a fully optimized C++ compiler would give you, while leveraging all the features of the host CPU. Let's look at some benchmarks:



This is an average benchmark, compiled using clang `-O3 --march=native` on WSL. On average, we see 80% native speed with sandboxing and 95% without. The C++ benchmark is actually run twice - we use the second run, after the cache has had time to warm up. Turning on `fastmath` for both inNative and clang makes both go faster, but the relative speed stays the same.

The only reason we haven't already gotten to 99% native speed is because WebAssembly's 32-bit integer indexes break LLVM's vectorization due to pointer aliasing. Once fixed-width SIMD instructions are added, native WebAssembly will close the gap entirely, because this vectorization analysis will have happened before the WebAssembly compilation step.

What a lot of people don't realize is that these benchmarks give the C++ compiler a ridiculous advantage it usually doesn't have: we're using `--march=native`. Most projects stick to SSE4 instructions, lest they turn out like [No Man's Sky](https://gearnuke.com/no-mans-sky-pc-fix-crashes-sse4-1-support/). inNative has the same advantage that JIT compilers have, which is that it can _always_ take full advantage of the native processor architecture. Yet, it also has all the advantages of a traditional ahead-of-time compiler, being able to perform extremely expensive brute force optimizations by caching it's compilation result. By compiling WebAssembly on the target machine _once_, we can get the advantages of both JIT and AOT compilers at the same time!

Of course, being fast isn't the only thing inNative does, it also allows webassembly modules to interface directly with the operating system. Here is a webassembly module that uses the win32 API to create a GUI:



Unfortunately, this kind of C interop is definitely not supported by the standard yet (but there is a proposal for it). As a result, inNative uses it's own unofficial extension to allow it to pass WebAssembly pointers into C functions. inNative also lets you write C libraries that expose themselves as WebAssembly modules, which would make it possible to build an interop library in C++.
