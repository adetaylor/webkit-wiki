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

* <[https://trac.webkit.org/r261670](https://trac.webkit.org/changeset/261670)> Web Inspector: show EventTarget listeners as an internal property
* <https://trac.webkit.org/r261499> Web Inspector: show JavaScript Worker name as an internal property
* <https://trac.webkit.org/r255986> Web Inspector: show JavaScript Worker terminated state as an internal property
* <https://trac.webkit.org/r224606> Web Inspector: Show Internal properties of PaymentRequest in Web Inspector Console
* <https://trac.webkit.org/r197061> Web Inspector: Expose Proxy target and handler internal properties to Inspector
* <https://trac.webkit.org/r187959> Web Inspector: Improve Support for PropertyName Iterator (Reflect.enumerate) in Inspector
* <https://trac.webkit.org/r181203> Web Inspector: ES6: Improved Support for Iterator Objects
* <https://trac.webkit.org/r180236> Web Inspector: Improved Console Support for Bound Function
* <https://trac.webkit.org/r180235> Web Inspector: ES6: Improved Console Support for Promise Objects