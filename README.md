<p align="center">
  <img src="https://github.com/user-attachments/assets/88b976b4-cc46-4706-a3e4-3cfa0e6877d5" alt="zanadir">
</p>

<p align="center">
  <a href="#coverage-score"><img src="https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/MustacheCase/zanadir/badges/zanadir-badge.json" alt="zanadir coverage"></a>
</p>

## Features

- 📂 **Scan**: Analyze the repository for CI/CD enhancement suggestions, including security services and best practices.
- ❓ **Help**: Get details on available commands and usage.
- 🔍 **CI Analysis**: Examines the repository's Continuous Integration (CI) setup and suggests improvements for security and best practices.
- 🚀 **Open Source**: Contributions are welcome to enhance Zanadir's capabilities!

## Supported CI Actions

Zanadir currently supports:

- GitHub Actions
- CircleCI
- GitLab
- Bitbucket Pipelines
- Azure Pipelines

Future work will include support for:

- Jenkins

## Categories We Suggest

Zanadir analyzes repositories in the following categories. The name in bold is
the exact value to pass to `--excluded-categories` (matching is
case-insensitive):

- 🛡️ **SCA**: Software Composition Analysis
- 🔐 **Secrets Detection**: Secrets Management
- 📜 **License Compliance**: License Compliance
- 🛠️ **End Of Life**: End-of-Life Software Packages
- 📊 **Coverage**: Test Coverage
- 📊 **Performance Testing**: Test Performance and Reliability
- 🧑‍💻 **Linter**: Code Linting
- 🧪 **Unit Tests**: Automated Unit Testing
- 🔎 **SAST**: Static Application Security Testing
- 🏗️ **IaC Security**: Infrastructure-as-Code Misconfiguration Scanning
- 🔗 **Supply Chain**: SBOM Generation, Signing and Provenance
- 🧱 **Workflow Security**: Hardening of the pipeline definition itself

## Usage Examples

### Basic Usage

Scan a repository for CI/CD improvement suggestions:

```sh
zanadir scan --dir /path/to/your/repo
```

### Output Formats

Zanadir supports three output formats: table (default), JSON and SARIF.

#### Table Output (Default)

```sh
zanadir scan --dir . --output table
```

**Sample Output:**
```
Coverage 10/11 - 1 category needs attention:

|--------------------------------|--------------------------------|-------------------|
|            CATEGORY            |          DESCRIPTION           |  SUGGESTED TOOLS  |
|--------------------------------|--------------------------------|-------------------|
| Performance and Reliability    | Tools for load, stress and     | k6, JMeter,       |
| Testing Tools                  | reliability testing, to verify | Gatling, Apache   |
|                                | a system holds up under        | Bench, Artillery, |
|                                | expected and peak traffic.     | BlazeMeter        |
|--------------------------------|--------------------------------|-------------------|
```

When nothing is missing, the scan says so rather than printing an empty table:

```
Coverage 11/11 - all categories are covered.
```

The headline is bold on an interactive terminal. Colour is omitted when the
output is piped or written with `--output-file`, and when `NO_COLOR` is set.

#### JSON Output

```sh
zanadir scan --dir . --output json
```

**Sample Output:**
```json
[
  {
    "ID": "Performance Testing",
    "Name": "Performance and Reliability Testing Tools",
    "Description": "Tools for measuring code coverage to ensure testing completeness and software quality.",
    "Suggestions": [
      {
        "Name": "k6",
        "Repository": "https://github.com/grafana/k6",
        "Description": "Grafana k6 is an open-source, developer-friendly, and extensible load testing tool. k6 allows you to prevent performance issues and proactively improve reliability.",
        "Language": ""
      },
      {
        "Name": "JMeter",
        "Repository": "https://github.com/apache/jmeter",
        "Description": "An Apache project designed to load test functional behavior and measure performance, with support for various protocols and servers.",
        "Language": ""
      }
    ]
  }
]
```

#### SARIF Output

```sh
zanadir scan --dir . --output sarif
```

SARIF is the standard interchange format for static analysis results. Emitting
it lets uncovered categories show up in GitHub's Security tab, and in any other
SARIF-consuming platform, alongside your other scanners.

Each uncovered category becomes one SARIF rule and one result at `warning`
level, with the suggested tools listed in the result's help text. Results are
anchored at a CI configuration file — the place the missing tool would be
added. A missing control is not a defect on a particular line, but every SARIF
consumer expects a location, and GitHub code scanning rejects a report whose
results have none.

#### Markdown Output

```sh
zanadir scan --dir . --output markdown
```

Markdown is the format for putting a scan in front of people rather than in a
file. Tools link to their repositories, and the report opens with a hidden
marker so a workflow can update the comment it posted last time instead of
adding a new one on every push.

**Sample Output:**
```markdown
<!-- zanadir-report -->
### zanadir

Coverage 10/11 - 1 category needs attention:

| Category | Description | Suggested tools |
| --- | --- | --- |
| Performance and Reliability Testing Tools | Tools for load, stress and reliability testing. | [k6](https://github.com/grafana/k6), [JMeter](https://github.com/apache/jmeter) |
```

When nothing is missing the table is omitted, leaving a single line.

#### Writing the report to a file

Use `--output-file` to write the report to a path instead of stdout. This works
with every format, and is the reliable way to produce a machine-readable report:
redirecting stdout captures the debug log as well, so `--output sarif --debug >
report.sarif` yields a file that is not valid JSON.

```sh
zanadir scan --dir . --output sarif --output-file zanadir.sarif
```

To publish the report from a GitHub Actions workflow:

```yaml
- name: Run zanadir
  run: zanadir scan --dir . --output sarif --output-file zanadir.sarif

- name: Upload SARIF
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: zanadir.sarif
```

Uncovered categories then appear in the repository's **Security** tab alongside
your other scanners.

### Posting the report to a pull request

SARIF puts uncovered categories in the **Security** tab, which people visit
occasionally. A pull request comment is where they already are.

```yaml
permissions:
  contents: read
  pull-requests: write

steps:
  - name: Run zanadir
    run: zanadir scan --dir . --output markdown --output-file zanadir.md

  - name: Post the report
    run: gh pr comment "$NUMBER" --body-file zanadir.md --edit-last --create-if-none
    env:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      NUMBER: ${{ github.event.pull_request.number }}
```

`--edit-last` keeps one comment up to date instead of adding one per push. It
matches on the last comment by the same user, so if something else comments as
`github-actions` too, match the marker instead:

```yaml
  - name: Find the previous report
    uses: peter-evans/find-comment@v4
    id: previous
    with:
      issue-number: ${{ github.event.pull_request.number }}
      body-includes: "<!-- zanadir-report -->"

  - name: Post the report
    uses: peter-evans/create-or-update-comment@v5
    with:
      comment-id: ${{ steps.previous.outputs.comment-id }}
      issue-number: ${{ github.event.pull_request.number }}
      body-path: zanadir.md
      edit-mode: replace
```

A pull request from a fork gets a read-only token, so the comment step cannot
post. Guard it with
`if: github.event.pull_request.head.repo.full_name == github.repository`
rather than reaching for `pull_request_target`, which would run your workflow
with write access to a fork's code.

## Coverage Score

Every scan is summarised as a coverage score: the categories that apply to the
repository, minus the ones still uncovered.

Three rules keep the number honest:

- **Excluded categories leave the denominator.** A repository is not marked
  down for a category it has declared irrelevant with `--excluded-categories`.
- **So do categories with nothing to scan.** A repository with no Terraform,
  Helm chart, Kubernetes manifest or Dockerfile is not told to add Checkov,
  and is scored out of ten rather than eleven. The scan says which:

  ```
  IaC Security is not applicable and is out of the score: nothing in this
  repository needs it.
  ```

  Two categories are ruled out this way. **IaC Security** needs
  infrastructure to scan, and the detection is deliberately broad — a
  directory called `infra`, `k8s` or `charts` is enough to keep it.
  **Workflow Security** needs GitHub Actions, because every tool in it reads
  that format, so a GitLab-only repository is not told to add actionlint.

  The rest apply to any codebase, and the ones that arguably do not — load
  testing a library, an SBOM for something that ships nothing — cannot be told
  apart from the ones that do, so guessing would rule out categories a
  repository does need.
- **Baseline-accepted gaps still count as uncovered.** A baseline records that
  a gap is tolerated, not that it was closed. A score that rose because someone
  committed a file would measure nothing.

### README Badge

`--badge` writes the score as a [shields.io endpoint][shields] file:

```sh
zanadir scan --dir . --badge zanadir-badge.json
```

```json
{
  "schemaVersion": 1,
  "label": "zanadir",
  "message": "10/11",
  "color": "green"
}
```

Publish that file from CI and point a badge at it. Put it on a branch of its
own rather than committing it to the default branch: a protected default
branch only takes changes through a pull request, so a workflow that pushes
the badge there is rejected.

```markdown
![zanadir](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/OWNER/REPO/badges/zanadir-badge.json)
```

A job that keeps it current on every push to the default branch, publishing
to a `badges` branch it creates on the first run:

```yaml
badge:
  if: github.event_name == 'push'
  runs-on: ubuntu-latest
  permissions:
    contents: write
  steps:
    - uses: actions/checkout@v4

    - name: Run zanadir
      run: zanadir scan --dir . --badge zanadir-badge.json

    - name: Publish the badge when it moved
      run: |
        set -euo pipefail
        cp zanadir-badge.json "$RUNNER_TEMP/zanadir-badge.json"
        rm -f zanadir-badge.json
        git config user.name github-actions
        git config user.email github-actions@github.com
        if git fetch --depth=1 origin badges 2>/dev/null; then
          git switch --discard-changes -C badges FETCH_HEAD
        else
          git switch --orphan badges
        fi
        cp "$RUNNER_TEMP/zanadir-badge.json" zanadir-badge.json
        git add zanadir-badge.json
        if git diff --cached --quiet; then exit 0; fi
        git commit -m "chore: update zanadir badge"
        git push origin badges
```

The `badges` branch holds that one file and nothing else. If a ruleset
covers every branch rather than only the default one, exclude `badges` from
it.

The colour tracks the score: green at 80% or above, yellow at 60%, orange at
40%, red below that, and bright green only at full coverage.

[shields]: https://shields.io/badges/endpoint-badge

## Effectiveness

Presence is not protection. A scanner wired up so it can never fail a build
leaves a repository worse off than one with no scanner at all, because
everybody believes the control is there. All of these used to report
**covered**:

```yaml
- uses: gitleaks/gitleaks-action@v2
  continue-on-error: true          # the job stays green whatever it finds
```

```yaml
on:
  schedule: [{cron: "0 3 * * *"}]  # nothing is blocked at merge time
```

```yaml
on:
  push: {branches: [main]}         # pull requests are never checked
```

A scan now grades each control it finds and reports the ones that protect less
than their presence suggests:

```
3 covered categories are weakened:
  Linter is partial (.github/workflows/pushonly.yml)
  SCA is advisory (.github/workflows/nightly.yml)
  Secrets Detection is advisory (.github/workflows/ci.yml)
```

| Verdict | Meaning |
|---|---|
| `enforcing` | the control can fail the build on the path that gates a change |
| `partial` | it can fail a build, but not on that path - pushes only, say |
| `advisory` | it runs, and cannot block anything |
| `unknown` | the configuration cannot be read well enough to say |

Supply Chain is graded against a different path. An SBOM, a signature or a
provenance attestation describes what ships rather than looking for something
to block, so it belongs where the artifact is published: running on a release,
a push, a tag, a pull request or a workflow chained after one is `enforcing`.
A manual run may be the release itself, which the file cannot tell, so it is
`unknown`. Only a schedule, detached from anything that ships, is `advisory`,
and `continue-on-error` is `advisory` here as everywhere: a signing step that
can fail silently publishes unsigned artifacts.

A weakened control does not fail a scan unless you ask it to. `--enforce` and
`--fail-on` act only on categories with no tooling; `--fail-on-weakened` is
what turns a weak control into a failure:

```sh
zanadir scan --dir . --fail-on-weakened
```

```
Enforcement failed: controls that cannot fail a build: Secrets Detection (advisory)
```

The two are independent. `--enforce` on its own still ignores weakened
controls, so upgrading does not start failing builds that passed before.
`--fail-on` narrows both.

A baseline accepts a weakness **at the verdict it had when accepted**:

```yaml
version: 1
categories:
  - Performance Testing
weakened:
    Secrets Detection: partial
```

So accepting `partial` does not also accept it becoming `advisory` later. If
the control gets worse than its recorded verdict, the scan fails again.
`--write-baseline` records the verdicts for you.

Some of this is generic - `continue-on-error` defeats any control, and so does
a workflow that only runs on a schedule. The rest is specific to one tool, and
a rule can say so:

```yaml
  - id: "tfsec-rule"
    regex: "(?i)\\btfsec\\b"
    weakenedWhen:
      - runMatches: "--soft-fail\\b"
        verdict: "advisory"
      - input: "soft_fail"
        equals: "true"
        verdict: "advisory"
```

A check is either a regex over the step's shell command or one action input
compared to a value - deliberately not an expression language. Covered today:
`trivy --exit-code 0`, `gitleaks --exit-code 0`, `tfsec` and `checkov`
`--soft-fail` / `soft_fail: true`, and `anchore/scan-action` with
`fail-build: false`.

Two deliberate silences. An `if:` condition is never interpreted, because it
can reference arbitrary context and reading one wrong is worse than admitting
it cannot be read - the control is reported `unknown` instead. And a category
is judged by its strongest control, so one properly wired scanner keeps the
category quiet however many weak ones sit beside it.

### Controls declared outside the pipeline

Dependabot and Renovate are detected from their configuration -
`.github/dependabot.yml`, `renovate.json` and the other names Renovate
accepts - not from a CI job, because that is not where they live.

They are graded `advisory`. Both run on the forge's own schedule and open pull
requests; neither can fail a build. So a repository whose only dependency
control is Dependabot is reported as covered **and** weakened, which is a more
useful answer than either "you have no SCA" or "you are covered":

```
1 covered category is weakened:
  SCA is advisory (.github/dependabot.yml)
```

### Platform coverage

GitHub Actions is graded in full. GitLab CI is graded for the two signals that
are unambiguous there - `allow_failure: true`, which is `continue-on-error`
under another name, and `when: manual`, a job that only runs when somebody
asks.

GitLab's `include:` is followed. A local file (`local:` or a bare path,
wildcards allowed) is read as more of the same configuration. GitLab's own
templates and CI/CD components cannot be read from the repository, so they are
recognised by name: `Jobs/SAST.gitlab-ci.yml` or
`components/sast/sast@...` counts as SAST, and likewise for Secret Detection,
Dependency and Container Scanning, SAST-IaC, License Scanning, Code Quality and
the performance-testing templates. A `project:` or `remote:` include is
skipped. A job zanadir cannot read is skipped rather than failing the scan.

GitLab's `rules:`, `only:` and `except:` are deliberately not interpreted.
Whether a branch pipeline gates a merge depends on a project setting that is
not in the repository, so reading them would be guessing. A GitLab job with
neither weakening field is reported `unknown`, which says nothing rather than
something wrong. CircleCI has no equivalent signal and is not graded.

Bitbucket Pipelines splits its file by trigger, so most of it is graded:

| Where the tool runs | Verdict |
| --- | --- |
| `pull-requests:` | `enforcing` |
| `branches:` for one named branch, such as `main` | `partial` - it runs after the merge |
| `tags:`, `custom:`, or a step or stage with `trigger: manual` | `advisory` |
| `after-script:` | `advisory` - a failing after-script does not fail the step |
| `default:`, or a wildcard branch pattern | `unknown` |

`default:` also runs on a pull request's source branch, and whether that build
blocks the merge is a merge check configured outside the repository, so it is
left `unknown` for the same reason as GitLab's branch pipelines. Pipes are read
like GitHub actions: `pipe: snyk/snyk-scan` counts as Snyk, and
`DONT_BREAK_BUILD: "true"` grades it `advisory`.

Azure Pipelines is read from `azure-pipelines.yml` (or `.yaml`) at the
repository root, following `template:` and `extends:` references to files in
the same repository. A template in another repository (`path@alias`) or one
named by an expression cannot be read and is skipped.

| Where the tool runs | Verdict |
| --- | --- |
| a pipeline whose `pr:` names branches | `enforcing` |
| `continueOnError: true` on the step or its job | `advisory` |
| `trigger: none`, `pr: none` and only `schedules:` | `advisory` |
| any `condition:` other than `succeeded()` | `unknown` |
| a pipeline with no `pr:` | `unknown` |

With no `pr:`, an Azure Repos pull request is validated by a branch policy
outside the repository, so the file cannot say whether the pipeline gates it.
`succeeded()` is ignored because it is the default every step already has. A
step with `enabled: false` never runs and counts for nothing. Tasks are read
like actions, with `inputs:` as their inputs, so `SnykSecurityScan` with
`failOnIssues: false` grades `advisory`.

In SARIF a weakened control is a `note`, pointing at the file that produced
the verdict, while a missing category stays a `warning`.

## Language-Aware Suggestions

Zanadir detects which languages a repository uses and suggests only tools that
apply to it — a Go project is not offered ESLint, and a JavaScript project is
not offered Pylint. Tools that work regardless of ecosystem (Trivy, Gitleaks,
Codecov, and so on) are always suggested.

Detection is based on dependency manifests in the repository root and its
immediate subdirectories, so common monorepo layouts work:

| Language   | Detected from                                              |
|------------|------------------------------------------------------------|
| Go         | `go.mod`                                                   |
| Python     | `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile` |
| JavaScript | `package.json`                                             |
| Ruby       | `Gemfile`, `*.gemspec`                                     |
| Java       | `pom.xml`, `build.gradle`, `build.gradle.kts`              |
| Rust       | `Cargo.toml`                                               |
| PHP        | `composer.json`                               