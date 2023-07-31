# policy-as-code

This Action was designed to allow users to configure their Risk threshold for security issues reported by [GitHub Code Scanning](https://docs.github.com/en/code-security/secure-coding/automatically-scanning-your-code-for-vulnerabilities-and-errors/about-code-scanning), [Secret Scanning](https://docs.github.com/en/code-security/secret-security/about-secret-scanning) and [Dependabot Security](https://docs.github.com/en/code-security/supply-chain-security/managing-vulnerabilities-in-your-projects-dependencies/configuring-dependabot-security-updates#about-configuring-dependabot-security-updates).

## Capability Demonstration

https://user-images.githubusercontent.com/2083085/131956624-e3f5140a-40e6-4067-9377-1093775aaa01.mp4

## Setup

### Action

Here is how you can quickly setup policy-as-code.

```yaml
# Policy as Code
- name: Advance Security Policy as Code
  uses: advanced-security/policy-as-code@v2.4.1
```

#### Action Examples

- [General Security](examples/workflows/security.yml)
- [Full Example with Details](examples/workflows/full.yml)
- [Licensing Compliance](examples/workflows/licensing.yml)
- [Policy as Code Compliance](examples/workflows/licensing.yml)

### CLI

The CLI tool primarily using pipenv to manage dependencies and pip virtual environments to not mismatch dependencies.

```bash
# Install dependencies and virtual environment
pipenv install
# [option] Install system wide
pipenv install --system
```

Once installed, you can just call the module using the following command(s):

```bash
# Using pipenv script
pipenv run main --help
# ... or
pipenv run python -m ghascompliance
```

#### CLI Examples

- [Code Scanning](examples/scripts/codescanning.sh)
- [Dependencies](examples/scripts/dependencies.sh)
- [Policies](examples/scripts/policies.sh)

## Features

| Feature         | github.com (Br/PR)                      | enterprise server (Br/PR)               | Description                                                                                                |
| :-------------- | :-------------------------------------- | :-------------------------------------- | :--------------------------------------------------------------------------------------------------------- |
| Code Scanning   | :white_check_mark: / :white_check_mark: | :white_check_mark: / :white_check_mark: | Code Scanning is a code analysis tool that scans your code for security vulnerabilities and coding errors. |
| Secret Scanning | :white_check_mark: / :white_check_mark: | :white_check_mark: / :white_check_mark: | Secret Scanning is a code analysis tool that scans your code for secrets.                                  |
| Dependabot      | :white_check_mark: / :white_check_mark: | :x: [2] / :white_check_mark: [3]        | Dependabot is a tool that automates discovery and remediation of vulnerabilities in your dependencies.     |
| Dependencies    | :white_check_mark: / :white_check_mark: | :x: [2] / :white_check_mark: [3]        | Dependencies is a tool that scans your code for dependencies.                                              |
| Licensing       | :white_check_mark: / :white_check_mark: | :x: [2] / :white_check_mark: [3]        | Licensing is a tool that scans your code for licensing issues.                                             |

_Notes:_

1. Br/PR = Branches / Pull Requests
2. :warning: GraphQL API not supported by GitHub Enterprise Server as of `3.7`
3. :white_check_mark: Supported as of [GHES `3.6`](https://docs.github.com/en/enterprise-server@3.6/rest/dependency-graph/dependency-review#get-a-diff-of-the-dependencies-between-commits)

## Policy as Code / PaC

Here is an example of using a simple yet cross-organization using Policy as Code:

```yaml
# Compliance
- name: Advance Security Policy as Code
  uses: advanced-security/policy-as-code@v2.4.1
  with:
    # The owner/repo of where the policy is stored
    policy: GeekMasher/security-queries
    # The local (within the workspace) or repository
    policy-path: policies/default.yml
    # The branch you want to target
    policy-branch: main
```

### PaC Configuration file

The Policy as Code configuration file is very simple yet powerful allowing a user to define 4 types of rules per technologies you want to use.

```yaml
# This is the technology you want to write a rule for
licensing:
  # The four main rules types to do everything you need to do for all things
  #  compliance

  # Warnings will always occur if the rule applies and continues executing to
  #  other rules.
  warnings:
    ids:
      - Other
      - NA
  # Ignores are run next so if an ignored rule is hit that matches the level,
  #  it will be skipped
  ignores:
    ids:
      - MIT License
  # Conditions will only trigger and raise an error when an exact match is hit
  conditions:
    ids:
      - GPL-2.0
    names:
      - tunnel-agent

  # The simplest and ultimate rule which checks the severity of the alert and
  #  reports an issue if the level matches or higher (see PaC Levels for more info)
  level: error
```

#### PaC Levels

There are many different levels of severities with the addition of `all` and `none` (self explanatory).
When a level is selected like for example `error`, all higher level severities (`critical` and `high` in this example) will also be added.

```yml
- critical
- high
- error
- medium
- moderate
- low
- warning
- notes
```

#### PaC Rule Blocks

For each rule you can choose either or both of the two different criteria's matches; `ids` and `names`

You can also use `imports` to side load data from other files to supplement the data already in the rule block

```yaml
codescanning:
  conditions:
    # When the `ids` of the technologies/tool alert matches any one of the ID's in
    #  the list specified, the rule will the triggered and report the alert.
    ids:
      # In this example case, the CodeQL rule ID below will always be reported if
      #  present event if the severity is low or even note.
      - js/sql-injection

      # Side note: Check to see what different tools consider id's verses names,
      #  for example `licensing` considers the "Licence" name itself as the id
      #  while the name of the package/library as the "name"

    # `names` allows you to specify the names of alerts or packages.
    names:
      - "Missing rate limiting"

    # The `imports` allows you to supplement your existing data with a list
    #  from a file on the system.
    imports:
      ids: "path/to/ids/supplement/file.txt"
      names: "path/to/names/supplement/file.txt"
```

#### Wildcards

For both types of criteria matching you can use wildcards to easily match requirements in a quicker way.
The matching is done using a Unix shell-style wildcards module called [fnmatch](https://docs.python.org/3/library/fnmatch.html) which supports `*` for matching everything.

```yaml
codescanning:
  conditions:
    ids:
      - "*/sql-injection"
```

#### Time to Remediate

The feature allows a user to define a time frame to which a security alert/vulnerability of a certain severity has before the alert triggered a violation in the Action.

By default, if this section is not defined in any part of the policy then no checks are done.
Existing policy files should act the same without the new section.

```yaml
general:
  # All other blocks will be inheriting the remediate section if they don't have
  #  their own defined.
  remediate:
    # Only `error`'s and above have got 7 days to remediate according to the
    #  policy. Any time before that, nothing will occur and post the remediation
    #  time frame the alert will be raised.
    error: 7

codescanning:
  # the `codescanning` block will inherit the `general` block
  # ...

dependabot:
  remediate:
    # high and critical security issues
    high: 7
    # moderate security issues
    moderate: 30
    # all other security issues
    all: 90

secretscanning:
  remediate:
    # All secrets by default are set to 'critical' severity so only `critical`
    #  or `all` will work
    critical: 7
```

##### Time to Remediate Examples

- [Time to Remediate Example](examples/policies/time-to-remediate.yml)

#### Data Importing

Some things to consider when using imports:

- Imports appending to existing lists and do not replace a previously generated list.
- Imports are relative to:
  - `Working Directory`
  - `GitHub Action / CLI directory`
  - `Cloned Repository Directory`
- Imports are only allowed from a number of predefined paths to prevent loading data on the system (AKA, path traversal).
