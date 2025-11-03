# WebKit Guidelines for Safer Swift Programming

See also [guidelines for Safer C++](Safer-CPP-Guidelines).

## Don't worry about memory safety unless you need to type `unsafe`

The point of using Strictly Safe Swift is that the compiler proves all your code is safe against memory safety errors. You don't need to think about it. Go and have a nice day instead.

## If you need to write `unsafe`

We are aiming to have a Swift codebase entirely free of `unsafe`. Our goal is to use Swift as the programming environment which verifies safety without exception. This is still work-in-progress.

If you find you need to write `unsafe`, do this:

1. Avoid using `unsafe` at all. You may be able to:
   * Restructure the code.
   * Use the Swift type system to prove truths at compile time (see the "use rich type system features" section below.)
   * Use an existing type which encapsulates the unsafety and already exposes a safe API for you to use.
   * [Annotate the C++ type such that its lifetime and bounds are understood by Swift](https://www.swift.org/documentation/cxx-interop/safe-interop/).
2. If you find the `unsafe` is still necessary,
   * Identify what abstraction is missing from the Swift standard library (or some other dependency) which would have allowed the `unsafe` to be avoided.
   * File a bug to ask for that improvement.
   * Attempt to locally build an abstraction which guarantees runtime safety, even if internally it includes unsafety.
   * Add a comment explaining the safety and referencing the radar you filed. Such comments should explain the safety invariants and reasoning which guarantee that they are true. These comments should not require global knowledge of the subsystem but should be auditable with reference just to obvious local truths.

Reviewers of unsafe code should:

* Reject `unsafe` code without comments expaining the safety.
* Fully understand the safety invariants which need to be upheld, and why they are always upheld.
* Reject the code if it requires anything beyond local reasoning. (That is, you can become confident it's safe without having to read code beyond the function or data type you're reviewing).
* Ensure that the author has followed the steps above.

## Use rich type system features for logical safety

Logic bugs can also sometimes be security bugs. Try to avoid them by using Swift's richer type system to encode invariants in the type system, so that it's impossible to make mistakes at runtime. This also helps minimize use of `unsafe` by reducing the possible runtime states that need to be handled.

At the moment, we don't have enough Swift experience in WebKit to give a firm policy here, but wherever possible please try to use Swift to avoid runtime conditions that can be avoided at compile-time. [Some examples can be found here](Safer-Swift-Rich-Type-System-Examples).

## Prefer Swift standard library types over WTF types

Swift is better able to:

* Prove safety
* Optimise away reference counts and bounds checks

if you use its built-in types. So, use `Swift.Array` rather than `WTF::Vector`; use `T?` rather than `RefPtr<T>`, etc. Similarly, if you have the choice between calling a `T* thingy()` versus `Ref<T> protectedThingy()` member function, call the former.

There will of course be occasions when it's more performant or significantly simpler to use WebKit and WTF types from Swift; that's OK.

Over time, we hope to make conversion code readily available.
