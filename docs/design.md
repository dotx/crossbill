# Design

Crossbill is one Go binary. It serves its admin page from assets embedded
in it, talks to that page over a JSON API, and starts its signer as a
child process of itself. There is no front-end build step and no second
service to deploy.

## The path of an opportunity

```
sources ─→ dedupe ─→ rules ─→ model scoring ─→ route
                                                ├─ notify (Telegram, Bark)
                                                ├─ track (1 h / 6 h / 24 h)
                                                ├─ execute (when trading is on)
                                                └─ drop
```

1. **Sources** turn the world into events. Each kind is a plugin: a
   public figure's posts, new pools and their volume, listing
   announcements, funding rates. Adding a source of a known kind is a row
   in the admin page; a new kind is a plugin.
2. **Dedupe** folds the same event seen many ways: dozens of tokens with
   one name, one announcement in five feeds. Embeddings group them.
3. **Rules** are deterministic: chain, liquidity, holders, contract checks
   (can it be sold, does the developer hold most of it, is liquidity
   locked). A rule starts from a template and is switched on and off.
4. **Scoring** asks a language model whether what's left matters to what
   you wrote down, and why. Cheap models screen; a stronger one judges the
   few that pass. Models are reached through any OpenAI-compatible
   endpoint; with [magpie](https://github.com/yetone/magpie) a tier is a
   routing group's name, and changing models is done there.
5. **Routing** sends each opportunity on by its score and type.

## Notifications

Two message types, configured separately:

| Type | What | Telegram | Bark |
| --- | --- | --- | --- |
| Opportunity | A token, its chain, why it fired, the checks, the score | A card with Track, Ignore, Open, and Buy once trading is on | The title, opening the card in the admin page |
| Alert | A tracked token moving hard, the developer selling, a post deleted, the signer locked, a service down | A card with Stop tracking, View | A critical alert, heard in silent mode |

For each type: which channels, the score it needs, quiet hours, Bark's
level. A button updates its message in place: working, done, refused.

## Tracking

Every opportunity, acted on or not, has its price recorded at 1 h, 6 h and
24 h. The results page counts, per source and per rule, how many fired and
how many went up afterwards. That is what says a source or rule is worth
keeping, and whether trading on it would have paid.

## Chains and execution

Three adapters, one per virtual machine:

| Adapter | Chains | Routes |
| --- | --- | --- |
| EVM | BSC, Base, Ethereum, Arbitrum, … | aggregators, the DEX router directly |
| Solana | Solana | aggregators, Jito bundles |
| Tron | Tron | SunSwap |

Another EVM chain is configuration: its id, RPC endpoints, router
addresses. An execution route lists its options in order and falls back
when one fails, the way magpie's routing groups fall back across
providers. A failure is classified before anything is rested: an RPC that
is down is rested, a token that can't be sold is not the RPC's fault.

## Wallets and signing

See [security](security.md).

## The admin page

| Group | Pages |
| --- | --- |
| Work | Opportunities, Tracking, Positions |
| Configure | Sources, Rules, Wallets, Chains, Notifications |
| Insights | Results, Model usage |
| Settings | Limits, keys, access |

Each row is a status dot, a name, one line of what it is doing, and a
switch. Detail opens in place. A "hide balances and addresses" toggle
masks the page for screenshots.

## Phases

1. Detect, track, notify. No wallets.
2. Wallets, the master password and the signer. Watch-only balances,
   manual buy and sell from the admin page.
3. Trading from Telegram and from rules, with exit rules (take profit in
   steps, trailing stop, time stop).

Each phase ships when the previous one has run for real.
