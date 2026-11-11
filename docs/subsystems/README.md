# Subsystem references

Each page describes a subsystem as it is now: what it is responsible for,
where its truth lives, its main paths, its states, and how it fails. Start
with the one for your change, then verify the source it links.

| Subsystem | Reference |
| --- | --- |

None yet; a subsystem gets its page in the commit that adds it.

## Writing a reference

Include:

- **Responsibilities and boundaries**: what it does, and what it leaves to
  others.
- **Sources of truth**: the files, tables and processes that hold its state,
  linked by path and named symbol.
- **Runtime paths**: from the input to the result, step by step.
- **States and transitions**: what moves it from one to the next.
- **Failure behavior**: what it does when a read fails, a peer is down or a
  value is malformed.
- **Verification**: the tests and commands that exercise it.

Link to source files and named symbols rather than copying details that
change. Keep each fact in one page and link to it from others. Describe
current behavior only; a proposal belongs in its issue.

## Keeping it current

A change that alters a responsibility, an interface, a state transition, a
runtime path or a failure behavior updates the reference in the same
commit. A refactor that keeps them needs no prose change.

## Reviewing a change

1. Find the subsystem and the real runtime path.
2. Compare the claimed before and after with the code, past the changed
   lines: callers, state, side effects.
3. Check the reference still matches and its links resolve.
4. Check the cited tests exercise the claim. Report what wasn't run.
