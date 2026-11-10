---
title: Workflows
datatable: true
---

This page demonstrates a searchable table that is populated from `_data/data_workflows.yaml`. See the [WorkflowHub workflows table documentation](workflowhub_integration) for manual and automated ways to maintain this file.

## Listing workflows

Use this snippet:

```liquid
{% raw %}{% include workflow-table.html %}{% endraw %}
```

This renders as:

{% include workflow-table.html %}
