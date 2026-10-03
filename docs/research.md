# Research and integration decisions

Checked 2026-10-03 using primary documentation, upstream source and GitHub metadata. **No Testificate or Floci runtime experiment was performed in this planning task.** Upstream tests are evidence of upstream implementation intent, not our execution evidence or an AWS parity certificate. [Pinned upstream metadata](upstreams.json) records licenses and revision dates; those are point-in-time facts, not maintenance guarantees.

## What can detect the ECS failure?

| Method | Realistic contribution | What it cannot establish |
| --- | --- | --- |
| TypeScript/type/static analysis | Invalid CDK code; contract-known missing/malformed configuration; template/provider-rule violations | Application startup from arbitrary string values; deployment success |
| Two synthesized infrastructure assemblies | Task/env/image/role/network/deployment differences, affected references, nested paths, possible replacements/deletions | Runtime values for unknown intrinsics, actual account state, reliable rollback |
| Local Docker | Actual process exit, command/env parsing, container health or declared HTTP readiness under recorded values | Fargate scheduling, IAM/execution-role permissions, ALB/VPC behavior, AWS secret injection |
| AWS emulation | Application calls against an implemented local service; some Docker-backed ECS lifecycle behavior | Unmeasured AWS fidelity, production conditions, arbitrary nested CloudFormation rollback |
| Integration tests | Specific workload interactions and assertions in their actual environment | Behavior outside the contract; an “integration” label alone says nothing about AWS |
| Real AWS validation | Account-specific resource/API checks; a real task can test AWS startup; a real service/stack update can test its deployment path | Untested regions/platforms, future reliability, all rollback paths; passing change-set validation does not ensure runtime success |

[CDK testing](https://docs.aws.amazon.com/cdk/v2/guide/testing.html) describes template assertions and snapshots. [ECS circuit-breaker documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-circuit-breaker.html) distinguishes failure to reach RUNNING from health failures and requires an eligible prior completed deployment for rollback. [CloudFormation change sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html) preview changes and perform some validation, while runtime failures can still occur. [Nested rollback recovery](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-continueupdaterollback.html) can require resource reconciliation; Testificate must not promise to prevent or automate that recovery.

[Current CDK diff](https://docs.aws.amazon.com/cdk/v2/guide/ref-cli-cmd-diff.html) defaults to change-set-backed analysis and can interact with AWS. Use offline assembly comparison, not that default, in v0.1. ECS circuit-breaker thresholds are now configurable; do not hard-code historical default formulas as a fidelity claim.

## Floci and testcontainers-floci

**Decision: optional near-term integration, preceded by a local fidelity spike; no fork and no mandatory emulator in v0.1.**

Floci and its modules use MIT licenses and had recent commits at the recorded snapshot. [Pinned ECS documentation](https://github.com/floci-io/floci/blob/543f01618f4070bde51d45a77464c70bb75516dd/docs/services/ecs.md) describes Docker-backed tasks and circuit-breaker/rollback logic. It also documents a single PRIMARY deployment representation, differing monitoring behavior, unenforced grace/placement settings, and synthesized task phases. These are specific limitations, not grounds to dismiss every local use.

[Pinned CloudFormation documentation](https://github.com/floci-io/floci/blob/543f01618f4070bde51d45a77464c70bb75516dd/docs/services/cloudformation.md) documents nested stacks but accepts unsupported resource types as stubs by default. It records update-time deletion/retention and replacement-policy differences. A successful emulated stack cannot demonstrate arbitrary AWS rollback. Require `FLOCI_SERVICES_CLOUDFORMATION_ALLOW_STUB_UNSUPPORTED_RESOURCE_TYPES=false` if CloudFormation is exercised, and inventory unsupported resources before execution.

[Circuit-breaker source tests](https://github.com/floci-io/floci/blob/543f01618f4070bde51d45a77464c70bb75516dd/src/test/java/io/github/hectorvent/floci/services/ecs/EcsServiceDeploymentCircuitBreakerTest.java) drive launch failures through a mocked container manager. They support the conclusion that circuit-breaker logic exists; they do not show a real Docker startup regression or a side-by-side AWS trial.

[testcontainers-floci](https://github.com/floci-io/testcontainers-floci/tree/96b92fa38c101f424d022cbb4fbc0e3d958447ed) is the Java module; do not add a JVM to a TypeScript product. The [Node module](https://github.com/floci-io/testcontainers-floci-node/tree/dc09277c97fc40ef322e8e6730425d2868ef0310) publishes `@floci/testcontainers`. Use it for lifecycle/network setup when demonstrated sufficient; a small wrapper around generic Node Testcontainers is the fallback. The spike must inspect support for ECS real-container mode, Docker socket/network configuration, readiness, image pinning and cleanup. Marketing examples for S3/SQS do not establish those requirements.

The first optional emulator workflow should test one declared application dependency, initially SQS, with a valid queue and a deliberate invalid identifier. Record endpoint/substitution overrides. Do not deploy the whole CDK stack merely to test that dependency. Any ECS emulation is an additional local observation with separate property IDs. Submit narrowly evidenced bugs upstream rather than maintaining a fork.

### Initial capability assessment (research, not runtime results)

| Capability | Evidence available now | Admission to a verification claim |
| --- | --- | --- |
| SQS client/API operations | Upstream docs/examples | Unknown behavioral result until Testificate spike executes a positive and negative contract |
| ECS runs actual containers | Documented implementation and source | Local behavior must be exercised with real mode and pinned Docker/image settings |
| ECS circuit breaker | Source tests with controlled launch failure | Local model only; AWS-comparison evidence absent |
| AWS scheduler, IAM, Fargate networking, ALB health | Partial modeled features and documented differences | Unknown for the specific change; never generalized from SDK compatibility |
| Unsupported CFN resource | Default stub behavior documented | Reject or unsupported gap, never demonstrated provisioning |
| Nested rollback/replacement/deletion | Documented support plus explicit differences | Unverifiable as AWS behavior; opt-in live study later |

## Candidate decisions

“Later” means reconsider only for a supported property/workflow; “research” does not commit an integration. Complexity and cost are measurements, not confidence.

| Candidate | Placement | Maintenance/license snapshot | Workflow fit, noise and support decision |
| --- | --- | --- | --- |
| [cfn-lint](https://github.com/aws-cloudformation/cfn-lint/tree/520b18f7371efa962c57ea947254b813876f39ce) | MVP, configured fallback/static adapter | Recent commits; MIT-0 | Emits provider/template-rule evidence on selected templates. Intrinsic evaluation is best effort; a clean result proves selected rules only. Pin rule/schema/config versions; Python provisioning and output compatibility add burden. Avoid duplicate execution when supported CDK validation already supplies equivalent evidence. |
| [cfn-guard](https://github.com/aws-cloudformation/cloudformation-guard/tree/814bd00a4e6d761e8b5c9f615dd510b1a2a7c374) | Near-term, opt-in | Recent commits; Apache-2.0 | Useful for explicit org/ECS template contracts. Rule applicability and missing-data semantics need tests; policy quality determines noise. Separate policy rules from runtime assertions; no huge default ruleset. |
| [Dependency vulnerability auditing](https://docs.npmjs.com/cli/audit/) | Near-term, lockfile-scoped npm adapter | Maintained npm CLI; Artistic-2.0; [pip-audit](https://github.com/pypa/pip-audit) Apache-2.0 if Python later | Known advisories for exact dependencies and database timestamp; not exploitability proof. Network/database freshness and transitive/dev dependency noise require explicit scope. No audit fix or silent registry upload. |
| [Ruff](https://docs.astral.sh/ruff/) | Later, only a Python target | Recent commits; MIT | Valuable Python lint evidence, irrelevant to the first TS CDK target. Reuse existing output before owning another tool/runtime; rule/suppression noise is configuration-dependent. |
| [Knip](https://knip.dev/guides/handling-issues) | Later, advisory TS cleanup | Recent commits; ISC | Unused files/dependencies do not prove startup. Dynamic imports and entrypoints require project configuration; false positives and plugin maintenance are ongoing costs. |
| [Vulture](https://github.com/jendrikseipp/vulture/tree/a70cc9ed09d1b24162608cdf88959ebd9e6405ab) | Unnecessary for planned scope | Recent commits; MIT | Python dead-code heuristics, runtime/whitelist burden and dynamic-dispatch noise; overlaps language tooling and no ECS property. Its confidence labels are tool heuristics, never Testificate confidence. |
| [RSLint](https://github.com/rslint/rslint/tree/7bc011e57c76ee114c440521a51907089584b10a) | Unnecessary | Default-branch commit 2022-03-28; MIT; README warns of early development | Additional parser/rules maintenance for no relevant new property. Prefer existing maintained TypeScript tooling; do not mistake non-archived status for active maintenance. |
| [Duplicalis](https://github.com/pfrankov/duplicalis/tree/29b7f7b4a8d9281a5b2a15820bc96133219699fa) | Experimental/research | Recent commits; Apache-2.0 | React code/style similarity, local model downloads or optional remote embeddings. Similarity is not behavioral equivalence; calibration, model/version fingerprints and false positives are substantial. No React product need in MVP; do not add inference runtime. |
| [Genesis Architect](https://github.com/maioio/genesis-architect/tree/816e2408097f3c7b4c2f012a0e7263e54f2f1e68) | Unnecessary integration | Recent commits; AGPL-3.0 at this revision | Broad architecture diagnosis/scaffolding overlaps the agent's planning and does not demonstrate ECS properties. Heuristic scores, runtime/tool churn and license compatibility increase burden. Do not copy or embed its code; no claim that AGPL prohibits commercial use. |
| [Cyclomatic complexity](https://eslint.org/docs/latest/rules/complexity) | Later, advisory existing-tool output | Maintained ESLint; MIT | Counts control-flow paths; can focus review/testing of changed functions. Parser variants and thresholds affect results, and refactor-driven deltas can mislead. Reuse measurements; no mandatory correctness threshold. |
| [Cognitive complexity](https://www.sonarsource.com/docs/CognitiveComplexity.pdf) | Experimental/research | Maintained Sonar methodology; implementation license must be checked individually | Human-understanding heuristic; nesting/style can shift it without behavioral change. Requires task-specific evaluation and avoids duplicating Sonar. Do not embed implementation on assumption of a methodology license. |
| Experimental token/code complexity | Experimental/research | No selected implementation/license | Tokenizer-dependent size is reproducible only with a named tokenizer/version; predictive correctness meaning unestablished. Study held-out regressions and false positives before any policy use. |
| Repository knowledge for agents | Later, bounded evidence export | Own TS report exporter, planned Apache-2.0 implementation | Export affected surfaces, dependencies, evidence links and unresolved assumptions with revision provenance. A wiki/graph maintained by inference introduces stale claims; no new knowledge engine. |
| [Google Open Knowledge Format](https://github.com/GoogleCloudPlatform/open-knowledge-format/blob/ad30107c31c06aec8a7d5636e0d1058118604e6f/SPEC.md) | Experimental/research | v0.2 at pinned spec; Apache-2.0 | Markdown/YAML knowledge interchange with provenance/trust concepts is relevant to export, not the execution-result schema. Evaluate lossless mapping and stale links; no BigQuery/Gemini reference-agent dependency or paid ingestion. |
| Agent token/cost telemetry | Unnecessary for verification product | Provider/version-dependent; no implementation chosen | Explains resource use, not properties demonstrated; pricing/model churn and privacy add burden. Record verification duration/resources in core; no prompt/token collection. |

[CloudFormation Validate](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-validate.html) is an additional relevant current option: Apache-2.0, offline structured validation and a default CDK plugin in supported current versions. Prefer capturing its existing synthesis report where available; retain cfn-lint as the explicit fallback. Recheck pinned CDK versions and platform packaging in the initial fixture spike. Do not require both engines or treat their agreement as independent correctness evidence.

## Overlap and actual differentiation

| Existing system | Established overlap | Testificate's chosen boundary |
| --- | --- | --- |
| [SonarQube](https://docs.sonarsource.com/sonarqube-cloud/standards/managing-quality-gates/introduction-to-quality-gates) | New-code quality gates, static issues, coverage/duplication metrics | Reuse findings as limited evidence if later needed; lead with the env-change startup counterexample and environment-bounded runtime observations. |
| [CodeRabbit](https://docs.coderabbit.ai/tools/pmd) | AI review plus integrated static tools | No generic reviewer or prose verdict engine. A reviewer/agent can consume actual executions and gaps from the local core. |
| [MegaLinter](https://megalinter.io/latest/) | Large maintained linter bundle, local/CI integrations | No linter aggregation product. Run only tools selected to assert properties of the affected change. |
| [Launchable](https://help.launchableinc.com/features/predictive-test-selection/how-launchable-selects-tests/) / [Trunk](https://docs.trunk.io/flaky-tests/overview) | Predictive selection, execution trends, flakiness operations | Deterministic property selection and conservative fallback first; no historical ML platform or flake dashboard. |
| [CDK diff](https://docs.aws.amazon.com/cdk/v2/guide/ref-cli-cmd-diff.html), Guard, cfn-lint, [Checkov](https://www.checkov.io/) | Infrastructure change preview and static policies | Tie those checks to runtime configuration and preserve what their environment cannot test. |
| [Testcontainers](https://node.testcontainers.org/), Floci / [LocalStack](https://docs.localstack.cloud/aws/services/ecs/) | Local infrastructure/runtime execution | Integrate lifecycle providers rather than reimplementing them; expose exact mappings and fidelity boundaries. |
| [Stryker](https://stryker-mutator.io/docs/), [in-toto](https://in-toto.io/), [SLSA provenance](https://slsa.dev/spec/v1.2/provenance) | Test strength and artifact provenance | Reuse existing concepts. Build provenance or mutation score alone does not establish an ECS deployment property. |

This is a product wedge, not a claim that competitors lack every similar feature. The release benchmark must show added value over compilation, synth, assertions, and a manually configured Docker smoke test. If Testificate cannot automatically preserve the change mapping, conditions, and gaps with less rediscovery, do not broaden the product.

## Remaining uncertainties

Input translation, nested assembly versions, CDK asset mapping, readiness oracle quality, Docker Desktop/Linux portability, current native static-validator packaging, and emulator qualification are engineering risks addressed by explicit roadmap gates. Account-specific AWS task execution, deployment circuit breaking and nested rollback require a later live study with separate authorization and budgets. No local or upstream source review removes those gaps.
