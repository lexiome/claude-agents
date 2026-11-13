# Claude Code agents team

A set of Claude Code role agents, skills and a CLAUDE.md template, used to build a health-records app as a solo founder. One agent, one job, written down.

**Honest note:** this was built for a solo health-tech startup. The domain rules (privacy, clinical safety, "not a doctor") are tuned for that. Adapt them to your own product, and delete the roles you do not need. Start with three: `cto`, `developer`, `code-reviewer`.

Full guide: https://lexiome.ai/build/agents?s=github

## How to use it

1. Copy `.claude/agents/` and `.claude/skills/` (and optionally `.claude/commands/`) into your repo.
2. Copy `CLAUDE.md` to your repo root and fill in the `<placeholders>`. Search all files for `<your-app>` and replace it with your product name.
3. Copy `docs/workflow.md` if you want the shared "repo as notice board" process the agents refer to.
4. The agents mention files such as `docs/roles/<role>.md`, `docs/decisions/`, `docs/automation-log.md` and `ROADMAP.md`. Create the ones you want, or edit the agent files to drop the reference.
5. Ask for work: "Use the CTO to plan a contact form, then the developer to build it, then the code-reviewer to check it."

Agents sound sure even when wrong. A different agent checks the work, reviewers are read-only, and "done" means you saw it work live.

## Agents

| Agent | Use it when |
|---|---|
| `cto` | Turning a goal into a plan and small tickets; reviewing work; gating merges |
| `developer` | Building one approved ticket with tests and surgical diffs |
| `code-reviewer` | Independent, read-only review of any diff before merge |
| `security-reviewer` | Changes touching data flow, logging, storage, dependencies, secrets |
| `legal` | Privacy regime, consent, retention, policy, contracts (not legal advice) |
| `clinical-validator` | Medical meaning of outputs; anything that could read as advice |
| `eval-qa` | Running your gold-set evals and catching accuracy regressions |
| `ocr-evaluator` | Measuring accuracy on real photographed documents |
| `scientist` | Scanning new papers and methods; filing research tickets |
| `market-analyst` | Competitors, pricing, market signals |
| `cfo` | Runway, burn, unit economics (read-only) |
| `product-manager` | What to build and in what order; specs; saying no |
| `shipper` | Cutting a plan to the smallest thing a user can touch this week |
| `user-researcher` | Interviews, usability tests, evidence for every spec |
| `product-designer` | UI, UX, design system, accessibility |
| `ux-copy-writer` | Every word a user reads: buttons, errors, consent, explainers |
| `journey-tester` | Clicking through the app as a non-technical user (read-only) |

## Skills

`ship-it`, `lean-spec`, `minimalist-check`, `first-users`, `one-glance`, `clinical-safety-gate`, `consumer-health-privacy`. Each is a checklist the agents are told to load for that kind of work.

## Not legal or medical advice

The privacy and clinical-safety skills are working notes from one product, not legal or medical advice. Check current law with a real lawyer.

No warranty. The agents and skills can be wrong; review everything they output.

## License

MIT. See `LICENSE`. The Lexiome name and logo are not covered by the licence.

Want to follow the build? https://lexiome.ai/build/agents?s=github
