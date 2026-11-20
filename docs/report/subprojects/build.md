# Build & coverage

## What is checked

Three checks read the saved result of `cpr build <N>`: whether the PR compiled, whether selected unit tests passed, and whether the lines the PR added ran under those tests.

For Cassandra Sidecar and Cassandra Analytics this section reads **"not built"**. `cpr build` supports only apache/cassandra's Ant build today; Gradle support for the subprojects is planned (OpenSpec change `gradle-build`). `cpr review` never builds, so without a saved result for the current head all three checks are not-applicable and nothing blocks.

What a build would need is also more than Cassandra's: both projects' integration tests depend on in-JVM dtest jars built from Cassandra, Analytics also on locally built Sidecar libraries, and Sidecar's container tests on Docker.

## Why

- The project's own CI builds the PR and runs its unit and integration tests on every PR to `trunk`; the CI aspect reads that result. Until `cpr build` handles Gradle, that is the build evidence for these projects.
- Building and testing locally needs the dtest jars first: `./scripts/build-dtest-jars.sh` for Sidecar, `./scripts/build-dependencies.sh` for Analytics. Sources: [Sidecar TESTING.md](https://github.com/apache/cassandra-sidecar/blob/trunk/TESTING.md), "Build Prerequisites"; [Analytics DEV-README.md](https://github.com/apache/cassandra-analytics/blob/trunk/DEV-README.md), "Dependencies".

## How each status is decided

A result counts only if it was saved for the PR's current head sha. No result means not built: all three checks are not-applicable with the reason "not built".

### `build.compiles`

**Blocking** when a result exists. Owner: contributor.

- **not-applicable**: not built for this head. This is the result for every Sidecar and Analytics PR today.

### `build.tests-pass`

**Blocking** when a result exists. Owner: contributor.

- **not-applicable**: not built for this head.

### `build.changed-line-coverage`

Advisory. Owner: contributor.

- **not-applicable**: not built for this head.

## How to fix

There is nothing for the contributor to fix here. To check a build yourself:

```
./gradlew codeCheckTasks build -x integrationTest   # Sidecar: code checks and unit tests
./gradlew clean assemble check                      # Analytics
```

Both need the dependencies above built first. Coverage: Sidecar's [TESTING.md](https://github.com/apache/cassandra-sidecar/blob/trunk/TESTING.md), "Code Coverage", gives the JaCoCo report command.

## Limits

- **No build, no coverage.** The section gives no compile, test or changed-line coverage evidence for these projects; read the CI aspect for whether the PR builds and its tests pass.
- `cpr build` on a Sidecar or Analytics PR builds nothing: it records "not built" for the head, with the reason that only apache/cassandra's Ant build is supported.
- PMD runs without compiled classes for the same reason (see the Code style aspect).
