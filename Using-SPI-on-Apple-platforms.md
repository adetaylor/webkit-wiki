On some Apple ports (e.g. iOS), WebKit tracks its usage of SPI (private API) via a build-time tool called [`audit-spi`](https://github.com/WebKit/WebKit/tree/main/Tools/Scripts/libraries/webkitapipy). We use SPI allowlists ([1], [2]) to document SPI that are in use and ensure there is a bug tracking their cleanup.

[1]: https://github.com/WebKit/WebKit/blob/main/Source/WebKit/Configurations/AllowedSPI.toml
[2]: https://github.com/WebKit/WebKit/blob/main/Source/WebCore/Configurations/AllowedSPI.toml

Allowlists are project-specific and stored in each project's `Configurations` directory. WebKitAdditions has its own project-specific allowlists as well.

## Allowlist format

An allowlist is a TOML document that groups SPI declarations by an _exception kind_ explaining why the SPI is being used and bugs tracking the eventual removal of the SPI. Each entry has three parts:

```toml
[[<kind>]]
# Bug URLS:
request = "rdar://xxxxxxxxxx"  # or https://bugs.webkit.org/...
cleanup = "rdar://yyyyyyyyyy"

# Allowed names:
symbols = [ "..." ]
classes = [ "..." ]
selectors = [ "..." ]

# Requirements:
requires = [ "..." ]
```

The `<kind>` is one of the categories of SPI exceptions:
  - **`temporary-usage`**: Used for temporary workarounds or to adopt SPI before a final API form is available.
  - **`not-web-essential`**: Functionality that another browser vendor would either not use or provide their own implementation.
  - **`equivalent-api`**: SPI that has the same behavior as API except in internal builds or testing workflows.


### Allowed names

`symbols` and `classes` lists denote string names of symbols or ObjC classes respectively.

`selectors` denote ObjC selectors. Each entry is the list contains a selector name and receiver class. For example:
```
{ name = "beginExtensionRequestWithInputItems:completion:", class = "NSExtension" }
```
A class name can be `"?"` to denote an unknown receiver. Because `audit-spi` detects ObjC method usage by analyzing a binary's `objc_selector` table, it cannot check whether an allowed selector is being sent to the expected class. However, the `class` field is used to disambiguate between multiple methods in the SDK with the same name.

### Bug URLs

Each allowlist entry is associated with up to two bugs. The meaning of the two bugs depends on the kind of exception:

|Bug type| Meaning for `temporary-usage` exceptions | Meaning for permanent exceptions: `not-web-essential` or `equivalent-api` |
|-| ------- | -------- |
|`request`|Tracks a request to make an API equivalent of the SPI being used.|Tracks a request on WebKit to approve this SPI for permanent use, including a justification for why it fits into its exception category.|
|`cleanup`|Tracks WebKit's adoption of the requested API, once it is available.|N/A|

In some circumstances, temporary exceptions don't have a meaningful request associated with them (for example if SPI is being temporarily adopted to work around an unrelated issue), so the `request` bug is optional.

### Requirements

An allowlist entry may have an optional `requires` list, which lists <wtf/Platform.h> conditions that must all be active for the allowed SPI to be considered. A condition can be inverted by prefixing it with `!`.

```toml
requires = [ "ENABLE_FOO", "!ENABLE_BAR" ]
```

## Exhaustiveness

All allowed names must be matched when checking SPI. In other words,
it is an error for a declaration to be allowed but not used by any of
the input files given to `audit-spi`. This helps keep the list of
temporary exceptions accurate, and it's also a line of defense against
accidentally leaking internal-only SPI into public allowlists.

Use `requires` directives to hide allowed SPI from build configurations where they are not used.

