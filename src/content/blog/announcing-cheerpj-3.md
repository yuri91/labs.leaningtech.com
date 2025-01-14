---
title: "Announcing CheerpJ 3.0: a JVM replacement in HTML5 and WebAssembly to run Java applications (and applets) on modern browsers"
description: |
  For the past year, we have been working on a new architecture for CheerpJ: our implementation of the JVM in HTML5/WebAssembly, designed to run Java applications on the browser. CheerpJ 3.0 will be released in the late summer of 2023, and will be easier to use, faster, and more compatible than ever before. Our live JavaFiddle demo has already been migrated to the current development version.
authors:
  - alessandro
pubDate: "May 9 2023"
heroImage: "/blog/blog-banners/cheerpj3.png"
featured: true
tags:
  - CheerpJ
---

_TLDR: for the past year, we have been working on a new architecture for CheerpJ: our implementation of the JVM in HTML5/WebAssembly, designed to run Java applications on the browser. CheerpJ 3.0 will be released in the late summer of 2023, and will be easier to use, faster, and more compatible than ever before. Our live [JavaFiddle](https://javafiddle.leaningtech.com) demo has already been migrated to the current development version._

[CheerpJ](https://leaningtech.com/cheerpj) is Leaning Technologies’ solution to run large-scale, unmodified Java applications and applets in the browser. The execution is fully client-side and there is no need for any server-side component beside a standard HTTP server. Over the last few years it has been our most successful product, not just commercially but also in the community, with over 100,000 users globally.

CheerpJ success stems from being able to efficiently run real-world Java applications with minimal effort, which is very useful to extend the life of legacy client-side Java applications. This is made possible by a few capabilities:

- **No source code required:** CheerpJ does not need access to the source code at all, and operates at the level of Java bytecode in .class and .jar files. Third-party libraries, dependencies and obfuscated code pose no issue.
- **Support for advanced Java features:** Any real world Java application, and the OpenJDK runtime itself, will make use of reflection, multithreading and runtime generated classes (used to implement lambdas/invokedynamic and proxies). CheerpJ fully supports all of these, requiring no adaptation of the application.
- **OpenJDK compatibility:** CheerpJ is based on an unmodified OpenJDK environment, guaranteeing the same behavior on the browser compared to a native JVM. It includes many emulation layers to ensure Filesystem, Networking, Printing, Clipboard and many other subsystems work seamlessly.

## Why is a new architecture needed?

As much as we are proud of the results we achieved with the current version of CheerpJ, over the years a few shortcomings have emerged, both in terms of technical capabilities and ease-of-use, in particular:

- **Execution model:** To achieve its performance, CheerpJ includes an AOT compiler that generates an optimized .jar.js for each .jar file of the original application. These files are loaded by CheerpJ at runtime together with their .jar counterparts, and used to speed-up execution. This model proved to be hard to understand, deploy, and integrate. The need to add CheerpJ as a post-processing step in a CI setup was often felt as an unwelcome burden, and many of our enterprise users found it hard to run the actual AOT compiler binaries on their controlled environments.
- **Limited support for ClassLoaders:** Resolution of Java class names into bytecode can be fully controlled at runtime via ClassLoaders. The AOT compilation model is not truly compatible with this level of flexibility. In real-world applications, especially those based on complex frameworks, this proved to be a more significant limitation than we expected. The AOT compilation model is also fragile when dealing with classes duplicated in multiple jars, which is quite common for logging libraries (log4j, slf4j).
- **Startup time and download size:** Java applications tend to be pretty liberal in adding dependencies, sometimes shipping a whole .jar while using just a few classes at runtime. In the current CheerpJ model, this results in many jar and jar.js files being downloaded, parsed and executed at runtime, which slows down the application startup.
- **Runtime support limited to Java 8:** The main obstacle to add support for Java 9 and later runtime versions in CheerpJ has been the implementation of Java ‘native’ methods (via the JNI), which would have required a sizeable repeated effort for each additional version/subversion of the runtime.

## The CheerpJ 3.0 architecture: a full JVM replacement in WebAssembly

We decided to take a holistic approach to solving all these problems, by redesigning the CheerpJ architecture from the ground up, while taking advantage of the lessons learned from CheerpX, our browser based x86 virtual machine.

The key features of the new CheerpJ architecture are:

- **Goodbye AOT, hello JIT compilation :** CheerpJ 3.0 features a fully transparent, multi-tier execution model, which starts with a fast interpreter for rarely used code, and combines a JIT compiler for frequently used code. With this new model no code is ever executed or generated for unused classes, improving startup performance. Because there is no AOT compiler or .jar.js files, the integration and deployment of CheerpJ 3.0 is now a matter of adding a few lines to an existing HTML page.
- **Full Classloader support:** Thanks to the new JIT approach, which mimicks how the JVM normally operates, we can now give full control to the appropriate Classloader for class resolution, including application provided ones. This completely eliminates incompatibilities caused by duplicated classes as well.
- **A new scalable JNI architecture:** We now compile 100% of the OpenJDK native code to WebAssembly, providing a viable path for supporting modern versions of Java and potentially specific point versions if a user requires so. This also means that CheerpJ now uses a completely unmodified version of OpenJDK, extending its level of compatibility even more.

Thanks to these architectural advancements, CheerpJ 3.0 can be considered a full WebAssembly-based replacement to the JVM, with a full OpenJDK runtime.

## Next steps

CheerpJ 3.0 is currently still in development, but it’s already sufficiently stable to be used in our [JavaFiddle](https://javafiddle.leaningtech.com) demo: a fully client side environment to compile, run and share Java code in the browser. The demo takes advantage of the fact that the java compiler itself is written in Java, and can hence run in CheerpJ as well.

![CheerpJ compiling and executing a Swing “Hello World” fully client-side in the browser](/blog/announcing-cheerpj-3/javafiddle.png)

We plan to officially release CheerpJ 3.0 later in summer 2023, and further announcements will be made in preparation to the release. We are extremely excited about this technology, not only for its proven capabilities of migrating existing Java applications to the browser, but also for the impact it might have on Java as a client-side language for Web development.

Our team is always available for any question, you can find us on [Discord](https://discord.leaningtech.com).

[Follow us](https://twitter.com/leaningtech) on Twitter to stay updated on all our products, including CheerpJ.

For more information: [https://leaningtech.com](https://leaningtech.com)
