Explanation of the project to replace the existing representations of CSS property values with a new implementation based on a value semantics and strong typing.

***

Today, roughly speaking, CSS property values are represented in two main forms:

1) [aka `CSS`] the CSSValue class hierarchy, stored in a CSSProperty.
2) [aka `Style`] the RenderStyle class family, stored in a RenderStyle. 

These two forms need to be bi-directionally convertible. `CSS` gets converted to `Style` via style building, and `Style` gets converted to `CSS` for computed style extraction.

In the current implantation, pretty much all of this uses reference semantics and is hand written, often relying on the programmer to perform unsafe casting to perform conversions.

The goal of this project is to replace both the `CSS` and `Style` parts with a new system based on value types. By utilizing strongly typed primitive values, the goal is to allow higher level types composed of the primitives to write as little code as possible, instead, focusing on declaratively describing the types.

For a contrived simple example, let's imagine a CSS property called `foo` with the grammar `<foo> = <length [0,∞]> <number>`.

In the old model we would probably have:
- `CSS` form: `Ref<CSSValuePair>` with two `Ref<CSSPrimitiveValue>` members
- `Style` form: a `Length` and a `float`, somewhere in RenderStyle.

In addition, we would have conversion code in `StyleBuilderConverter` which would do three unsafe casts, one for the CSSValuePair, one for each primitive value. It would further assume the types of the primitive values. And if it got everything right, it would also have a blending in CSSPropertyAnimation that remember to call the `blend` function that takes a `ValueRange`. It would also have custom code to convert from the `Style` to `CSS` representation in `ComputedStyleExtractor`. 

In the new model, we would have:
- `CSS` form:
```c++
namespace WebCore::CSS {
    using Foo = SpaceSeparatedTuple<Length<CSS::Nonnegative>, Number<>>;
}
```
- `Style` form:
```c++
namespace WebCore::Style {
    using Foo = SpaceSeparatedTuple<Length<CSS::Nonnegative>, Number<>>;
}
```

No additional conversion logic is required. No custom blending is required. 

For an instance of `CSS::Foo`, `foo`:
- to serialize it, one can call `CSS::serializationForCSS(foo)` or `CSS::serializationForCSS(builder, foo)` (depending on if you have a builder you want to append to or not).
- to check for computed style dependencies, one can call `CSS::collectComputedStyleDependencies(dependencies, foo)`.
- to convert from `CSS` to `Style`, one can call `Style::toStyle(foo, styleBuilder)`.

For an instance of `Style::Foo`, `foo`:
- to convert from `Style` to `CSS`, one can call `Style::toCSS(foo, renderStyle)`.
- to blend it with another instance of `Style::Foo`, `bar`, one can call `Style::blend(foo, bar, blendingContext)`.

All those calls will utilize the wealth of builtin primitives to correctly implement the functionality.

***

Now, as stated, that is a bit of a contrived example. In practice, many complex types that need to be used for more than just conversion, serialization and blending, may want to have real names for member variables.

To allow that, we allow annotating types to make them "tuple-like".

What is a “tuple-like” type? It’s any types that returns valid results for calls to `get<>()`, `std::tuple_size` and `std::tuple_element`. 

In practice, this means implementing a `get<>()` function and instantiating a macro. For example, for struct `Bar`:

```c++
namespace WebCore::CSS {
    struct Bar {
        LengthPercentage<> wibble;
        LengthPercentage<> wobble;
    };

    template<size_t I> const auto& get(const Bar& bar)
    {
        if constexpr (!I)
            return bar.wibble;
        else if constexpr (I == 1)
            return bar.wobble;
    }
}
CSS_TUPLE_LIKE_CONFORMANCE(Bar, 2) /* the 2 here is the total number of members */
```

In this case, you would also need to implement serialization yourself, since we haven't given enough information here to have the system figure out how you want your elements separated. To do that, we can introduce the secret sauce used for the system, template specialization. For `Bar`, serialization, assuming we want the members comma separated, would look like:

```c++
namespace WebCore::CSS {
    template<> struct Serialize<Bar> {
        void operator()(StringBuilder& builder, const Bar& bar)
        {
            serializeForCSS(builder, bar.wibble);
            builder.append(", "_s);
            serializeForCSS(builder, bar.wobble);
        }
    };
}
```

We call `Serialize` an "interface", and say that `Bar` is conforming to it by specializing it like this. In fact, all the builtin types we have seen so far, `SpaceSeparatedTuple`, `Length<>`, etc. have specializations, which is how the automatic generation works. In the case of something like `SpaceSeparatedTuple`, it's what is called a "partial specialization", meaning there is still some generic aspect that allows it to call down to its elements.

For the `Style` side of `Bar`, things would look pretty similar:

```c++
namespace WebCore::Style {
    struct Bar {
        LengthPercentage<> wibble;
        LengthPercentage<> wobble;
    };

    template<size_t I> const auto& get(const Bar& bar)
    {
        if constexpr (!I)
            return bar.wibble;
        else if constexpr (I == 1)
            return bar.wobble;
    }
}
STYLE_TUPLE_LIKE_CONFORMANCE(Bar, 2) /* the 2 here is the total number of members */
```

We need one additional thing here, which is to tell the system that `CSS::Bar` and `Style::Bar` are a pair (alas, just the names being the same is not enough). Since both types have the same structure, and conform the "tuple-like" protocol, we can use a helper macro:

```c++
DEFINE_CSS_STYLE_MAPPING(CSS::Bar, Bar)
```

Under the covers, this expands to:

```c++
template<> struct CSSToStyleMapping<CSS::Bar> { using type = Bar; };
template<> struct StyleToCSSMapping<Bar> { using type = CSS::Bar; };
```

which is enough to allow bi-directional conversion.

But if one needs more control over conversion, specialization can be used just like with `Serialization`. In this case, one would specialize the types `ToCSS` and `ToStyle`. For `Bar` that might look like this (though, remember, there would be no reason to do this for `Bar`):

``c++
template<> struct ToCSS<Bar> { 
    auto operator()(const Bar& bar, const RenderStyle& style) -> CSS::Bar
    {
        return CSS::Bar { toCSS(bar.wibble, style), toCSS(bar.wobble, style) };
    }
};

template<> struct ToStyle<CSS::Bar> { 
    auto operator()(const CSS::Bar& bar, const BuilderState& state, const CSSCalcSymbolTable& symbolTable) -> Bar
    {
        return Bar { toStyle(bar.wibble, state, symbolTable), toStyle(bar.wobble, state, symbolTable) };
    }
};
```
