To effectively reduce build times, it is first important to understand where that time is being spent. This page documents various tools and techniques used to perform detailed analysis of where the complier is spending its time.

# Clang 9+

Clang 9 has introduced a new flag, [-ftime-trace](https://releases.llvm.org/9.0.0/tools/clang/docs/ReleaseNotes.html#new-compiler-flags), which will generate time-profile information as a .json artifact during compilation. These artifacts include timing information on a per-header, per-function, per-template, or even per-optimization level.

# Building ClangBuildAnalyzer

While per-object-file trace information is useful for uncovering what causes compiling a specific source file to take as long as it does, tracing overall build time requires summarizing those traces across the entire project. The same person who originally authored the Clang tracing patch also built a summarization tool using those per-object-file traces as input, [ClangBuildAnalyzer](https://github.com/aras-p/ClangBuildAnalyzer).  Here's how to build ClangBuildAnalyzer with Xcode on macOS:

1. `git clone https://github.com/aras-p/ClangBuildAnalyzer`
2. `cd ClangBuildAnalyzer/projects/xcode`
3. `xcodebuild`
4. Copy `build/Release/ClangBuildAnalyzer` to a location in `$PATH`

# Building WebKit with tracing enabled

1. Clean your build directory (but make sure it still ''exists'').
2. `ClangBuildAnalyzer --start path/to/WebKitBuild`
3. `make debug ARGS='OTHER_CFLAGS="-ftime-trace" OTHER_CPLUSPLUSFLAGS=-ftime-trace' `
or:
3. Create a file called `LocalOverrides.xcconfig` in the root of the WebKit checkout with the following contents, and build with Xcode:
``` cpp
OTHER_CFLAGS=$(inherited) -ftime-trace
OTHER_CPLUSPLUSFLAGS=$(inherited) -ftime-trace
```

Then, when the build is complete:
4. `ClangBuildAnalyzer --stop path/to/WebKitBuild path/to/output/file`
5. `ClangBuildAnalyzer --analyze path/to/output/file > path/to/text/file`

ClangBuildAnalyzer writes a file with the current time to your build directory when run with `--start`. Then when run with `--stop`, it collects all of the trace files generated during that time window, and collates them into the output file. `--analyze` turns that into human-readable output. Profiling individual projects within WebKit would involve running steps 1-5 from within, e.g., the Source/WebCore directory.

# Resolving Expensive Headers

ClangBuildAnalyzer will generate a list of the ten most expensive (in terms of compilation time) headers encountered during the build. The current list of most expensive headers for WebKit projects is tracked at [[Expensive Headers]].

For example:

``` cpp
*** Expensive headers:
832243 ms: /Volumes/Data/WebKit/Source/WebCore/bindings/js/JSDOMGlobalObject.h (included 246 times, avg 3383 ms), included via:
  JSMallocStatistics.o JSMallocStatistics.h JSDOMWrapper.h  (6515 ms)
  JSMemoryInfo.o JSMemoryInfo.h JSDOMWrapper.h  (6490 ms)
  JSInternalSettingsGenerated.o JSInternalSettingsGenerated.h JSDOMWrapper.h  (6473 ms)
  JSApplePayCancelEvent.cpp JSApplePayCancelEvent.h JSDOMWrapper.h  (6455 ms)
  JSInternalsSetLike.o JSInternalsSetLike.h JSDOMWrapper.h  (6435 ms)
  JSMockPageOverlay.o JSMockPageOverlay.h JSDOMWrapper.h  (6430 ms)
  ...
```

From this we can see that JSDOMGlobalObject.h is a very expensive header; it contributes about 3.3s of compile time, on average, to every source file which includes it. And it is included 246 times, which given the WebKit unified build system, means it is included by a majority of source files in the WebCore project. For these expensive headers, its often the case that the "expensive" header is expensive due to including other expensive headers, and one approach to make that header less expensive is to forward declare types rather than include their definitions. In cases where inline implementations of methods make forward declaration impossible, those inline definitions can be moved into a `<Type>Inlines.h` file, and the original declarations annotated with `inline`. Source files which contain references to those inline functions must include the `<Type>Inlines.h` file, or the compile will generate a `-Wundefined-inline` error.

## Header Best Practices

While resolving some expensive headers, a few best practices stood out to reduce compilation times without regressing runtime performance:

### Forward-declare all the things

Forward-declaring types used by your class's methods allows clients who don't call those methods to not incur the compile-time cost of including those types' headers.

C++ class members must be fully defined, so e.g. consider using `UniqueRef<MyClass>`, and forward-declaring `MyClass`, instead of including `MyClass.h`.

C++ function parameters and return values do not need to be fully defined, and can be forward-declared. So e.g. if a method in your class uses `MyClass` as function parameter or return value, consider forward-declaring `MyClass`, instead of including `MyClass.h`.

C++ virtual functions are rarely able to be inlined, even if decorated with `inline`, so avoid inline definitions of virtual functions. So e.g. if defining a virtual base class method that returns a `Ref<MyClass>`, put the default implementation inside the class implementation file.

Bad:
``` cpp
// MyClass.h
#include "YourClass.h"
class MyClass {
public:
    void passByValue(YourClass);
    void passByReference(const YourClass&);
    YourClass getByValue();
    virtual std::unique_ptr<YourClass> createUnique() { return nullptr; }
};
```

Good:
``` cpp
// MyClass.h
class YourClass;

class MyClass {
public:
    void passByValue(YourClass);
    void passByReference(const YourClass&);
    YourClass getByValue();
    virtual std::unique_ptr<YourClass> createUnique();
};

// MyClass.cpp
#include "YourClass.h"
std::unique_ptr<YourClass> MyClass::createUnique()
{
    return nullptr;
}
```


### Avoid class-scoped enums

When defining a public enumeration for a class, do so at namespace scope rather than inside the class, and always specify an explicit enum size. E.g.:

Bad:
``` cpp
// MyClass.h
class MyClass {
public:
    enum Options {
        Option1,
        Option2,
        Option3,
    };
...
};
```

Good:
``` cpp
// MyClass.h
enum class MyClassOptions : uint8_t {
    Option1,
    Option2,
    Option3,
};
class MyClass {
public:
...
};
```

This allows clients to forward declare the enum type, rather than include `MyClass.h`:
``` cpp
// YourClass.h
enum class MyClassOptions : uint8_t;
class YourClass {
    void functionWithMyClassOptions(MyClassOptions);
};
```

### Add Inlines.h headers

When explicitly inlining a function definition for performance reasons, and that definition requires including an external header, putting the inline definition in an Inlines.h header file. Annotate the class method declaration with `inline`, which will cause a compiler warning if the Inline.h header is not included by the caller. E.g.:

Bad:
``` cpp
// MyClass.h
#include "YourClass.h"
class MyClass {
public:
    YourClass getFoo() { return YourClass::foo(); }
};
```

Good:
``` cpp
// MyClass.h
class YourClass;

class MyClass {
public:
    inline YourClass getFoo();
};

// MyClassInlines.h
#include "YourClass.h"
inline YourClass& MyClass::getFoo() { return YourClass::foo(); }
```

### Avoid Virtual Inlines

Virtual functions will almost never gain any benefit from being inlined (unless callers cast the function itself, e.g.: `foo->Derived::bar()` instead of `foo->bar()`, which is a very uncommon practice). If a virtual function definition in a header file requires including an external header, consider moving the definition into the implementation file. E.g.:

Bad:
``` cpp
// MyClass.h
#include "YourClass.h"
class MyClass final : public BaseClass {
public:
    void doFoo() final { m_yourClass->doFoo(); }

private:
    Ref<YourClass> m_yourClass;
};
```

Good:
``` cpp
// MyClass.h
class MyClass final : public BaseClass {
public:
    void doFoo() final;

private:
    Ref<YourClass> m_yourClass;
};

// MyClass.cpp
#include "MyClass.h"
#include "YourClass.h"
void MyClass::doFoo() { m_yourClass->doFoo(); }
```

### Avoid Private Inlines

Private functions, by definition, can only be called from class methods and declared friends, so defining private functions in the class header file is of limited value. If those private functions are called by friends or other inlined functions, consider moving the private function definition into an Inline.h header. Otherwise, consider moving the definition into the implementation file.

Bad:
``` cpp
// MyClass.h
class MyClass {
private:
    void doFoo() { foo(); }
};
```

Good:
``` cpp
// MyClass.h
class MyClass {
private:
    void doFoo();
};

// MyClass.cpp
#include "MyClass.h"
void MyClass::doFoo() { foo(); }
```

### Avoid Default Parameters

Default parameters for non-built-in types usually require including in the full declaration for those types into the header declaring the function which takes that type. This is because the caller must have the full definition of that type at the call site. Instead, overloading can be used to provide the same mechanic at the call site without requiring the caller to have access to the parameter type definition, allowing the parameter type to be forward-declared by that header.

E.g.:
``` cpp
// ParameterType.h
enum class EnumParameter : bool {
    Bad,
    Good,
};

struct StructParameter {
    StructParameter(int value) : m_value(value) { }
    int m_value { 0 };
};
```

Bad:
``` cpp
// MyClass.h
#include "ParameterType.h"
class MyClass {
private:
    void doEnum(EnumParameter = EnumParameter::Good);
    void doStruct(StructParameter&& = StructParameter { 1 });
};
```

Good:
``` cpp
// MyClass.h
enum class EnumParameter : bool;
struct StructParameter;

class MyClass {
private:
    void doEnum();
    void doEnum(EnumParameter);
    void doStruct();
    void doStruct(StructParameter&&);
};

// MyClass.cpp
#include "MyClass.h"
void MyClass::doEnum() { doEnum(EnumParameter::Good); }
void MyClass::doStruct() { doStruct(StructParameter { 1 }); }
```

