# ADR-001 Github Workflow

| Property        | Value                                 |
| --              | --                                    |
| *Components*    | Backend, Web, iOS, Android            |
| *Related*       |                                       |
| *Superceded by* |                                       | 

## Context

*What is the issue that we're seeing that is motivating this decision or change?*

As we are growing the team that is working on the project and forking the project away from NYPL it is helpful to have a common 
branching and releasing strategy across the project. This will make it easier to jump into a project repo and know the workflow 
for submitting a fix or using a release.

As we are setting up a new GitHub organization for The Palace Project its a good time to agree on a workflow for developing code 
that will come into the project.

## Alternatives Considered

*Any alternatives considered any details about why they were set aside.*

- Keeping with the same git workflow that NYPL has been using.
    - Advantages: Easier to stay aligned with upstream since we would be using same strategy.
    - Disadvantages: The workflow isn't consistent and well defined across the project. Each repo has its own 
      branch names and release strategy.
- [Gitflow](https://nvie.com/posts/a-successful-git-branching-model/)
    - Advantages: Widely known and used. Extensions have been developed to help with management and it is supported on many 
      git tools.
    - Disadvantages: The author himself admits that this workflow was developed in a different time and may not be as relevant 
      today where most software isn't concurrently supporting many versions. Doesn't fit nicely into the github model.
- [One Flow](https://www.endoflineblog.com/oneflow-a-git-branching-model-and-workflow)
    - Advantages: It is flexible according to team decisions. More closely aligned to the current development model than git 
      flow.
    - Disadvantages: It isn’t recommended for projects with Continuous Delivery or Continuous Deployment. Doesn't take advantage 
      of Github features the project is using.

## Decision

*What is the change that we're proposing and/or doing? This may have several subsections giving more detail about the decision.*

We will use [Github Flow](https://guides.github.com/introduction/flow/) customized for our team and project as outlined in the 
following sections. Although there is a lot of text included below, this is to make sure we are on the same page and to help 
orient new developers on the project. This document should be treated as a guidance that can be ignored if a situation demands 
something different. The guidelines should work for us and help us to make progress, not against us.

### The `main` branch

We will have one eternal branch on our repositories called `main`. Anything in the main branch should always be treated as deployable 
and releasable. Where possible build artifacts will be created for every commit to the `main` branch. 

The `main` branch will use the following GitHub branch protection rules:

- Require pull request reviews before merging (at least one reviewer)
- Require status checks to pass before merging
- Require linear history

### New features / bug fixes

Feature branches (also sometimes called topic branches) are where the day-to-day development work happens. They are used to 
develop new features and bug fixes for the upcoming release. Your branch name should be descriptive (e.g., `refactor-authentication`, 
`user-content-cache-key`, `make-retina-avatars`), so that others can see what is being worked on. 

#### Starting a feature branch

To start a feature branch, simply create a new branch from `main`:

```bash
$ git checkout -b refactor-authentication main
```

#### Open a Pull Request

Pull Requests initiate discussion about your commits. Because they're tightly integrated with the underlying Git repository, anyone 
can see exactly what changes would be merged if they accept your request.

You can open a Pull Request at any point during the development process. You can open a draft pull request when you have little or 
no code but want to share some screenshots or general ideas, when you're stuck and need help or advice, or you can open a regular 
pull request when you're ready for someone to review your work. 

#### Discuss and review your code

Once a Pull Request has been opened, the person or team reviewing your changes may have questions or comments. Perhaps the coding 
style doesn't match project guidelines, the change is missing unit tests, or maybe everything looks great and props are in order. 

#### Deploy

You can deploy from a branch for final testing in before merging to main. How this testing is done will depend on the particular 
repository, however where possible CI will build artifacts from feature branches, so that they can easily be deployed and tested 
before merging to `main`.

#### Merge

Several conditions should be met before code in a pull request is merged back into the `main` branch. Some of these conditions 
are enforced by branch protection on the main branch.

1. The code should be automatically tested and all the CI status checks passed
2. The code should be reviewed and approved by another developer on the project
3. When possible the code should be deployed to either a production or testing environment for final testing before merging

Finally, after these conditions are met, the code can be merged into `main` by any member of the team. As a general guideline, the PR 
author will be the one to do the merge. The code will be merged using the Github "squash merge" feature and the commit message written 
to document the changes made and the reasons why.

## Squash Merge

Squash merge allows us to have a clear and concise git history that clearly and easily documents the changes made and the reasons why. 
This concise history can be used to easily put together release notes for each release of the project.

Squash merges allow the developers of a pull request to commit early and commit often, and push their commits to the pull request 
regularly, keeping track of work in chunks of progress that make sense to during development. This often includes commits that are 
"wip" (work-in-progress) or "part A done" or "typo, minor fix". These make sense to the author of the pull request, but give little 
context to the work done when merged into the final main branch git history.

Using squash merges allows us to have the best of both worlds. Code is developed and pushed using many small commits and then 
contextualized into the overall project history by the person merging the pull request with focus on clarity in the final commit 
message of the changes done and the reasons why.

## Releases

Releases are created with version numbers reflecting [semantic versioning](https://semver.org/). These releases are created on using 
the Github releases tool. A change log is created that describes the bug fixes and features in the release and a tag created off of 
the `main` branch. CI will be setup where possible to build artifacts from the releases and upload them to the appropriate repository.

# Consequences

What becomes easier or more difficult to do because of this change?

Adopting this workflow across the project will make it easier to onboard new committers, either from the broader community that we hope 
to build around the project, or as new staff members or contractors. It will make it easier to jump into a project repo and know the 
workflow for submitting a fix or using a release.

This workflow will move us further away from what the upstream NYPL project is doing, since they have their own conventions and 
workflow. This may make it harder to pull in changes that have been made by NYPL into this project, or vice versa.
