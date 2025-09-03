---
title: Gitea Actions, a buggy mess
tags:
    - CI/CD
    - DevOps
    - Gitea
    - GitHub Actions
    - Workflows
date: 2025-09-03
slug: gitea-actions-a-buggy-mess
image: thumbnail.jpg
keywords:
    - Gitea
    - Actions
    - bugs
    - renovate
description: Gitea Actions look good on the surface till you actually try to use it in production.
---

My team uses Jenkins for CI/CD, and we want to migrate away from it due to its insanely complex and hard to understand scripts.

We use Gitea as our git hosting platform, the direct CI/CD offer for it is Gitea Actions, it's already there, integrated into Gitea and gives us access to the huge community Ecosystem around GitHub Actions.

Gitea says everything works apart from a few limitations, then everything else should work, right?

https://docs.gitea.com/usage/actions/comparison

WRONG!!!!

## Private reusable workflows

In GitHub Actions you can use reusable workflows from private repos, as long as the GitHub token has access to the repo.

Gitea Actions don't have such a token, but you can just add a PAT to the actions URL for this to work, right?

Wrong, it just doesn't work no matter what, as the YAML parser doesn't allow anything other than a specific syntax.

This also affects limited Orgs (Organizations that are only visible to authenticated users). You would think the runner is authenticated, but apparently not.

https://github.com/go-gitea/gitea/issues/25929

Reported in 2023, still broken.

Sure no biggie, we can just create a dedicated public organization for the workflows.

## Reusable workflow steps are all shown as one step

Gitea parses the steps through the workflow file, and then sorts the logs under each step.
When you are using a reusable workflow, Gitea only parses one step, and all logs are shown under it.

This makes debugging a nightmare when workflows fail.

https://github.com/go-gitea/gitea/issues/26187
Reported in 2023, still broken.

Well, we just have to get used to the mangled logs, maybe just add an emoji to each step, and then search for it in the browser to find the step logs.

## Actions are only pulled once

When using a major tag for an action like v4, the latest release belonging to that version will be pulled, including patches and minor releases.

Guess what? That doesn't work in the Gitea Actions act_runner, since the act_runner has a runtime cache, and the runner won't check for updates after the first pull.

This is an even bigger problem for reusable workflows, where you might prefer to use a branch instead of tags, and just assume the runner will use the latest commit on each run.

https://gitea.com/gitea/act_runner/issues/726

Simple fix, just tag the actions to the last patch release, and run Renovate to automatically update the tag used, that surely gotta work?

Only for actions hosted on GitHub, not on Gitea.

### Renovate can't update workflow tags in Gitea

https://github.com/renovatebot/renovate/discussions/27734

Renovate still looks for packages in GitHub and not Gitea for reusable workflows and actions.

Meaning auto updates for Gitea-hosted actions/workflows aren't possible.

Yeah, I've had enough, Gitea ain't it then.

## Conclusion

Gitea Actions might work for simple workflows that are only used in one repo, don't need concurrency limits, don't need to get updated regularly, and don't rely on anything non-public.

In addition to these crazy bugs, dealing with HTTP proxies in Gitea Actions is just insane. The NOPROXY proxy exception env variable is very finicky where it's just ignored most of the time, but I think it's more of an Actions problem than a Gitea Actions problem.

After hitting so many bugs that were reported years ago! I just can't continue to invest time in Gitea Actions. Who knows what issues will show up when these workflows become part of the team's development process? It seems like GitLab is really the way to go.