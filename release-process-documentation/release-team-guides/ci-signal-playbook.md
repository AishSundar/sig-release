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
- Makes recommendations to Sig-Release for promoting and demoting Release blocking and Merge blocking tests as per the [Blocking tests Proposal](https://docs.google.com/document/d/1kCDdmlpTnHPQt5z8JzODdFCc3T2D4MKR53twsDZu20c/edit)

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

## Overview of tasks across release timeline

For any release, its schedule and activities/deliverable for each week will be published in the release directory, e.g: [1.11 schedule](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.11/release-1.11.md#timeline). This section talks about specific CI Signal lead deliverable for each milestone in the release cycle.

### Pre Feature Freeze
Here are some good early deliverables from the CI Signal lead between start of release and feature freeze
- Maintain a master tracking sheet and keep it up-to-date with  issues tracking any test issue (failure/flake) - [Sample sheet](https://docs.google.com/spreadsheets/d/1j2K8cxraSp8jZR2S-kJUT6GNjtXYU9hocNRiVUGZWvc/edit#gid=127492362)
- Copy over any open test issues from previous release (ask previous CI Signal lead for the tracker sheet) and follow up on them with owners
- Monitor [master-blocking](https://k8s-testgrid.appspot.com/sig-release-master-blocking) and [master-upgrade](https://k8s-testgrid.appspot.com/sig-release-master-upgrade) dashboards **twice a week** and ensure all failures and flakes have issues open
  - Make sure all open issues have a kind/bug, priority/flake or priority/failing-test, priority/important-soon labels 
  - Make sure the issue is assigned against the current milestone 1.xx, using /milestone
  - Assign the issue to appropriate SIG using /sig label
  - Add @kubernetes/sig-foo-test-failures to get SIG’s attention on the issue
  - CC the release manager and bug triage lead
  - Draw SIG's attention to the test failure by post in SIG’s Slack channel. This can also help routing the issue to the rightful owner(s) in the SIG
  - [Sample test failure issue](https://github.com/kubernetes/kubernetes/issues/63611)
- Build and maintain a document of area experts / owners across SIGs for future needs eg: Scalability experts, upgrade test experts etc
  
#### **_Best Practice:_** 
The SLA and involvement of signal lead at this stage might vary from release to release (and the CI Signal lead). However in an effort to establish a baseline of the test health, the signal lead can take an initial stab at the tests runs at the start of the release, open issues, gather and report on the current status. Post that, it might suffice to check on the tests **twice a week** due to high code churn and expected test instability.

### Feature Freeze to Code Freeze
Day to day tasks remain pretty much the same as before, with the following slight changes
- Post 1.xx.0-beta.0 release, [master-blocking](https://k8s-testgrid.appspot.com/sig-release-master-blocking) and [master-upgrade](https://k8s-testgrid.appspot.com/sig-release-master-upgrade) dashboards **every other day**.
- Starting now, curate and send a **Weekly CI Signal report** escalating current test failures and flakes to the entire kubernetes-dev community. Such conitnuous and early reporting of test health went a long way in rallying SIG attention to test failures and stabilizing the release much early in the release cycle.
  - 

### Tips and Tricks when checking test dashboards

- Quirk: if a job is listed as FAILING but doesn't have "Overall" as one of its ongoing failures, it's not actually failing. It might be "red" from some previous test runs failures and will clear up after a few "green" runs
- if a job is failing in one of the meta-stages (Up, Down, DumpClusterLogs, etc), find the owning SIG since it is a infra failure
- if a job is failing because a specific test case is failing, and that test case has a [sig-foo] in its name, tag sig-foo in the issue and find appropriate owner within the sig
- with unit test case failures, try to infer the SIG from the path or OWNERS files in it. Otherwise find the owning SIG to help
- with verify failures, try to infer the failure from the log. Otherwise find the owning SIG to help
- if a test case is failing in one job consistently, but not others, both the job owner and test case owner are responsible for identifying why this combination is different

### Routing test issues to SIG/owner

This can be done in 2 ways (i) including @kubernetes/sig-foo-test-failures teams in the issue files and (ii) going to the #sig-foo slack channel if you need help routing the failue to appropriate owner.

If a job as a whole is failing, file a v1.y issue in kubernetes/kubernetes titled:`[job failure] job name issue`
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

If a test case is failing, file a v1.y milestone issue in kubernetes/kubernetes titled: `[e2e failure] full test case name`
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
