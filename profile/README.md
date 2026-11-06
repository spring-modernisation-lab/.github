# spring-modernisation-lab

A prototype that shows **where AI adds real value in a framework upgrade, and where it must not decide**. It migrates a small Spring estate from Spring Boot 2.7 / Java 11 to Spring Boot 3.5 / Java 21.

> Automation does what it reliably can. A small AI handles the long tail one scoped task at a time. Facts, Maven and the recorded behaviour decide what is right. People decide what only people can, and own every merge.

## How the pieces fit

The **agent** assesses the estate, then migrates the **services** in waves: the parent and shared library first, then the services that depend on them. For each repository it records the Boot 2.7 behaviour, runs OpenRewrite, ports what is left with a scoped AI, and opens a **draft pull request** for a person to review. It never merges.

## Repositories

| Repository | What it is | Why it exists |
|---|---|---|
| [spring-modernisation-agent](https://github.com/spring-modernisation-lab/spring-modernisation-agent) | The migration pipeline (.NET): assess, transform, port, report and hand over | The subject of the prototype. Start with its README, which has the process diagram, the aim and the key findings from the CI runs |
| [platform-parent](https://github.com/spring-modernisation-lab/platform-parent) | Legacy Spring parent POM, version 1.0.0 | Wave 1. Every service inherits from it, so it has to migrate before they can |
| [shared-security](https://github.com/spring-modernisation-lab/shared-security) | Shared security library built on `spring-boot-starter-security` | Wave 1. A library that `account-service` and `customer-service` depend on, so it also has to move first |
| [account-service](https://github.com/spring-modernisation-lab/account-service) | Spring Boot 2.7 service on the parent and `shared-security` | Part of the estate that is assessed and ordered into waves |
| [customer-service](https://github.com/spring-modernisation-lab/customer-service) | Spring Boot 2.7 service on the parent and `shared-security` | Part of the estate that is assessed and ordered into waves |
| [orders-service](https://github.com/spring-modernisation-lab/orders-service) | Order management service: REST API under `/api/orders`, H2, HTTP Basic | The migration target. It has no internal dependencies and carries six deliberately seeded traps (Hibernate `UserType`, HttpClient 5, Spring Security, Ehcache 2, a trailing-slash URL and others) that OpenRewrite cannot fully fix. Its migration PR is where the results show up |

## Actions

| Workflow | Repository | What it does | Why |
|---|---|---|---|
| [Spring modernisation](https://github.com/spring-modernisation-lab/spring-modernisation-agent/actions/workflows/portfolio-transform.yml) | spring-modernisation-agent | The full run: assess, record behaviour, OpenRewrite, optional scoped AI ports, report, and optional draft PRs (`workflow_dispatch`) | The main entry point. Each numbered CI run is one run of this workflow |
| [Portfolio Baseline Build](https://github.com/spring-modernisation-lab/spring-modernisation-agent/actions/workflows/portfolio-baseline-build.yml) | spring-modernisation-agent | Builds every repository on Java 11 at its current commit and records the result | Proves each baseline builds before migrating it. Also called by the full run, so readiness is judged on evidence for the exact commits being migrated |
| [OpenRewrite probe](https://github.com/spring-modernisation-lab/spring-modernisation-agent/actions/workflows/openrewrite-probe.yml) | spring-modernisation-agent | Runs one migration route's recipes on one repository, then builds it. It can also measure candidate recipes and compare behaviour. It never pushes | A discovery tool: finds what the deterministic recipes leave behind, and checks new recipes before they are adopted |
| [Baseline Maven Build](https://github.com/spring-modernisation-lab/platform-parent/actions/workflows/baseline-build.yml) | platform-parent | Maven build on every push and pull request to `main` | Keeps the parent POM buildable |

## Where to start

1. Read the [agent README](https://github.com/spring-modernisation-lab/spring-modernisation-agent#readme) for the aim, the process diagram and the key findings.
2. Open the [migration draft PR on orders-service](https://github.com/spring-modernisation-lab/orders-service/pulls) to see what a hand-over looks like: the outcome and its reasons, the behaviour compared with Boot 2.7, the AI's patches, and the decisions a person must make.
