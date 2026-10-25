# Hurahopi agent instructions

This repository is currently a planning workspace. Read [README.md](README.md) for status and document ownership before proposing or implementing changes. Extend that one README when setup changes; keep specifications in their topic documents.

## Project skill

For Hurahopi product, UX, schema, API, or application work, read [skills/hurahopi/SKILL.md](skills/hurahopi/SKILL.md). Read only the referenced specifications relevant to the task. Current user instructions and authorization take precedence over local skill guidance; do not reopen decisions already made in this conversation.

## Design direction

[docs/desain.md](docs/desain.md) is the project's design direction. References to `DESIGN.md` in third-party skills mean this file; do not create a second copy. Screen behavior belongs to [docs/ux-spec.md](docs/ux-spec.md), and the database specification is [docs/database/design.md](docs/database/design.md).

The contrast target is at least 4.5:1 for all product text. Use the W3C interpretation linked in desain.md; the upstream shorthand of 18px for large text must not weaken this target.

<!-- antislop:start -->
## antislop

Anti-slop is installed locally under `skills/`, not as a global plugin. For interface, copy, accessibility, responsive layout, or code-comment work, read [the core](skills/antislop/SKILL.md), then load only the relevant modules:

- UI and visual composition: [antislop-ui](skills/antislop-ui/SKILL.md).
- Product language and prose: [antislop-copywriting](skills/antislop-copywriting/SKILL.md).
- Accessibility: [antislop-human](skills/antislop-human/SKILL.md).
- Responsive layouts: [antislop-layoutmobile](skills/antislop-layoutmobile/SKILL.md).
- Code comments: [antislop-code](skills/antislop-code/SKILL.md).

When upstream refers to `antislop.md`, load `skills/antislop/SKILL.md`. The installed source and hashes are recorded in [source.json](skills/antislop/source.json); preserve [the MIT license](skills/antislop/LICENSE).

Resolve during/after using the core's mode rules. The user selected **during for the conversation installing these skills**; do not ask again while that conversation continues. This does not establish a global preference or select a mode for unrelated future conversations. Do not write global settings without an explicit request to remember the choice.

Use the contrast formula/reference table in antislop-human; standalone documentation/contrast scripts are not part of this project. Use the existing Hopi direction and supplied references; generating new assets remains a separate task. Report the delivery gate against what was actually delivered. For documentation, validate documents and token contrast; browser/build/click-through evidence is required when delivering an implemented UI. Do not label unbuilt interface behavior as tested.
<!-- antislop:end -->
