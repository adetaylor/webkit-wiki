The current implementation of CSS Values in WebKit is organic and adhoc, and while it has served us well, it could use investment to make it easier to maintain and make it easier to add new properties and values. In this proposal I will outline a vision for this infrastructure.

## Goals

* Generate as much as possible from CSSProperties.json, including the property parsers
* Strongly typed, immutable, and comprehensive output from the property parsers
  - We should never have to "trust" that the parser only produces "valid" values, the type system should enforce it for us
* Thread safe (the need for the CSS property parser from Workers is clear, let's make it foolproof)
* Memory efficient (can't regress things)

## Details

CSS properties in WebCore are currently modeled as a tuple of `CSSPropertyID` & `metadata` & `CSSValue`, where `CSSPropertyID` is an integer constant (currently 0-513, amusingly 1 larger than 2^9th, so this requires 10 bits), `metadata` is a set 6 bits describing longhand/shorthand, implicitness and importance, and `CSSValue` is pointer to a reference counted type hierarchy (though non-virtualized) representation (64 bits inline, but more when you look at the pointed to value, somewhat mitigated by the use of CSSValuePool). 

One thing to notice is that there is an inherit inefficiency in using a fully general CSSValue for every property, as not every property can be represented using any CSS value. 

CSSValues are expressed in a polymorphic fashion, encouraging use of CSSValue or CSSPrimitiveValue (a common subclass) as members of other types, despite those types not actually supporting all possible values (violating Liskov). 

I propose we change CSSValue to instead be union of all the possible top level value types. In practice, this might look like removing the inheritance relationship, e.g. `CSSCubicBezierTimingFunctionValue` no longer inherits from `CSSValue`, making `CSSValue` itself no longer reference counted or heap allocated, and introducing a `std::variant` in `CSSValue` that has all the types. While doing this, I would also replace `CSSPrimitiveValue` with a family of new types, one for each kind of `CSSPrimitiveValue`, e.g. a new `CSSColorValue`, a new `CSSFloatValue`, a new `CSSAngleValue`, etc. Given we already have a union in `CSSValue`, having a second one at the `CSSPrimitiveValue` would serve no purpose. We can also apply this same rule to other sub-hierarchies like `CSSBasicShape`.

For `CSSValue` types that currently contain other `CSSValue` (or other polymorphic subtypes like `CSSPrimitiveValue`) members, rather than continuing to use them, the types should instead utilize the smallest possible subset of the `CSSValue` union as they can. For instance, `CSSRayValue` currently contains the member `Ref<CSSPrimitiveValue> m_angle`, but instead, it could now contain `std::variant<CSSAngle, CSSNumber, CSSCalc<CSSAngle>> m_angle`. (FIXME: Find a better example)

Making this proposed changed naively leads to two problems: 1) the size of CSSValue skyrockets, and 2) recursively defined values are no longer possible (for example, the `<image>` grammar is `<image> = <url> | <image()> | <image-set()> | <cross-fade()> | <element()> | <gradient>` with the `<image-set()` grammar of `image-set() = image-set( [ [ <image> | <string> ] [ <resolution> || type(<string>) ] ]# )
`).

To at least partially solve these problems, we can re-introduce some heap allocation, starting with values that contain themselves. Additionally, we will likely want to cap the maximum size of a type in the `CSSValue` `std::variant` to limit the size of `CSSValue`. Tuning this threshold will be important, so we should aim to make whether or not something is heap allocated hidden and compile time driven. A good starting point would likely be a maximum payload of 8 bytes, allowing simple types like `integers`, `floats` and `doubles` (and even some small aggregates of those) to be stored without allocation. This would still be larger than the current `CSSValues` inline cost, as the total size will be `8 bytes + sizeof(std::variant tag)`, so we may need to consider a smaller payload and a custom variant implementation using WTF::CompactPointerTuple, but we should hold off on that until measurements can be done, as we will have wins elsewhere.

One place where we can look for memory wins is in `CSSProperty`. As noted above, `CSSProperty` allows for all property ids to have any possible CSSValue associated with them, but that is not required. Most (if not all) properties only support a small subset of the possible values, meaning that `CSSValue's` `std::variant` tag is wasting bits. A `-text-decoration-skip` can only ever have one kind of a value, and identifier, so when packaged together in a `CSSProperty`, the tags bits of the `CSSValue` are of no use.

To make this kind of optimization feasible, we would want to generate a struct type for each CSSPropertyID + Supported Top Level Value, and utilize a single tag to identify the pair. Whether this will fit into 2^10 bits remains to be calculated.

### Generation

To create all the types described above by hand would be quite tedious. Instead, we should augment and utilize CSSProperty.json to meet our needs.

To accomplish this we will need to add metadata for each property about what value types are supported. 

One way to go would be to follow the specs. For example, the CSS Background and Borders spec uses the following notation to describe the allowed values for the `background-size` property: `[<bg-size>]#` (see https://drafts.csswg.org/css-backgrounds-3/#background-size), which in turn is defined as `<bg-size> = [ <length-percentage [0,∞]> | auto ]{1,2} | cover | contain`. This tells us the following type would satisfy the needs of `background-size`: 

```c++
enum class AutoKeyword { Auto };
enum class CoverOrContainKeyword { Cover, Contain };

std::variant<
    std::variant<CSSNonNegativeLengthPercentage, AutoKeyword>,
    std::pair<std::variant<CSSNonNegativeLengthPercentage, AutoKeyword>, std::variant<CSSNonNegativeLengthPercentage, AutoKeyword>>,
    CoverOrContainKeyword
>
```

The downside of using the spec grammar syntax directly would be the complexity of writing the parser and generator on top of it.

Another possibility, and perhaps a better starting point, would be to create a simpler DSL in the JSON file to describe the output. Perhaps something like:

```json
"background-size": {
    "values": [
        "cover",
        "contain",
        {
            "value": "<range>",
            "min": 1,
            "max": 2,
            "alternatives": [
                "auto",
                "<length-percentage [0,∞]>"
            ]
        }
    ],
    ...
},
```

Here, we utilize the existing `values` key and extend it with special strings that start and end with `<` and `>` (mimicking the grammar a bit).

In both cases, we can utilize the new metadata to generate the types we need, but additionally, we can start to generate the actual property parsers themselves. 