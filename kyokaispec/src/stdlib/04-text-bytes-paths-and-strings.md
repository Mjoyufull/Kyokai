# Text, Bytes, Paths, And Strings

[Rikona Kurasaki / Mjoyufull]
Text looks gentle until it crosses a boundary. Then the old lies appear: bytes pretending to be words, paths pretending to be strings, C strings hiding a zero byte like a knife under the table. Kyokai keeps those shapes separate.

> Trace: D30-D30a, D54, D68-D69, D120, D201
> Covers: Text, byte, C-string, OS-string, path, parsing, and conversion APIs use separate named contracts.

## Text Model

`String` is owned linear UTF-8 text with stored allocator identity. Moving a `String` transfers ownership of its allocation and allocator identity. Destroying a `String` releases its owned storage through the allocator stored in the value.

> Trace: D30-D30a, D44, D201
> Covers: `String` ownership, encoding, and allocation identity are explicit.

`StaticString` is immutable compile-time UTF-8 text produced by `static "..."`. It is not allocator-owned, not destroyable storage, and not retargeted by contextual typing into `String`. Allocating an owned `String` from a `StaticString` requires an explicit destination allocator.

> Trace: D120, D201
> Covers: Static text and owned text are separate values with explicit allocation on conversion.

Text indexing is not byte indexing unless the API name says byte. Character, scalar, grapheme, and byte views are different contracts. No stdlib API may silently normalize Unicode, silently accept invalid UTF-8, or silently reinterpret bytes as text.

> Trace: D30-D30a, D54, D85
> Covers: Text view boundaries and Unicode behavior are explicit.

| API Family | Ownership | Allocation | Failure | Capabilities | Platform | Tests | Trace |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `String` construction | Consumes or borrows source according to `from*`, `to*In`, or `into*`. | Fresh owned results take explicit allocator and store it. | `AllocError`; invalid UTF-8 returns `TextError` unless compile-time literal validation rejects it. | None. | UTF-8 on all targets. | valid/invalid UTF-8, empty, embedded NUL, allocation failure. | D30-D30a, D54, D74, D201 |
| `StaticString` | Borrowed immutable static data. | None. | Literal syntax errors are compile-time diagnostics. | None. | UTF-8 on all targets. | literal escaping, zero length, non-ASCII, invalid escape rejection. | D54, D120 |
| Text slices/views | Borrow owner; cannot outlive owner. | None. | Bad checked bounds are TPOE; validation functions return `Result`. | None. | Same as owner. | boundary indices, scalar boundaries, invalid byte offsets. | D30-D30a, D77 |
| Text formatting/parsing | Borrows or consumes by named contract. | Formatting to owned text takes allocator; sink formatting does not allocate by itself. | `ParseError`, `AllocError`, or sink error as named. | Capability only through sink value. | Locale-independent unless an API explicitly names locale data. | parse failures, round trips, sink prefix failure. | D40-D40a, D69, D102 |

> Trace: D30-D30a, D40-D40a, D54, D69, D74, D77, D102, D120, D201
> Covers: Text API families publish ownership, allocation, failure, authority, platform, and test contracts.

## Bytes

`Byte` data is not text. Byte arrays, byte spans, and byte buffers expose raw octets and do not promise UTF-8 validity. APIs that validate bytes as text have names such as `validateUtf8`, `fromUtf8In`, or another explicit text-conversion form.

> Trace: D30-D30a, D54, D201
> Covers: Bytes and text do not collapse into each other.

`ByteSpan` is a borrowed view. `ByteBuffer` is owned growable storage with stored allocator identity. Operations that append, encode, decode, compress, hash, or parse bytes must state whether they allocate, whether partial output can be written before failure, and how existing views are invalidated.

> Trace: D44, D74, D77, D85, D201
> Covers: Byte containers inherit allocator and invalidation rules from the memory-container contract.

## C Strings

`CString` is owned NUL-terminated storage suitable for C interop. `CStr` is a borrowed validated C-string view. Constructing `CString` from Kyokai text or bytes checks for interior NUL unless the function name explicitly says it accepts raw bytes and states the resulting contract.

> Trace: D20, D30-D30a, D68, D201
> Covers: C string interop uses dedicated wrappers with validation, ownership, and allocator contracts.

Borrowed `CStr` values are valid only for the lifetime of the underlying storage and must not be used after the foreign call invalidates that storage. Owned `CString` values are linear and destroy through their stored allocator.

> Trace: D68, D77, D245
> Covers: C-string lifetimes and ownership are explicit across FFI boundaries.

## OS Strings And Paths

`OsString` and `OsStr` represent platform-native argument and environment text where the host does not promise UTF-8. `PathBuf` and `Path` represent filesystem paths. A path is not a `String`; APIs that accept filesystem names take path types.

> Trace: D30-D30a, D67, D171
> Covers: Platform-native strings and filesystem paths are separate from ordinary UTF-8 text.

Pure path manipulation can be capability-free when it only joins, normalizes lexical components, checks extension-like syntax, or views components. Anything that observes or changes the filesystem, resolves symlinks, reads metadata, opens a handle, changes permissions, or depends on the process environment requires the relevant capability.

> Trace: D67, D85, D171, D211
> Covers: Path syntax work is pure; filesystem authority requires capabilities.

Safe Kyokai has no ambient current-directory semantics. Relative paths are resolved only against an explicit `Directory` handle or a named capability-gated API that returns such a handle. Top-level `FileCapability` path operations accept absolute paths only.

> Trace: D171
> Covers: Relative filesystem behavior names its base directory explicitly.

| API Family | Ownership | Allocation | Failure | Capabilities | Platform | Edge Cases | Trace |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `OsString`/`OsStr` | Owned or borrowed platform string storage. | Owned conversions take allocator. | Encoding conversion returns `TextError` when UTF-8 cannot be produced. | None for pure conversion. | Native OS encoding/representation is documented per target. | invalid native bytes, empty value, embedded NUL where OS rejects it. | D30-D30a, D67 |
| `PathBuf`/`Path` | Owned or borrowed path storage. | Owned path construction takes allocator. | Invalid path syntax returns `PathError` where target rejects it. | None for lexical operations. | Separator, root, drive, and encoding rules are target-specific but specified. | empty, root-only, relative, absolute, reserved names, invalid separators. | D171, D201 |
| Filesystem path operations | Borrows or consumes path and handle as named. | Output paths/strings take allocator. | `IoError`, `PathError`, or `AllocError`. | `FileCapability` or explicit `Directory` handle. | Target support documented per operation. | symlinks, races, missing path, permission denial, non-directory base. | D67, D171, D211 |

> Trace: D30-D30a, D67, D74, D85, D171, D201, D211
> Covers: OS-string and path families publish ownership, allocation, failure, capability, platform, and edge-case contracts.

## Parsing

`Parsable[T]` parses text into a value and returns `Result[T, ParseError]`. Numeric parsers must state accepted syntax, radix rules, signs, separators, overflow behavior, whitespace behavior, and whether the parser consumes the whole input.

> Trace: D69, D75-D76, D85
> Covers: Parsing is typed, fallible, and syntax-explicit.

Parsing bytes requires a byte parser contract. Parsing text requires valid UTF-8. A parser that accepts both must expose two named entrypoints or a parameter whose type makes the boundary visible.

> Trace: D30-D30a, D69
> Covers: Byte parsing and text parsing do not silently share one boundary.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
The standard library should never make the programmer guess whether a value is language text, protocol bytes, a C argument, or a path crossing into the operating system. Those are different rooms. Kyokai keeps the doors labeled.

> Trace: D30-D30a, D68-D69, D171
> Covers: Separate text, byte, C-string, OS-string, and path contracts prevent hidden boundary behavior.
