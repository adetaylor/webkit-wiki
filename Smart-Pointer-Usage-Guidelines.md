# Smart Pointers in WebKit

There are 3 primary smart pointer types in WebKit.
* `Ref` and `RefPtr` - These smart pointers are used like [`std::shared_ptr`](https://en.cppreference.com/w/cpp/memory/shared_ptr). They extend the lifetime of an object so long as there is an outstanding `Ref` or `RefPtr`. To use `Ref` or `RefPtr` with an object, make the class inherit from `RefCounted<T>` or `ThreadSafeRefCounted<>` or define `ref()` and `deref()` functions which implement the semantics of `RefCounted`. RetainPtr works similarly for Objective-C objects. OSObjectPtr work similarly for Darwin OS objects.
* `WeakPtr` and `ThreadSafeWeakPtr` - These smart pointers are used like [`std::weak_ptr`](https://en.cppreference.com/w/cpp/memory/weak_ptr). They become `nullptr` when the object they point to dies. To use `WeakPtr` or `ThreadSafeWeakPtr`, make the class inherit from `CanMakeWeakPtr<T>` or `CanMakeThreadSafeWeakPtr<T>`, whichever is appropriate.  Note that classes that want to implement both `ThreadSafeRefCounted` and `ThreadSafeWeakPtr` must inherit from `ThreadSafeRefCountedAndCanMakeThreadSafeWeakPtr<>`.
* `CheckedRef` and `CheckedPtr` - These smart pointers are used like raw C++ references and pointers. They are intended for single-ownership use cases like the render tree, where you should have no need to extend or observe the lifetime of an object. They ensure memory safety by release-asserting at object destruction time that no outstanding references remain. To use `CheckedRef` or `CheckedPtr`, make the class inherit from `CanMakeCheckedPtr` or `CanMakeThreadSafeCheckedPtr`, whichever is appropriate; or define `incrementPtrCount` and `decrementPtrCount` and and implement the semantics of `CanMakeCheckedPtr`.

Which smart pointer to use depends on intended semantics. For shared ownership, `Ref` and `RefPtr` are most appropriate, so long as they do not create a [reference cycle](https://en.wikipedia.org/wiki/Reference_counting#Dealing_with_reference_cycles). To break a reference cycle using a [weak reference](https://en.wikipedia.org/wiki/Weak_reference), use `WeakPtr` or `ThreadSafeWeakPtr`. If neither shared nor weak semantics are needed, use `CheckedRef` or `CheckedPtr`. `CheckedRef` and `CheckedPtr` are slightly more efficient than `WeakPtr` or `ThreadSafeWeakPtr` since the latter two require a separate heap memory allocation (`WeakPtrImpl`).

In general, you should use some form of smart pointer for:
* class data members
* global variables
* local variables

# Safe Use of Smart Pointers

## What is Dangerous?

So **what is** a *dangerous* use of references and pointers you may ask? It’s any use that we can’t trivially conclude that it doesn’t lead to a use-after-free.

For now, we don’t detect dangerous use of non-ref counted objects including ones that can vend WeakPtr. It’s on us, humans, to decide which objects need to be ref counted or need to be CanMakeWeakPtr.

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
3. Every object passed to a non-trivial function as an argument (including "this" pointer) should be stored as a `Ref`, `RefPtr`, `CheckedRef`, or `CheckedPtr` in the caller’s local scope unless it's an argument to the caller itself by the transitive property [1]. A trivial function is any function which simply returns the value of a member variable. e.g. `Foo* foo() { return m_foo.get(); }`[alpha.webkit.UncountedCallArgsChecker](https://clang.llvm.org/docs/analyzer/checkers.html#alpha-webkit-uncountedcallargschecker)
4. Every object must be captured using `Ref`, `RefPtr`, `CheckedRef`, `CheckedPtr`, or `WeakPtr` for a lambda function unless lambda function is only invoked locally and not stored anywhere. [webkit.UncountedLambdaCapturesChecker](https://clang.llvm.org/docs/analyzer/checkers.html#id129)
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

To understand (3), examine the following code, in which, `setForm` is called with the result of `findAssociatedForm`, which returns `HTMLFormElement*` without storing it in a `Ref` or `RefPtr`. If `setForm` can somehow cause `HTMLFormElement` to be deleted before completing its work, then this can result in a use-after-free within setForm.

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

For (4), capturing a ref counted object as a raw pointer or a reference, the tool may generate a warning like this:
```
Source/WebKit/UIProcess/API/APIHTTPCookieStore.cpp:115:74: warning: [WebKit] captured a raw pointer to a ref-countable type [-Wlambda-capture-ptr-to-refcntbl]
    cookieManager->deleteCookie(m_owningDataStore->sessionID(), cookie, [pool = WTFMove(pool), completionHandler = WTFMove(completionHandler)]() mutable {
```

This is warning about `webPage` object getting captured using a raw reference in the following function:

```cpp
void HTTPCookieStore::deleteCookie(const WebCore::Cookie& cookie, CompletionHandler<void()>&& completionHandler)
{
    auto* pool = m_owningDataStore->processPoolForCookieStorageOperations();
    if (!pool) {
        if (m_owningDataStore->sessionID() == PAL::SessionID::defaultSessionID() && !cookie.session)
            deleteCookieFromDefaultUIProcessCookieStore(cookie);
        else
            m_owningDataStore->removePendingCookie(cookie);

        RunLoop::main().dispatch([completionHandler = WTFMove(completionHandler)] () mutable {
            completionHandler();
        });
        return;
    }

    auto* cookieManager = pool->supplement<WebKit::WebCookieManagerProxy>();
    cookieManager->deleteCookie(m_owningDataStore->sessionID(), cookie, [pool = WTFMove(pool), completionHandler = WTFMove(completionHandler)]() mutable {
        completionHandler();
    });
}
```
Here, we should have stored WebProcessPool using `Ref`, `RefPtr`, `CheckedRef`, `CheckedPtr`, or `WeakPtr` instead

In the following example, `this` object can be deleted between the time this lambda function is created and called. To avoid use-after-free of `this`, we need to store it using `Ref`, `RefPtr`, `CheckedRef`, `CheckedPtr`, or `WeakPtr` instead:

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

(5) is about ensuring safety of local variables. We allow use of a raw pointer or a reference but only if its embedding scope contains an equivalent Ref or RefPtr. In the following code, cssValueList is a raw pointer but it has an alias cssValue which is a RefPtr so we allow it.

```cpp
static Vector<String> authoredGridTrackSizes(Node* node, GridTrackSizingDirection direction, unsigned expectedTrackCount)
{
    auto* element = dynamicDowncast<StyledElement>(node);
    if (!element)
        return { };

    auto directionCSSPropertyID = direction == GridTrackSizingDirection::ForColumns ? CSSPropertyID::CSSPropertyGridTemplateColumns : CSSPropertyID::CSSPropertyGridTemplateRows;
    RefPtr<CSSValue> cssValue;
    if (auto* inlineStyle = element->inlineStyle())
        cssValue = inlineStyle->getPropertyCSSValue(directionCSSPropertyID);

    if (!cssValue) {
        auto styleRules = element->styleResolver().styleRulesForElement(element);
        styleRules.reverse();
        for (auto styleRule : styleRules) {
            ASSERT(styleRule);
            if (!styleRule)
                continue;
            cssValue = styleRule->properties().getPropertyCSSValue(directionCSSPropertyID);
            if (cssValue)
                break;
        }
    }

    auto* cssValueList = dynamicDowncast<CSSValueList>(cssValue.get());
    if (!cssValueList)
        return { };
```

On the other hand, here is an example of unsafe use of raw pointers:

```cpp
if (auto* srcList = downcast<CSSValueList>(m_fontFaceRule->properties().getPropertyCSSValue(CSSPropertySrc).get())) {
    for (auto& item : *srcList)
        downcast<CSSFontFaceSrcLocalValue>(const_cast<CSSValue&>(item)).setSVGFontFaceElement(*this);
}
```

Here, we're we’re storing the result of getPropertyCSSValue as CSSValueList*. But if setSVGFontFaceEleme was a non-trivial function that could mutate the said property or its value, we may end up having a use-after-free bug. The solution is to deploy RefPtr instead as follows:

```cpp
if (auto* srcList = downcast<CSSValueList>(m_fontFaceRule->properties().getPropertyCSSValue(CSSPropertySrc).get())) {
    for (auto& item : *srcList)
        downcast<CSSFontFaceSrcLocalValue>(const_cast<CSSValue&>(item)).setSVGFontFaceElement(*this);
}
```

## When to Use Which Smart Pointer

WebKit supports a number of smart pointers with their own preconditions to use them:

* `RefPtr` / `Ref` - Object must implement `ref()` and `deref()` functions whereby last call to `deref()` will delete this. Typically accomplished by inheriting from `RefCounted<T>`.
* `WeakPtr` - Object must inherit from `CanMakeWeakPtr<T>`.
* `ThreadSafeWeakPtr` - Object must inherit from `ThreadSafeRefCountedAndCanMakeThreadSafeWeakPtr<T>`.
* `CheckedPtr` / `CheckedRef` - Object must `incrementPtrCount` and `decrementPtrCount` with semantics so that the destructor will release assert that `decrementPtrCount` has been called as many times as `incrementPtrCount`. Typically accomplished by inheriting from `CanMakeCheckedPtr`.

Each smart pointer type is catered towards specific use cases in mind.

### Shared Ownership with `Ref` and `RefPtr`
`RefPtr` and `Ref` are useful when there could be multiple owners for a given object, or multiple heap allocated objects need to keep the object alive. RefCounted and ThreadSafeRefCounted, or alternatively any class which implements the semantics of `ref()` and `deref()` can be used to implement an object which supports RefPtr and Ref. There is no `ThreadSafeRefPtr` or `ThreadSafeRef`. Regular RefPtr and Ref are thread safe as long as `ref()` and `deref()` are thread safe (e.g. uses `ThreadSafeRefCounted`).

### Weak Relationship with `WeakPtr` and `ThreadSafeWeakPtr`
`WeakPtr` (single threaded) and `ThreadSafeWeakPtr` (concurrency safe) are useful when an object needs to be referenced by some other object but without keeping the object alive. Instead, it would automatically start returning nullptr when the object it points to has been destroyed. An object which inherits from `CanMakeWeakPtr` and `ThreadSafeRefCountedAndCanMakeThreadSafeWeakPtr`, respectively, support these pointers. Note that thread safety is built into pointer types themselves unlike `Ref` and `RefPtr`. In addition, any object which inherits from `ThreadSafeRefCountedAndCanMakeThreadSafeWeakPtr` supports Ref and RefPtr as well as `ThreadSafeWeakPtr`.

### Cheaper Weak Relationship with CheckedPtr and CheckedRef
Whilst `WeakPtr` and `ThreadSafeWeakPtr` are preferred way of establishing a weak relationship between objects, it has both runtime cost (one extra indirect load) and memory (allocates one `WeakPtrImplBase` object). When these runtime costs are not permissible or semantics of reference is desirable (i.e. the value of it should never be nullptr in normal circumstances), then `CheckedPtr` and `CheckedRef` provide a viable alternative. These smart pointers act a lot like `RefPtr` and `Ref` and call `incrementPtrCount()` and `decrementPtrCount()` on the object it points to but it doesn’t extend the lifetime of the object. Instead, when the object is about to get destroyed, its destructor will release assert that there are not outstanding `CheckedPtr` and `CheckedRef` left, thus preventing the use of freed memory region down the line.

One drawback of `CheckedPtr` and CheckedRef is that each user is responsible for clearing its values before the object gets destroyed, and if and when the release assert fails, it doesn’t provide any backtrace or other information regarding which client still has an outstanding instance of `CheckedPtr` and `CheckedRef`. We must audit all uses of `CheckedPtr` and `CheckedRef` to find out which ones are violating the contract. Like `RefPtr` and `Ref`, `CheckedPtr` and `CheckedRef` are thread safe as long as `incrementPtrCount()` and `decrementPtrCount()` are thread safe (e.g. uses `CanMakeThreadSafeCheckedPtr`).
