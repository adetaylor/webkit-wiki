# WebKit Guidelines for Safer IPC Programming

See also [guidelines for Safer C++](Safer-CPP-Guidelines.md).

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

IPC endpoints can be conditionally enabled using `[EnabledBy=FooEnabled]` in the `*.messages.in` file.

**Right:**
```cpp
messages -> GPUConnectionToWebProcess {
  [EnabledBy=WebGPUEnabled] void CreateRemoteGPU(WebKit::WebGPUIdentifier identifier)
  [EnabledBy=WebGPUEnabled] void ReleaseRemoteGPU(WebKit::WebGPUIdentifier identifier)
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
