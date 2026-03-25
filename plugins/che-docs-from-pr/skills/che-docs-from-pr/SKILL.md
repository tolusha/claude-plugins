---
name: che-docs-from-pr
description: Generate che-docs AsciiDoc articles from GitHub PR code changes. Analyzes one or more PRs, suggests article type and target module, then creates the .adoc file and updates navigation.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# che-docs-from-pr

Generate an AsciiDoc article for the `eclipse-che/che-docs` repository based on one or more GitHub pull request URLs.

**Important:** You are running inside a cloned `eclipse-che/che-docs` repository. All file paths are relative to this repo root.

## Workflow

Follow these steps in order. Do not skip steps.

---

### Step 1: Fetch PR Data

For **each** PR URL provided by the user:

1. Run:
   ```
   gh pr view <URL> --json title,body,files,labels,state
   ```
2. Run:
   ```
   gh pr diff <URL>
   ```
3. Extract and record:
   - PR title
   - PR description/body
   - List of changed files (with paths)
   - Full diff content
   - Labels and state

4. After processing all PRs, build a **combined summary** of all changes. Group related changes together. Identify:
   - What component or feature is affected
   - What kind of change it is (new feature, configuration change, API update, behavioral change, bug fix, deprecation, etc.)
   - Key technical details from the diff (new flags, config options, API fields, environment variables, CLI arguments, etc.)

---

### Step 2: Analyze and Suggest

Based on the combined summary from Step 1, determine the best article to write.

#### 2a. Determine article type

Choose one:

| Type | When to use | Examples |
|------|-------------|----------|
| **Procedure** | New features or changed workflows that users need to follow step-by-step | "Configuring X", "Setting up Y", "Using Z" |
| **Concept** | Architectural or behavioral changes that need explanation | "Understanding X", "How Y works" |
| **Reference** | New config options, API fields, CLI flags, environment variables | Tables of options, parameter lists |
| **Assembly** | Large features that span multiple topics | Combines multiple includes into one page |

#### 2b. Determine target module

List available modules by running:
```
ls modules/
```

Common modules:
- `end-user-guide` — for features users interact with directly
- `administration-guide` — for admin/operator configuration
- `overview` — for high-level concepts and architecture

Choose the module that best fits the content.

#### 2c. Suggest a filename

- Use lowercase with dashes: `configuring-workspace-idle-timeout.adoc`
- Use a verb prefix matching the article type:
  - Procedure: `configuring-*`, `setting-up-*`, `using-*`, `creating-*`
  - Concept: `understanding-*`, or a noun phrase like `workspace-idle-timeout`
  - Reference: noun phrase like `devfile-reference`, `cli-flags`
  - Assembly: noun phrase describing the feature area

#### 2d. Present suggestions to the user

Use `AskUserQuestion` to present your suggestions. Format the question exactly like this:

```
Based on my analysis of the PR(s), here are my suggestions:

**PR Summary:** <one-paragraph summary of what the PR(s) change>

**Article type:** <Procedure | Concept | Reference | Assembly>
**Rationale:** <why this type fits>

**Target module:** <module name>
**Filename:** <suggested-filename.adoc>
**Article title:** <Suggested Title>

Do you agree with these suggestions? You can:
- Confirm as-is
- Change the article type, module, filename, or title
- Provide additional context about what the article should cover
```

---

### Step 3: User Confirms

Wait for the user's response. They may:
- Confirm everything as-is
- Override any of: article type, target module, filename, article title
- Provide additional context or instructions

Apply any overrides before proceeding.

---

### Step 4: Generate the Article

#### 4a. Read the template

Read the matching template from the che-docs repo:
```
templates/concept.adoc
templates/procedure.adoc
templates/reference.adoc
templates/assembly.adoc
```

Use `Read` to get the template content for the chosen article type. If the template file does not exist, use the format rules below as a fallback.

#### 4b. Read product variables

Read the `antora.yml` file at the repo root to discover available product attributes. Common ones include:
- `{prod}` — full product name
- `{prod-short}` — short product name
- `{prod-id-short}` — product ID (for URLs, IDs)
- `{prod-url}` — product URL
- `{orch-name}` — orchestrator name (e.g., Kubernetes)
- `{ocp}` — OpenShift Container Platform
- `{platforms-name}` — platform name
- `{prod-checluster}` — CheCluster CR name

**Critical rule:** NEVER hardcode product names, platform names, or orchestrator names. Always use the attribute variables from `antora.yml`. For example, write `{prod-short}` instead of "Che" or "Eclipse Che".

#### 4c. Study existing articles

Scan 2-3 existing articles in the target module to match the writing style:
```
ls modules/<module>/pages/
```
Then read 2-3 representative files to understand:
- How they structure content
- How they use metadata headers
- How they format sections, lists, code blocks
- How they reference other pages with `xref:`

#### 4d. Write the article

Generate the `.adoc` file following this structure:

```adoc
:description: <A concise description derived from the PR summary>
:keywords: <comma-separated relevant keywords>
:navtitle: <Short navigation title>
:page-aliases:

[id="<filename-without-adoc-extension>"]
= <Article Title>

<body content>
```

**Content rules:**

1. **Metadata headers** — Always include `:description:`, `:keywords:`, `:navtitle:`, and `:page-aliases:` (leave `:page-aliases:` empty for new articles).

2. **Section ID** — The first `[id="..."]` must match the filename without the `.adoc` extension. All subsequent section IDs must also use the `[id="section-name"]` format.

3. **Product variables** — Use attribute references from `antora.yml`. Never write literal product names.

4. **Content by article type:**

   - **Procedure:**
     ```adoc
     [id="<filename-without-extension>"]
     = <Title>

     <Introductory sentence explaining what this procedure achieves.>

     .Prerequisites
     * <prerequisite 1>
     * <prerequisite 2>

     .Procedure
     . <Step 1 — use imperative mood: "Configure", "Create", "Run">
     +
     <optional additional detail, code blocks, etc.>
     . <Step 2>

     .Verification
     * <How to verify the procedure worked>
     ```
     **Note:** Only include the `.Verification` section if there is a meaningful, practical way for the user to verify the result. If the change is purely declarative (e.g., patching a CheCluster CR field) and there is no observable outcome to check beyond "the command succeeded", omit the `.Verification` section entirely.

   - **Concept:**
     ```adoc
     [id="<filename-without-extension>"]
     = <Title>

     <Explanatory content. Describe what it is, why it matters, how it works.>

     == <Subsection>

     <More detail>
     ```

   - **Reference:**
     ```adoc
     [id="<filename-without-extension>"]
     = <Title>

     <Brief intro sentence.>

     [cols="1,1,2", options="header"]
     |===
     | Parameter | Default | Description
     | `<param>` | `<default>` | <description>
     |===
     ```

   - **Assembly:**
     ```adoc
     [id="<filename-without-extension>"]
     = <Title>

     <Introductory paragraph.>

     include::partial$<included-file>.adoc[]
     ```

5. **Cross-references** — When referring to other existing che-docs pages, use `xref:<page>.adoc[]` syntax. Search existing pages to find relevant ones:
   ```
   grep -r "<keyword>" modules/*/pages/ --include="*.adoc" -l
   ```

6. **Code blocks** — Use AsciiDoc source blocks:
   ```adoc
   [source,yaml]
   ----
   key: value
   ----
   ```

7. **Admonitions** — Use standard AsciiDoc admonitions:
   ```adoc
   NOTE: <text>
   TIP: <text>
   IMPORTANT: <text>
   WARNING: <text>
   ```

8. **CheCluster changes** — When the PR modifies CheCluster CR fields, do NOT print the full CheCluster YAML. Instead, provide an `oc patch` / `kubectl patch` command that applies only the relevant change. For example:
   ```adoc
   [source,shell,subs="+quotes,+attributes,+macros"]
   ----
   {orch-cli} patch checluster {prod-checluster} \
     --namespace {prod-namespace} \
     --type merge \
     --patch '{
       "spec": {
         "components": {
           "cheServer": {
             "extraProperties": {
               "CHE_LIMITS_WORKSPACE_IDLE_TIMEOUT": "1800000"
             }
           }
         }
       }
     }'
   ----
   ```
   Always use `{orch-cli}` instead of literal `oc` or `kubectl`.

9. **Cluster access prerequisite** — When a procedure or other article requires access to the cluster (e.g., running `{orch-cli}` commands, patching CheCluster, creating resources), add the following to the `.Prerequisites` section:
   ```adoc
   * An active `{orch-cli}` session with administrative permissions to the destination {orch-name} cluster. See {orch-cli-link}.
   ```

10. **TODO comments** — Where information from the PR is insufficient or ambiguous, insert:
   ```adoc
   // TODO: <describe what needs to be clarified or added>
   ```

11. **Writing style:**
   - Follow the Red Hat Style Guide and Modular Documentation Initiative
   - Use active voice ("Configure the timeout" not "The timeout is configured")
   - Be concise and direct
   - Do not invent information beyond what the PR changes show
   - Write for the target audience of the module (end users vs. administrators)
   - Use second person ("you") when addressing the reader

#### 4e. Save the article

Write the file to:
```
modules/<module>/pages/<filename>.adoc
```

---

### Step 5: Update Navigation

1. Read the navigation file:
   ```
   modules/<module>/nav.adoc
   ```

2. Determine the best position for the new entry. Look at the existing structure and place the new `xref:` near related topics.

3. Add the entry:
   ```adoc
   * xref:<filename>.adoc[]
   ```

4. Save the updated `nav.adoc`.

If the navigation file has a nested structure, place the entry at the appropriate nesting level. If unsure, add it at the end of the most relevant section.

---

### Step 6: Build and Preview

Run the preview script to build the docs and verify the new article renders correctly:

```
tools/runnerpreview.sh
```

Check the build output for errors. If the build fails:
- Fix any AsciiDoc syntax errors in the generated article
- Fix any broken `xref:` references
- Re-run `tools/runnerpreview.sh` until the build succeeds

Once the build succeeds, the preview will be available locally. Inform the user so they can review the rendered output before proceeding.

---

### Step 7: Create a PR against che-docs

After completing all steps, commit the changes and create a PR against `eclipse-che/che-docs`.

#### 7a. Determine PR title prefix

Choose the correct prefix based on the article type:

| Article type | PR title prefix | Why |
|---|---|---|
| **Procedure** | `procedures:` | Includes procedures — engineering and QE review mandatory |
| **Concept** | `docs:` | Documentation without procedures — engineering review mandatory |
| **Reference** | `docs:` | Documentation without procedures — engineering review mandatory |
| **Assembly** | `docs:` or `procedures:` | Use `procedures:` if it contains procedures, otherwise `docs:` |

#### 7b. Format the PR body

Use the che-docs PR template format. The PR body **must** follow this structure:

```
## What does this pull request change?

<Describe the article created and what documentation it adds. Reference the source PR(s) that prompted this change.>

## What issues does this pull request fix or reference?

<List the source PR URLs. If they reference issues, list those too. Use "N/A" if none.>

## Specify the version of the product this pull request applies to

<Specify the version, or "next" if it applies to the upcoming release.>

## Pull Request checklist

The author and the reviewers validate the content of this pull request with the following checklist, in addition to the [automated tests](code_review_checklist.adoc).

- Any procedure:
  - [ ] Successfully tested.
- Any page or link rename:
  - [ ] The page contains a redirection for the previous URL.
  - Propagate the URL change in:
    - [ ] Dashboard [default branding data](https://github.com/eclipse-che/che-dashboard/blob/main/packages/dashboard-frontend/src/services/bootstrap/branding.constant.ts)
- [ ] Builds on [Eclipse Che hosted by Red Hat](https://workspaces.openshift.com).
- [ ] the *`Validate language on files added or modified`* step reports no vale warnings.
```

#### 7c. Create the PR

Commit and push the changes, then create the PR:
```
gh pr create --repo eclipse-che/che-docs --title "<prefix> <Short description>" --body "<formatted body from 7b>"
```

#### 7d. Print summary

```
## Done

**Article created:** `modules/<module>/pages/<filename>.adoc`
**Navigation updated:** `modules/<module>/nav.adoc`
**PR created:** <PR URL>

### Next steps
1. Review the generated article and address any `// TODO:` comments
2. Wait for CI checks and address any vale warnings
```

---

## Error Handling

- If `gh` is not installed or not authenticated, tell the user to install and authenticate the GitHub CLI first.
- If the PR URL is invalid or the PR cannot be fetched, report the error and ask the user to verify the URL.
- If a template file is missing from `templates/`, proceed using the format rules in Step 4d as a fallback.
- If `antora.yml` is missing, warn the user but proceed — use `{prod}` and `{prod-short}` as defaults and add a `// TODO: verify product variables` comment.
- If the target module directory does not exist, ask the user to pick from the available modules.
