# Notes for coding agents

Crossbill holds private keys and can spend them. Read this page before
changing anything, and the pages it links before changing what they cover.

## Before a change

- [Design](docs/design.md): the parts and how an opportunity moves
  through them.
- [Security](docs/security.md): the master password, the signer and its
  policy, masking, the threat model. A change to any path that reaches a
  signature, a key or a limit is checked against it, and updates it in the
  same commit when it changes what it says.
- [Subsystem references](docs/subsystems/README.md): read the one for the
  subsystem you change, then the source it links.
- [Code standards](docs/code-standards.md): what a change, a test and a
  commit must show.
- [Lessons](LESSONS.md): what earlier work got wrong and the rule that
  would have caught it.

## Money paths

A money path is any code that can lead to a signature: the signer, its
policy and limits, wallet storage, the execution adapters, and every entry
that asks for a trade (admin page, Telegram buttons, rules). On a money
path:

- The signer's policy is the only gate. A check made in a caller (a
  button handler, a page) is a convenience, never the guard.
- A language model's output never becomes an order, a limit, an address or
  an amount. It scores and explains.
- A key, a seed phrase or the master password never leaves the signer
  process: not in a log, an error, a test failure, a notification, a
  prompt or a panic message.
- Tests use testnets, local chains or throwaway keys made in the test.
  Never a real key, never mainnet.

## When unsure

Stop and say what you don't know. A guess about how a chain, a router or
an API behaves is checked against its source or docs before code is built
on it.
