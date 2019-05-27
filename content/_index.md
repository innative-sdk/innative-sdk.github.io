---
title: "inNative WebAssembly Runtime"
date: 2019-05-17T05:09:04-07:00
tags: ""
draft: false
type: index
summary: "Compile WebAssembly to native binaries, or embed inNative and provide your own WebAssembly execution environment."
---
inNative is an AOT (ahead-of-time) compiler for WebAssembly that creates **C compatible binaries**, either as **sandboxed plugins** you can dynamically load, or as **stand-alone executables** that interface *directly with the operating system*. This allows webassembly modules to participate in C linking and the build process, either statically, dynamically, or with access to the host operating system. The runtime can be installed standalone on a user's machine, or it can be embedded inside your program. It's highly customizable, letting you choose the features, isolation level, and optimization amount you need for your use-case.

<p>Join a team of webassembly enthusiasts working hard to make inNative a more viable target for compilers, and building better integration options for non-web embeddings in general. We maintain an [active discord server](https://discord.gg/teQ9Uz5), and post articles from community members [on our website](/news/). If you find a bug, or your program can't compile on inNative until we implement a specific feature, [file an issue](https://github.com/innative-sdk/innative/issues/new) on GitHub so we can track the needs of developers.

### Documentation
The primary source of documentation for inNative is the [GitHub Wiki](https://github.com/innative-sdk/innative/wiki), which lists all the externally accessible functions and how to use them. This wiki should be kept up to date, but always double-check the source code comments if something seems amiss. Feel free to [file an issue](https://github.com/innative-sdk/innative/issues/new) if there is misleading or missing documentation.