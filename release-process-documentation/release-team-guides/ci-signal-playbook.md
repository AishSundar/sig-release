# CI Signal Lead Playbook

## Overview of CI Signal responsibilities
CI Signal lead assumes the responsibility of the quality gate for the release. This person is responsible for:
- Monitors various e2e tests in sig-release dashboard ([master-blocking](https://k8s-testgrid.appspot.com/sig-release-master-blocking), [master-upgrade](https://k8s-testgrid.appspot.com/sig-release-master-upgrade), [x.y-blocking](https://k8s-testgrid.appspot.com/sig-release-1.11-blocking))on a regular basis and provides early and continuous signals on release and test health to both the Release team and SIGs
- Ensures that all release blocking tests provide a clear Go/No-Go signal for the release
- Flags regression as close to source as possible i.e., as soon as the offending code was merged
- Files and triages issues proactively for test failures and flakes, finds appropriate SIG/owner, follows up on status and verifies fixes on newer test runs
- Studies patterns/history of test health from previous releases and works closely with SIGs and test owners to 
  - Understand and fix the test for the current release
  - Understand if the test needs to be release blocking
  - Advise Release team in making informed go/no-go calls 
  - Work with sig-testing on any possible test infra improvement to help improve test pass rate
- Makesrecommendations to Sig-Release for promoting and demoting Release blocking and Merge blocking tests as per the [Blocking tests Proposal](https://docs.google.com/document/d/1kCDdmlpTnHPQt5z8JzODdFCc3T2D4MKR53twsDZu20c/edit)

### Explicit detail is important:
- If you're looking for answer that's not in this document, don't just ask me, please file an issue so we can keep the document current.
- If I don't file an issue for a failing job or test, _I will lose track of it_, so I err on the side of filing an issue even if someone told me something in some other medium
- If a dashboard isn't listed here, or a test isn't on one of the listed dashboards, **_I'm not looking at it_**

## Requirements
- I need the ability to notify kubernetes org github teams, so I must be a kubernetes org collaborator or member
  - The process to become one of these is in [our community membership ladder](https://github.com/kubernetes/community/blob/master/community-membership.md#requirements-for-outside-collaborators)
- I need the ability to add a milestone to issues, so I must be a member of the [kubernetes-pm github team](https://github.com/orgs/kubernetes/teams/kubernetes-pm/members)
  - The process to join this group is in [our community project-managers page](https://github.com/kubernetes/community/blob/master/project-managers/README.md#joining-the-group)
- I need to understand what tests matter and generally how our testing infra is wired together
  - I can ask previous CI Signal leads for advice
  - I can ask sig-testing for guidance

## Status Quo

Currently, here's what I'm doing as the CI Signal role for release 1.y:

### Checking test dashboards

I check [sig-release-master-blocking](https://k8s-testgrid.appspot.com/sig-release-master-blocking) daily.

- quirk: if a job is listed as FAILING but doesn't have "Overall" as one of its ongoing failures, it's not actually failing
- if a job is failing in one of the meta-stages (Up, Down, DumpClusterLogs, etc), I find the owning sig
- if a job is failing because a specific test case is failing, and that test case has a [sig-foo] in its name, I go bug sig-foo
- with unit test case failures, I try to infer the sig from the path, or OWNERS files in it, otherwise I find the owning sig to help
- with verify failures, I try to infer the failure from the log, otherwise I find the owning sig to help
- if a test case is failing in one job consistently, but not others, both the job owner and test case owner are responsible for identifying why this combination is different

I will ocasionally check [sig-release-master-upgrade](https://k8s-testgrid.appspot.com/sig-release-master-upgrade)

### Pinging SIG's

I do this by using @kubernetes/sig-foo-test-failures teams when I file issues, or going to the #sig-foo slack channel if I need routing help and know someone's around

If a job as a whole is failing, I file a v1.y issue in kubernetes/kubernetes titled:`[job failure] job name issue`
```
/priority critical-urgent
/priority failing-test
@kubernetes/sig-FOO-test-failures

This job is on the [sig-release-master-blocking dashboard](https://k8s-testgrid.appspot.com/sig-release-master-blocking),
and prevents us from cutting [WHICH RELEASE] (kubernetes/sig-release#NNN). Is there work ongoing to bring this job back to green?

https://k8s-testgrid.appspot.com/sig-sig-release-master-blocking#[THE FAILING JOB]

...any additional debugging info I can offer...
```
Examples:
- [[job failure] ci-kubernetes-e2e-kubeadm-gce](https://github.com/kubernetes/kubernetes/issues/54905)

If a test case is failing, I file a v1.y milestone issue in kubernetes/kubernetes titled: `[e2e failure] full test case name`
```
/priority critical-urgent
/priority failing-test
@kubernetes/sig-FOO-test-failures

This test case has been failing [SINCE WHEN OR FOR HOW LONG] and affects [WHICH JOBS]: [triage report](LINK TO TRIAGE REPORT)

This is affecting [WHICH JOBS] on the [sig-release-master-blocking dashboard](https://k8s-testgrid.appspot.com/sig-release-master-blocking), 
and prevents us from cutting [WHICH RELEASE] (kubernetes/sig-release#NNN). Is there work ongoing to bring this test back to green?

...any additional debugging info I can offer...

eg: if there's a really obvious single cluster / cause of failures
[triage cluster 12345](LINK TO TRIAGE CLUSTER)
---
the text from the triage cluster
---
```
Examples:
- [[e2e failure] [sig-network] Networking Granular Checks: Services [Slow] should function for client IP based session affinity: udp](https://github.com/kubernetes/kubernetes/issues/54524)
- [[e2e failure] [sig-network] Networking Granular Checks: Services [Slow] should function for client IP based session affinity: http](https://github.com/kubernetes/kubernetes/issues/54571)


### Reporting Status

I'm not sure I want to act as a human dashboard generator on an ongoing basis, so I'd like to find a better way to do this.

I have been updating the sig-release issue for the latest release each time I scrub through dashboards, eg: [1.9.0-alpha.2 issue](https://github.com/kubernetes/sig-release/issues/22#issuecomment-340970138)

## Code Slush

I check [sig-release-master-blocking](https://k8s-testgrid.appspot.com/sig-release-master-blocking) and [sig-release-master-upgrade](https://k8s-testgrid.appspot.com/sig-release-master-upgrade) daily.

I will ocasionally check [sig-release-1.y-blocking](https://k8s-testgrid.appspot.com/sig-release-1.y-blocking) and work with the Test Infra role to make sure the list of jobs looks correct

## Code Freeze

I check [sig-release-master-blocking](https://k8s-testgrid.appspot.com/sig-release-master-blocking) and [sig-release-master-upgrade](https://k8s-testgrid.appspot.com/sig-release-master-upgrade) daily.

I expect more responsiveness from SIG's at this point. In addition to filing github issues, I start pinging issues for updates, and pinging sig slack channels.

## Exit Code Freeze

I stop looking at release-master-blocking

I look solely at release-1.y-blocking and release-master-upgrade
