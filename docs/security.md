# Security

Crossbill holds private keys and can spend them. This page says how they
are kept, what stands between a request and a signature, and what
Crossbill does not protect against. A change that alters any of it updates
this page in the same commit.

## Defaults

- **Detection only.** A fresh install finds and notifies; it signs
  nothing. Trading is turned on in the admin page, after limits are set.
- **The admin page listens on `127.0.0.1`.** Serving it to a network
  (Tailscale, a LAN) needs an access key, carried in a sign-in link as a
  cookie is set. Its `/api` answers only its own origin: a request whose
  `Origin` is `null`, another origin, or a `Host` that isn't the address
  it serves is refused (a page rebound to `127.0.0.1` by DNS sends a `Host`
  of its own name).
- **No auto-update.** A new version is installed by the user.

## The master password

One password protects every key. It is set in the admin page before the
first wallet is created or imported; there is no way to hold a key without
it.

- **What is on disk.** A random 256-bit data key encrypts each wallet's
  key (XChaCha20-Poly1305). The data key is itself encrypted with a key
  derived from the password (Argon2id, parameters stored beside it). Only
  these ciphertexts are written. Changing the password re-encrypts the data
  key alone.
- **No recovery.** Crossbill keeps nothing that can stand in for the
  password. A wallet created in Crossbill shows its seed phrase once, at
  creation, to be written down; with the password lost, that phrase is the
  only way back to the funds.
- **Unlocking.** After a restart the keys are locked. The password is
  entered once, in the admin page or with `crossbill unlock`. Until then
  detection, tracking and notifications go on, and nothing is signed:
  no buy, no sell, no stop loss. Bark sends a critical alert, repeated
  until it is unlocked.
- **Wrong passwords.** Each failure waits longer before the next try is
  taken, and is notified.
- **Sensitive actions** ask for the password again each time: turning
  trading on, raising a limit, changing the withdrawal allowlist,
  withdrawing, exporting a key.

Unlocking without a person present (a TPM, a password file, an
environment variable) and passkey unlock are not part of the first version.
The data-key layout lets them be added as another wrapping of the same
data key.

## The signer

Keys live in one process only: the signer, a child process of the
`crossbill` binary, started by it and spoken to over a pipe. It has three
requests: unlock, sign an intent, lock.

- It opens no network connection.
- It is not dumpable (`PR_SET_DUMPABLE` 0, core size 0), so another
  process of the same user can't attach to it or read a core of it.
- The keys and the data key sit in locked memory (`mlock`), never in swap,
  which on many machines is not encrypted.
- It signs **intents**, never raw transactions handed to it: buy this
  token with this much of this wallet's balance, sell this share of a
  position. It builds the transaction itself from the intent and the
  chain's quote, then checks it against the policy:
  - per-trade and per-day limits, per wallet and per chain;
  - only the router contracts on the allowlist;
  - slippage at most the limit set;
  - a transfer only to an address on the withdrawal allowlist.

  Anything else is refused, whoever asked. The admin page, the Telegram
  buttons and the automatic strategies reach the signer only through this
  check, so a limit holds on every route to a signature.
- Limits are read from local configuration. The admin page can lower
  them without the password; raising one needs it. A limits file that
  can't be read means no signing, never "no limits".

## Wallets

- **Watch-only wallets** are addresses Crossbill reads balances of. It
  holds no key for them. Most funds belong there, or on an exchange or a
  hardware wallet.
- **Hot wallets**, one or more per chain, are the ones Crossbill signs
  for. Keep in them only what you can lose.
- **The withdrawal allowlist** is your own addresses (an exchange deposit
  address, a watch-only wallet). Profits go only there.

## Telegram and Bark

- Telegram buttons act only within the limits, and only for the Telegram
  account ids configured. Anything over a limit replies "confirm in the
  admin page". A stolen Telegram account can spend at most what the limits
  allow.
- A button pressed twice trades once. A button older than its quote, or a
  price moved past the slippage limit since the alert, is refused.
- Bark has no buttons; it only alerts.

## Models

- A language model scores and explains; it never holds a key, never signs
  and never places an order. Buying is decided by rules and by you.
- What goes to the model has wallet addresses, balances and amounts
  replaced by placeholders. It doesn't need them to judge a token.

## Masking

Every way out of the process passes one masker: logs, the database,
Telegram, Bark, model prompts, exports. It finds private keys and seed
phrases in the forms wallets export them, and checks each find: a Solana
key's second half is the public key its first half derives, a WIF key's
checksum holds, an EVM key's 64 hex digits count only after a name that
says it's a key (so a transaction hash isn't masked), a seed phrase's
words are all in the BIP-39 list.

## Backups

A backup carries configuration. It carries the encrypted wallets only when
asked for, and never anything that unlocks them.

## Supply chain

Dependencies are pinned. Releases are built reproducibly and signed. An
update replaces the signer too, so it is installed by hand, never
automatically.

## Threat model

Protected against:

- the disk, or a copy of the files, taken while the keys are locked or
  not: only ciphertext is there;
- a key leaking through logs, notifications, model prompts or backups;
- a stolen Telegram account, or a bug or compromise in the admin page or
  the notification handlers: they reach only the signer's policy, bounded
  by the limits;
- another program of the same user reading the signer's memory.

Not protected against:

- root on the machine while unlocked: root reads any process's memory;
- a malicious build of Crossbill you installed;
- losing the password with no seed phrase written down;
- a token that can't be sold, a rug pull or a bad trade: the limits bound
  the loss, they don't prevent it.

## Reporting a vulnerability

Report privately through the repository's **Security → Report a
vulnerability**. Please don't open a public issue for it.
