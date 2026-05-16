# Testing, Coverage, And Benchmarks

[Rikona Kurasaki / Mjoyufull]
Tests in Kyokai are ordinary code standing under ordinary rules. That matters. A test harness that sneaks in authority, hidden imports, fake ownership, or catchable panics would teach a different language than the one people ship.

> Trace: D28, D137
> Covers: Kyokai tests use ordinary language semantics and explicit authority.

## Test Declarations

Inline tests are declared in module bodies. They are excluded from production artifacts unless a test build explicitly includes them. Tests may access private declarations in the same module body and public/internal declarations according to the same visibility rules as ordinary same-package code.

> Trace: D17, D28, D78
> Covers: Inline tests live in bodies, are test-only artifacts, and use normal visibility.

A pure test has no root capability and receives no ambient authority. If a test needs process, filesystem, environment, network, clock, random, terminal, or other authority-bearing operations, it must declare an explicit capability parameter such as `with (root: RootCapability)` or a narrower capability shape admitted by the stdlib contract.

> Trace: D48, D67, D137, D211
> Covers: Tests do not gain ambient authority; capability-using tests spell it.

Each capability-using test receives a fresh root or declared capability instance for that test execution. Capabilities are not shared between tests unless a future harness chapter defines an explicit fixture with linear ownership and cleanup rules.

> Trace: D137, D211
> Covers: Test authority is isolated per test.

Test success means the test returns normally with all linear obligations discharged. A failed `require`, failed `ensure`, `panic`, TPOE, failed assertion helper, uncaught runtime-fatal process termination, timeout, leaked live linear value at test exit, or harness setup failure makes the test fail according to its category.

> Trace: D2, D53, D84, D137
> Covers: Test completion follows ordinary cleanup, contract, panic, TPOE, and linearity rules.

## Running Tests

`kyokai test` builds selected packages in the selected test profile and runs discovered inline tests. In workspace scope, packages are tested in deterministic dependency order. Independent tests may run in parallel only when the harness can preserve per-test capability isolation, deterministic reporting, and no shared mutable harness state outside explicit fixtures.

> Trace: D28, D83, D137
> Covers: Test execution is deterministic in reporting and explicit about parallel safety.

Required flags include:

| Flag | Meaning | Trace |
| --- | --- | --- |
| `--filter <pattern>` | Run tests whose fully qualified test name matches the pattern. | D28 |
| `--exact <name>` | Run exactly one fully qualified test name. | D28 |
| `--doc` | Include documentation tests extracted by `kyokai doc` rules. | D218 |
| `--no-run` | Build test artifacts without executing them. | D28, D80 |
| `--jobs <n>` | Limit parallel test execution. | D28, D83 |
| `--timeout <duration>` | Apply an explicit per-test timeout where the target runner supports it. | D28, D137 |
| `--format human|json` | Select report format. | D29 |
| `--list` | List discovered tests without building runner executables unless discovery requires checking. | D28, D270 |
| `--failed` | Re-run tests recorded as failed by the previous compatible test report. | D28, D270 |
| `--seed <value>` | Set the deterministic property/fuzz seed for reproducible runs. | D220, D270 |
| `--replay <id-or-file>` | Replay a recorded property failure, fuzz crash, or minimized reproducer. | D220, D270 |
| `--fuzz` | Run fuzz targets instead of ordinary tests, using coverage-guided mutation where supported. | D220, D270 |
| `--corpus <path>` | Use an explicit fuzz corpus directory. | D220, D270 |
| `--minimize <id-or-file>` | Minimize a recorded failing input without changing the language semantics being tested. | D220, D270 |

> Trace: D28, D29, D83, D137, D218, D220, D270
> Covers: Test command selection, doc-test inclusion, concurrency, timeouts, property/fuzz replay, corpus handling, minimization, and reporting are specified.

A timeout is a harness failure, not a language-level catch of `panic` or TPOE. If the target runner kills the test process, the report must classify the failure as timeout or runner termination and must not pretend the program returned a Kyokai value.

> Trace: D84, D137
> Covers: Test timeouts stay outside language failure semantics.

## Property Testing And Fuzzing

Property tests use typed generators, deterministic seeds, shrinking for admitted data types, and replayable failure records. A property failure report must include the fully qualified test name, seed, shrink path or minimized input where available, target, backend, profile, and toolchain version. Re-running with the same seed and compatible source must reproduce the same generated input sequence until a generator contract changes under an explicit compatibility rule.

> Trace: D83, D220, D270
> Covers: Property tests are reproducible daily tooling, not one-off random executions.

Fuzz targets are explicit test declarations or test-adjacent declarations under the toolchain discovery rules. `kyokai test --fuzz` runs those targets with an explicit corpus, crash directory, seed, run budget, and report format. A crash reproducer is an ordinary artifact under the output report tree or selected corpus path; it must be replayable by `kyokai test --replay`.

> Trace: D29, D83, D220, D270
> Covers: Coverage-guided fuzzing has explicit target discovery, corpus state, crash artifacts, and deterministic replay.

Minimization is a harness operation over a recorded failing input. It must preserve the failure category being minimized, such as compile error, test assertion, panic, TPOE, runtime-fatal, or sanitizer/backend failure where that target runner reports one. If minimization changes the failure category, the report must say the minimization is not valid for the original failure.

> Trace: D84, D137, D220, D270
> Covers: Fuzz and property minimization do not blur distinct Kyokai failure classes.

## Documentation Tests

Documentation tests are extracted from documentation comments only when the fenced block or directive marks the code as Kyokai test code. The extracted code is compiled with the package edition and the same public interface visibility available to documentation examples, unless the doc comment explicitly declares a same-package test context.

> Trace: D105, D218
> Covers: Doc tests are explicit, edition-aware, and visibility-aware.

`kyokai test --doc` uses the same compiler engine as ordinary tests. A doc test cannot bypass imports, capability declarations, contracts, or linearity. Examples that are meant to fail must be marked as compile-fail or run-fail in the doc-test directive and checked against the expected failure category.

> Trace: D28, D29, D137, D218
> Covers: Doc tests are real compiler tests, including negative examples.

## Coverage

Coverage reports are produced only when requested. Coverage instrumentation must not change language semantics, evaluation order, borrow/linearity behavior, panic/TPOE behavior, volatile semantics, atomic semantics, or capability flow. If a target/backend cannot provide conforming coverage, the command must fail or report unsupported coverage for that target.

> Trace: D28, D73, D83, D137, D141
> Covers: Coverage instrumentation is explicit and semantics-preserving.

Coverage output records package, module, declaration, branch, expression/statement region where available, test profile, target, backend, compiler version, and source revision identity. Human reports may aggregate; JSON reports must keep stable identifiers for tools.

> Trace: D29, D83, D225
> Covers: Coverage reports are useful for humans and stable for CI tooling.

## Benchmarks

Benchmarks are ordinary test-like declarations or explicitly marked tests run by `kyokai bench`. They must not use hidden optimizer barriers that change language semantics. The harness may use timing loops, warmup, repetition, and statistical reporting, but these are harness behavior, not Kyokai language constructs.

> Trace: D28, D83
> Covers: Benchmarks are explicit harness operations over ordinary Kyokai code.

A benchmark that needs authority must declare capabilities just like a test. The harness must report target, backend, profile, runner, CPU/OS facts when available, iteration policy, and whether measurements are comparable across runs. Bench numbers are presentation/report artifacts and are not reproducible build artifacts.

> Trace: D80, D83, D137
> Covers: Benchmarks report their environment and do not claim deterministic artifact identity.

## Why This Shape

[Rikona Kurasaki / Mjoyufull]
Tests should be close enough to the code to catch the truth, but not so privileged that they become a different court. Kyokai lets them see private rooms when they live in the same module, but it does not hand them the master key to the house unless the test asks for it in the open.

> Trace: D28, D137, D211
> Covers: The test model balances practical module testing with explicit capability security.
