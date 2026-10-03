# Testificate

Change verification with explicit evidence and explicit gaps.

Testificate is planned to answer: **What properties of this specific change have been demonstrated, under which conditions, and what remains unverified?** It applies equally to human and agent changes; line authorship is irrelevant.

**Status:** product definition and roadmap only. No application has been implemented.

## Start with the roadmap

The [GitHub Project](https://github.com/users/BrandonCasa/projects/3/views/1) is the development roadmap. Read [the build sequence](docs/roadmap.md), [the architecture decisions](docs/design.md), and [the researched integration decisions](docs/research.md).

## First useful release

The first target is a TypeScript AWS CDK v2 app deploying a Linux ECS workload. A change to an environment variable can compile, synthesize, and satisfy template assertions while causing the application to exit or never become ready.

The v0.1 CLI will compare two committed revisions, synthesize both without AWS access, identify affected ECS task configuration, and run the selected immutable container image with the base and head configuration. A declared readiness contract supplies the oracle. The report connects the observed startup failure to the changed task properties and preserves logs, image identity, input fingerprints, and limitations.

Planned interface:

```text
testificate verify --base <ref> --head <ref>
```

Example report meaning:

| Property | Base | Head | Conditions |
| --- | --- | --- | --- |
| CDK synthesizes | Demonstrated | Demonstrated | Pinned CDK, dependencies, context |
| Selected template satisfies configured validation rules | Demonstrated | Demonstrated | Exact validator and rule set recorded |
| Selected process stays alive and becomes ready | Demonstrated | Failed | Local Docker, recorded image digest, supplied test values |
| Task starts on AWS Fargate | Unverifiable | Unverifiable | No real AWS execution |
| ECS circuit breaker and nested-stack rollback succeed | Unverifiable | Unverifiable | No real AWS deployment |

A local policy may pass while AWS properties remain unverified. Every report names that policy; there is no unqualified “safe to deploy” badge or invented confidence score.

## Deliberate MVP limits

- One selected ECS service/task and one essential Linux container, with a pinned image and an explicit bounded startup/readiness contract.
- Local substitutions for unresolved values are allowed only when explicitly supplied and recorded as conditions. Secret injection by AWS remains unverified.
- Read nested templates from the CDK cloud assembly, but flag deletions, logical-ID churn, ambiguous mappings, possible replacements, and unresolved references conservatively.
- No AWS credentials, provisioning, asset publication, automatic remediation, cloud dashboard, model calls, broad linter bundle, or plugin marketplace.
- GitHub Actions is an adapter after the local release, not part of the core contract.

Floci is a promising optional environment for AWS dependency integration tests. It is **not a prerequisite for v0.1**: its documented ECS and CloudFormation differences make it unsuitable as proof of real AWS deployment or rollback. The roadmap includes a bounded fidelity spike and an upstream integration using the Node Testcontainers module if the spike succeeds.

## Outcomes in order

1. **v0.1: Local ECS change evidence.** Reproduce and detect the configuration regression before deployment, with reproducible evidence and honest gaps.
2. **v0.2: Repeatable team verification.** Reuse the same CLI in CI; add bounded emulator dependency tests and explicit policy/audit evidence where useful.
3. **v0.3: Portable review evidence.** Expose existing results to agents and support a measured extension to task topology.
4. **Research, without release commitments.** Real AWS validation, stronger test oracles, advisory complexity signals, and OKF interchange.

Research was checked on 2026-10-03. No AWS resources were created and no paid services were used to develop this plan.
