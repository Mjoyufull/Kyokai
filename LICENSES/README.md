# License Boundary

This directory stores the full license texts used by Kyokai.

## License Texts

- `GPL-3.0-or-later.txt`: GNU General Public License version 3 text.
- `GCC-exception-3.1.txt`: GCC Runtime Library Exception 3.1 text.

## Default Path Classes

Kyokai-owned files should use these default SPDX identifiers unless a file carries a more specific compatible notice.

| Path class | SPDX identifier | Notes |
| --- | --- | --- |
| `bin/`, `lib/`, compiler drivers, package/build tooling, formatter, LSP, docs tooling | `GPL-3.0-or-later` | Toolchain-only source. |
| `standard/`, future `runtime/`, startup code, compiler support library, target-side TPOE/panic helpers, allocation/runtime shims | `GPL-3.0-or-later WITH GCC-exception-3.1` | Code that can be linked, copied, embedded, or combined into user target programs. |
| Compiler-emitted target helper code | `GPL-3.0-or-later WITH GCC-exception-3.1` | Generated helper code that becomes part of target programs. |
| `test/`, `test-programs/`, examples | Same as nearest owner unless a file states otherwise. | Tests and examples should not imply a different product license. |

Inherited Austral files may still carry `Apache-2.0 WITH LLVM-exception` or other compatible notices. We Preserve those notices until the file is replaced or relicensed by the relevant copyright holder.

Before a release, release tooling should generate a manifest from these path classes plus per-file SPDX notices so compiler/toolchain artifacts and target-linked runtime/stdlib artifacts are not blurred.
