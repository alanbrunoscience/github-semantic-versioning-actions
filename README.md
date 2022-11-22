# Action Responsible for Creating a Semantic Version Through Conventional Commit Messages

This action is responsible for generating a semantic version from a Git repository commit history and also is used the **[Paul Hatch/semantic-version](https://github.com/PaulHatch/semantic-version)** action to auxiliary in this process.

<br>

## Outputs

In this action, there is only one output, called **`version`** and this value will be the output of the step **"Set version as output"**:

<br>

```yaml
outputs:
  version:
    description: "Version of the workflow or action"
    value: ${{ steps.generate_version_output.outputs.version }}
```

<br>

## What does this action do?

This composite action runs three steps:

<br>

### 1) Checkout

In this step will be used the **`checkout`** action to allow the workflow access to the contents of the repository, since this action checks out the repository under **`$GITHUB_WORKSPACE`**, so the workflow can access it. Also, only a single commit is fetched by default, for the **ref/SHA** that triggered the workflow. Thus, it's necessary to set **`fetch-depth: 0`** to fetch all history for all branches and tags:

<br>

```yaml
- name: Checkout
  uses: actions/checkout@v3
  with:
    # Fetch all commits
    fetch-depth: 0
```

<br>

### 2) Semantic versioning

In order to generate a semantic version from a Git repository commit history is used the **[Paul Hatch/semantic-version](https://github.com/PaulHatch/semantic-version)** action. So, to update the **Major**, **Minor**, and **Patch**, the messages of the commits must have the following structure, respectively:

<br>

```yaml
major_pattern: "break:"
minor_pattern: "feat:"
patch_pattern: "fix:"
```

<br>

Any message that doesn't start with these words will **only increment the Patch**. The string to determine the version output format consists of **`format: "${major}.${minor}.${patch}"`**:

<br>

```yaml
- name: Semantic versioning
  id: versioning
  uses: paulhatch/semantic-version@v4.0.2
  with:
    tag_prefix: "v"
    major_pattern: "break:"
    minor_pattern: "feat:"
    patch_pattern: "fix:"
    format: "${major}.${minor}.${patch}"
```

<br>

### 3) Set version as output

After performing the above step, the output of the **Semantic Version** step will be exported as an output of this step called **`version`**. Also, an **`ID`** has been placed (in which its value will be **`generate_version_output`**) to reference this step:

<br>

```yaml
- name: Set version as output
  id: generate_version_output
  run: echo "::set-output name=version::v${{ steps.versioning.outputs.version }}"
  shell: bash
```

<br>

## Dependencies and Requirements

It was necessary to make this repository accessible from other repositories in the organization through **Settings > Actions > General > Access > Accessible from repositories in the organization 'Name_Organization'**.
