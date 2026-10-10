# Doc42

Doc42 generates engineering documents, such as requirements specifications and
design descriptions, from a SysML v2 repository. What a document contains and
how it is structured is defined in SysML v2; tables and diagrams are projected
from the model, so the document cannot drift from it. The output is PDF.

This repository publishes the Doc42 command-line tool for Linux and a GitHub
Action that installs it. Doc42 is closed-source software by
[Elan8](https://elan8.com).

## Use in GitHub Actions

```yaml
jobs:
  documents:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: elan8/doc42-action@v0.1.0

      - run: doc42 check model

      - run: doc42 build model --doc ProductDocs::srs --out out
        env:
          ELAN8_LICENSE: ${{ secrets.ELAN8_LICENSE }}

      - uses: actions/upload-artifact@v4
        with:
          name: documents
          path: out/*.pdf
```

The action puts `doc42` on the `PATH`. To run a single command in the same step,
pass it as `args` and set `ELAN8_LICENSE` on that step:

```yaml
- uses: elan8/doc42-action@v0.1.0
  with:
    args: build model --doc ProductDocs::srs --out out
  env:
    ELAN8_LICENSE: ${{ secrets.ELAN8_LICENSE }}
```

| Input | Default | Meaning |
| --- | --- | --- |
| `version` | `0.1.0` | Doc42 version to install |
| `args` | empty | Arguments for one `doc42` run; empty installs only |
| `working-directory` | `.` | Directory to run `doc42` in |

The output `executable-path` is the path of the installed executable.

A working example is the
[robot vacuum cleaner showcase](https://github.com/elan8/sysml-robot-vacuum-cleaner),
which builds a customer requirements, a system requirements and a system design
specification on every push.

## License key

`doc42 list`, `doc42 check` and `doc42 library export` work without a license.
`doc42 build` and `doc42 ir` need one.

A license is a token that starts with `elan8ci_`. Store it as a repository or
organization secret named `ELAN8_LICENSE` and pass it to the step through the
environment, as shown above. Doc42 verifies the token offline against Elan8's
public keys; it does not contact any server and does not send your model
anywhere. `doc42 license` shows who a token is issued to and how long it is
valid.

Secrets are not available to pull requests from forks. Skip the build step for
them:

```yaml
if: github.event_name != 'pull_request' || github.event.pull_request.head.repo.full_name == github.repository
```

To obtain a license, contact Elan8 through [elan8.com/contact](https://elan8.com/contact).

## Use outside GitHub Actions

Download `doc42-<version>-linux-x64.tar.gz` and `SHA256SUMS.txt` from the
[releases](https://github.com/elan8/doc42-action/releases), check the archive
with `sha256sum --check`, and unpack it. The executable needs glibc 2.35 or
later (Ubuntu 22.04, Debian 12 and newer). Windows and macOS builds are not
published yet.

## Writing documents

A document is a SysML view annotated with `@Doc42::Document`; its nested views
are the sections.

```sysml
package ProductDocs {
    private import Doc42::*;
    private import StandardViewDefinitions::*;

    view srs {
        @Document {
            title = "Product";
            organization = "Example Engineering";
            number = "PRD-SRS-001";
            revision = "A";
            template = "srs";
        }
        view purpose {
            @Section { title = "Purpose and scope"; }
            doc /* What the system is and what this document covers. */
        }
        view requirements : GeneralView {
            @Section { title = "System requirements"; }
            @RequirementTable;
            expose Product::ResponseRequirement;
        }
        view architecture : InterconnectionView {
            @Section { title = "System interconnection"; }
            expose Product::system;
        }
    }
}
```

Built-in templates: `crs` (customer requirements), `srs` (system requirements),
`sds` (system design), `test` (test specification), `verification-report` and
the general `elan8-standard`. `doc42 library export <dir>` writes the `Doc42`
SysML library for use on an editor's library path.

Doc42 reads models with [Spec42](https://github.com/elan8/spec42), the
open-source SysML v2 toolchain by Elan8. Pass additional SysML libraries with
`--library-path`.

## Legal

The Doc42 executable is proprietary software, © Elan8, all rights reserved. See
[LICENSE.md](LICENSE.md). Third-party components are listed in the
`THIRD_PARTY_NOTICES.md` file inside each release archive. The action definition
and the examples in this repository may be copied freely.
