# Smart Pointers in WebKit

There are basic 3 types of smart pointers in WebKit.
* `Ref` and `RefPtr` - These smart pointers are used like [`std::shared_ptr`](https://en.cppreference.com/w/cpp/memory/shared_ptr), and extends the lifetime of an object so long as there is an outstanding `Ref` or `RefPtr`. To use `Ref` or `RefPtr` with an object, make the class inherit from `RefCounted<T>` or define `ref()` and `deref()` functions which implement the semantics of `RefCounted`.
* `WeakPtr` and `ThreadSafeWeakPtr` - These smart pointers are used like [`std::weak_ptr`](https://en.cppreference.com/w/cpp/memory/weak_ptr) and becomes `nullptr` when the object it points to dies. To use `WeakPtr` or `ThreadSafeWeakPtr`, make the class inherit from `CanMakeWeakPtr<T>` or `CanMakeThreadSafeWeakPtr<T>`, whichever is appropriate.
* `CheckedRef` and `CheckedPtr` - These are substitute for raw pointers and raw references with object's destructor release-asserting that there is no outstanding reference left. To use `CheckedRef` or `CheckedPtr`, make the class inherit from `CanMakeCheckedPtr` or `CanMakeThreadSafeCheckedPtr`, whichever is appropriate; or define `incrementPtrCount` and `decrementPtrCount` and and implement the semantics of `CanMakeCheckedPtr`.

Which smart pointer should be used generally depends on semantics of objects. If there is a shared ownership, then `Ref` and `RefPtr` is most appropriate so long as it won't create a [reference cycle](https://en.wikipedia.org/wiki/Reference_counting#Dealing_with_reference_cycles). When a relationship is that of [weak reference](https://en.wikipedia.org/wiki/Weak_reference), we can use `WeakPtr` or `ThreadSafeWeakPtr` if weak semantics (i.e. pointer becomes nullptr once the object is deleted) is required. In the cases where weak semantics is not needed (i.e. we're clearing those references manually before/when object dies), we can use `CheckedRef` or `CheckedPtr`. `CheckedRef` and `CheckedPtr` are slightly more efficient than `WeakPtr` or `ThreadSafeWeakPtr` since the latter two require a separate heap memory allocation (`WeakPtrImpl`).

# Safe Use of Smart Pointers

## What is Dangerous?

So **what is** a *dangerous* use of references and pointers you may ask? It’s any use that we can’t trivially conclude that it doesn’t lead to a use-after-free.

Consider the following example. This code may lead to a use-after-free of “parent” in the third line because the code doesn’t keep the parent alive. Because `updateLayout` can execute arbitrary script execution, it may remove the parent node from the document so that the parent is no longer alive by the time third line is executed.

```cpp
Node* parent = element->parentElement();
document.updateLayout();
parent->scrollIntoView();
```

In general, relying on a complex data structure such as DOM tree to keep `RefCounted` objects alive while we call a non-trivial function is not safe. All it takes for the code to have a use-after-free is for someone to start updating style, layout, etc... inside the function either directly or indirectly. And we don’t want to make WebKit development really hard by forcing anyone who modifies a function to check every caller of the function and their callers, etc... to make sure it’s safe to do so.

For this reason, it’s dangerous to store a raw pointer or a reference to a ref counted object as a local variable and use it after calling a non-trivial function. We did a similar analysis of a number of other patterns and usage of ref counted objects in WebKit and came up with the following basic rules for using ref counted objects in a safe manner. We’re hoping that these rules would be eventually incorporated into our coding style guideline: https://webkit.org/code-style-guidelines/

## Rules for Using Ref Counted Objects

1. Every data member to another object must use either `Ref`, `RefPtr`, `CheckedRef`, `CheckedPtr`, or `WeakPtr`. This also includes types used in container types such as `Vector` and `HashMap`. [webkit.NoUncountedMemberChecker](https://clang.llvm.org/docs/analyzer/checkers.html#id128)(https://clang.llvm.org/docs/analyzer/checkers.html#webkit-nouncountedmemberchecker)
2. Every ref counted base class, if it has a derived class, must define a virtual destructor. [webkit.RefCntblBaseVirtualDtor](https://clang.llvm.org/docs/analyzer/checkers.html#webkit-refcntblbasevirtualdtor)
3. Every object passed to a non-trivial function as an argument (including "this" pointer) should be stored as a `Ref`, `RefPtr`, `CheckedRef`, or `CheckedPtr` in the caller’s local scope unless it's an argument to the caller itself by the transitive property [1]. [alpha.webkit.UncountedCallArgsChecker](https://clang.llvm.org/docs/analyzer/checkers.html#alpha-webkit-uncountedcallargschecker)
4. Every object must be captured using `Ref`, `RefPtr`, `CheckedRef`, `CheckedPtr`, or `WeakPtr` for a lambda function. [webkit.UncountedLambdaCapturesChecker](https://clang.llvm.org/docs/analyzer/checkers.html#id129)
5. When there is a raw pointer or a raw reference local variable, there must be a `Ref`, `RefPtr`, `CheckedRef`, or `CheckedPtr` in the outer scope or it must be a function argument. [alpha.webkit.UncountedLocalVarsChecker](https://clang.llvm.org/docs/analyzer/checkers.html#alpha-webkit-uncountedlocalvarschecker)

(1) is pretty trivial. Every ref counted data member should be stored using `Ref`, `RefPtr`, `CheckedRef`, `CheckedPtr`, `WeakPtr` since it would not be trivially obvious that life cycles of two or more objects are correctly tied or managed together.

(2) is also pretty easy to understand. In the following example, if someone destroys an instance of B using Ref<A>, then it would result in an undefined behavior so we forbid that.

```cpp
struct A : public RefCounted<A> {
    Vector<int> someData;
};

struct B : public A {
     Vector<int> otherData;
};
```

To understand (3), examine the following code, in which, `setForm` is called with the result of `findAssociatedForm`, which returns HTMLFormElement* without storing it in a `Ref` or `RefPtr`. If `setForm` can somehow cause `HTMLFormElement` to be deleted before completing its work, then this can result in a use-after-free within setForm.

```cpp
void FormAssociatedElement::resetFormOwner()
{
    RefPtr<HTMLFormElement> originalForm = m_form.get();
    setForm(findAssociatedForm(&asHTMLElement(), originalForm.get())); // This line
    HTMLElement& element = asHTMLElement();
    auto* newForm = m_form.get();
    if (newForm && newForm != originalForm && newForm->isConnected())
        element.document().didAssociateFormControl(element);
}
```

Why, you may ask, we don't put `HTMLFormElement*` in a `Ref` or `RefPtr` in `setForm` instead? This is because a lot of code has calls to a function with a local variable. Because the callee doesn’t know whether the caller has already stored each argument in Ref / RefPtr or not, we need to be safe and store them again in `Ref` / `RefPtr`, resulting in an unnecessary ref-churn. Additionally, if we took the approach of the callee being responsible for keeping every argument alive, then every non-trivial member function of a ref counted object must have `protectedThis` at the beginning of the function, which would be particularly wasteful if those member functions are sometimes called by other member functions.

Once this rule is applied everywhere, for example, we can get rid of `protectedThis` from our codebase because it would be redundant. Furthermore, this rule transitively allows a raw pointer or a reference passed in as an argument to be used as arguments to call another function without first storing it in Ref or RefPtr.

For example, the following function satisfies this rule because both `oldDocument` and `newDocument` are raw references passed to this function as arguments, and therefore the caller of this function must have already stored them as local variables using `Ref` or `RefPtr`.

```cpp
void HTMLMediaElement::didMoveToNewDocument(Document& oldDocument, Document& newDocument)
{
    ASSERT_WITH_SECURITY_IMPLICATION(&document() == &newDocument);
    if (m_shouldDelayLoadEvent) {
        oldDocument.decrementLoadEventDelayCount();
        newDocument.incrementLoadEventDelayCount();
    }

    unregisterWithDocument(oldDocument);
    registerWithDocument(newDocument);

    HTMLElement::didMoveToNewDocument(oldDocument, newDocument);
    updateShouldAutoplay();
}
```

(4) can considered as generalization for (3) for lambda captured variables. In the following example, `this` object can be deleted between the time this lambda function is created and called. To avoid use-after-free of `this`, we need to store it using `Ref`, `RefPtr`, `CheckedRef`, `CheckedPtr`, or `WeakPtr` instead:

```cpp
    auto completionHandlerWrapper = [this, completionHandler = WTFMove(completionHandler)] (const IPC::DataReference& resumeData) mutable {
        completionHandler(resumeData);
        if (!weakThis || m_ignoreDidFailCallback == IgnoreDidFailCallback::No)
            return;
        DOWNLOAD_RELEASE_LOG("didCancel: (id = %" PRIu64 ")", downloadID().toUInt64());
        if (auto extension = std::exchange(m_sandboxExtension, nullptr))
            extension->revoke();
        m_downloadManager.downloadFinished(*this);
    };
```
