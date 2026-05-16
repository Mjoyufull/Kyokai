# Linear Resources Rationale

[Rikona Kurasaki / Mjoyufull]
The inherited resource argument is still the center of Kyokai. A file handle, pointer, lock guard, channel endpoint, process handle, directory handle, pinned object, generator, or capability has a life. It is created, used, transferred, split if the API permits that, and finally consumed. The bug appears when that life is only a story in the programmer's head.

> Trace: D2, D6, D77, D89, D194
> Covers: Linear values encode resource lifecycles in the type system and checker rather than in comments or habit.

## Why Linear, Not Just Analysis

[Rikona Kurasaki / Mjoyufull]
Static analysis can find bugs, but a moving pile of heuristics is not the same thing as a type rule a programmer can learn. Kyokai follows Austral in wanting a fixed, teachable rule: a linear owner is used exactly once, and a type that contains linear state becomes linear unless a closed built-in rule says otherwise.

> Trace: D6, D77, D194
> Covers: Universe classification and exactly-once use are static type/checker rules.

That does not make every resource easy. It makes the hard parts visible. When a value moves, the old place is unusable. When a container stores linear elements, the container must explain how those elements are consumed. When a loop might consume a value zero or many times, the checker refuses unless the loop form proves the path.

> Trace: D77, D89/D199, D205-D206
> Covers: Moves, containers, patterns, and loops preserve exactly-once ownership.

## Borrows Are Not Owners

[Rikona Kurasaki / Mjoyufull]
Borrowing is how Kyokai lets code look like code instead of a ceremony of passing owners in circles. A borrow gives temporary access and then ends. It does not become a second owner, and it does not let mutation happen through the wall while some other view still lives.

> Trace: D6, D14, D187/D238-D240
> Covers: Borrow types are non-owning references with explicit mutable and immutable forms.

Named regions exist only when an API needs to relate borrowed lifetimes, such as returning a view tied to an input. Anonymous regions cover local borrows. This keeps borrow syntax visible without making every simple function carry a lifetime ledger in public.

> Trace: D6, D14
> Covers: Anonymous regions are complete source types and named regions express API relationships.

## Cleanup Is Source

[Rikona Kurasaki / Mjoyufull]
Kyokai does not smuggle cleanup through a destructor. If cleanup matters, the source says it. `defer` and `errdefer` are not hidden destructors; they are visible statements with checker state and exact exit-path behavior. A resource can also be consumed by an explicit close, destroy, release, drain, or surrender API.

> Trace: D2, D2a/D2b/D207, D246
> Covers: Cleanup uses visible constructs and explicit consuming APIs, not compiler-inserted ordinary-scope destructors.

This is less convenient in the small and cleaner in the large. A reader can see when a resource leaves the room. A compiler can prove it. A standard-library contract can say what happens if cleanup fails, if early exit happens, or if a linear buffer still contains values.

> Trace: D77, D85, D146, D229
> Covers: Linear cleanup obligations extend through stdlib contracts, channels, iterators, and containers.

## Stable Address And Pinning

[Rikona Kurasaki / Mjoyufull]
Moves are as-if bytewise relocation. That is simple until a value points into itself or external code remembers its address. Kyokai names the stable-address boundary instead of pretending ordinary ownership also means ordinary address permanence. `PinBox[T]` and pinned types carry that promise.

> Trace: D89/D199, D89a/D89b
> Covers: Safe self-reference requires explicit stable-address/pinning rules.

## Result

[Rikona Kurasaki / Mjoyufull]
Linear resources are Kyokai's answer to lifecycle bugs: not a linter pass, not a destructor convention, not a culture of careful people, but a rule the compiler enforces and the spec explains. The program says who owns the thing, who borrows it, who moves it, and who consumes it.

> Trace: D6, D77, D89, D194
> Covers: Linear ownership and borrowing make resource lifecycle boundaries explicit and statically checked.
