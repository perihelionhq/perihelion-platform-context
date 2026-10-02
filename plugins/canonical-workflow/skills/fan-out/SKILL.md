---
name: fan-out
description: Route work that crosses repositories by opening a labelled `factory` issue in each repository that must follow an upstream change, such as a new perihelion-protos release. Use as the action for a tag `push` trigger in perihelion-protos, or when an issue's work belongs in other repositories.
---

# Fan out cross-repository work

A run only works in the repository whose event started it. When a change must land in other repositories, open one `factory` issue in each, and let their own `implement` runs do the work. The issues are the routing layer.

## Inputs

- The triggering event at `.cloud-launch/trigger-events.json`. For a release, require a `push` whose `payload.ref` starts `refs/tags/`, and read the tag from it.
- An environment whose GitHub access can open issues across the `perihelionhq` organisation. It is not the repository's own `factory` environment.

## Procedure

1. **Describe the change.** For a protos release, compare the tag with the previous tag and summarise what changed in `demo.proto`: added, changed and removed messages, fields and RPCs. Flag anything that breaks existing consumers.
2. **Find the consumers.** Search the organisation for repositories that vendor `demo.proto` (`gh search code --owner perihelionhq --filename demo.proto`). Skip this repository and any repository already at the tag.
3. **Check for existing issues.** In each consumer, look for an open issue whose title names the tag. Skip consumers that have one.
4. **Open the issues.** In each remaining consumer, open an issue titled `Bump vendored demo.proto to <tag>`. The body holds the summary from step 1, the parts of it that touch services this repository implements or calls, and a link to the release. Label it `factory`, which starts `implement` there.
5. **Report.** Comment on the triggering issue, or log for a tag push, a list of the issues opened and the consumers skipped with the reason.

## Do not

Do not change code in any repository, and never open a second issue for the same tag in the same repository.
