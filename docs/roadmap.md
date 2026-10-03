# Build sequence

The [GitHub Project](https://github.com/users/BrandonCasa/projects/3/views/1) is the live roadmap. This index records the initial dependency-compatible sequence; use issue status and native blocked-by links for current progress. No dates or time estimates are implied.

## v0.1 - Local ECS change evidence

Start with **#1 and #2**. #3 and #7 establish evidence/execution contracts; #4 and #5 establish reproducible synthesis/impact; #6 establishes selection and weakening protection; #8 and #9 produce runtime/static evidence; #10 presents policy results; #11 qualifies the release. The fixture spike is a prerequisite to the runtime/static adapters, not a reason to postpone independent core setup.

| Order / issue | Priority | Prerequisites | Outcome |
| --- | --- | --- | --- |
| [#1](https://github.com/BrandonCasa/Testificate/issues/1) Prove the offline ECS startup regression and define the supported fixture contract | P0 | Ready | Evidence-gathering gate |
| [#2](https://github.com/BrandonCasa/Testificate/issues/2) Establish the local TypeScript core and fixture-driven development gates | P0 | Ready | Required MVP work |
| [#3](https://github.com/BrandonCasa/Testificate/issues/3) Implement versioned property evidence, artifact provenance, and stale-result invalidation | P0 | #1, #2 | Required MVP work |
| [#4](https://github.com/BrandonCasa/Testificate/issues/4) Capture exact base/head revisions and synthesize isolated CDK assemblies offline | P0 | #2, #3 | Required MVP work |
| [#5](https://github.com/BrandonCasa/Testificate/issues/5) Model affected ECS surfaces across templates, references, and nested stacks | P0 | #4 | Required MVP work |
| [#6](https://github.com/BrandonCasa/Testificate/issues/6) Select checks deterministically and detect weakened verification contracts | P0 | #3, #5 | Required MVP work |
| [#7](https://github.com/BrandonCasa/Testificate/issues/7) Execute planned checks with bounded processes, redaction, and reliable cleanup | P0 | #2, #3 | Required MVP work |
| [#8](https://github.com/BrandonCasa/Testificate/issues/8) Run paired Docker startup/readiness checks from selected ECS configuration | P0 | #1, #5, #6, #7 | Required MVP work |
| [#9](https://github.com/BrandonCasa/Testificate/issues/9) Translate CDK validation and cfn-lint results into scoped static evidence | P1 | #1, #4, #6, #7 | Required MVP work |
| [#10](https://github.com/BrandonCasa/Testificate/issues/10) Produce change-linked reports and evaluate named policies without hiding gaps | P0 | #6, #8, #9 | Required MVP work |
| [#11](https://github.com/BrandonCasa/Testificate/issues/11) Qualify and release v0.1 with the ECS regression and honest-gap acceptance suite | P0 | #10 | Required MVP work |

All 11 issues are required to reach the first useful release. #11 includes original bad-env/healthy controls, exact package provenance, all applicable local gates, supported-host qualification, and explicit unverified AWS behavior. None of the following phases is an MVP prerequisite.

## v0.2 - Repeatable team verification

- [#12](https://github.com/BrandonCasa/Testificate/issues/12) Wrap the released CLI for GitHub and generic CI with exact revision reporting (P1); depends on #11.
- [#13](https://github.com/BrandonCasa/Testificate/issues/13) Qualify Floci and the Node Testcontainers module for a bounded dependency workflow (P1); depends on #11.
- [#14](https://github.com/BrandonCasa/Testificate/issues/14) Integrate opt-in Floci dependency checks with explicit capability limits (P1); depends on #13.
- [#15](https://github.com/BrandonCasa/Testificate/issues/15) Add opt-in Guard contracts as explicit template policy evidence (P2); depends on #11.
- [#16](https://github.com/BrandonCasa/Testificate/issues/16) Add lockfile-scoped dependency audit evidence with database freshness (P2); depends on #11.

The outcome is portable CI evidence and explicit optional dependency/policy checks. Floci implementation #14 cannot start before spike #13; a failed spike narrows/defers that integration. Guard and audit are opt-in and have separate property claims. v0.2 readiness requires qualification of the selected optional contracts, not a claim that all AWS APIs are supported.

## v0.3 - Portable review evidence

- [#17](https://github.com/BrandonCasa/Testificate/issues/17) Expose existing plans and evidence to agents through a thin MCP/read interface (P2); depends on #12.
- [#18](https://github.com/BrandonCasa/Testificate/issues/18) Measure support boundaries for sidecars and reproducible CDK image assets (P2); depends on #11.
- [#19](https://github.com/BrandonCasa/Testificate/issues/19) Extend ECS verification to the task topology and assets qualified by the spike (P2); depends on #18.

This phase exposes existing evidence to agents and adds only the task/image mappings qualified by the preceding spike. Preserve the v0.1 gate and revalidate scope before starting.

## Exploratory; no release commitment

- [#20](https://github.com/BrandonCasa/Testificate/issues/20) Design a bounded real-AWS validation study before adding live execution; depends on #11, #13.
- [#21](https://github.com/BrandonCasa/Testificate/issues/21) Evaluate independent contracts and mutation controls for correlated code/test changes; depends on #11.
- [#22](https://github.com/BrandonCasa/Testificate/issues/22) Test whether complexity and similarity signals improve verification selection; depends on #11.
- [#23](https://github.com/BrandonCasa/Testificate/issues/23) Evaluate lossless OKF export of revision-bound verification knowledge; depends on #17.

Live AWS remains a designed experiment until separately authorized and budgeted. Similarity/complexity, stronger test-oracle methods, and OKF exports require evidence of usefulness before implementation. Ruff/Knip are later candidate integrations only when a supported workflow warrants them; Vulture, RSLint, Genesis Architect embedding and agent cost telemetry are deliberately not implementation issues.

## Project conventions

The saved views are [MVP build path](https://github.com/users/BrandonCasa/projects/3/views/1), [Release phases](https://github.com/users/BrandonCasa/projects/3/views/2), and [Exploratory research](https://github.com/users/BrandonCasa/projects/3/views/3). Each shows ordered titles, status, priority/scope labels and milestone; the research view deliberately has no release assignments.

Use `scope:mvp`, `scope:next`, `scope:later`, or `scope:explore` plus P0/P1/P2 and `research` for spikes. Numbered titles provide stable build ordering. Ready/Blocked describe current MVP readiness; Backlog is future planned work; Exploring is uncommitted research; In progress/Done record actual execution. Dependencies are both native GitHub relationships and issue-body links. Milestones represent independently useful outcomes, and exploratory issues have no release milestone.

The initial plan has 23 issues and 34 acyclic dependency links. No product implementation is included in this planning commit.
