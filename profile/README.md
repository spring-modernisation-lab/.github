# spring-modernisation-lab

A prototype that upgrades a group of Spring applications from Spring Boot 2.7 / Java 11 to Spring Boot 3.5 / Java 21. Automated tools do the routine work, a tightly limited AI handles the leftovers, and people make the decisions.

> **In one sentence:** automation does what it reliably can, a small AI fixes what is left one clearly defined job at a time, the build and a recorded snapshot of the application's behaviour decide whether it worked, and people decide anything that is a judgement call.

## What we are trying to show

Where AI adds real value in a framework upgrade, and where it must not be trusted to decide. In short:

- Automated tools can do most of the routine work, and say clearly where they stop.
- A limited AI can handle the leftovers, one small job at a time, as long as its change is kept only when the problem is fixed, nothing new breaks and the behaviour is unchanged.
- Facts decide whether an upgrade worked, not the AI. That includes catching a build that passes but has lost behaviour.
- Anything only a person can decide is handed over for review. Nothing is ever merged automatically.
- The output is readable by people: documents and pull requests built from recorded facts, with a reason for every outcome.

## How it works

1. **Assess.** Scan the applications, work out which depend on which, and group them into **waves**. Shared parents and libraries go first.
2. **Record.** Before changing anything, record what each old application does: whether it starts, who can call what, and its key settings.
3. **Rewrite.** An automatic code-rewriting tool (OpenRewrite) applies the routine changes.
4. **Fix the leftovers.** Whatever is left is sorted into kinds. For each known kind, the AI is given one small job, limited to specific files. Its change is kept only if the problem is gone, nothing new fails and behaviour is unchanged.
5. **Hand over.** A draft pull request and a results document explain what changed and why. A person reviews, decides and merges.

## The repositories

| Repository | What it is | Why it is here |
|---|---|---|
| [spring-modernisation-agent](https://github.com/spring-modernisation-lab/spring-modernisation-agent) | The upgrade pipeline itself | This is the prototype. **Start with its README**, which has the process diagram, the findings from each test run and what is left to do |
| [platform-parent](https://github.com/spring-modernisation-lab/platform-parent) | The shared parent configuration that every service builds on | Must be upgraded first, because everything else depends on it |
| [shared-security](https://github.com/spring-modernisation-lab/shared-security) | A shared security library used by two of the services | Also upgraded early, for the same reason |
| [account-service](https://github.com/spring-modernisation-lab/account-service) | A Spring Boot 2.7 application | Part of the set being assessed and ordered into waves |
| [customer-service](https://github.com/spring-modernisation-lab/customer-service) | A Spring Boot 2.7 application | Part of the set being assessed and ordered into waves |
| [orders-service](https://github.com/spring-modernisation-lab/orders-service) | An order management application | The main test subject. It deliberately contains six awkward cases that automation cannot fully fix, so its draft upgrade PR shows the whole process |

## The automated workflows

| Workflow | Where | What it does | Why it is there |
|---|---|---|---|
| [Spring modernisation](https://github.com/spring-modernisation-lab/spring-modernisation-agent/actions/workflows/portfolio-transform.yml) | spring-modernisation-agent | Runs the whole process: assess, record, rewrite, fix the leftovers, report, and optionally open draft PRs | The main entry point. Each numbered test run is one run of this |
| [Portfolio Baseline Build](https://github.com/spring-modernisation-lab/spring-modernisation-agent/actions/workflows/portfolio-baseline-build.yml) | spring-modernisation-agent | Builds every repository as it is today and records the result | Proves each application builds before we try to upgrade it |
| [OpenRewrite probe](https://github.com/spring-modernisation-lab/spring-modernisation-agent/actions/workflows/openrewrite-probe.yml) | spring-modernisation-agent | Tries the automatic rewrite rules on one repository and builds the result. It never changes the repository | Finds out what the automatic rules leave behind, and tests new rules before they are adopted |
| [Baseline Maven Build](https://github.com/spring-modernisation-lab/platform-parent/actions/workflows/baseline-build.yml) | platform-parent | Builds the parent on every change to the main branch | Keeps the parent buildable |

## What is proven, and what is not yet

**Proven on the demo set:** the automatic rewrite did most of the routine work. The AI's small, scoped fixes were accepted on the first attempt, and the behaviour snapshot caught problems a passing build missed. Cases that need a person (a library Spring Boot 3 dropped, a business choice about URLs) were reported, not worked around.

**Still to do:**

- Work on any chosen set of repositories. The demo is built around these six, and one workflow has its list written in.
- Remember progress between runs, so a later run picks up where the last stopped.
- Wait for upgraded shared components to be released before upgrading what depends on them, and bump those versions automatically.
- Support deploying and cutting over the upgraded components. People still own the merge, the release and retiring the old version, but the pipeline should say when each is possible.
- Keep a migration up to date while normal development carries on.

## Where to start

1. Read the [agent README](https://github.com/spring-modernisation-lab/spring-modernisation-agent#readme) for the full picture.
2. Open the [draft upgrade PR on orders-service](https://github.com/spring-modernisation-lab/orders-service/pulls) to see what a hand-over looks like: the outcome and its reasons, how behaviour compares with the old version, the AI's changes, and the decisions left for a person.
