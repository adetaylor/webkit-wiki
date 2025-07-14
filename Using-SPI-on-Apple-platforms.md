On some Apple ports (e.g. iOS), WebKit tracks its usage of SPI (private API) via a build-time tool called [`audit-spi`](https://github.com/WebKit/WebKit/tree/main/Tools/Scripts/libraries/webkitapipy). We use SPI allowlists ([1], [2]) to document SPI that are in use and ensure there is a bug tracking their cleanup.

[1]: https://github.com/WebKit/WebKit/blob/main/Source/WebKit/Configurations/AllowedSPI.toml
[2]: https://github.com/WebKit/WebKit/blob/main/Source/WebCore/Configurations/AllowedSPI.toml

Allowlists are project-specific and stored in each project's `Configurations` directory. WebKitAdditions has its own project-specific allowlists as well.

### Allowlist format

An allowlist is a TOML document that organizes allowed declarations into
functional categories and associates each with an *exception status*
that justifies why the SPI is allowed. Each entry has three parts:

```toml
[<category>.<exception>]
symbols = [ "..." ]
classes = [ "..." ]
selectors = [ "..." ]
requires = [ "..." ]
```

The `<category>` key is an arbitrary string. It documents the reason or subsystem that a particular SPI is being used for.

The `<exception>` is either:
- A bug URL, denoting a temporary exception that will
be cleaned up when the bug is closed.
- A permanent exception string, denoting SPI use that we never intend to clean up. Exception types are:

  - **`not-web-essential`**: Functionality that another browser vendor would either not use or provide their own implementation
  - **`equivalent-api`**: SPI that has the same behavior as API except in internal builds or testing workflows.

#### Allowed names

`symbols`, `classes`, and `selectors` lists denote string names of symbols, ObjC classes, and ObjC selectors respectively.

All allowed names must be matched when checking SPI. In other words,
it is an error for a declaration to be allowed but not used by any of
the input files given to `audit-spi`. This helps keep the list of
temporary exceptions accurate, and it's also a line of defense against
accidentally leaking internal-only SPI into public allowlists.

#### Requirements

An allowlist entry may have an optional `requires` list, which lists <wtf/Platform.h> conditions that must all be active for the allowed SPI to be considered. A condition can be inverted by prefixing it with `!`.

```toml
requires = [ "ENABLE_FOO", "!ENABLE_BAR" ]
```

#### Example

```toml
[graphics."rdar://147613178"]
symbols = ["_kCAContentsFormatRGBA10XR"]
```

This entry allows use of the symbol `_kCAContentsFormatRGBA10XR`, associates it with
the "graphics" category, and denotes a temporary exception with
[rdar://147613178](https://rdar.apple.com/147613178) as the cleanup bug.

