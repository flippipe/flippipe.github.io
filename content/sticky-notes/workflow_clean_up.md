---
title: "GitHub Actions Clean Up"
date: 2022-07-15T13:56:25+01:00
summary: "How to delete a workflow from GitHub Actions"
description: "How to delete a workflow from GitHub Actions"
tags: ["github"]
draft: false
---

There is no simple button to push, and clean all stuff in a workflow. 

So we need to clear all individual workflow runs, then the workflow will disappear. 😿

To delete all runs in a workflow, [Samuel Müller](https://stackoverflow.com/users/727795/samuel-m%c3%bcller) gave us a clean [script](https://stackoverflow.com/a/65539398) to do it:

``` sh
OWNER=<your user/org name>
REPO=<repo name>

# list workflows
gh api -X GET /repos/$OWNER/$REPO/actions/workflows | jq '.workflows[] | .name,.id'

# copy the ID of the workflow you want to clear and set it
WORKFLOW_ID=<workflow id>

# list runs
gh api -X GET /repos/$OWNER/$REPO/actions/workflows/$WORKFLOW_ID/runs | jq '.workflow_runs[] | .id'

# delete runs (you'll have to run this multiple times if there's many because of pagination)
gh api -X GET /repos/$OWNER/$REPO/actions/workflows/$WORKFLOW_ID/runs | jq '.workflow_runs[] | .id' | xargs -I{} gh api -X DELETE /repos/$OWNER/$REPO/actions/runs/{}

```

It dependes on [GitHub CLI](https://cli.github.com/manual/), which is available [here](https://github.com/cli/cli/releases/latest).