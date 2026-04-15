# Contributing


## Linuxfabrik Standards

The following standards apply to all Linuxfabrik repositories.


### Code of Conduct

Please read and follow our [Code of Conduct](CODE_OF_CONDUCT.md).


### Issue Tracking

Open issues are tracked on GitHub Issues in the respective repository.


### Pre-commit

Some repositories use [pre-commit](https://pre-commit.com/) for automated linting and formatting checks. If the repository contains a `.pre-commit-config.yaml`, install [pre-commit](https://pre-commit.com/#install) and configure the hooks after cloning:

```bash
pre-commit install
```


### Commit Messages

Commit messages follow the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) specification:

```
<type>(<scope>): <subject>
```

If there is a related issue, append `(fix #N)`:

```
<type>(<scope>): <subject> (fix #N)
```

`<type>` must be one of:

- `chore`: Changes to the build process or auxiliary tools and libraries
- `docs`: Documentation only changes
- `feat`: A new feature
- `fix`: A bug fix
- `perf`: A code change that improves performance
- `refactor`: A code change that neither fixes a bug nor adds a feature
- `style`: Changes that do not affect the meaning of the code (whitespace, formatting, etc.)
- `test`: Adding missing tests


### Changelog

Document all changes in `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Sort entries within sections alphabetically.


### Language

Code, comments, commit messages, and documentation must be written in English.


### CI Supply Chain

GitHub Actions in `.github/workflows/` are pinned by commit SHA, not by tag. Dependabot's `github-actions` ecosystem keeps these pins up to date.

Python packages installed via `pip` inside workflows follow a two-tier policy:

- `pre-commit` is installed from a hash-pinned requirements file at `.github/pre-commit/requirements.txt`, generated with `pip-compile --generate-hashes --strip-extras` from `.github/pre-commit/requirements.in`. Dependabot's `pip` ecosystem watches that directory and maintains both files.
- One-shot installs such as `ansible-builder`, `build`, `mkdocs`, `pdoc`, and `ruff` in release, docs, or test workflows are version-pinned only (`package==X.Y.Z`) and kept fresh by Dependabot. Scorecard's `pipCommand not pinned by hash` findings for these are considered acceptable risk and may be dismissed.


### Versioning the Kickstart File

All three installer config files (`lf-rhel.cfg`, `lf-debian.cfg`, `lf-ubuntu.cfg`) carry a shared running build stamp under the name `LF_KICKSTART_VERSION`. The same stamp value applies to all three files at any given time — it is a repo-wide build marker, not a per-file version.

The format is `YYYYMMDDNN`, where `YYYYMMDD` is the build date and `NN` is a two-digit daily sequence number starting at `01`. Example: `2026041501` for the first build on 2026-04-15, `2026041502` for the second on the same day.

On every supported target, the stamp lands in two places: the installer-time logs and the file `/root/lf-install-version` on the installed system. The installer-time logs differ by target:

- `lf-rhel.cfg`: exported as a shell variable in `%pre`, echoed into `/tmp/kickstart.install.pre.log`, re-echoed by the generated `%post` (so it shows up in `/var/log/anaconda/ks-script-*.log`), and embedded as a comment in `/root/dynamic.ks`.
- `lf-debian.cfg`: logged via `logger(1)` in `d-i preseed/early_command`, which lands in `/var/log/syslog` during install and later in `/var/log/installer/syslog`.
- `lf-ubuntu.cfg`: logged via `logger(1)` in `early-commands`, which lands in the live installer journal and later in `/var/log/installer/installer-journal.txt`.

The stamp appears as a literal in multiple places within each of the three files (header comment, early/late commands, and for `lf-rhel.cfg` the `%pre` shell variable). All occurrences must be kept in lockstep when bumping — `grep -n LF_KICKSTART_VERSION lf-*.cfg` is a quick way to find them.

Rules:

- Bump `LF_KICKSTART_VERSION` in every commit that changes the effective content of `lf-rhel.cfg`, `lf-debian.cfg` or `lf-ubuntu.cfg` (kickstart/preseed/autoinstall logic, embedded shell/Python, comments that end up in the generated output).
- Pure documentation commits (`README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`) do not bump the version.
- If multiple content-changing commits happen on the same day, increment `NN` accordingly.
- Releases (as described in the `Changelog` section) also imply a version bump, because they are also content changes.


### Coding Conventions

- Sort variables, parameters, lists, and similar items alphabetically where possible.
- Always use long parameters when using shell commands.
- Use RFC [5737](https://datatracker.ietf.org/doc/html/rfc5737), [3849](https://datatracker.ietf.org/doc/html/rfc3849), [7042](https://datatracker.ietf.org/doc/html/rfc7042#section-2.1.1), and [2606](https://datatracker.ietf.org/doc/html/rfc2606) in examples and documentation:
    - IPv4: `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`
    - IPv6: `2001:DB8::/32`
    - MAC: `00-00-5E-00-53-00` through `00-00-5E-00-53-FF` (unicast), `01-00-5E-90-10-00` through `01-00-5E-90-10-FF` (multicast)
    - Domains: `*.example`, `example.com`
