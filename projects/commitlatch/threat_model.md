# CommitLatch security audit scope

## Project and intended guarantees

CommitLatch is an experimental Apache-2.0 Python SDK and CLI with a local SQLite
intent journal. It records a business operation before dispatch, fences local
state transitions, and reconciles an ambiguous outcome using a trusted adapter.
Its main security-relevant property is conservative redispatch control: an
unknown outcome must not become permission to execute the operation again.
It is a new alpha project; no independent adoption or production-readiness claim
is made. Inclusion in OSS Scanner is requested for case-by-case review.

## Trust boundaries and untrusted inputs

Inspect strict-JSON arguments and results, business keys, namespaces, adapter
identities, provider receipts, exception handling, CLI input and generated HTML.
Application-supplied data and provider responses can contain adversarial strings.
Adapters are trusted executable Python code; the engine cannot independently
prove receipt authenticity or a claim that a request cannot apply later.

All cooperating callers for one operation must share its database and business
key. A namespace is not an authentication or tenant isolation boundary. Local
database files contain plaintext arguments, results and evidence. Database/host
administrators are trusted. Multi-host/NFS operation, malicious Python adapters,
direct writes outside the SDK, new business keys, deleted journals and stale
backup restoration are outside the promised duplicate-prevention boundary.

## Priority components

- `src/commitlatch/engine.py`: uncertain results, reconciliation, explicit retry,
  immutable payload/adapter binding and fail-closed error handling.
- `src/commitlatch/ledger.py`: transactional claims, expired/late workers,
  namespace/key scoping, SQL queries and read-only event playback.
- `src/commitlatch/model.py`: strict JSON and fingerprinting.
- `src/commitlatch/report.py` and CLI: untrusted content escaping and filesystem
  handling within documented caller privileges.

`demo.py` and `http_demo.py` are synthetic testing fixtures, not production
provider services. Their deliberate lost responses, non-idempotent writes and
opaque receipt modes are not vulnerabilities by themselves. Look for violations
of the SDK contract rather than assuming the fixture is a hardened web server.

## Reproduction environment

The image contains the installed package and optional pinned LangGraph example.
All commands run offline using local files/processes and loopback HTTP only:

```sh
cd /src
python -m unittest discover -s tests -v
python examples/quickstart.py
python examples/langgraph_recovery.py
python examples/langgraph_recovery.py --opaque
python -m commitlatch http-demo
```

The graph example tests the external action boundary, not graph checkpoint
recovery. Use temporary directories and synthetic provider data. Do not contact
real provider accounts or require API keys. Dependencies are not this project's
code; identify an integration flaw in CommitLatch when reporting dependency use.

## Findings and severity

For each report, provide affected source lines, the violated documented invariant,
an offline minimal reproducer, expected/observed effects, and a regression test
with a minimal patch where feasible. Demonstrated unauthorized code execution or
credential disclosure should state the actual trust boundary crossed. A duplicate
external mutation caused by bypassing the documented claim/reconciliation rules
can be high severity depending on prerequisites and impact. Do not label every
liveness failure critical. Report local privilege/resource-exhaustion assumptions
explicitly. Known conservative UNKNOWN outcomes are expected behavior.
