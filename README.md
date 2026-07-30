# Grimoire index template

Run your own [Grimoire](https://grimoire.rs) package index: a searchable site
of AI-config packages — skills, rules, agents, MCP servers, bundles — that
`grim` installs from, with a contribution gate that decides what auto-merges.

The index holds **pointers**. The packages themselves live in an OCI registry
you already have (GHCR, GitLab's registry, Harbor, anything OCI). Nothing here
needs a secret to run.

## Start

1. **Use this template** → name your repository → create it.
2. Clone it and run:

   ```sh
   npm install
   npm run setup
   ```

`setup` reads your `origin` remote, so it already knows which forge you are on
and where GitHub Pages will serve you — the questions it asks are about
branding, and most of them are Enter. It then writes the whole index: the
`index/` tree, `index.config.json`, the trust policy, and the CI, and replaces
this README with one about your index.

3. Commit and push what it wrote, **including `package-lock.json`** — CI
   installs from it, so it is not optional.
4. In **Settings → Pages**, set **Source: GitHub Actions**. Until you do, the
   `pages` workflow runs green and deploys nothing.

That is the whole setup. Your index is live at the URL `setup` printed.

## Then

Publishers announce into your index with `grim publish --announce`, which opens
a pull request adding one pointer file. The `validate` workflow is the gate:
exit 0 means the change is eligible to merge — the author owns the namespace on
your forge, the path is in bounds, the registry host is on your committed
allowlist, and the OCI reference actually resolves. Anything else means a human
looks at it. Turn on `"autoMerge": true` in the `ci` block when you want passing
contributions to land by themselves.

To require the gate, add `validate` as a required status check in branch
protection. Add it and the branch rule together — a required context that never
reports blocks every pull request forever.

Consumers add your index with one line:

```sh
grim config registry add <alias> --index https://<your-pages-url>
```

## What you get

| File | What it is |
|---|---|
| `index/<host>/<namespace>/<package>/metadata.json` | The data. One pointer per package |
| `index.config.json` | Branding, your registry, and the `ci` block the workflows are rendered from |
| `index-policy.json` | The committed allowlist the gate reads — registry hosts, reserved namespaces, trusted bots |
| `.github/workflows/` | Generated from the `ci` block. Change the config and run `npm run ci`; do not hand-edit them |

`npm run dev` previews the site locally. `npm run ci:check` is what CI runs to
catch a hand-edited workflow.

## GitLab

Works the same way. `setup` reads a GitLab remote and renders `.gitlab-ci.yml`
instead — same gate, GitLab Pages deploy. One caveat, and it is structural: a
merge request supplies its own `.gitlab-ci.yml`, so the gate is advisory unless
you enforce it instance-side (a required merge check, protected-branch approval
rules, or a compliance pipeline). Auto-merge is therefore GitHub-only.

## Notes

- Pick a licence. This template ships none on purpose — the index metadata and
  your repository are yours to licence. The first-party index publishes its
  metadata under CC0.
- The renderer, the build and the gate all live in
  [`@grimoire-rs/indexer`](https://github.com/grimoire-rs/indexer).
  `package-lock.json` pins which version judges your contributions; bump it the
  ordinary npm way and re-run `npm run ci`.
- `npm run setup` is a one-shot bootstrap: it overwrites this file and
  `package.json`. To change something later, edit `index.config.json` and run
  `npm run ci`.
