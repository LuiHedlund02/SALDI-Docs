# SALDI

SALDI is a free Danish accounting system for bookkeeping, invoicing, inventory, purchasing, production, bookings, and integrations. This repository contains the application codebase and the supporting documentation set.

## Key module areas

- `admin/` — administration, setup, licensing, backups, restore
- `finans/`, `debitor/`, `kreditor/` — core accounting, AR, and AP flows
- `lager/`, `produktion/` — stock, goods, and production workflows
- `booking/`, `rental/`, `sager/`, `projectManager/` — operational business modules
- `api/`, `restapi/`, `soapserver/`, `soapklient/`, `sync_shop/`, `dandomain/` — integration surfaces
- `systemdata/`, `includes/`, `stdFunc/` — shared runtime, configuration, and utilities

## Installation

Start with [`INSTALLATION.txt`](INSTALLATION.txt) for the current setup instructions.
For the documentation-led install flow, see [`docs/getting-started/installation.md`](docs/getting-started/installation.md).

## Documentation

- [`docs/`](docs/) — current technical documentation and the preferred source of truth for maintainers
- [`doc/`](doc/) — legacy release notes, historical reference material, and older HTML/PDF assets
- [`guides/`](guides/) — user-facing guides and PDFs

The docs hub is [`docs/README.md`](docs/README.md).

## Contribution

Keep changes small and practical. When code changes affect setup, configuration, integrations, or user flows, update the relevant docs in the same pull request.

## Links

- Project website: [saldi.dk](https://saldi.dk)
- Historical Danish entry doc: [`LAESMIG.txt`](LAESMIG.txt)
