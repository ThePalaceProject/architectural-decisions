# Architectural Decision Records

This repository is used to collect decision records recording "architecturally significant" decisions made in the Palace Project: those that affect the structure, non-functional characteristics, dependencies, interfaces, or construction techniques.

## Workflow

ADRs are proposed and accepted using the github pull request system. ADRs go through the following states in their lifecycle: 

- *In development* - Developed in a draft PR.
- *Proposal* - Feedback is gathered on the pull request. Once the PR is approved, it can be merged by its author. 
- *Active* - An ADR that has been merged into the `main` branch.
- *Superceded* - A new ADR is accepted to replace the old one and the superceded ADR is linked to the new ADR.

A PR can be created to modify an existing ADR to clarify meaning and add to the description. However, if a decision is being reversed the existing ADR should be superceded and a new one created to replace it.

## Format 

### Markdown

Each ADR will be created in github markdown, using the template set out in [template.md](TEMPLATE.md).

### Naming

Each ADR is given a sequential and monotonically increasing number, `#`. Numbers will not be reused. Each ADR will use a filename in the format: `ADR-# Short description of ADR.md`.
