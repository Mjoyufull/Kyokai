# Unsafe, FFI, And ABI

[Rikona Kurasaki / Mjoyufull]
Foreign code is a gate in the wall. Austral already treated that gate as unsafe. Kyokai keeps that old warning sign, then bolts the whole frame down: the module must say it is unsafe, the call site must carry unsafe authority, the source must record the contract, and the wrapper must translate the foreign world back into Kyokai values.

Unsafe code is not a room where the language stops speaking. It is a narrow bridge with posted weight limits. If the bridge cannot say what it carries, who owns it, what can fail, and what happens when the other side lies, it is not a Kyokai bridge yet.

> Trace: D20/D20a/D20b, D73, D242/D242a, D245
> Covers: Raw FFI and unsafe operations are specified boundaries with explicit authority, ABI, ownership, failure, and audit contracts.

This chapter defines unsafe modules, unsafe contracts, unsafe capability flow, raw C foreign blocks, imports, exports, callbacks, raw pointers, volatile access, inline assembly, dynamic loading, plugins, foreign ownership transfer, sum-type boundaries, and the C ABI surface.

> Trace: D22, D31, D42, D80, D94/D257, D113a/D113b, D139, D228, D255
> Covers: Unsafe facilities are individually admitted and tied to target/backend contracts rather than hidden behind backend folklore.

## Unsafe Boundary

`pragma Unsafe_Module;` marks a module body that contains raw unsafe operations, raw foreign declarations, inline assembly, volatile memory operations, raw dynamic loading, raw signal-handler registration, raw pointer/address conversions, trusted capability acquisition, or another unsafe primitive admitted by this specification.

> Trace: D20, D73, D245
> Covers: Unsafe is module-scoped as an implementation boundary, not an expression-level escape from the language.

The pragma does not grant visibility, does not grant host authority, does not allow capability forgery, does not bypass imports, and does not make private declarations public. It only permits the module body to contain unsafe operations that safe modules cannot contain.

> Trace: D17, D20, D245, D255
> Covers: Unsafe module status changes which raw operations may appear, not the module system or authority model.

A safe module may import public or same-package `internal` safe wrappers from an unsafe module. It may not call raw foreign declarations or unsafe primitives merely because it can name the module.

> Trace: D17, D20, D245
> Covers: Safe APIs exported from unsafe modules are ordinary declarations; raw unsafe machinery remains gated.

Kyokai has no open-ended language-level undefined behavior category. Unsafe means the programmer is using a specified operation with specified preconditions, effects, and failure behavior. Misuse must be classified as compile error, TPOE, panic, runtime-fatal termination, ordinary foreign-contract violation, or another explicitly named outcome.

> Trace: D73, D84, D228
> Covers: Unsafe code is still specified Kyokai behavior; it is not C-style UB imported into the language.

If a proposed unsafe operation cannot state its alignment, provenance, lifetime, ownership, initialization, aliasing, failure, and backend-lowering rules, it is not part of Kyokai.

> Trace: D73, D228
> Covers: Vague unsafe primitives are rejected until their contract is precise.

## Unsafe Contracts

Every unsafe module must contain at least one source-level `unsafe contract ... audit;` block. The contract block is part of the module body source. It is not a comment convention, not an external `SAFETY.md`, and not optional review prose.

> Trace: D245
> Covers: Unsafe reasoning is machine-discoverable source metadata.

An unsafe contract must enumerate each unsafe facility used by the module: raw foreign declarations, unsafe intrinsics, inline assembly, volatile operations, raw dynamic loading, raw signal handlers, raw pointer/address conversions, trusted capability acquisition, and any future unsafe primitive.

> Trace: D20, D22, D94/D257, D113a, D245
> Covers: Unsafe operations are covered individually, not hidden under a single vague module note.

The compiler rejects an unsafe operation that is not covered by an unsafe contract entry. The compiler also rejects a contract entry that names no unsafe operation in the module unless that entry is explicitly marked as a module-wide invariant.

> Trace: D245
> Covers: Contract coverage is checked in both directions.

An unsafe contract states at least:

| Field | Required meaning |
| --- | --- |
| `covers` | The unsafe declaration, primitive, raw call, assembly block, or dynamic-loading operation covered. |
| `requires` | Preconditions the module must establish before the unsafe operation. |
| `ensures` | Invariants restored before safe callers can observe results. |
| `authority` | Capability values or trusted authority used by the operation. |
| `ownership` | Whether Kyokai retains, transfers, borrows, receives, or destroys ownership. |
| `failure` | How foreign failure conventions become `Result`, `Optional`, TPOE, `panic`, or runtime-fatal behavior. |
| `backend` | Backend, target, ABI, alignment, calling-convention, or toolchain facts the operation depends on. |
| `exports` | Safe wrappers whose soundness relies on the contract. |

> Trace: D20/D20b, D73, D85, D211, D245
> Covers: Unsafe contracts state authority, ownership, failure, and backend assumptions explicitly.

An unsafe contract is not a formal proof and does not weaken any unsafe primitive's own semantics. The primitive remains governed by its individual rule. The contract records how this module satisfies that rule and what safe wrapper behavior depends on it.

> Trace: D73, D245
> Covers: Audit metadata explains unsafe use; it does not relax the language contract.

## Unsafe Capability

Kyokai provides `UnsafeCapability` as a sealed linear authority token for raw unsafe calls. Safe code cannot construct it, clone it, or obtain it by importing an unsafe module. The token enters only through runtime/toolchain-authorized unsafe entrypoints or trusted standard-library authority paths that this specification admits.

> Trace: D20, D211, D245, D255
> Covers: Raw unsafe authority is explicit and unforgeable.

Every raw foreign declaration is called in Kyokai source as if it has an additional leading parameter of type `&![UnsafeCapability]`. This parameter is part of the Kyokai source contract and audit trail. It is erased before the actual foreign ABI call.

> Trace: D20
> Covers: Raw calls require source-level unsafe authority without changing the C ABI.

Unsafe primitives other than FFI may also require `&![UnsafeCapability]` when the primitive touches external authority, raw memory, machine state, device state, or backend-specific behavior. A primitive that does not require it must still be individually specified.

> Trace: D20, D73, D94/D257, D211
> Covers: Unsafe authority is attached where the operation crosses a trust boundary.

`UnsafeCapability` is not `RootCapability`. It is not broad host authority. It authorizes use of unsafe machinery under contract; the actual filesystem, network, clock, dynamic-loader, device, signal, or process authority still comes from the relevant capability values.

> Trace: D20, D67, D211, D255
> Covers: Unsafe permission and external host permission are separate authority facts.

## Foreign Blocks

A raw C boundary is written as a foreign block:

```kyokai
foreign "C" is
    function c_open(path: Address[Nat8], flags: Int32): Int32;
    function c_close(fd: Int32): Int32;
mon;
```

> Trace: D20, D111/D127
> Covers: Raw C declarations live in a visible `foreign "C" is ... mon;` boundary.

Only `foreign "C"` is admitted by the current grammar. A future ABI string requires a separate decision covering its calling convention, type mapping, failure conventions, callback behavior, and backend support.

> Trace: D20a, D80
> Covers: The raw foreign ABI set is closed until specified.

A `foreign "C"` block is legal only in a module body marked `pragma Unsafe_Module;`. Raw foreign declarations are body declarations. They are private implementation machinery unless ordinary interface declarations expose safe wrappers.

> Trace: D17, D20, D245
> Covers: Foreign declarations do not become public API by existing.

Foreign declarations may declare functions, foreign constants with ABI-fixed scalar representation, and other raw ABI items explicitly admitted by this chapter. Imported functions may not be generic. Exported functions may not be generic.

> Trace: D20, D82
> Covers: Raw ABI declarations are monomorphic and concrete.

A raw foreign declaration is not a safe Kyokai contract. It says how to call a foreign symbol. It does not say the symbol is memory-safe, retry-safe, thread-safe, capability-safe, allocation-free, or error-checked. Safe wrappers must state those facts.

> Trace: D20b, D85, D245
> Covers: ABI declaration and safe API contract are different layers.

## C ABI Type Surface

`foreign "C"` uses the selected target's C ABI exactly. The selected target triple, ABI enum, backend, C compiler family, version floor, flags, and admitted extension families belong to the target/toolchain contract.

> Trace: D20a, D31, D80, D139
> Covers: The C boundary is target-specific but not guessed.

The raw C ABI surface is limited to:

| Kyokai form | Raw C ABI status |
| --- | --- |
| Fixed-width integer types | Admitted with target contract for exact C representation typedefs. |
| `Bool` | Admitted through Kyokai's specified C boolean representation. |
| `Index`, `ByteSize` | Admitted through the selected target's specified size representation. |
| `Float32`, `Float64` | Admitted when the target contract provides the required IEEE 754 representation. |
| `Address[T]` | Admitted as nullable raw address/pointer form. |
| `Pointer[T]` | Admitted as non-null raw pointer form under wrapper-enforced validity. |
| `FnPtr(...)` | Admitted as a bare C function pointer shape. |
| `extern record` | Admitted by value only when every field is FFI-admitted. |
| Pointer/address to `extern type` | Admitted for opaque foreign handles. |

> Trace: D20a, D42, D75, D80
> Covers: Raw C declarations use a closed admitted type surface.

`extern type Name;` declares an opaque foreign type with unknown Kyokai size and layout. It may appear only behind FFI-admitted pointer/address forms or other specifically admitted ABI wrappers. It may not be passed by value, stored by value as an ordinary Kyokai object, destructured, measured with ordinary layout introspection, or pattern matched.

> Trace: D20a, D42
> Covers: Opaque C types stay opaque and pointer-shaped.

A record may cross raw C FFI by value only if it is declared `extern record` and every field is FFI-admitted. Ordinary `record` values use Kyokai layout and do not silently become C structs. `packed record` is byte-tight Kyokai layout and does not imply C ABI compatibility.

> Trace: D20a, D42
> Covers: By-value aggregate FFI requires explicit `extern record` layout.

Arrays do not decay implicitly in foreign declarations. If the C ABI wants a pointer, the declaration must spell `Address[T]`, `Pointer[T]`, or another explicitly specified pointer-like form.

> Trace: D20a, D55
> Covers: C array decay is not imported as Kyokai magic.

`Callable`, `CallableMut`, and `CallableOnce` do not cross raw FFI directly. Only `FnPtr(...)` may cross the raw foreign boundary as a typed bare callback value.

> Trace: D20a, D21, D118/D126/D197
> Covers: Capturing Kyokai closures are not C function pointers.

Passing or returning Kyokai unions, `Optional`, `Result`, or any other sum type by value across raw FFI is illegal. C tagged-union APIs must be modeled with explicit ABI-shaped declarations: integer tags, `extern record` containers, and raw payload storage where the C ABI actually specifies it. Safe wrappers validate tags and payload contracts before constructing Kyokai unions.

> Trace: D20a, D65, D242a
> Covers: Kyokai sum types have no implicit C ABI.

Raw foreign declarations may not take or return Kyokai `Linear` values by value. Foreign code is never assumed to understand Kyokai linearity, exactly-once consumption, borrow exclusivity, or cleanup obligations.

> Trace: D20, D195, D242
> Covers: Raw C cannot pretend to consume or produce Kyokai-owned linear values.

A safe wrapper may consume a linear Kyokai value before a foreign call only by explicitly decomposing it into FFI-legal raw parts inside an unsafe module. The unsafe contract must say whether ownership is retained by Kyokai, transferred to foreign code, borrowed only for call duration, or returned through a named handle/result.

> Trace: D20, D89, D242, D245
> Covers: Ownership transfer across FFI is wrapper-modeled and contract-covered.

## Raw Addresses And Pointers

`Address[T]` is a nullable raw address. `Pointer[T]` is a non-null raw pointer. Both are `Free`; neither owns storage, extends lifetime, grants access rights, proves alignment, proves initialization, or proves aliasing safety.

> Trace: D6, D20a, D73
> Covers: Raw pointer-like values are non-owning machine values, not borrows or owners.

Safe Kyokai may pass raw address values around only where an API explicitly exposes them. Dereference, conversion to a borrow, conversion to an owning value, pointer arithmetic, unaligned access, byte reinterpretation, or lifetime extension is unsafe and must be admitted as an individually specified primitive or safe wrapper.

> Trace: D73, D77, D245
> Covers: Validity-sensitive raw memory use is confined to unsafe contracts.

Kyokai has no general `transmute`, no general type-punning primitive, and no arbitrary pointer-to-integer or integer-to-pointer roundtrip in the safe language. Any future representation reinterpretation primitive must state exact provenance, alignment, lifetime, initialization, and backend rules before it ships.

> Trace: D73, D228
> Covers: The unsafe memory model stays closed and specified.

A raw pointer retained by foreign code after a call is a contract fact. The wrapper must represent it as a linear handle, borrowed relationship, callback registration, or another explicit state object. It may not leave the retention rule as prose outside the source contract.

> Trace: D20b, D85, D242, D245
> Covers: Foreign retention of pointers is represented in Kyokai state.

## Foreign Failure

`errno`, null returns, negative returns, sentinel values, status integers, out-parameters, host exceptions, callback error protocols, and other foreign failure conventions are foreign runtime state. They are not Kyokai semantics until an unsafe wrapper translates them.

> Trace: D20b, D24, D84
> Covers: Raw foreign errors become Kyokai values or fatal paths only through wrappers.

A safe wrapper must translate foreign failure into `Result`, `Optional`, TPOE, `panic`, runtime-fatal termination, or another specified Kyokai outcome. It must also state whether foreign calls can allocate, block, mutate global state, use thread-local state, read environment state, depend on signal behavior, or require retry on interruption.

> Trace: D20b, D85, D91, D211
> Covers: Safe wrappers surface failure, blocking, allocation, and hidden foreign state.

A raw foreign call carries no implied guarantee about retry safety, `EINTR`, allocation behavior, thread safety, reentrancy, cancellation, callback lifetime, or resource ownership. The wrapper contract must say what is known.

> Trace: D20b, D85, D91, D245
> Covers: Foreign conventions are not inherited silently.

Foreign code may not unwind, `longjmp`, throw an exception, or otherwise skip Kyokai frames. If that happens, the foreign boundary contract has been violated. Kyokai does not define ordinary recovery from that violation.

> Trace: D20b, D84, D245
> Covers: Foreign frames may not bypass Kyokai cleanup and ownership state.

Kyokai `panic`, TPOE, runtime-fatal termination, and internal compiler/runtime fatal paths must not unwind through foreign frames. If such a path occurs while foreign code is on the stack, the runtime terminates through the specified fatal path rather than crossing the foreign boundary with stack unwinding.

> Trace: D20b, D84, D253
> Covers: Fatal Kyokai termination does not become foreign unwinding.

## Callbacks And Exports

A raw callback accepted by foreign code must use `FnPtr(...)`. Capturing closures and callable-family values do not cross as callbacks directly.

> Trace: D20a, D21, D118/D126/D197
> Covers: Foreign callbacks are bare function-pointer ABI values.

Any foreign API that can call back into Kyokai must specify callback ABI, callback lifetime, thread of execution, reentrancy, authority passed to the callback, termination behavior, and whether the callback may run concurrently with the registering Kyokai task.

> Trace: D20b, D90/D90a, D164, D245
> Covers: Callback contracts state lifetime, thread, authority, and failure behavior.

A callback must not receive a borrowed Kyokai reference unless the wrapper can prove the foreign API calls it only within the borrow's live region and never stores the pointer. If the foreign API can retain a callback or user-data pointer, the wrapper must represent that retention as a linear registration handle with explicit unregister/destroy behavior.

> Trace: D14, D20b, D195, D242
> Covers: Borrow lifetime is not trusted to foreign callback folklore.

Exporting Kyokai functions through a C ABI wrapper is legal only for signatures whose parameters and return type are in the exported C ABI surface. Exported functions may not be generic, may not expose Kyokai sum types by value, may not expose `Linear` values by value, and may not expose borrow references directly unless a future rule admits a specific wrapper pattern.

> Trace: D20a, D242, D242a
> Covers: Exported functions use the same raw ABI honesty as imports.

If an exported Kyokai function panics, triggers TPOE, or hits runtime-fatal termination, the generated wrapper must terminate according to Kyokai fatal-path rules. It must not unwind into C as an ordinary C return or exception.

> Trace: D20b, D84, D253
> Covers: Export wrappers preserve Kyokai fatal semantics.

## Dynamic Loading

Raw dynamic library loading is unsafe. It requires dynamic-loader authority, an unsafe module, an unsafe contract, and linear handles for opened libraries and resolved resources.

> Trace: D113a, D211, D245
> Covers: `dlopen`-style behavior is explicit authority-bearing unsafe machinery.

A raw loaded-library handle is linear. It must be closed through an explicit operation or moved into another owner that has a visible destruction obligation. Safe Kyokai does not silently leak or auto-drop dynamic loader handles.

> Trace: D2, D89, D113a, D195
> Covers: Dynamic library handles obey ordinary linear ownership.

Resolving a symbol from a raw dynamic library returns raw address state. Casting that state to a typed `FnPtr` or pointer form is unsafe and must be covered by a contract naming the expected symbol type, ABI, lifetime, and target/platform assumptions.

> Trace: D20a, D73, D113a, D245
> Covers: Dynamic symbol typing is an unsafe promise, not loader proof.

Safe runtime extensibility uses verified Kyokai plugin contracts, not arbitrary raw dynamic libraries. A safe plugin loader must check exact compiler/toolchain version compatibility, language edition, ABI compatibility, required capabilities, exported initialization/shutdown surface, and unsafe/dynamic-loading audit metadata before admitting the plugin.

> Trace: D113b, D150, D211, D245
> Covers: Safe plugins are verified Kyokai units, not unstructured shared objects.

A loaded plugin is represented by a linear handle. Initialization and shutdown are explicit. The host passes only the capabilities the plugin receives; dynamic loading does not grant ambient root authority.

> Trace: D113b, D211, D255
> Covers: Plugin authority and lifetime are explicit values.

## Volatile Access

Volatile memory access exists as unsafe operation-level primitives, not as a type qualifier:

```kyokai
readVolatile(addr: Address[T]): T
writeVolatile(addr: Address[T], value: T): Unit
```

> Trace: D94/D257
> Covers: Volatile is named on the operation, not infected into reference types.

A volatile access is externally observable. The compiler may not elide it, merge it with another volatile access, or move another volatile access across it.

> Trace: D94/D257
> Covers: Volatile operations preserve externally visible access order among themselves.

Volatile is not synchronization. It creates no happens-before edge and does not replace atomics, fences, mutexes, rwlocks, channels, task joins, or other concurrency primitives.

> Trace: D90/D90a, D94/D257, D247
> Covers: MMIO visibility is not thread synchronization.

The volatile-legal type domain is closed: fixed-width integer types, `Bool`, raw address/pointer-like machine values explicitly admitted by the memory model, and `extern record` or `packed record` aggregates whose fields are recursively volatile-legal. Ordinary records, unions, borrows, capabilities, and linear resources are not volatile-legal unless a later rule admits them explicitly.

> Trace: D42, D73, D94/D257
> Covers: Volatile does not become a back door for arbitrary value representation.

Volatile operations require naturally aligned addresses for `T` unless a separately specified unaligned volatile primitive is used. If the address is invalid, misaligned, unmapped, or faults at hardware/runtime level, the result is runtime-fatal termination, not language-level UB.

> Trace: D73, D84, D94/D257
> Covers: Invalid volatile access has a specified fatal category.

## Inline Assembly

`asm(...)` is an unsafe backend-independent inline assembly form. It is legal only inside `pragma Unsafe_Module;` and must be covered by an unsafe contract.

> Trace: D22, D245
> Covers: Inline assembly is unsafe, audited source, not a backend-specific escape hatch.

The first argument is a compile-time guard, normally over `target.arch`, `target.os`, `target.abi`, or `target.endianness`. If the guard is false, the block is not compiled for that target.

> Trace: D18/D18a, D19/D19a, D22, D80
> Covers: ISA-specific assembly is target-gated at compile time.

Operands use named forms such as `in("reg") expr`, `out("reg") binding`, `inout("reg") binding`, and `clobber("reg", ...)`. The compiler trusts the operand and clobber declarations; the unsafe contract must state the machine-state effects the safe wrapper relies on.

> Trace: D22, D245
> Covers: Assembly effects are declared by operands, clobbers, and unsafe contract.

The C backend lowers inline assembly through the selected supported C compiler's admitted inline-assembly facility. The LLVM backend lowers it through LLVM inline assembly. If the selected backend/target/toolchain cannot implement the block with the declared semantics, the build fails.

> Trace: D22, D31, D80, D139, D228
> Covers: Assembly lowering is explicit and cannot silently degrade.

## Signals And Raw Runtime Hooks

Safe signal handling is defined by the concurrency and capability chapters through `SignalWatcher`. Raw signal-handler registration is unsafe-only and must be covered by an unsafe contract.

> Trace: D95/D256, D245
> Covers: Arbitrary signal handlers are not safe Kyokai.

Raw signal handlers must obey the selected platform's async-signal-safety rules. They may not touch ordinary Kyokai linear state, allocate, call arbitrary Kyokai code, acquire capabilities, run `defer`, or interact with the borrow checker as if they were ordinary control flow.

> Trace: D73, D95/D256, D245
> Covers: Raw signal handlers stay outside the safe execution model.

Synchronous fault signals such as segmentation fault, bus error, illegal instruction, floating-point hardware exception, and abort are runtime-fatal conditions, not recoverable safe signal events.

> Trace: D84, D95/D256, D253
> Covers: Fault signals are fatal paths, not safe callbacks.

## Transitional FFI

Transitional FFI is allowed for bootstrap, hard OS or hardware boundaries, mature external dependencies needed before Kyokai can self-host, and temporary bridges recorded in the implementation plan. It is not a general excuse to wrap C for pure computation.

> Trace: D64, D230
> Covers: RIIK stays the destination for pure computation while bootstrap remains practical.

Every transitional FFI wrapper must live in an unsafe module, have unsafe contracts, expose a safe Kyokai API when used by safe code, and document whether it is permanent boundary code or replacement-target bootstrap code.

> Trace: D20, D64, D230, D245
> Covers: Transitional wrappers are tracked and audited.

Transitional FFI does not relax the raw ABI surface, ownership-transfer restrictions, sum-type restrictions, no-language-UB model, capability model, or unsafe-contract requirement.

> Trace: D20/D20a/D20b, D64, D73, D211, D242/D242a, D245
> Covers: Bootstrap pragmatism does not weaken the unsafe boundary.

## FFI Examples

A raw C declaration stays private behind unsafe authority:

```kyokai
pragma Unsafe_Module;

module body Kyokai.Posix.RawFile is
    foreign "C" is
        function c_open(path: Address[Nat8], flags: Int32): Int32;
        function c_close(fd: Int32): Int32;
    mon;

    unsafe contract PosixFileRaw is
        covers foreign c_open;
        covers foreign c_close;
        authority FileSystemCapability;
        maps errno_to IoError;
        forbids foreign_unwind;
        ownership retained_by Kyokai;
    audit;
seal;
```

> Trace: D20, D20b, D245
> Covers: Raw symbols, authority, failure mapping, and unwind prohibition are source-visible.

A safe wrapper exposes ordinary Kyokai behavior:

```kyokai
function openFile(fs: &![FileSystemCapability], path: &[Path]): Result[File, IoError]
    require path.isNormalized();
is
    // body validates path encoding, calls raw C with UnsafeCapability,
    // checks errno/status, and returns a linear File handle on success.
qed;
```

> Trace: D20b, D53, D85, D211, D242
> Covers: Safe APIs expose capability, ownership, contracts, and recoverable failure as Kyokai facts.
