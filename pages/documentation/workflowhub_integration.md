---
title: WorkflowHub workflows table
datatable: true
---

The theme can display workflows in a searchable table. Workflow metadata is stored in `_data/data_workflows.yaml`, following the same data-file pattern as the [tools and resources table](resource_table).

The data file can be maintained manually. Sites that use a public WorkflowHub collection can optionally use the supplied GitHub Actions workflow to keep it up to date.

## Way of working

The `{% raw %}{% include workflow-table.html %}{% endraw %}` include reads the committed `_data/data_workflows.yaml` file. The file can be edited manually or generated from a WorkflowHub collection. Jekyll does not contact WorkflowHub when it builds the site.

## Prerequisites

* Use a recent version of the ELIXIR Toolkit Theme.
* Create `_data/data_workflows.yaml` manually or generate it from a WorkflowHub collection.
* Add `datatable: true` to the front matter of pages that render the table.

## The main YAML file

`_data/data_workflows.yaml` contains a top-level list of workflow records. Only `title` and `url` are required. The supported fields are:

* **`title`**: Display name of the workflow.
* **`url`**: Absolute WorkflowHub URL, or a path relative to the configured `workflowhub.base_url`.
* **`class`**: Workflow type, such as `nextflow`, `galaxy` or `snakemake`.
* **`edam_top`**: List of EDAM topics.
* **`edam_ops`**: List of EDAM operations.
* **`license`**: Workflow licence identifier.
* **`updated_at`**: Last update date in `YYYY-MM-DD` format.
* **`projects`**: List of contributing projects or teams.
* **`doi`**: DOI without the `https://doi.org/` prefix.
* **`remote_link`**: Link to the source repository.
* **`tools`**: Tool metadata supplied by WorkflowHub. This is retained in the data file but is not currently displayed by the table.

For example:

```yml
- title: Example workflow
  url: https://workflowhub.eu/workflows/123?version=1
  class: nextflow
  edam_top:
    - Sequence analysis
  edam_ops:
    - Sequence alignment
  license: MIT
  updated_at: '2026-01-31'
  projects:
    - Example project
  doi: 10.48546/workflowhub.workflow.123.1
  remote_link: https://github.com/example/example-workflow
  tools: ''
```

## Maintaining workflows manually

1. Create `_data/data_workflows.yaml` if it does not already exist.
2. Add or update records using the fields above. Use two spaces for YAML indentation.
3. Check that every record has a unique, non-empty `url` and a non-empty `title`.
4. Preview the site locally and commit the updated data file.

Manual data maintenance does not require `workflowhub.enabled` or a `collection_id`. When a record uses a relative `url`, the table resolves it against `workflowhub.base_url`, which defaults to `https://workflowhub.eu`.

## Listing workflows

Add `datatable: true` to the page front matter:

```yml
---
title: Workflows
datatable: true
---
```

Then add the table include where it should appear:

```liquid
{% raw %}{% include workflow-table.html %}{% endraw %}
```

This renders as:

{% include workflow-table.html %}

See the [workflows example page](workflows) for the rendered result.

## Generating the data manually

Instead of maintaining individual records, the supplied generator can retrieve all workflows in a public WorkflowHub collection. This requires a source checkout of the theme because maintenance scripts are not installed by the gem or remote theme.

First configure the collection in the website's `_config.yml`:

```yml
workflowhub:
  enabled: true
  collection_id: 40
  # base_url: https://workflowhub.eu
```

Replace `40` with the collection ID for the site. `base_url` is optional and defaults to `https://workflowhub.eu`.

Create a Python environment and install the generator dependencies:

```sh
python3 -m venv .venv
. .venv/bin/activate
python -m pip install "requests>=2.32,<3" "PyYAML>=6,<7"
```

From the root of the website repository, run the script from the theme checkout:

```sh
python /path/to/elixir-toolkit-theme/.github/actions/workflowhub/workflowhub_finder.py
```

The command writes `_data/data_workflows.yaml`. Review the resulting diff, preview the site and commit the data file. Re-run the command whenever the collection needs to be refreshed.

## Optional automated updates

The GitHub Actions workflow automates the same generation process and opens a pull request when the collection changes. It is optional; the table and local generator do not depend on it.

When it runs, the workflow:

1. Checks out the website's default branch.
2. Loads the generator from the same theme commit as the workflow.
3. Reads the website's WorkflowHub settings and generates `_data/data_workflows.yaml`.
4. Opens or updates a pull request if the data changed. It does not write directly to the default branch.

The workflow uses the repository's `GITHUB_TOKEN`; a separate WorkflowHub or personal access token is not required.

### Forks

Forks inherit `.github/workflows/workflowhub.yml` and the generator, so they do not need a caller workflow. Enable GitHub Actions and permission for Actions to create pull requests, then run **Update WorkflowHub data** from the Actions tab.

### Gem and remote-theme sites

Gem and remote-theme installations do not copy GitHub workflow files into the website repository. After adding the WorkflowHub configuration above, add this small caller workflow to `.github/workflows/` in the website repository:

```yml
name: Update WorkflowHub data

on:
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  update:
    uses: ELIXIR-Belgium/elixir-toolkit-theme/.github/workflows/workflowhub.yml@main
```

Pin the reusable workflow to the same released theme tag used by the website instead of `main` for a production site. A `schedule` trigger can also be added if regular refreshes are useful.

The repository must allow GitHub Actions to create pull requests under **Settings → Actions → General → Workflow permissions**. Run the caller manually from the Actions tab for the first update and review the resulting pull request before merging it.

The generator validates required values and duplicate URLs. It also refuses to replace an existing non-empty data file with an empty response. Once automation is enabled, avoid editing the generated file manually because the next update will overwrite those changes.
