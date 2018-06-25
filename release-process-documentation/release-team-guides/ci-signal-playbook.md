# CI Signal Lead Playbook

## Overview of CI Signal responsibilities

CI Signal lead assumes the responsibility of the quality gate for the release. This person is responsible for:
- Monitoring various e2e tests in sig-release dashboards ([master-blocking](https://k8s-testgrid.appspot.com/sig-release-master-blocking), [master-upgrade](https://k8s-testgrid.appspot.com/sig-release-master-upgrade), [x.y-blocking](https://k8s-testgrid.appspot.com/sig-release-1.11-blocking)) on a regular basis and providing early and continuous signals on release and test health to both Release team and SIGs
- Ensuring that all release blocking tests provide a clear Go/No-Go signal for the release
- Flagding regression as close to source as possible i.e., as soon as the offending code was merged
- Filing issues proactively for test failures and flakes, triaging issues to appropriate SIG/owner, following up on status and verifying fixes on newer test runs
- Studying patterns/history of test health from previous releases and working closely with SIGs and test owners to 
  - Understand and fix the test for the current release
  - Understand if the test needs to be release blocking
  - Advise Release team in making informed go/no-go calls 
  - Work with SIG-Testing on any possible test infra improvement to help improve test pass rate
- Making recommendations to SIG-Release for promoting and demoting release blocking and merge blocking tests as per the [Blocking tests Proposal](https://docs.google.com/document/d/1kCDdmlpTnHPQt5z8JzODdFCc3T2D4MKR53twsDZu20c/edit)

### Explicit detail is important:

- If you're looking for answer that's not in this document, please file an issue so we can keep the document current.
- Generally CI Signal lead errs on the side of filing an issue for each test failure or flake before following up with SIGs/owners. This way we dont _lose track of an issue_
- If a dashboard isn't listed here, or a test isn't on one of the listed dashboards, **_CI Signal lead is not looking at it_**

## Requirements

- Signal lead needs the ability to notify kubernetes org github teams, so must be a kubernetes org collaborator or member
  - The process to become one of these is in [our community membership ladder](https://github.com/kubernetes/community/blob/master/community-membership.md#requirements-for-outside-collaborators)
- Signal lead needs the ability to add a milestone to issues, so must be a member of the [kubernetes-pm github team](https://github.com/orgs/kubernetes/teams/kubernetes-pm/members)
  - The process to join this group is in [our community project-managers page](https://github.com/kubernetes/community/blob/master/project-managers/README.md#joining-the-group)
- Signal lead need to understand what tests matter and generally how our testing infra is wired together
  - He/she can ask previous CI Signal leads for advice
  - He/she can ask SIG-Testing for guidance

## Overview of tasks across release timeline

For any release, its schedule and activities/deliverable for each week will be published in the release directory, e.g: [1.11 schedule](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.11/release-1.11.md#timeline). This section talks about specific CI Signal lead deliverable for each milestone in the release cycle.

### Pre Feature Freeze

Here are some good early deliverables from the CI Signal lead between start of the release to feature freeze.
- Maintain a master tracking sheet and keep it up-to-date with issues tracking any test failure/flake - [Sample sheet](https://docs.google.com/spreadsheets/d/1j2K8cxraSp8jZR2S-kJUT6GNjtXYU9hocNRiVUGZWvc/edit#gid=127492362)
- Copy over any open test issues from previous release (ask previous CI Signal lead for the tracker sheet) and follow up on them with owners
- Monitor [master-blocking](https://k8s-testgrid.appspot.com/sig-release-master-blocking) and [master-upgrade](https://k8s-testgrid.appspot.com/sig-release-master-upgrade) dashboards **twice a week** and ensure all failures and flakes are tracked via open issues
  - Make sure all open issues have a kind/bug, priority/flake or priority/failing-test, priority/important-soon labels 
  - Make sure the issue is assigned against the current milestone 1.x, using /milestone
  - Assign the issue to appropriate SIG using /sig label
  - CC the release manager and bug triage lead
  - Add @kubernetes/sig-foo-test-failures to draw SIG-foo’s attention to the issue
  - Post the test failure in SIG’s Slack channel to get help in routing the issue to the rightful owner(s)
  - [Sample test failure issue](https://github.com/kubernetes/kubernetes/issues/63611)
- Build and maintain a document of area experts / owners across SIGs for future needs e.g.: Scalability experts, upgrade test experts etc
  
#### **_Best Practice:_** 
The SLA and involvement of signal lead at this stage might vary from release to release (and the CI Signal lead). However in an effort to establish an early baseline of the test health the signal lead can take an initial stab at the tests runs at the start of the release, open issues, gather and report on the current status. Post that, it might suffice to check on the tests **twice a week** due to high code churn and expected test instability.

### Feature Freeze to Code Slush

Day to day tasks remain pretty much the same as before, with the following slight changes
- Monitor [master-blocking](https://k8s-testgrid.appspot.com/sig-release-master-blocking) dashboard **daily**
- Monitor [master-upgrade](https://k8s-testgrid.appspot.com/sig-release-master-upgrade) **every other day**
- Starting now, curate a **weekly CI Signal report** escalating current test failures and flakes and send to kubernetes-dev@googlegroups.com community. [See below](#reporting-status) for more details on the report format and contents

### Code Slush to Code Freeze

This is when things really begin to ramp up in the release cycle with active bug triaging and followup on possible release blocking issues to assess release health. Day to day tasks of CI Signal lead include
- Auditing test status of master-blocking, master-upgrade and release-1.x.y-blocking dashboards on **daily basis**
- Keeping issues' status upto-date on GitHub
- Working closely with SIGs, test owners, bug triage lead and Release team in triaging, punting and closing issues
- Updating issue tracker frequently so anyone on the release team can get to speed and help followup on open issues
- Continuing to send [weekly CI signal report](#reporting-status)

### During Code Freeze

- Continue best practices from Code slush stage. Monitor master-blocking, master-upgrade and release-1.x.y-blocking dashboards on **daily basis**
- Quickly escalate any new failures to release team and SIGs
- Continuing to send [weekly CI signal report](#reporting-status)

### Exit Code Freeze

- Once 1.x.y-rc.1 release branch is cut and master re-opens for next release PRs, stop monitoring master-blocking and **focus only on master-upgrade and 1.x.y-blocking dashboards on daily basis**
- If tests have stabilized (they should have by now), stop sending weekly CI report

## Special high risk test categories to monitor

Historically there are a couple of families of test that are hard to stabilize, regression prone and pose a high risk of delaying and/or derailing a release. As CI Signal lead it is highly recommended to pay close attention and extra caution when dealing with test failures in the following areas.

### Scalability tests
Scalability testing is inherently challenging and regressions in this area are potentially a huge project risk 
- Requires lots and lots of servers running tests, and hence expensive
- Tests are long running, so especially hard/expensive/slow to resolve issues via Git bisection
- Examination of results is actually the bigger more expensive part of the situation

The following scalability jobs/tests could regress quite easily (due to seemingly unrelated PRs anywhere in k8s codebase), require constant manual monitoring/triaging and domain expertise to investigate and resolve.
- [master-blocking gci-gce-100](https://k8s-testgrid.appspot.com/sig-release-master-blocking#gci-gce-100)
- [master-blocking gce-scale-correctness](https://k8s-testgrid.appspot.com/sig-release-master-blocking#gce-scale-correctness)
- [master-blocking gce-scale-performance](https://k8s-testgrid.appspot.com/sig-release-master-blocking#gce-scale-performance)
- [release 1.xx-blocking gce-scalability-1.xx](https://k8s-testgrid.appspot.com/sig-release-1.10-blocking#gci-gce-scalability-1.10) (Post release branch cut)

The CI Signal lead should 
- Continuously monitor these tests early in the release cycle, ensure issues are filed and escalated to the Release team and right owners in SIG-Scalability (@bobwise, @shyamjvs, @wojtek-t) 
- Work with SIG-Scalability to understand if the failure is a product regression versus a test issue (flake) and in either case followup closely on a fix
- Additionally, it might help to monitor SIG-Scalability’s performance dashboard to flag if and when there is considerable performance degradation

Starting in 1.11, scalability tests are now blocking OSS presubmits. Specifically we are running performance tests on gce-100 and kubemark-500 setups. This is a step towards catching regressions sooner and stabilizing the release faster.

### Upgrade/Downgrade tests
[Upgrade tests](https://k8s-testgrid.appspot.com/sig-release-master-upgrade#Summary) are inherently tricky in the way they are setup to run and test. We run multiple  combinations that test upgrading and downgrading of
- just the master
- entire cluster
- on gce and gke
- in parallel and otherwise

We also run **skewed tests** which involve running the old k8s version’s tests i.e., 1.x-1.y tests, against the new (upcoming) version i.e., 1.x.y. Many of those tests will fail because of intentional changes in the new version, putting us in the position of back-patching tests to old versions and thus forcing us to add lots of new code to a stable release. 

The CI Signal lead should 
- Continuously monitor the [master-upgrade tests](https://k8s-testgrid.appspot.com/sig-release-master-upgrade#Summary) throughout the cycle
- If an upgrade/downgrade job is listed as FAILING, check the following and escalate accordingly
  - if "UpgradeTest", "hpa-upgrade", "ingress-upgrade" tests are failing in an upgrade/downgrade job, then indeed master/cluster is failing to upgrade or downgrade. This is potentially release blocking. Escalate to SIG-cluster-lifecycle 
  - if "UpgradeTest", "hpa-upgrade", "ingress-upgrade" tests are passing and other SIG specific tests are failing, then upgrade/downgrade completed succesfully. Such failures are historically mostly flakes, resolve in subsequent runs and hence mostly not release blocking. Nevertheless, file issues (even for flakes) and escalate to appropriate test owning SIG for triage, assessment and resolution
  - To get a better and quicker feedback on upgrade/downgrade tests, we can look at upgrade jobs that have "parallel" in their title eg: [gce-1.10-master-upgrade-cluster-parallel](https://k8s-testgrid.appspot.com/sig-release-master-upgrade#gce-1.10-master-upgrade-cluster-parallel). These complete faster and are more stable since they don't run slow long running tests. If the parallel upgrade/downgrade jobs are green, this indicates we are mostly good with respect to upgrade functionality
  - Yet another tip to assess seriousness of the failure is to see if both gce and gke counterparts of a upgrade/downgrade job are failing eg: [gce-1.10-master-upgrade-master](https://k8s-testgrid.appspot.com/sig-release-master-upgrade#gce-1.10-master-upgrade-master) and [gke-gci-1.10-gci-master-upgrade-master](https://k8s-testgrid.appspot.com/sig-release-master-upgrade#gke-gci-1.10-gci-master-upgrade-master). If both are failing then its indicative of a systemic failure in upgrade functionality. If only gke jobs are failing then it might be a GKE only issue and does not block k8s release. In this case escalate to GKE Cluster Lifecycle team  (@roberthbailey, @krousey)

## Tips and Tricks of the game

### Checking test dashboards

- Quirk: if a job is listed as FAILING but doesn't have "Overall" as one of its ongoing failures, it's not actually failing. It might be "red" from some previous test runs failures and will clear up after a few "green" runs
- if a job is failing in one of the meta-stages (Up, Down, DumpClusterLogs, etc), find the owning SIG since it is a infra failure
- if a job is failing because a specific test case is failing, and that test case has a [sig-foo] in its name, tag SIG-foo in the issue and find appropriate owner within the SIG
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

### Monitoring Commits for test failure triangulation
Yet another effective way for a signal lead to stay on top of code churn and regression is by monitoring commits to master and specific branch periodically.
- Once a day look at https://github.com/kubernetes/kubernetes/commits/master/ for master or a specific branch
- Look for any new end-to-end tests added by a PR

This helps us
- Stay aware of tests’ history and identify ownership when they start failing
- Identify possible culprit PRs incase of a regression
- Usually if the volume of things merged is very low, then that’s a signal that something is terribly broken as well

## Reporting Status
Since 1.11 we started curating and broadcasting a weekly CI Signal report to the entire kubernetes developer community highlighting
  - Top failing and flaky jobs and tests
  - New failing and flaky jobs and tests in the past week
  - Call out SIGs actively fixing and ignoring issues
  - Call out potential release blocking issues
  - Call out upcoming key dates e.g., Code slush, freeze etc
  - [Link to the previous reports](https://docs.google.com/document/d/1y044OcaKGEUgj094JH1ZxnnLRHnqi0Kq0f4ov56kvxE/edit) 
  - CCing individual SIGs (kubernetes-sig-foo@googlegroups.com) that own failing tests help in getting their attention
  
[Sample Weekly CI Signal Report](https://groups.google.com/forum/#!topic/kubernetes-dev/S9AuFMiIJkQ)

Early and continuous reporting of test health proved greatly effective in rallying SIG attention to test failures and stabilizing the release much early in the release cycle. 
