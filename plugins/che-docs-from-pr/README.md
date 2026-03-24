# che-docs-from-pr

A Claude Code plugin that generates [che-docs](https://github.com/eclipse-che/che-docs) AsciiDoc articles from GitHub pull request code changes.

## What it does

Given one or more PR links, the skill:

1. Fetches PR data (title, description, diff) using `gh` CLI
2. Analyzes the changes and suggests an article type (procedure, concept, reference, or assembly) and target module
3. Lets you confirm or override the suggestions
4. Generates a `.adoc` file following che-docs conventions (metadata, product variables, section IDs, Red Hat Style Guide)
5. Updates the module's `nav.adoc` with the new page

## Prerequisites

- [GitHub CLI (`gh`)](https://cli.github.com/) installed and authenticated
- A local clone of [eclipse-che/che-docs](https://github.com/eclipse-che/che-docs)
- Run Claude Code from inside the `che-docs` directory

## Installation

```
/plugin marketplace add tolusha/claude-plugins
/plugin install che-docs-from-pr@claude-plugins
```

## Usage

```
/che-docs-from-pr <PR_URL_1> [PR_URL_2] ...
```

### Examples

Single PR:
```
/che-docs-from-pr https://github.com/eclipse-che/che-server/pull/456
```

Multiple PRs (combined into one article):
```
/che-docs-from-pr https://github.com/eclipse-che/che-server/pull/456 https://github.com/eclipse-che/che-operator/pull/789
```

## License

[EPL-2.0](../../LICENSE)
