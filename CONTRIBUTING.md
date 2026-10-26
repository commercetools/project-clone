# Contributing

This is a commercetools-labs project, provided as-is and unsupported (see [SUPPORT.md](SUPPORT.md)).
Contributions are welcome and reviewed on a best-effort basis.

Please only contribute code you have the right to license under MIT, and keep any third-party
code's own license file and copyright notice intact.

Never commit credentials or personal data. `.env.source`, `.env.target`, `data/` and `logs/` are
gitignored because they hold client secrets and a real project's catalog, customers and orders,
and test fixtures stay synthetic (`jane@example.com`).

## Before you open a PR

```bash
npm test
```

That is `node --test 'test/*.test.mjs'`. It needs no credentials and no network, finishes in
about a second, and is the whole check for this repository, so please run it before you open a
PR. The code is plain JavaScript with no dependencies, so there is no build, typecheck or lint
step alongside it.

**Testing against real projects is a deliberate, manual step.** Nothing in `npm test` talks to
commercetools. If your change touches the export, import or wipe paths, exercise it against two
projects you are happy to have written to: `npm run inventory` first to confirm which ones those
are, then a narrow `--only <resource>` run rather than a full clone. Credential-dependent tests
stay out of `test/`, which keeps the offline suite fast.

## What good looks like here

This repository is written to be read. People land in it to understand how a key-based transfer
works before writing their own, so:

- **Comment the *why*, not the *what*.** `// add the key` is noise. "A commercetools `id`
  means nothing outside the project that issued it and a `key` means the same thing
  everywhere, so the transfer runs on keys" is the reason the code is shaped the way it is,
  and it cannot be recovered from reading it.
- **Match the surrounding code.** Its naming, its comment density, its idioms. One style
  throughout keeps the codebase easy to read, even where a new style would be nicer in
  isolation.
- **Keep it dependency-free.** This runs on plain Node 20+ ESM with no `node_modules`. That is
  deliberate: it makes a tool that holds `manage_project` credentials on two projects auditable
  by reading it. A new dependency needs a clear reason in the PR.
- **Every resource belongs in `src/resources.mjs`.** The resource table carries the endpoint,
  the dependency `order`, the key strategy and the draft shaping for each type, in one place.
  Adding a resource means adding an entry rather than a new code path, and the `order` number
  is what guarantees a tax category exists before the product that points at it.
- **Keep every stage idempotent and resumable.** You resume a run that stops halfway by
  running it again, because what the target already holds is filtered out before sending.
  This is the property that makes the tool usable on a catalog large enough to fail partway.
  A change that makes a second run create duplicates is a bug, however much faster it is.
- **Reference rewriting goes through `src/refs.mjs`.** Every `{typeId, id}` becomes
  `{typeId, key}` in one place. A resource that reaches into a draft and rewrites a reference
  itself is how one resource type quietly stops resolving.
- **Transforms are testable offline, and tested.** Reference rewriting, locale discovery, enum
  denormalisation, key derivation and draft shaping are all pure functions over data, which is
  why `test/` can cover them without credentials. If you add one, extend the suite as well, so
  the coverage lives in the repository rather than in a run against one project.
- **Keep it project-agnostic.** Nothing about a particular project, region or catalog belongs in
  the code. Credentials and URLs come from `.env.source` / `.env.target` or the `SOURCE_CTP_*` /
  `TARGET_CTP_*` environment variables; anything an operator should be able to change belongs
  on the command line, not in a constant.
- **Anything destructive stays guarded and visible.** `wipe` requires `--confirm <projectKey>`
  matching the target exactly, and `clone` makes the operator type the target key. Both guards
  stay as they are, and `--yes` stays as narrow as it is.

## Reporting a security issue

Security issues in this code go in a GitHub issue. Issues are public and this repository has no
private disclosure channel, so please describe a sensitive finding with that in mind.

Vulnerabilities in the commercetools platform itself go to
[commercetools' security contact](https://commercetools.com/privacy-and-security).

By contributing you agree your contribution is licensed under the [MIT License](LICENSE).
