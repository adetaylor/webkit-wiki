When objects are logged to the Web Inspector Console, visual representations are generated for those objects, including listing member properties and the prototype chain.  In addition to values accessible by other JavaScript, since all of this is happening inside Web Inspector, additional internal details can be exposed to make development/debugging easier.

This is done in two places:

* for core JavaScript `JSInjectedScriptHost::getInternalProperties`
* for all other web `WebInjectedScriptHost::getInternalProperties`

Internal properties are shown as greyed-out properties when viewing any JavaScript object in Web Inspector. 

Internal property “descriptors” take the form of a `JSArray` of:

```json
{
    "name": "...",
    "value": <JSValue>
}
```

(which is the same format as `Runtime.InternalPropertyDescriptor` as that's eventually what they're converted into when sent to Web Inspector)

Much of the work is done for you by doing (replace `foo`/`Foo`/`bar`/`baz` with whatever object you're inspecting):

```cpp
if (auto* foo = JSFoo::toWrapped(vm, value)) {
    auto* array = constructEmptyArray(globalObject, nullptr);
    RETURN_IF_EXCEPTION(scope, { });
    
    unsigned index = 0;
    
    array->putDirectIndex(globalObject, index++, constructInternalProperty(globalObject, "bar"_s, foo->bar()));
    RETURN_IF_EXCEPTION(scope, { });
    
    array->putDirectIndex(globalObject, index++, constructInternalProperty(globalObject, "baz"_s, foo->baz()));
    RETURN_IF_EXCEPTION(scope, { });
    
    return array;
}
```

Web Inspector ultimately consumes this array of internal property “descriptors” as part of the return value for

* `internalProperties` of `Runtime.getProperties`
* `internalProperties` of `Runtime.getDisplayableProperties`


Past Examples:

* [282553@main](https://commits.webkit.org/282553@main) Web Inspector: Console: show boundThis for arrow functions
* [282000@main](https://commits.webkit.org/282000@main) Web Inspector: Console: add internal properties for registrations of a FinalizationRegistry
* [264171@main](https://commits.webkit.org/264171@main) Web Inspector: Console: add an internal property for the target of a WeakRef
* [224767@main](https://commits.webkit.org/224767@main) Web Inspector: show EventTarget listeners as an internal property
* [224645@main](https://commits.webkit.org/224645@main) Web Inspector: show JavaScript Worker name as an internal property
* [220345@main](https://commits.webkit.org/220345@main) Web Inspector: show JavaScript Worker terminated state as an internal property
* [195505@main](https://commits.webkit.org/195505@main) Web Inspector: Show Internal properties of PaymentRequest in Web Inspector Console
* [172764@main](https://commits.webkit.org/172764@main) Web Inspector: Expose Proxy target and handler internal properties to Inspector
* [165749@main](https://commits.webkit.org/165749@main) Web Inspector: Improve Support for PropertyName Iterator (Reflect.enumerate) in Inspector
* [160483@main](https://commits.webkit.org/160483@main) Web Inspector: ES6: Improved Support for Iterator Objects
* [159739@main](https://commits.webkit.org/159739@main) Web Inspector: Improved Console Support for Bound Function
* [159738@main](https://commits.webkit.org/159738@main) Web Inspector: ES6: Improved Console Support for Promise Objects