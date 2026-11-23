# DevSecOps security pipeline

The `DevSecOps Security Pipeline` workflow runs for every pull request and can
also be started manually from the GitHub Actions page.

## Security tools and gates

| Test | Tool | Report | Gate |
| --- | --- | --- | --- |
| SCA | OWASP Dependency-Check 13.0.0 | XML and HTML | Warning when CVSS is 7.0 or higher |
| SAST | Semgrep CE 1.185.0 | JSON | Warning when one or more `ERROR` findings are present |
| DAST | OWASP ZAP 2.17.0 | XML and HTML | Blocking when one or more High-risk alerts are present |

SCA and SAST intentionally use warning gates so the vulnerable application can
reach the dynamic test. ZAP is the blocking gate and therefore determines the
final result of the `Build, test and DAST - ZAP (blocking)` check.

All scanner container images and third-party GitHub Actions are pinned to an
immutable digest or commit. Generated reports are retained as workflow
artifacts for 30 days.

## Required GitHub Actions secrets

- `DEFECTDOJO_URL`: Base URL of the DefectDojo instance, for example
  `https://defectdojo.example.com`.
- `DEFECTDOJO_API_TOKEN`: DefectDojo API v2 token with permission to create and
  reimport tests for this asset.
- `NVD_API_KEY`: NVD API key used by Dependency-Check. The workflow works
  without it, but initial database updates are significantly slower.

Each scanner report is reimported into DefectDojo even when its quality gate
fires. DefectDojo creates separate tests named `OWASP Dependency-Check`,
`Semgrep Community Edition`, and `OWASP ZAP Full Scan`.

GitHub does not provide secrets to pull requests from forks. The scans and
artifact uploads still run for those pull requests, but the DefectDojo import
steps are skipped.

## ZAP scope and safety

The ZAP plan in `zap/automation-plan.yaml` targets only the ephemeral Juice Shop
container on the private `devsecops` Docker network. It runs a traditional
spider, an Ajax spider, passive scanning, and an active scan. Do not change its
target to a third-party system without explicit authorization.

## Merge protection

After the workflow has run at least once, configure the repository's `master`
branch rule to require the `Build, test and DAST - ZAP (blocking)` status check.
