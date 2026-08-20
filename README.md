# printf-docs

Source for [docs.printf.dev](https://docs.printf.dev).

```bash
npm install
npm run dev       # :3000
npm run build
```

## How pages stay honest

The API reference under `docs/api/` is generated from `order-api/openapi.yaml`
on every merge to that repo's main. **Do not hand-edit it** — your change will
be overwritten on the next spec change, silently, and you will not find out
until a customer does.

Everything under `docs/guides/` is hand-written, which means nothing regenerates
it and nothing notices when it drifts. Every guide carries `last_reviewed` in
frontmatter. A guide unreviewed for 90 days is a bug with a slower fuse.

## Frontmatter

```yaml
---
title: Sizing and fit
section: guides
last_reviewed: 2026-05-14
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---
```

`covers_endpoints` and `covers_sdks` are what let us answer "which pages does
this API change affect" without a human reading all forty of them.
