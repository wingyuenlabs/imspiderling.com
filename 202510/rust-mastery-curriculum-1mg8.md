---
Title: Rust Mastery Curriculum
Description: 
Author: Cargo
Date: 2025-10-24T21:24:12.000Z
Robots: noindex,nofollow
Template: index
---
<h1>
  
  
  Deep Rust Mastery Curriculum (Free Resources Edition)
</h1>




<h2>
  
  
  <strong>Philosophy &amp; Approach</strong>
</h2>

<p>This curriculum prioritizes <strong>understanding over memorization</strong>. You'll build mental models of how Rust works at a fundamental level, enabling you to reason about complex problems rather than pattern-match solutions.</p>




<h2>
  
  
  <strong>Phase 1: Foundation &amp; Mental Models (4-6 weeks)</strong>
</h2>

<h3>
  
  
  Week 1-2: The Ownership Revolution
</h3>

<p><strong>Goal:</strong> Internalize why ownership exists, not just how it works</p>

<p><strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://doc.rust-lang.org/book/" rel="noopener noreferrer">The Rust Programming Language</a></strong> - Chapters 1-6</li>
<li>📚 <strong><a href="https://doc.rust-lang.org/rust-by-example/" rel="noopener noreferrer">Rust By Example</a></strong> - Parallel reading for hands-on practice</li>
<li>🎥 <strong><a href="https://www.youtube.com/c/LetsGetRusty" rel="noopener noreferrer">Let's Get Rusty - Ownership playlist</a></strong> - Visual explanations</li>
<li>🎓 <strong><a href="https://github.com/rust-lang/rustlings" rel="noopener noreferrer">Rustlings</a></strong> - Exercises on ownership, move semantics</li>
</ul>

<p><strong>Deep Dive Activities:</strong></p>

<ul>
<li>Trace memory allocation/deallocation manually on paper for simple programs</li>
<li>Compare Rust's approach to C++ RAII, garbage collection, and manual memory management</li>
<li>Read <a href="https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html" rel="noopener noreferrer">this article on ownership</a> multiple times</li>
</ul>

<p><strong>Exercise:</strong> </p>

<ul>
<li>Complete all ownership-related Rustlings exercises</li>
<li>Build a simple vector-like type from scratch</li>
<li>
<strong><a href="https://rust-unofficial.github.io/too-many-lists/" rel="noopener noreferrer">Learn Rust With Entirely Too Many Linked Lists</a></strong> - Start with "A Bad Stack"</li>
</ul>

<p><strong>Reflection:</strong> Write explanations of ownership as if teaching a 10-year-old, then a C programmer, then a Python developer</p>




<h3>
  
  
  Week 3-4: Borrowing &amp; Lifetimes—The Hard Parts
</h3>

<p><strong>Goal:</strong> Develop intuition for the borrow checker</p>

<p><strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://doc.rust-lang.org/book/" rel="noopener noreferrer">The Book</a></strong> - Chapters 7-11</li>
<li>📚 <strong><a href="https://rust-unofficial.github.io/too-many-lists/" rel="noopener noreferrer">Too Many Linked Lists</a></strong> - Continue through all chapters</li>
<li>📝 <strong><a href="https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md" rel="noopener noreferrer">Common Rust Lifetime Misconceptions</a></strong> - Essential reading!</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=rAl-9HwD858" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Lifetime Annotations</a></strong>
</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=iVYWDIW71jk" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Subtyping and Variance</a></strong>
</li>
</ul>

<p><strong>Deep Dive:</strong></p>

<ul>
<li>Manually desugar lifetime elision rules</li>
<li>Study the borrowing flowchart: when is <code>&amp;T</code> sufficient vs <code>&amp;mut T</code> vs owned <code>T</code>?</li>
<li>Work through <strong><a href="https://practice.rs/ownership/ownership.html" rel="noopener noreferrer">Rust by Practice - Ownership section</a></strong>
</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Build a simple text editor buffer with efficient insertions</li>
<li>Implement a graph structure with multiple ownership patterns (<code>Rc</code>, <code>RefCell</code>)</li>
<li>Create a custom iterator that borrows from its parent structure</li>
<li>Complete <strong><a href="https://exercism.org/tracks/rust" rel="noopener noreferrer">Exercism - Rust Track</a></strong> exercises: Clock, Circular Buffer, Doubly Linked List</li>
</ul>




<h3>
  
  
  Week 5-6: Type System Mastery
</h3>

<p><strong>Goal:</strong> Think in types, not just code</p>

<p><strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://doc.rust-lang.org/book/" rel="noopener noreferrer">The Book</a></strong> - Chapters 10, 17, 19</li>
<li>📝 <strong><a href="https://rust-unofficial.github.io/patterns/" rel="noopener noreferrer">Rust Design Patterns</a></strong> - All patterns</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=yozQ9C69pNs" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Iterators</a></strong>
</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=8O0Nt9qY_vo" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Smart Pointers and Interior Mutability</a></strong>
</li>
<li>📝 <strong><a href="https://rust-lang.github.io/api-guidelines/" rel="noopener noreferrer">Rust API Guidelines</a></strong> - Design principles</li>
<li>📝 <strong><a href="https://blog.rust-lang.org/2015/05/11/traits.html" rel="noopener noreferrer">Abstraction without overhead: traits in Rust</a></strong>
</li>
</ul>

<p><strong>Deep Dive:</strong></p>

<ul>
<li>Trait objects vs generics: monomorphization costs and tradeoffs</li>
<li>Marker traits (<code>Send</code>, <code>Sync</code>, <code>Copy</code>, <code>Sized</code>) and their implications</li>
<li>Associated types vs generic parameters—when to use each</li>
<li>Read the <a href="https://doc.rust-lang.org/std/iter/" rel="noopener noreferrer">std::iter module docs</a> completely</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Implement a small serialization library using traits (inspired by serde)</li>
<li>Build a plugin system using trait objects</li>
<li>Create your own <code>Result</code>-like type with custom error handling</li>
<li>Study <a href="https://github.com/serde-rs/serde" rel="noopener noreferrer">Serde's source code</a> to understand trait magic</li>
</ul>




<h2>
  
  
  <strong>Phase 2: Concurrency &amp; Async (6-8 weeks)</strong>
</h2>

<h3>
  
  
  Week 7-9: Fearless Concurrency Foundations
</h3>

<p><strong>Goal:</strong> Understand why Rust makes concurrency safe</p>

<p><strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://doc.rust-lang.org/book/ch16-00-concurrency.html" rel="noopener noreferrer">The Book Chapter 16</a></strong> - Fearless Concurrency</li>
<li>📚 <strong><a href="https://marabos.nl/atomics/" rel="noopener noreferrer">Rust Atomics and Locks</a></strong> by Mara Bos - FREE ONLINE! Read chapters 1-6</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=b4mS5UPHh20" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Channels</a></strong>
</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=rMGWeSjctlY" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Atomics and Memory Ordering</a></strong>
</li>
<li>📝 <strong><a href="https://doc.rust-lang.org/nomicon/concurrency.html" rel="noopener noreferrer">The Rustonomicon - Concurrency</a></strong>
</li>
</ul>

<p><strong>Deep Dive:</strong></p>

<ul>
<li>How <code>Send</code> and <code>Sync</code> prevent data races at compile time</li>
<li>Memory ordering: Relaxed, Acquire, Release, SeqCst</li>
<li>Interior mutability pattern: <code>Cell</code>, <code>RefCell</code>, <code>Mutex</code>, <code>RwLock</code>
</li>
<li>Read <a href="https://doc.rust-lang.org/std/sync/" rel="noopener noreferrer">std::sync module docs</a> thoroughly</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Build a thread pool from scratch (follow along with <strong><a href="https://doc.rust-lang.org/book/ch20-00-final-project-a-web-server.html" rel="noopener noreferrer">Rust Book Chapter 20</a></strong>)</li>
<li>Implement a concurrent hash map using <code>RwLock</code>
</li>
<li>Create a work-stealing scheduler (study <a href="https://github.com/rayon-rs/rayon" rel="noopener noreferrer">rayon's approach</a>)</li>
<li>Complete exercises from <strong><a href="https://marabos.nl/atomics/" rel="noopener noreferrer">Rust Atomics and Locks</a></strong>
</li>
</ul>




<h3>
  
  
  Week 10-12: Async Rust—The Complete Picture
</h3>

<p><strong>Goal:</strong> Understand async from first principles</p>

<p><strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://rust-lang.github.io/async-book/" rel="noopener noreferrer">Asynchronous Programming in Rust</a></strong> - Official async book</li>
<li>📚 <strong><a href="https://tokio.rs/tokio/tutorial" rel="noopener noreferrer">Tokio Tutorial</a></strong> - Build mini-Redis (DO THIS!)</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=ThjvMReOXYM" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Async/Await</a></strong>
</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=9_3krAQtD2k" rel="noopener noreferrer">Jon Gjengset - The What and How of Futures and async/await in Rust</a></strong>
</li>
<li>📝 <strong><a href="https://tokio.rs/tokio/tutorial/async" rel="noopener noreferrer">Tokio internals: Understanding Rust's async runtime</a></strong>
</li>
<li>📝 <strong><a href="https://blog.cloudflare.com/pin-and-unpin-in-rust/" rel="noopener noreferrer">Pin, Unpin, and why Rust needs them</a></strong>
</li>
</ul>

<p><strong>Deep Dive:</strong></p>

<ul>
<li>What is a <code>Future</code>? Implement your own simple future following <strong><a href="https://rust-lang.github.io/async-book/02_execution/02_future.html" rel="noopener noreferrer">this guide</a></strong>
</li>
<li>How polling works: <code>Poll::Ready</code> vs <code>Poll::Pending</code>
</li>
<li>The <code>Waker</code> API and how runtimes schedule tasks</li>
<li>Pin and why it exists (this is hard—spend time here)</li>
<li>Read <a href="https://docs.rs/futures/latest/futures/" rel="noopener noreferrer">futures crate docs</a>
</li>
</ul>

<p><strong>Critical Understanding:</strong></p>

<ul>
<li>Async is about <strong>I/O concurrency</strong>, not parallelism</li>
<li>The "function coloring" problem and its implications</li>
<li>Structured concurrency with <code>select!</code> and <code>join!</code>
</li>
</ul>

<p><strong>Required Reading:</strong></p>

<ul>
<li>📝 <strong><a href="https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/" rel="noopener noreferrer">What Color is Your Function?</a></strong> - Understanding async limitations</li>
<li>📝 <strong><a href="https://ryhl.io/blog/async-what-is-blocking/" rel="noopener noreferrer">Async: What is blocking?</a></strong>
</li>
</ul>




<h3>
  
  
  Week 13-14: Async Ecosystems &amp; Patterns
</h3>

<p><strong>Goal:</strong> Master practical async programming</p>

<p><strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://tokio.rs/tokio/topics/bridging" rel="noopener noreferrer">Tokio Topics</a></strong> - All advanced topics</li>
<li>📝 <strong><a href="https://tokio.rs/tokio/topics/tracing" rel="noopener noreferrer">Tokio's tracing documentation</a></strong>
</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=bzja9fQWzdA" rel="noopener noreferrer">Jon Gjengset - Implementing TCP in Rust</a></strong> - Deep async networking</li>
<li>📝 <strong><a href="https://ryhl.io/blog/" rel="noopener noreferrer">Alice Ryhl's Tokio blog posts</a></strong> - All of them!</li>
<li>📚 <strong><a href="https://book.async.rs/" rel="noopener noreferrer">async-std book</a></strong> - Compare with Tokio</li>
</ul>

<p><strong>Study:</strong></p>

<ul>
<li>Tokio vs async-std vs smol: architectural differences</li>
<li>Channels: mpsc, broadcast, watch, oneshot</li>
<li>Async traits and why they're challenging</li>
<li>Read <a href="https://github.com/tokio-rs/tokio" rel="noopener noreferrer">tokio source code</a> - start with <code>tokio::time</code>
</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Complete the <strong><a href="https://tokio.rs/tokio/tutorial" rel="noopener noreferrer">Tokio mini-Redis tutorial</a></strong> fully</li>
<li>Build an async HTTP server from scratch (without frameworks) - follow <strong><a href="https://www.youtube.com/watch?v=MZPXLNLlagc" rel="noopener noreferrer">Jon Gjengset's stream</a></strong>
</li>
<li>Implement a connection pool with timeouts and backpressure</li>
<li>Create a pub-sub system using Tokio's broadcast channels</li>
<li>Build a rate limiter using async semaphores</li>
</ul>




<h2>
  
  
  <strong>Phase 3: Advanced Patterns &amp; Systems (8-12 weeks)</strong>
</h2>

<h3>
  
  
  Week 15-17: Zero-Cost Abstractions
</h3>

<p><strong>Goal:</strong> Write high-level code that compiles to optimal machine code</p>

<p><strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://nnethercote.github.io/perf-book/" rel="noopener noreferrer">The Rust Performance Book</a></strong> - Entire book</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=h4RkCyJyXmM" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Sorting Algorithms</a></strong> - Performance patterns</li>
<li>📝 <strong><a href="https://deterministic.space/high-performance-rust.html" rel="noopener noreferrer">Cheap tricks for high-performance Rust</a></strong>
</li>
<li>📝 <strong><a href="https://nnethercote.github.io/perf-book/profiling.html" rel="noopener noreferrer">The Rust Performance Book - Profiling</a></strong>
</li>
<li>📝 <strong><a href="https://doc.rust-lang.org/reference/attributes/codegen.html" rel="noopener noreferrer">Optimization - Rust Language Reference</a></strong>
</li>
</ul>

<p><strong>Deep Dive:</strong></p>

<ul>
<li>Understanding LLVM IR: use <a href="https://godbolt.org/" rel="noopener noreferrer">Compiler Explorer (Godbolt)</a> with Rust</li>
<li>Inlining, devirtualization, and constant folding</li>
<li>Profile-guided optimization</li>
<li>Reading assembly output with <code>cargo asm</code> (install with <code>cargo install cargo-show-asm</code>)</li>
<li>Study <a href="https://github.com/bheisler/criterion.rs" rel="noopener noreferrer">criterion.rs</a> for benchmarking</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Benchmark different iterator patterns (<code>.map().filter()</code> vs manual loops)</li>
<li>Optimize hot paths using profiling tools (install <code>cargo-flamegraph</code>)</li>
<li>Write a zero-allocation parser</li>
<li>Study <a href="https://github.com/serde-rs/json" rel="noopener noreferrer">serde_json performance</a> and try to beat it</li>
<li>Complete <strong><a href="https://exercism.org/tracks/rust" rel="noopener noreferrer">Exercism Performance-focused exercises</a></strong>
</li>
</ul>




<h3>
  
  
  Week 18-20: Unsafe Rust &amp; FFI
</h3>

<p><strong>Goal:</strong> Know when and how to break the rules safely</p>

<p><strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://doc.rust-lang.org/nomicon/" rel="noopener noreferrer">The Rustonomicon</a></strong> - Read cover-to-cover (seriously!)</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=QAz-maaH0KM" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Unsafe Code</a></strong>
</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=rAl-9HwD858" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Lifetime Annotations (revisit)</a></strong>
</li>
<li>📝 <strong><a href="http://cliffle.com/p/dangerust/" rel="noopener noreferrer">Learn Rust the Dangerous Way</a></strong> - LWN articles</li>
<li>📝 <strong><a href="https://rust-lang.github.io/unsafe-code-guidelines/" rel="noopener noreferrer">Unsafe Code Guidelines</a></strong>
</li>
</ul>

<p><strong>Deep Dive:</strong></p>

<ul>
<li>Undefined behavior and how to avoid it</li>
<li>Raw pointers and their limitations</li>
<li>Writing sound unsafe abstractions</li>
<li>FFI with C libraries</li>
<li>Study <a href="https://doc.rust-lang.org/src/std/" rel="noopener noreferrer">std library unsafe usage</a> - search for <code>unsafe</code>
</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Implement a custom allocator (use <code>std::alloc::GlobalAlloc</code>)</li>
<li>Create Rust bindings for a C library using <code>bindgen</code>
</li>
<li>Build a simple arena allocator</li>
<li>Study <a href="https://github.com/crossbeam-rs/crossbeam" rel="noopener noreferrer">crossbeam's lock-free structures</a> - careful unsafe usage</li>
<li>Contribute to <strong><a href="https://github.com/rust-lang/miri" rel="noopener noreferrer">Miri</a></strong> test cases (undefined behavior detector)</li>
</ul>




<h3>
  
  
  Week 21-23: Macros—Metaprogramming Mastery
</h3>

<p><strong>Goal:</strong> Generate code at compile time elegantly</p>

<p><strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://veykril.github.io/tlborm/" rel="noopener noreferrer">The Little Book of Rust Macros</a></strong> - Complete guide</li>
<li>📚 <strong><a href="https://doc.rust-lang.org/reference/macros.html" rel="noopener noreferrer">The Rust Reference - Macros</a></strong>
</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=q6paRBbLgNw" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Declarative Macros</a></strong>
</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=geovSK3wMB8" rel="noopener noreferrer">Jon Gjengset - Crust of Rust: Procedural Macros</a></strong>
</li>
<li>📝 <strong><a href="https://github.com/dtolnay/proc-macro-workshop" rel="noopener noreferrer">Procedural Macros Workshop</a></strong> - Hands-on exercises</li>
</ul>

<p><strong>Deep Dive:</strong></p>

<ul>
<li>Declarative macros: pattern matching on syntax</li>
<li>Procedural macros: derive, attribute, function-like</li>
<li>Hygiene and macro expansion order</li>
<li>Use <code>cargo expand</code> to see macro output</li>
<li>Study <a href="https://docs.rs/syn/latest/syn/" rel="noopener noreferrer">syn</a> and <a href="https://docs.rs/quote/latest/quote/" rel="noopener noreferrer">quote</a> crates</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Complete <strong><a href="https://github.com/dtolnay/proc-macro-workshop" rel="noopener noreferrer">dtolnay's proc-macro-workshop</a></strong> - All 5 projects</li>
<li>Write a custom derive macro for serialization</li>
<li>Build a builder pattern generator (study <a href="https://github.com/colin-kiegel/rust-derive-builder" rel="noopener noreferrer">derive_builder</a>)</li>
<li>Create a DSL using macros (e.g., SQL query builder)</li>
<li>Study <a href="https://github.com/serde-rs/serde/tree/master/serde_derive" rel="noopener noreferrer">serde_derive source</a>
</li>
</ul>




<h3>
  
  
  Week 24-26: Error Handling &amp; Observability
</h3>

<p><strong>Goal:</strong> Build production-ready error handling</p>

<p><strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://doc.rust-lang.org/book/ch09-00-error-handling.html" rel="noopener noreferrer">Rust Book - Error Handling</a></strong>
</li>
<li>📝 <strong><a href="https://blog.burntsushi.net/rust-error-handling/" rel="noopener noreferrer">Error Handling in Rust</a></strong> - Andrew Gallant's guide</li>
<li>📝 <strong><a href="https://nick.groenen.me/posts/rust-error-handling/" rel="noopener noreferrer">Rust Error Handling</a></strong> - Modern patterns</li>
<li>📚 Read docs for: <a href="https://docs.rs/anyhow/" rel="noopener noreferrer">anyhow</a>, <a href="https://docs.rs/thiserror/" rel="noopener noreferrer">thiserror</a>, <a href="https://docs.rs/eyre/" rel="noopener noreferrer">eyre</a>
</li>
<li>📚 <strong><a href="https://docs.rs/tracing/latest/tracing/" rel="noopener noreferrer">Tracing documentation</a></strong> - Structured logging</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=jpVzSse7oJ4" rel="noopener noreferrer">Jon Gjengset - Error Handling in Rust</a></strong>
</li>
</ul>

<p><strong>Study:</strong></p>

<ul>
<li>Error trait design: <code>std::error::Error</code>
</li>
<li>When to use <code>anyhow</code> vs <code>thiserror</code>
</li>
<li>Structured logging with <code>tracing</code>
</li>
<li>Distributed tracing concepts</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Design a comprehensive error type for a multi-layer application</li>
<li>Instrument async code with tracing spans</li>
<li>Build a metrics collection system using <code>tracing-subscriber</code>
</li>
<li>Study <a href="https://github.com/tokio-rs/tracing/tree/master/examples" rel="noopener noreferrer">tokio-tracing examples</a>
</li>
<li>Create error report with full context chain (study <code>eyre</code> and <code>color-eyre</code>)</li>
</ul>




<h2>
  
  
  <strong>Phase 4: Real-World Systems (Ongoing)</strong>
</h2>

<h3>
  
  
  Domain-Specific Mastery
</h3>

<p><strong>Systems Programming:</strong><br>
<strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://os.phil-opp.com/" rel="noopener noreferrer">Writing an OS in Rust</a></strong> - Complete tutorial series</li>
<li>📚 <strong><a href="https://doc.rust-lang.org/embedded-book/" rel="noopener noreferrer">Embedded Rust Book</a></strong>
</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=bzja9fQWzdA" rel="noopener noreferrer">Jon Gjengset - Implementing TCP</a></strong>
</li>
<li>Study <a href="https://github.com/redox-os/redox" rel="noopener noreferrer">redox-os source</a> - OS written in Rust</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Follow <strong><a href="https://os.phil-opp.com/" rel="noopener noreferrer">Writing an OS in Rust</a></strong> completely</li>
<li>Implement a file system in userspace (FUSE)</li>
<li>Build a network protocol from scratch</li>
<li>Contribute to <a href="https://github.com/rust-lang/rust" rel="noopener noreferrer">rust-lang std library</a>
</li>
</ul>




<p><strong>Web &amp; Backend:</strong><br>
<strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://actix.rs/docs/" rel="noopener noreferrer">Actix Web Documentation</a></strong>
</li>
<li>📚 <strong><a href="https://docs.rs/axum/latest/axum/" rel="noopener noreferrer">Axum Documentation</a></strong>
</li>
<li>📚 <strong><a href="https://rocket.rs/v0.5/guide/" rel="noopener noreferrer">Rocket Guide</a></strong>
</li>
<li>🎥 <strong><a href="https://www.youtube.com/watch?v=MZPXLNLlagc" rel="noopener noreferrer">Let's build a web server from scratch</a></strong> - Jon Gjengset</li>
<li>📝 <strong><a href="https://rust-cli.github.io/book/" rel="noopener noreferrer">Command Line Apps in Rust</a></strong>
</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Build a complete REST API with Axum</li>
<li>Implement WebSocket server with state management</li>
<li>Create a simple database driver from scratch (study <a href="https://github.com/sfackler/rust-postgres" rel="noopener noreferrer">tokio-postgres</a>)</li>
<li>Build a web scraper with <a href="https://docs.rs/reqwest/" rel="noopener noreferrer">reqwest</a> and <a href="https://docs.rs/scraper/" rel="noopener noreferrer">scraper</a>
</li>
<li>Study <a href="https://github.com/rustls/rustls" rel="noopener noreferrer">rustls source</a> - TLS implementation</li>
</ul>




<p><strong>Blockchain/DeFi:</strong><br>
<strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://solanacookbook.com/" rel="noopener noreferrer">Solana Cookbook</a></strong> - Free comprehensive guide</li>
<li>📚 <strong><a href="https://book.anchor-lang.com/" rel="noopener noreferrer">Anchor Book</a></strong> - Framework documentation</li>
<li>📚 <strong><a href="https://github.com/solana-labs/solana-program-library" rel="noopener noreferrer">Solana Program Library</a></strong> - Study source code</li>
<li>📝 <strong><a href="https://solana.com/docs/core/accounts" rel="noopener noreferrer">Understanding Solana's Account Model</a></strong>
</li>
<li>📝 <strong><a href="https://borsh.io/" rel="noopener noreferrer">Borsh Specification</a></strong>
</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Deep dive into Solana's runtime and BPF</li>
<li>Study Anchor framework internals (read <a href="https://github.com/coral-xyz/anchor" rel="noopener noreferrer">source</a>)</li>
<li>Implement AMM math from scratch (constant product, stable swap)</li>
<li>Build cross-program invocation patterns</li>
<li>Study successful programs: <a href="https://github.com/project-serum/serum-dex" rel="noopener noreferrer">Serum DEX</a>, <a href="https://github.com/blockworks-foundation/mango-v3" rel="noopener noreferrer">Mango Markets</a>
</li>
<li>Understand account rent and program derived addresses</li>
</ul>




<p><strong>Performance Engineering:</strong><br>
<strong>Resources:</strong></p>

<ul>
<li>📚 <strong><a href="https://nnethercote.github.io/perf-book/" rel="noopener noreferrer">The Rust Performance Book</a></strong> (revisit)</li>
<li>📝 <strong><a href="https://doc.rust-lang.org/std/simd/index.html" rel="noopener noreferrer">Rust SIMD Guide</a></strong>
</li>
<li>📝 <strong><a href="https://www.lurklurk.org/effective-rust/cache-friendly.html" rel="noopener noreferrer">Cache-Friendly Code</a></strong>
</li>
<li>📝 Study <a href="https://github.com/BurntSushi/ripgrep" rel="noopener noreferrer">ripgrep source</a> - peak Rust performance</li>
</ul>

<p><strong>Projects:</strong></p>

<ul>
<li>Profile real applications with <code>perf</code>, <code>cargo flamegraph</code>
</li>
<li>Optimize memory allocations using <code>heaptrack</code>
</li>
<li>Study cache-friendly data structures</li>
<li>Learn SIMD programming with <code>std::simd</code>
</li>
<li>Contribute performance improvements to open-source Rust projects</li>
</ul>




<h2>
  
  
  <strong>Continuous Learning Practices</strong>
</h2>

<h3>
  
  
  Daily Habits (Pick 2-3)
</h3>

<ol>
<li>
<p><strong>Read source code</strong> (30 min): Study well-written Rust projects</p>

<ul>
<li>
<strong>Essential repos to study:</strong>

<ul>
<li>
<a href="https://github.com/tokio-rs/tokio" rel="noopener noreferrer">tokio</a> - Async runtime</li>
<li>
<a href="https://github.com/serde-rs/serde" rel="noopener noreferrer">serde</a> - Serialization</li>
<li>
<a href="https://github.com/clap-rs/clap" rel="noopener noreferrer">clap</a> - CLI parser</li>
<li>
<a href="https://github.com/tokio-rs/axum" rel="noopener noreferrer">axum</a> - Web framework</li>
<li>
<a href="https://github.com/BurntSushi/ripgrep" rel="noopener noreferrer">ripgrep</a> - Fast grep</li>
<li>
<a href="https://github.com/sharkdp/bat" rel="noopener noreferrer">bat</a> - Better cat</li>
<li>
<a href="https://github.com/sharkdp/fd" rel="noopener noreferrer">fd</a> - Better find</li>
</ul>
</li>
</ul>
</li>
<li>
<p><strong>Write code</strong> (1-2 hours): Build small experiments</p>

<ul>
<li>
<strong><a href="https://github.com/rust-lang/rustlings" rel="noopener noreferrer">Rustlings</a></strong> daily</li>
<li>Implement algorithms in Rust</li>
<li>Recreate crates you use to understand them</li>
</ul>
</li>
<li>
<p><strong>Teach others</strong>: </p>

<ul>
<li>Answer questions on <strong><a href="https://users.rust-lang.org/" rel="noopener noreferrer">Rust Users Forum</a></strong>
</li>
<li>Write blog posts (host free on GitHub Pages)</li>
<li>Contribute to <strong><a href="https://github.com/rust-lang/rust-by-example" rel="noopener noreferrer">Rust by Example</a></strong>
</li>
</ul>
</li>
</ol>

<h3>
  
  
  Weekly Activities
</h3>

<ul>
<li>
<p><strong>Code review</strong>: Review Rust PRs on GitHub</p>

<ul>
<li>Find projects with "good first issue" label</li>
<li>Study how maintainers review code</li>
</ul>


</li>

<li>

<p><strong>Deep dive</strong>: Pick one concept and understand it completely</p>

<ul>
<li>Use <strong><a href="https://doc.rust-lang.org/book/" rel="noopener noreferrer">The Book</a></strong>, <strong><a href="https://doc.rust-lang.org/nomicon/" rel="noopener noreferrer">Rustonomicon</a></strong>, or <strong><a href="https://doc.rust-lang.org/reference/" rel="noopener noreferrer">Reference</a></strong>
</li>
</ul>


</li>

<li>

<p><strong>Build something</strong>: Work on a meaningful project</p>

<ul>
<li>Ideas: <strong><a href="https://rust-cli.github.io/book/" rel="noopener noreferrer">CLI apps</a></strong>, web services, games</li>
</ul>


</li>

</ul>

<h3>
  
  
  Monthly Goals
</h3>

<ul>
<li>
<p><strong>Read papers</strong>: </p>

<ul>
<li><a href="https://www.rust-lang.org/learn" rel="noopener noreferrer">Rust papers/talks</a></li>
<li>Distributed systems papers from <a href="https://github.com/papers-we-love/papers-we-love" rel="noopener noreferrer">Papers We Love</a>
</li>
</ul>


</li>

<li>

<p><strong>Contribute to OSS</strong>: </p>

<ul>
<li>Find Rust projects needing help on <a href="https://github.com/topics/rust?o=desc&amp;s=stars" rel="noopener noreferrer">GitHub</a>
</li>
<li>Start with documentation improvements</li>
</ul>


</li>

<li>

<p><strong>Write</strong>: </p>

<ul>
<li>Blog posts (free: <a href="https://dev.to/">dev.to</a>, <a href="https://medium.com/" rel="noopener noreferrer">Medium</a>, <a href="https://pages.github.com/" rel="noopener noreferrer">GitHub Pages</a>)</li>
<li>Share learnings on <strong><a href="https://www.reddit.com/r/rust/" rel="noopener noreferrer">r/rust</a></strong>
</li>
</ul>


</li>

</ul>




<h2>
  
  
  <strong>Essential Free Resources Summary</strong>
</h2>

<h3>
  
  
  📚 Must-Read Books (All Free!)
</h3>

<ol>
<li><a href="https://doc.rust-lang.org/book/" rel="noopener noreferrer">The Rust Programming Language</a></li>
<li><a href="https://doc.rust-lang.org/rust-by-example/" rel="noopener noreferrer">Rust By Example</a></li>
<li><a href="https://doc.rust-lang.org/nomicon/" rel="noopener noreferrer">The Rustonomicon</a></li>
<li><a href="https://marabos.nl/atomics/" rel="noopener noreferrer">Rust Atomics and Locks</a></li>
<li><a href="https://rust-lang.github.io/async-book/" rel="noopener noreferrer">Asynchronous Programming in Rust</a></li>
<li><a href="https://veykril.github.io/tlborm/" rel="noopener noreferrer">The Little Book of Rust Macros</a></li>
<li><a href="https://rust-unofficial.github.io/too-many-lists/" rel="noopener noreferrer">Too Many Linked Lists</a></li>
<li><a href="https://nnethercote.github.io/perf-book/" rel="noopener noreferrer">The Rust Performance Book</a></li>
</ol>

<h3>
  
  
  🎥 Video Series
</h3>

<ul>
<li>
<a href="https://www.youtube.com/c/JonGjengset" rel="noopener noreferrer">Jon Gjengset - Crust of Rust</a> - THE best advanced content</li>
<li>
<a href="https://www.youtube.com/c/LetsGetRusty" rel="noopener noreferrer">Let's Get Rusty</a> - Great for beginners</li>
<li>
<a href="https://www.youtube.com/c/NoBoilerplate" rel="noopener noreferrer">No Boilerplate</a> - Energizing short videos</li>
</ul>

