# Crossbill

[中文](README.zh-CN.md)

Crossbill watches several chains and the people who move them, tells you
when something worth a look happens, and can trade from wallets it holds
for you, within limits you set.

> **Status: design.** Nothing here runs yet. The documents below are the
> decisions made so far; code follows them.

## What it does

- **Detects** opportunities: a public figure's post matched to a token
  launched minutes later, a pool's volume or holders jumping, a listing
  announcement, a funding rate far from normal.
- **Filters** them: safety checks on the token, then a language model
  scores what's left against what you said you care about. Thousands of
  events a day become a few.
- **Notifies** you on Telegram and Bark, each message type to the
  channels you pick.
- **Tracks** every opportunity afterwards (price at 1 h, 6 h, 24 h), so
  you can see which sources and rules were worth following.
- **Trades**, once you turn it on: from per-chain hot wallets it holds
  encrypted, through a signer that refuses anything over your limits.

Chains: EVM (BSC, Base, Ethereum, Arbitrum, …), Solana and Tron.

## Defaults

- Detection only. Trading stays off until you turn it on and set limits.
- The admin page listens on `127.0.0.1`. Opening it to a network needs an
  access key.
- Crossbill sends nothing anywhere you didn't configure: your RPC
  endpoints, data sources, model endpoint and notification channels.
- You bring every key: data APIs, model endpoint, bot tokens.

## Documents

- [Design](docs/design.md): the parts and how they fit.
- [Security](docs/security.md): the master password, the signer, what is
  protected and what isn't.

## Not advice

Crossbill is a tool. It finds and acts on signals; it does not tell you
they're good. Most new tokens go to zero. Some services it can use are
not available in every country; check before you use them.

## License

[MIT](LICENSE)
