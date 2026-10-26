# Support

Thanks for taking a look at this project. It is commercetools-labs reference code: shared in the
hope that it is useful, and **unsupported**.

In practice that means:

- It is freely available under the MIT license ([LICENSE](LICENSE)) and is provided as-is, without
  warranty.
- It isn't covered by commercetools Support, your account team or any service-level agreement, so
  we can't promise replies, fixes or new releases.
- Issues and pull requests are welcome, and we'll take a look when we can.

## Before you point it at a project that matters

`ctclone` writes to two projects, deletes from one of them on request, and needs
`manage_project` on both, so check these first:

- **`wipe` deletes everything cloneable from the target.** Its guard is that
  `--confirm <projectKey>` must match the target's key exactly (`src/wipe.mjs`), and that key
  comes from whatever `.env.target` contains. So the guard confirms you typed the key
  `.env.target` names; making sure that is the project you mean is up to you.
- **Check which project `.env.target` resolves to, every time, before you run anything.**
  `npm run inventory` prints both sides and writes nothing. A typo in a project key is the
  one mistake here with no undo. `clone` asks you to type the target key to continue; `--yes`
  skips that prompt, so leave it off when you are running by hand.
- **`keyfix` writes to the *source* project.** It adds a `key` to source entities that lack
  one, because the whole transfer runs on keys rather than ids (`src/keyfix.mjs`). It is the
  only write made to the source, it is additive, and `--dry-run` shows you the set first. On a
  project you did not plan to modify, run that first.
- **Every cloned customer is created with the same password.** Source passwords are one-way
  hashes and cannot be carried over, so each customer is created with `--customer-password`,
  which **defaults to `123`** (`bin/ctclone.mjs`). If anyone other than you can reach the
  target, pass a real one. Customers on `ExternalAuth` are created without a password.
- **Scope the API clients, and retire them afterwards.** Both sides need `manage_project`:
  the source to read everything and set keys, the target to write every resource type. That
  is close to the broadest credential the platform issues. Mint a pair for the migration,
  keep them out of any deployed application, and delete them when the clone is done. Nothing
  in this tool needs to run continuously.
- **`.env.source` and `.env.target` are gitignored, and are best kept that way.** They hold
  client secrets in plain text.
- **The clone is faithful, not identical.** Carts, quotes, extensions, subscriptions,
  messages and order edits are not transferred, orders arrive External-taxed whatever they
  were in the source, and some order fields have nowhere to go. Each order records what it
  lost under `_lossy` in the export. The README's "What does not, and why" table is the
  complete list, with the reason for each.
- **Verify the result before you rely on it.** `verify` compares counts, `verify --deep` diffs
  every entity field by field, and `smoke` drives the live API: it logs a cloned customer in,
  resolves a product's references, adds a SKU to a cart and runs a search. A matching count
  is the weakest of the three, and it agrees with a product that arrived without prices.

## Finding your way around the code

- The code is commented with the reasoning as well as the mechanics. `src/resources.mjs` is
  the one file to start with: the resource table, the dependency order and the draft shaping
  all live there. Then `src/refs.mjs` for how an `id` reference becomes a `key` reference,
  `src/keyfix.mjs` for key derivation, and `src/importApi.mjs` for the bulk path.
- The tests show the transforms at work. `test/transform.test.mjs` and
  `test/bulkimport.test.mjs` cover reference rewriting, locale discovery, enum
  denormalisation, key derivation and draft shaping. They run offline against no project, in
  about a second, with `npm test`.
- [FEATURES.md](FEATURES.md) is a code-derived inventory of what is implemented, and the
  quickest answer to "does this handle X?".

## If you need help

- **Questions and bugs:** open a GitHub issue. Another reader or a maintainer may well have an
  answer, though we can't commit to one.
- **Using it in production:** feel free to fork it and adapt it to your needs. Once it's in
  production it's yours, so review the code, test it against projects of your own and keep
  your copy up to date.
- **commercetools products:** for questions about Composable Commerce itself, the
  [commercetools documentation](https://docs.commercetools.com), the community and
  commercetools Support are the right place, as usual.
