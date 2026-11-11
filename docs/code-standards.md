# Code standards

What a change, a test and a commit must show. A reviewer checks each rule
that applies and names the one a change misses.

## What a change contains

| Rule | What a reviewer looks at |
| --- | --- |
| A change does what its goal needs and nothing else. | Every change in behavior has a reason. Refactors, extra features and policy changes go in a change of their own. |
| A change holds on the whole path. | Follow it from the input (a source event, a button, a page) to the end (a notification, a record, a signature), through the callers and the state it changes. One correct function doesn't make the operation correct. |
| A fix reaches every sibling of the bug. | Search for the same pattern: every chain adapter, every entry to the signer, every source of the same kind gets the fix in the same change. |
| A change re-derives what is built on it. | When a formula, a list, a unit or the timing of a step changes, find every caller, stored copy and test that assumed the old meaning. |
| A failed read, parse or check means "unknown". | Never "empty", "none", "zero" or "allowed". A limits file, an allowlist or a wallet store that can't be read stops signing; it is never read as no limits. Keep the last good read, and copy a file that doesn't parse aside before writing over it. |
| Files are written whole. | Write to a temporary file in the same folder, sync it, rename it over the old one. A reader may meet a file another process is writing. |
| Another system's values come from that system. | A chain's units, a router's calldata, an API's fields and error codes are taken from its source, docs or recorded replies, never guessed from a name. |
| The user's value wins. | A default never overrides a value the user set. A migration that would change one says so and leaves it. |
| Nothing that can block runs under a lock its reader needs. | No channel send, network call or disk wait while holding a lock another goroutine takes first. A change that starts a goroutine or timer runs `-race`. |

## Money and keys

| Rule | What a reviewer looks at |
| --- | --- |
| The signer's policy is the only gate. | Every route to a signature (admin page, Telegram, rules, CLI) reaches the same check. A limit added in one caller and not in the signer is not a limit. A test for a limit goes through each route. |
| Amounts are integers in the chain's smallest unit. | No float on a money path. Conversions to and from display units happen at the edge, with the token's decimals read from the chain. |
| Secrets never leave the signer process. | Not in logs, errors, panics, test output, notifications, prompts or backups. Error values from the signer carry a reason, never the input. A test feeds a real-shaped throwaway key through each exit and checks it is masked. |
| A model's output is data, never a command. | A score or a reason may route a notification. It never sets an amount, an address, a limit or a wallet, and never triggers a signature by itself. |
| A failure is classified before anything is rested. | An RPC that is down, a route that refuses, and a token that can't be sold are different failures. Only the first two rest a provider. |
| A retry never sends twice. | A trade that may have reached the chain is looked up before it is sent again. A button pressed twice trades once. |
| A security exemption is scoped by where a value sits, never by a field's name. | An exemption (a value not masked, a request not checked) is tested with the protected value placed where untrusted data goes. |
| A known gap is an open issue. | A sentence in a doc that names a weakness comes with an issue in the same commit. |

## Tests

| Rule | What a reviewer looks at |
| --- | --- |
| A fix comes with a test that fails without it. | Put the old code back, run the test, see it fail with a real failure that names the behavior. A compile error or a skip doesn't count. |
| A test never reaches the network. | Chains run locally (anvil, solana-test-validator) or are served from recorded replies through httptest. An API's reply is built from bytes saved from the real one. |
| A test never touches real funds or real config. | Keys are made in the test. The data folder is a temporary one. Nothing reads or writes the user's Crossbill folder. |
| Tests use real inputs. | Fixtures look like real events, real chain replies and real files, with their missing fields and odd values. |
| Tests check behavior, not implementation. | Assert what the caller needs. A refactor that keeps behavior keeps the tests green. |
| A red test on `main` is a bug now. | "It fails on main too" is not a baseline. Fix it or file it, in a commit of its own, before building on it. |
| Whatever a test starts ends with it. | Goroutines, processes and timers a test starts are stopped before it returns. |
| Time is injected. | A test about timing uses a clock it controls, never a sleep it hopes is long enough. |

## Acceptance

A change is merged only when it shows each of these, with what was run
and what it printed. A check not run is listed as not run.

1. A test that fails without the change, and passes with it.
2. `gofmt -l` prints nothing for the changed files; `go vet ./...` and
   `go build ./...` pass; `go test -race ./...` passes. Run the whole
   module, not the packages you think you touched.
3. A change to the admin page runs its browser tests in Chromium and
   WebKit, in English and Chinese.
4. A change on a money path also runs against a local chain end to end:
   from the request to the confirmed transaction, and the refused cases
   (over a limit, a router not allowed, an address not allowed, locked).
5. Real use where it is safe: a source against the real feed, an adapter
   against a testnet. What wasn't tried for real is said, and isn't
   described to users as working.

## Go

- Write `any`, not `interface{}`.
- Errors are wrapped with what was being done: `fmt.Errorf("read limits: %w", err)`.
- A comment says what code is for in a sentence or two, and names
  functions rather than line numbers.
- No package-level mutable state outside `main`, except what a test can
  replace through a documented seam.

## Admin page

- Every string the user sees goes through `t()` and has its Chinese.
- Dropdowns use the page's own menu, not a native `<select>`.
- A click never scrolls the page.
- State is a dot or a swatch, not a colored stripe.
- Amounts, balances and addresses pass through the hide toggle.

## Commits

A subject names the area, then what the user now sees, as plain behavior:
`signer: a buy over the daily limit is refused with the limit it hit`.
The body says how it was verified and which test fails without it.

One commit, one goal. Two fixes are two commits, so either can be reverted
alone. Commit and push as each step is done: work that exists only on one
machine is work that can be lost.
