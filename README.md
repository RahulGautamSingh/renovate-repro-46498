# Minimal reproduction: Gradle rich version constraints

Reproduction for [renovatebot/renovate#46498](https://github.com/renovatebot/renovate/pull/46498), from [discussion #45481](https://github.com/renovatebot/renovate/discussions/45481).

## Current behavior

Renovate does not extract dependencies whose version is declared in a `version { ... }` block.
`build.gradle` has four of them, and Renovate finds none, so `log4j-core:2.14.1` (CVE-2021-44228) never gets a vulnerability fix.

## Expected behavior

| Dependency | Declared | Expected |
| --- | --- | --- |
| `org.apache.logging.log4j:log4j-core` | `strictly '2.14.1'` | disabled by default, but gets a vulnerability-fix PR |
| `org.slf4j:slf4j-api` | `strictly '1.7.25'` | disabled by default, no PR |
| `com.google.code.gson:gson` | `require '2.10'` | updated like a plain version |
| `commons-io:commons-io` | `strictly '[2.6, 3.0['` + `prefer '2.6'` | skipped as `multiple-constraint-dep` |

Vulnerabilities come from OSV (`osvVulnerabilityAlerts: true`), so no GitHub Dependabot alerts are needed.

## Verified

Run with Renovate from source against [`648bc88`](../../commit/648bc881ce909bef6822a3b07a270ca5d91b0acc).

| Renovate | Deps extracted | `log4j-core` | `slf4j-api` | `gson` | `commons-io` |
| --- | --- | --- | --- | --- | --- |
| `main` (8f2ffc0e90) | 0 | not extracted | not extracted | not extracted | not extracted |
| with [#46498](https://github.com/renovatebot/renovate/pull/46498) (91d690ee0b) | 4 | [#1](../../pull/1) `strictly '2.25.4'` [SECURITY] | `disabled` | [#4](../../pull/4) `require '2.14.0'` | `multiple-constraint-dep` |

With #46498, Renovate logs `Dependency: org.apache.logging.log4j:log4j-core, is disabled by default, but has a vulnerability alert`.
On `main`, the same run would autoclose both PRs, because the dependencies are no longer found.

## Opt-in rule

[`efd06e5`](../../commit/efd06e5) adds the package rule from the [Gradle manager docs](https://github.com/renovatebot/renovate/blob/feat/gradle-rich-versions/lib/modules/manager/gradle/readme.md), which turns `strictly` and `prefer` constraints back on for regular updates:

```json
{
  "matchManagers": ["gradle"],
  "matchJsonata": ["managerData.versionConstraint in ['strictly', 'prefer']"],
  "enabled": true
}
```

Run with #46498 (193b2b4b6f): `slf4j-api` is no longer disabled and gets regular update PRs, which rewrite the `strictly` version:

- [#5](../../pull/5) `strictly '1.7.25'` → `strictly '1.7.36'`
- [#6](../../pull/6) `strictly '1.7.25'` → `strictly '2.0.20'`
