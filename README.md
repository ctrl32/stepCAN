# stepCAN

Specification for a CANopen distributed I/O bus whose node addresses are derived
from the hardware rather than configured — usable both as a DIN-rail backplane
and as a bus spread across a machine or a building.

**Revision 0.2 · 2026-09-15**

## What is in here

`docs/` holds the specification, written in the **Sourcebook dialect**: valid
Markdown first, with YAML front matter and folder structure carrying the
navigation. It builds unchanged with the Sourcebook platform, with MkDocs
Material, or with Docusaurus.

```
docs/stepcan/
├── index.md                      landing page
├── overview/                     what it is, architecture, deployment profiles
├── physical-layer/               connectors, cable, CAN, chain, protection
├── protocol/                     enumeration, CANopen layer, indicators
├── implementation/               power, module design, firmware, test, BOM
└── reference/                    verification status, sources
```

Navigation comes from the folder tree plus `sidebar_position` and
`sidebar_label` in each file's front matter. A file whose `slug` is `index`
*is* its section and contributes nothing to the address.

## Building

### Sourcebook

Point the platform at this repository. Only `main` is fetched and published.
Tags matching `docs/*` become released versions:

```bash
git tag docs/0.2 && git push origin docs/0.2
```

### MkDocs Material

`mkdocs.yml` carries an equivalent explicit `nav`, so the same tree builds
without the platform:

```bash
pip install -r requirements.txt
mkdocs serve          # http://127.0.0.1:8000
mkdocs build --strict
```

`.github/workflows/deploy.yml` builds with `--strict` and publishes to GitHub
Pages on every push to `main`. Enable Pages in the repository settings with
**Source: GitHub Actions**.

### Docusaurus

The front matter fields (`title`, `slug`, `sidebar_position`, `sidebar_label`,
`description`, `tags`) are the ones Docusaurus already uses, so `docs/` drops in
as-is. Note that the admonitions here use the MkDocs Material `!!!` spelling;
Docusaurus needs the `:::` spelling, which the Sourcebook parser accepts for
both.

## Conventions used in the source

- **Admonitions** use `!!! type "Title"` with a four-space indented body.
  Types used: `note`, `tip`, `info`, `warning`, `danger`.
- **Requirements** are numbered `TR-1` … `TR-7` and each sits in an
  `!!! info` or `!!! danger` block so it is visually distinct from discussion.
- **Every electrical figure** is either quoted from a named source, or derived
  with the calculation shown. Nothing is asserted without one or the other.
- **Cross-document links** are relative and point at `.md` files, e.g.
  `[clock accuracy](../physical-layer/can.md#clock-accuracy-a-crystal-is-mandatory)`.

## Status

Revision 0.2 closed all eight items that revision 0.1 left open. Three of them
changed the design — a mandatory crystal, a changed CAN transceiver, and a
withdrawn upgrade recommendation. See
[Verification status](docs/reference/verification.md).

Four documentary points remain partially settled; none blocks the design, and
each is listed with the document that would settle it.

## Where it is published

ctrl32.com serves this specification through **Sourcebook**, the
documentation engine of fabrikaERP, which reads these files straight from
this repository — no export step, no second copy. The pages appear under
`/docs/stepcan/`.

The book lives in `docs/stepcan/` rather than directly in `docs/` on
purpose: Sourcebook builds a page's address by walking its folder index
pages, so the folder is what keeps this specification a book of its own
beside the ctrl32 manual instead of six loose sections mixed into it. Each
folder's `index.md` declares the `slug:` that names its address segment.

`.github/workflows/portability.yml` publishes nothing. It builds the same
files with plain MkDocs Material and `--strict`, so a broken internal link
turns the build red — that is the promise Sourcebook rests on: the files
are the source, not an import.
