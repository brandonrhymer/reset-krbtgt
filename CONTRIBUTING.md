# Contributing

## Development environment

Use Windows PowerShell 5.1 Desktop edition, Pester 3.4.0, and
PSScriptAnalyzer 1.25.0. Have approved copies installed on the development
workstation before running verification. Pester 5 uses a different test API;
upgrading the test framework requires a separate migration and validation.

The mocked suite does not require AD RSAT, domain membership, administrative
rights, or access to domain controllers. Do not run operator entry scripts as
development smoke tests. Preflight reads the directory; Test mutates it;
Reset changes KRBTGT credentials. Even Reset `-WhatIf` requires directory
access and writes local evidence.

## Change workflow

1. Create a focused branch from current `main`.
2. Keep runtime changes scoped and add meaningful regression coverage for
   changed behavior. Update the relevant runbook, contracts, or architecture
   decisions when behavior changes.
3. Review the diff and run the checks below in Windows PowerShell 5.1.
4. Open a pull request explaining the problem, resulting behavior, validation,
   and remaining limitations. Require a successful `verify` check before merge.
5. Review the pull request before merging. Do not push changes directly to
   `main`, force-push shared history, or use a failing check as a release gate.

```powershell
# Check the existing package before editing.
.\tools\Test-ReleaseHashes.ps1

# After intentional changes to files listed in release-manifest.txt:
.\tools\New-ReleaseHashes.ps1
.\tools\Invoke-ReleaseVerification.ps1
git diff --check
```

Review changed hash entries alongside the source diff. Never regenerate hashes
to dismiss an unexplained integrity failure. Add distributable files to
`release-manifest.txt`; CI configuration and local development artifacts do
not belong in the air-gap package. The checksums verify file consistency;
they do not authenticate a publisher or replace code signing.

## Continuous integration

GitHub Actions runs the local verification gate on a hosted Windows runner
for `main` pushes, pull requests, and manual dispatch. Branch pushes update the
pull-request check without starting a duplicate push run. It uses read-only repository
permissions and explicitly versioned development dependencies. It has no AD
credentials and performs no deployment. Network dependency installation occurs
only during CI setup, outside the offline runtime package.

## Releases

- Merge reviewed changes only after required verification succeeds.
- For runtime changes, update both the module manifest version and module
  `ToolVersion`; change schema versions only when their contracts change.
- Regenerate and review hashes after the final package changes. Verify again
  from a clean checkout of the exact commit to be released.
- Create a version tag and GitHub release from that reviewed commit. Package
  only the files listed in `release-manifest.txt`, retaining relative paths.
  Run `Test-ReleaseHashes.ps1` against the extracted package before publishing.
- Release notes must identify changes, the exact commit, verification results,
  and the lack of representative AD validation. Never imply that a successful
  CI run certifies operational use.

Do not commit real receipts, domain details, credentials, customer information,
or generated test output. Keep fixtures fictional and report sensitive issues
privately to the repository owner.
