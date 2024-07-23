# WebKit Guidelines for Safer C++ Programming


## Use smart pointers for object lifetime management

> In the following guidelines, the definition of a trivial member function is a function that is inline and cannot cause `this` to get destroyed. It usually applies to simple getters / setters.

### When calling a non-trivial member function on an object, hold a smart pointer to the object on the stack

**Reasoning:**

This makes sure the member function cannot make a use-after-free use of `this` during its execution. Use a `Ref` / `RefPtr` on the stack if the object is ref-counted, a `CheckedRef` / `CheckedPtr` otherwise.

Similarly, we should be using `RetainPtr` for Objective C objects, `OSObjectPtr` for Darwin OS objects, `GRefPtr` for various GLib types, and `CachedResourceHandle` for CachedResource objects.

Note that it is important for the smart pointer to be a stack variable. Calling a function on a data member that has a smart pointer type is not truly safe, because this data member could get reassigned while the function is running.

**Right:**
```cpp
if (RefPtr element = m_element) { // Even if m_element is already a RefPtr.
    element->focus();
    // Use protectedFoo() functions that return a Ref / RefPtr
    // to avoid local variables.
    element->protectedDocument()->didFocus();
    if (CheckedPtr renderer = element->renderer())
        renderer->didFocus();
    // Use checkedFoo() functions that return a CheckedRef / CheckedPtr
    // to avoid local variables.
    element->checkedFocusContoller().didFocus();
}

if (RefPtr document = ownerDocument())
    document->foo();
```

**Wrong:**
```cpp
if (m_element) {
    m_element->focus();
    m_element->document()->didFocus();
    if (auto* renderer = m_element->renderer())
        renderer->didFocus();
    m_element->focusContoller().didFocus();
}

// Calling protectedOwnerDocument() here is redundant, calling ownerDocument()
// is sufficient. protectedFoo() functions are only useful to avoid local
// smart pointer variables.
if (RefPtr document = protectedOwnerDocument())
    document->foo();
```


### When passing an object to non-trivial function, hold a smart pointer to this object on the stack

**Reasoning:**

This makes sure that the object passed to the function cannot be used-after-free during the execution of the function. Use a `Ref` / `RefPtr` on the stack if the object is ref-counted, a `CheckedRef` / `CheckedPtr` otherwise.

Similarly, we should be using `RetainPtr` for Objective C objects, `OSObjectPtr` for Darwin OS objects, `GRefPtr` for various GLib types, and `CachedResourceHandle` for `CachedResource` objects.

**Right:**
```cpp
Ref element = task.element();
adjustDirectionality(element);
// Use protectedFoo() / checkedFoo() functions to avoid local variables.
registerWithDocument(element->protectedDocument());
```

**Wrong:**
```cpp
auto& element = task.element();
adjustDirectionality(element);
registerWithDocument(element->document());
```

### Use smart pointers instead of raw pointers for all data members

**Reasoning:**

This makes sure we don’t use-after-free data members, by enforcing that pointers cannot become stale. Use `Ref` / `RefPtr` for ref-counted objects that you wish to keep alive. Use `WeakRef` / `WeakPtr` / `CheckedRef` / `CheckedPtr` for other pointers, or when you need to avoid reference cycles. There are several things you should consider when choosing when deciding whether to use `Checked` pointers or `Weak` ones:
- Does the type subclass `CanMakeWeakPtr` or `CanMakeCheckedPtr` already? You may consider using the pointer type which doesn’t require subclasses a new base class.
- Checked pointers are normally more performant than Weak ones and may be required in performance-sensitive code.
- Checked pointers will crash in the case where Weak ones would harmlessly become null, and those crashes can be hard to debug.

**Right:**
```cpp
class Foo {
private:
    Ref<Document> m_document;
    RefPtr<Node> m_node;
    WeakPtr<RenderObject> m_renderer;
    WeakRef<RenderObject> m_rootRenderer;
    WeakHashSet<Listener> m_listeners;
    CheckedPtr<Owner> m_owner;
    CheckedRef<Owner> m_rootOwner;
    HashSet<CheckedRef<Owner>> m_owners;
};
```

**Wrong:**
```cpp
class Foo {
private:
    Document& m_document;
    Node* m_node;
    RenderObject* m_renderer;
    RenderObject& m_rootRenderer;
    HashSet<Listener*> m_listeners;
};
```

### Capture `protectedThis` or `weakThis` in asynchronous lambdas where you need to use `this`

**Reasoning:**

Before using `this` in an asynchronous lambda, we should make sure that `this` is still alive. You should capture `protectedThis = Ref { *this }` to make sure `this` to keep `this` alive until the lambda is executed.
Alternatively, if you don’t need to keep `this` alive, you can capture `weakThis = WeakPtr { *this }`. When capturing `weakThis`, you should NOT capture `this` as its use would be too error-prone. You should capture `weakThis` only then in the lambda, you can construct a `protectedThis` from the `weakThis` and null check it right away.

**Right:**
```cpp
// Keep `this` alive.
doAsyncWork([this, protectedThis] {
    foo(); // Member function on `this`.
    bar(); // Member function on `this`.
});

// Do not keep `this` alive but use it safely.
doAsyncWork([weakThis = WeakPtr { *this }] {
    RefPtr protectedThis = weakThis.get();
    if (!protectedThis)
        return;
    protectedThis->foo();
    protectedThis->bar();
});
```

**Wrong:**
```cpp
// Fails to keep `this` alive.
doAsyncWork([this] {
    foo(); // Member function on `this`.
});

// Shouldn’t capture `this` as it is too easy to use-after-free.
doAsyncWork([this, weakThis = WeakPtr { *this }] {
    if (!weakThis)
        return;
    foo(); // Member function on `this`.
    bar(); // Member function on `this`.
});
```

## Manage resources automatically using resource handles and RAII

**Reasoning:**

This avoids leaks, double frees and complexity related to manual resource management.

A few examples of this are:
* Use `Ref` / `RefPtr` instead of explicit `ref()` / `deref()` calls.
* Use `WTF::UniqueRef` / `std::unique_ptr` to avoid explicit `new` / `delete` calls.
* Use `Locker` to avoid explicit calls to `Lock::lock()` / `Lock:unlock()`.

In general, this applies to any 2 operations / function calls that need to be balanced in order to avoid a bug / leak. It is too easy for calls to get unbalanced, particularly due to early returns. RAII objects / handles avoids this class of bugs.

**Right:**
```cpp
Locker locker { m_valueLock };
m_value = 0;

auto resource = makeUnique<Resource>();
RunLoop::main().dispatch([resource = WTFMove(resource)] {
    // Use resource.
});
```

**Wrong:**
```cpp
m_valueLock.lock();
m_value = 0;
m_valueLock.unlock();

auto* resource = new Resource;
RunLoop::main().dispatch([resource] {
    // Use resource.
    delete resource;
});
```

The GTK/WPE ports also have some types to make it easier to interact with GLib style C APIs.

**Right:**
```cpp
GUniqueOutPtr<GError> error;
GUniquePtr<char> address(g_dbus_address_get_for_bus_sync(G_BUS_TYPE_SESSION, nullptr, &error.outPtr()));
if (error)
  g_warning("Unable to get session D-Bus address: %s", error->message);
else
  // Use address.
```

**Wrong:**
```cpp
GError* error = nullptr;
char* address = g_dbus_address_get_for_bus_sync(G_BUS_TYPE_SESSION, nullptr, &error);
if (error) {
  g_warning("Unable to get session D-Bus address: %s", error->message);
  g_error_free(error);
} else {
  // Use address.
  g_free(address);
}
```


## Guard against type confusion

### Use `downcast<>()` when casting to a subclass type

**Reasoning:**

`downcast<>()` validates the type at runtime in release builds so that bad casts no longer result in type confusion security bugs.

**Right:**
```cpp
Document* document() { return downcast<Document>(scriptExecutionContext()); }
```

**Wrong:**
```cpp
Document* document() { return static_cast<Document*>(scriptExecutionContext()); }
```

**Note:**

JavaScriptCore uses `jsCast<>()` instead of `downcast<>()` for `JSValue`.


### Prefer `dynamicDowncast<>()` over `is<>()` + `downcast<>()`

**Reasoning:**

`is<>()` + `downcast<>()` ends up checking the type twice, which may be inefficient. It is also more error prone than a single call to `dynamicDowncast<>()`. `dynamicDowncast<>()` often results in more concise code too.

**Right:**
```cpp
return dynamicDowncast<Element>(node);
```

**Wrong:**
```cpp
return is<Element>(node) ? &downcast<Element>(node) : nullptr;
```

**Note:**

JavaScriptCore uses `jsDynamicCast<>()` instead of `dynamicDowncast<>()` for `JSValue`.


### Use strongly-typed identifiers instead of `uint64_t`

**Reasoning:**

Using weakly-typed identifiers such as `uint64_t` leads to confusion between identifier types and bugs.

**Right:**
```cpp
enum class NodeIdentifierType { };
using NodeIdentifier = ObjectIdentifier<NodeIdentifierType>;

class Node : public Identified<NodeIdentifier> { };
// Node::identifier() returns a NodeIdentifier type.

enum class TaskIdentifierType { };
using TaskIdentifier = AtomicObjectIdentifier<TaskIdentifierType>;

TaskIdentifier generateTask()
{
    auto taskIdentifier = TaskIdentifier::generate();
    Locker locker { m_tasksLock };
    m_tasks.add(taskIdentifier, Task::create());
    return taskIdentifier;
}
```

**Wrong:**
```cpp
uint64_t m_identifier;
```


### Use enum classes instead of old-style enumerations

**Reasoning:**

Avoids type confusing, easier to pass over WebKit IPC.

**Right:**
```cpp
enum class Border : uint8_t { Left, Right, Top, Bottom };
```

**Wrong:**
```cpp
enum Border { BorderLeft, BorderRight, BorderTop, BorderBottom };
```


### Consider using `std::variant` instead of unions

**Reasoning:**

Unions are prone to type confusion bugs. `std::variant` fixes that.

**Right:**
```cpp
std::variant<int, double> m_value;
```

**Wrong:**
```cpp
union {
    int m_intValue;
    double m_doubleValue;
};
bool m_isDouble;
```

**Notes:**

A `std::variant<>` may be larger than an equivalent union and may thus be unsuited for classes where the size matters for performance.


## Use `ASCIILiteral` instead of `const char*` for string literals

**Reasoning:**

Using `ASCIILiteral` makes it clear that the string we’re dealing with is ASCII-only and that the string is immortal.
This allows code to leverage these facts and be more efficient. It also documents in code what the lifetime of the string is.
Finally, `ASCIILiteral` goes bounds checking on indexed access, thus avoiding out-of-bounds access bugs.

**Right:**
```cpp
static constexpr auto foo = "foo"_s; // `_s` suffix gives you an ASCIILiteral. 

static ASCIILiteral toString(EnumType value)
{
    switch (value) {
    case EnumType::A:
        return "A"_s;
    case EnumType::B:
        return "B"_s;
    default:
        return "Others"_s;
    }
}
```

**Wrong:**
```cpp
static const char* foo = "foo";

static const char* toString(EnumType value)
{
    switch (value) {
    case EnumType::A:
        return "A";
    case EnumType::B:
        return "B";
    default:
        return "Others";
    }
}
```

## Use `std::exchange()` instead of `WTFMove()` when the "moved-from" variable may get reused

**Reasoning:**

Using a variable after it’s been "moved from" is bad practice in C++ and may result in unexpected behavior depending on how the move constructor is implemented. std::optional move constructor, for example, doesn’t reset the "moved-from" value to `std::nullopt`.

**Right:**
```cpp
if (RefPtr data = std::exchange(m_data, nullptr))
    data->detach();
```

**Wrong:**
```cpp
if (RefPtr data = WTFMove(m_data))
    data->detach();
```


## Subclassing `RefCounted` / `ThreadSafeRefCounted`

### Subclasses of `RefCounted` / `ThreadSafeRefCounted` should not have a public constructor

**Reasoning:**

A public `create()` factory function should be exposed, to create instances of such classes to make sure that we adopt the object after construction and store it in a `Ref` / `RefPtr`. Mixing explicit memory management and ref-counting can result in use-after-free bugs.

**Right:**
```cpp
class Foo : public RefCounted<Foo> {
public:
    static Ref<Foo> create();
private:
    Foo();
}
```

**Wrong:**
```cpp
class Foo : public RefCounted<Foo> {
public:
    Foo();
}
```

### Subclasses of `RefCounted` / `ThreadSafeRefCounted` should have a virtual destructor if they’re not `final`

**Reasoning:**

`RefCounted<T>` / `ThreadSafeRefCounted<T>` call `delete static_cast<T*>(this)`. If the subclass `T` is not final (has subclasses), then those subclasses’s destructors will not run unless `T`’s destructor is marked as `virtual`.

**Right:**
```cpp
class Foo : public RefCounted<Foo> {
public:
    virtual ~Foo(); // Needs to be virtual since Foo is not final.
};

class Bar : public Foo { };

class FinalBaz final : public RefCounted<FinalBaz> {
public:
    ~FinalBaz(); // Doesn't need to be virtual since the class is final.
}
```

**Wrong:**
```cpp
class Foo : public RefCounted<Foo> {
public:
    ~Foo();
};

class Bar : public Foo { };
```

## Guard against out-of-bounds access

### Use `std::span` instead of a raw pointer and a size

**Reasoning:**

`std::span` does bounds checking, less risk of size and buffer getting out of sync. std::span also provides API such as `subspan()`, `first()` and `last()` which can be used to avoid doing unsafe pointer arithmetics.

**Right:**
```cpp
void didReceiveData(std::span<const uint8_t>);
```

**Wrong:**
```cpp
void didReceiveData(const uint8_t* data, size_t size);
```

### Use `std::array` instead of C array

**Reasoning:**

`std::array` does bounds checking, whereas C-style arrays do not.

**Right:**
```cpp
std::array values { 1, 2, 3 };
```

**Wrong:**
```cpp
int values[] = { 1, 2, 3 };
```

### Every container / view type should do boundary check on element access

**Reasoning:**

Bounds checking in release builds plays a critical part of achieving memory safety.

**Right:**
```cpp
class StringView {
public:
    UChar operator[](size_t index) const
    {
        RELEASE_ASSERT(index < length());
        if (is8Bit())
            return characters8()[index];
        return characters16()[index];
    }
private:
    const LChar* characters8() const;
    const UChar* characters16() const;
}
```

**Wrong:**
```cpp
class StringView {
public:
    UChar operator[](size_t index) const
    {
        if (is8Bit())
            return characters8()[index];
        return characters16()[index];
    }
private:
    const LChar* characters8() const;
    const UChar* characters16() const;
}
```

**Note:**

The view / container type may wrap another container / view which already does bounds checking, in which case you do not need to duplicate the check:
```cpp
class NodeList {
public:
    Node* operator[](size_t index) const
    {
        // No need for a RELEASE_ASSERT() here.
        return m_nodes[index];
    }
private:
    Vector<Node> m_nodes; // Vector::operator[] already validates bounds.
}
```

## Do not use protected visibility for data members

**Reasoning:**

Poor data encapsulation is a frequent source of bugs.

**Right:**
```cpp
class Parent {
protected:
    const String& value() const { return m_value; }
    void setValue(String&& value) { m_value = WTFMove(value); }
private:
    String m_value;
};

class Child : public Parent {
// Child uses value() / setValue().
};
```

**Wrong:**
```cpp
class Parent {
protected:
    String m_value;
};

class Child : public Parent {
// Child uses m_value.
};
```


## Improve thread safety

### Use `crossThreadCopy()` passing an object to another thread

**Reasoning:**

Many WebKit types require copying to be safely passed to another thread. In the past, we’ve sometimes determined that it was sufficient to `WTFMove()` some types in some cases. There is no longer any performance win associated with this so we should now do the safe thing and call `crossThreadCopy()` unconditionally.

**Right:**
```cpp
void MyClass::didReceiveData(String&& data)
{
    m_workQueue->dispatch([data = crossThreadCopy(WTFMove(data))] {
        // Do something.
    });
}
```

**Wrong:**
```cpp
void MyClass::didReceiveData(String&& data)
{
    m_workQueue->dispatch([data = WTFMove(data)] {
        // Do something.
    });
}
```

**Notes:**

Many WebKit types (such as `String`) have `isolatedCopy()` overloads which may result in better performance when calling `crossThreadCopy()` on a r-value reference. 


### Use thread safety analysis macros in `<wtf/ThreadSafetyAnalysis.h>`

**Reasoning:**

Multi-threading is a common source of (security) bugs and these macros can help catch a lot of the bugs at build time. They also help document the code.

**Right:**
```cpp
class Foo {
protected:
    void initializeValuesWhileLocked() WTF_REQUIRES_LOCK(m_valuesLock);
private:
    Lock m_valuesLock;
    std::array<int, 10> m_values WTF_GUARDED_BY_LOCK(m_valuesLock);
};
```

**Wrong:**
```cpp
class Foo {
protected:
    void initializeValuesWhileLocked();
private:
    Lock m_valuesLock;
    std::array<int, 10> m_values;
};
```

## IPC Hardening

### Always validate data coming from IPC

**Reasoning:**

A child process can become compromised and we should therefore not trust any data it sends other processes via IPC. Failure to properly validate received data can lead to sandbox escapes or denial of service attacks (by crashing the recipient process). In general, IPC from WebContent processes have the highest risk. However, other child processes can in theory get compromised too and send bad IPC.

It is never acceptable for bad IPC from a compromised process to cause the recipient to crash, even in a non-exploitable way (such as a `RELEASE_ASSERT()`). Also, instead of ignoring bad IPC, it is better practice to kill the compromised process so it cannot keep trying to find an exploit.

Validation can happen at two levels:
* Ideally during IPC deserialization, in generated code (using `[Validator=X]` in `*.serialization.in` files)
* Otherwise, after deserialization using a `MESSAGE_CHECK()`

Both will cause the termination of the sender.

**Right:**
```cpp
class WebCore::ShareableBitmapConfiguration {
    [Validator='m_size->width() >= 0 && m_size->height() >= 0'] WebCore::IntSize m_size;
};
```

**Wrong:**
```cpp
class WebCore::ShareableBitmapConfiguration {
    WebCore::IntSize m_size; // Then assume m_size contains positive values.
};
```

**Right:**
```cpp
void Foo::setCookiesFromDOM(const URL& firstParty, const String& cookies)
{
    MESSAGE_CHECK(allowsFirstPartyForCookies(m_processIdentifier, firstParty));
    // Set cookies...
}
```

**Wrong:**
```cpp
void Foo::setCookiesFromDOM(const URL& firstParty, const String& cookies)
{
    // Assume this WebContent process is allowed to set cookies for this
    // |firstParty| and set cookies...
}
```

### Disable IPC endpoints that are no longer relevant when a particular feature is disabled

**Reasoning:**

All IPC endpoints are potential attack vectors. As a result, if certain features are disabled (which is often the case for experimental ones), its related IPC endpoints (or messages) should also be disabled to reduce the surface of attack.

IPC endpoints can be conditionally enabled using `[EnabledIf='isFooEnabled()']` in the `*.messages.in` file.

**Right:**
```cpp
messages -> GPUConnectionToWebProcess {
  [EnabledIf='isWebGPUEnabled()'] void CreateRemoteGPU(WebKit::WebGPUIdentifier identifier)
  [EnabledIf='isWebGPUEnabled()'] void ReleaseRemoteGPU(WebKit::WebGPUIdentifier identifier)
}
```

**Wrong:**
```cpp
messages -> GPUConnectionToWebProcess {
  // These messages are only sent when the WebGPU feature is turned on.
  void CreateRemoteGPU(WebKit::WebGPUIdentifier identifier)
  void ReleaseRemoteGPU(WebKit::WebGPUIdentifier identifier)
}
```

### Manual `encode()` / `decode()` functions should no longer be used for IPC serialization

**Reasoning:**

Manually writing `encode()` or `decode()` for a class / struct is no longer acceptable in new code. Most of our IPC serializers / deserializers get generated from an IDL format in `*.serialization.in` files (e.g. `WebCoreArgumentCoders.serialization.in`), and any remaining classes with manual `encode()` / `decode()` methods will be transitioned soon.

Generating serialization methods reduces the risk of bugs, automatically includes validation logic when encoding and decoding, and supports automatic testing of serialized types.

**Right:**
```cpp
// In WebCoreArgumentCoders.in:
struct WebCore::FloatPoint {
    float m_x;
    float m_y;
}
```

**Wrong:**
```cpp
// In FloatPoint.h:
namespace WebCore {
struct FloatPoint {
    float m_x;
    float m_y;
    
    template<typename Encoder>
    void encode(Encoder& encoder) const
    {
        encoder << m_x << m_y;
    }
    
    template<typename Decoder>
    static std::optional<FloatPoint> decode(Decoder& decoder)
    {
        std::optional<float> x;
        decoder >> x;
        if (!x)
            return std::nullopt;
        std::optional<float> y;
        decoder >> y;
        if (!y)
            return std::nullopt;
        return FloatPoint { *x, *y };
    }
}
} // namespace WebCore
```