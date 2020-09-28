<!--
**Note:** When your KEP is complete, all of these comment blocks should be removed.

To get started with this template:

- [ ] **Pick a hosting SIG.**
  Make sure that the problem space is something the SIG is interested in taking
  up. KEPs should not be checked in without a sponsoring SIG.
- [ ] **Create an issue in kubernetes/enhancements**
  When filing an enhancement tracking issue, please make sure to complete all
  fields in that template. One of the fields asks for a link to the KEP. You
  can leave that blank until this KEP is filed, and then go back to the
  enhancement and add the link.
- [ ] **Make a copy of this template directory.**
  Copy this template into the owning SIG's directory and name it
  `NNNN-short-descriptive-title`, where `NNNN` is the issue number (with no
  leading-zero padding) assigned to your enhancement above.
- [ ] **Fill out as much of the kep.yaml file as you can.**
  At minimum, you should fill in the "Title", "Authors", "Owning-sig",
  "Status", and date-related fields.
- [ ] **Fill out this file as best you can.**
  At minimum, you should fill in the "Summary" and "Motivation" sections.
  These should be easy if you've preflighted the idea of the KEP with the
  appropriate SIG(s).
- [ ] **Create a PR for this KEP.**
  Assign it to people in the SIG who are sponsoring this process.
- [ ] **Merge early and iterate.**
  Avoid getting hung up on specific details and instead aim to get the goals of
  the KEP clarified and merged quickly. The best way to do this is to just
  start with the high-level sections and fill out details incrementally in
  subsequent PRs.

Just because a KEP is merged does not mean it is complete or approved. Any KEP
marked as `provisional` is a working document and subject to change. You can
denote sections that are under active debate as follows:

```
<<[UNRESOLVED optional short context or usernames ]>>
Stuff that is being argued.
<<[/UNRESOLVED]>>
```

When editing KEPS, aim for tightly-scoped, single-topic PRs to keep discussions
focused. If you disagree with what is already in a document, open a new PR
with suggested changes.

One KEP corresponds to one "feature" or "enhancement" for its whole lifecycle.
You do not need a new KEP to move from beta to GA, for example. If
new details emerge that belong in the KEP, edit the KEP. Once a feature has become
"implemented", major changes should get new KEPs.

The canonical place for the latest set of instructions (and the likely source
of this file) is [here](/keps/NNNN-kep-template/README.md).

**Note:** Any PRs to move a KEP to `implementable`, or significant changes once
it is marked `implementable`, must be approved by each of the KEP approvers.
If none of those approvers are still appropriate, then changes to that list
should be approved by the remaining approvers and/or the owning SIG (or
SIG Architecture for cross-cutting KEPs).
-->
# KEP-127: Support User Namespaces

<!--
This is the title of your KEP. Keep it short, simple, and descriptive. A good
title can help communicate what the KEP is and should be considered as part of
any review.
-->

<!--
A table of contents is helpful for quickly jumping to sections of a KEP and for
highlighting any additional information provided beyond the standard KEP
template.

Ensure the TOC is wrapped with
  <code>&lt;!-- toc --&rt;&lt;!-- /toc --&rt;</code>
tags, and then generate with `hack/update-toc.sh`.
-->

<!-- toc -->
- [Release Signoff Checklist](#release-signoff-checklist)
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Proposal](#proposal)
  - [User Stories](#user-stories)
    - [Story 1](#story-1)
    - [Story 2](#story-2)
  - [Notes/Constraints/Caveats](#notesconstraintscaveats)
    - [Volumes Support](#volumes-support)
    - [Container Runtime Support](#container-runtime-support)
  - [Risks and Mitigations](#risks-and-mitigations)
    - [Breaking Existing Workloads](#breaking-existing-workloads)
    - [Duplicated Snapshots of Container Images](#duplicated-snapshots-of-container-images)
    - [Container Images with High IDs](#container-images-with-high-ids)
- [Implementation Phases](#implementation-phases)
  - [Phase 1](#phase-1)
  - [Future Phases](#future-phases)
- [Design Details](#design-details)
  - [Summary of the Proposed Changes](#summary-of-the-proposed-changes)
  - [PodSpec Changes](#podspec-changes)
  - [CRI API Changes](#cri-api-changes)
  - [Test Plan](#test-plan)
  - [Graduation Criteria](#graduation-criteria)
    - [Alpha -&gt; Beta](#alpha---beta)
    - [Beta -&gt; GA](#beta---ga)
  - [Upgrade / Downgrade Strategy](#upgrade--downgrade-strategy)
  - [Version Skew Strategy](#version-skew-strategy)
- [Production Readiness Review Questionnaire](#production-readiness-review-questionnaire)
  - [Feature Enablement and Rollback](#feature-enablement-and-rollback)
  - [Rollout, Upgrade and Rollback Planning](#rollout-upgrade-and-rollback-planning)
  - [Monitoring Requirements](#monitoring-requirements)
  - [Dependencies](#dependencies)
  - [Scalability](#scalability)
  - [Troubleshooting](#troubleshooting)
- [Implementation History](#implementation-history)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
  - [Differences with Previous Proposal](#differences-with-previous-proposal)
  - [Default Value for userNamespaceMode](#default-value-for-usernamespacemode)
  - [Host Defaulting Mechanishm](#host-defaulting-mechanishm)
- [References](#references)
<!-- /toc -->

## Release Signoff Checklist

<!--
**ACTION REQUIRED:** In order to merge code into a release, there must be an
issue in [kubernetes/enhancements] referencing this KEP and targeting a release
milestone **before the [Enhancement Freeze](https://git.k8s.io/sig-release/releases)
of the targeted release**.

For enhancements that make changes to code or processes/procedures in core
Kubernetes—i.e., [kubernetes/kubernetes], we require the following Release
Signoff checklist to be completed.

Check these off as they are completed for the Release Team to track. These
checklist items _must_ be updated for the enhancement to be released.
-->

Items marked with (R) are required *prior to targeting to a milestone / release*.

- [ ] (R) Enhancement issue in release milestone, which links to KEP dir in [kubernetes/enhancements] (not the initial KEP PR)
- [ ] (R) KEP approvers have approved the KEP status as `implementable`
- [ ] (R) Design details are appropriately documented
- [ ] (R) Test plan is in place, giving consideration to SIG Architecture and SIG Testing input
- [ ] (R) Graduation criteria is in place
- [ ] (R) Production readiness review completed
- [ ] Production readiness review approved
- [ ] "Implementation History" section is up-to-date for milestone
- [ ] User-facing documentation has been created in [kubernetes/website], for publication to [kubernetes.io]
- [ ] Supporting documentation—e.g., additional design documents, links to mailing list discussions/SIG meetings, relevant PRs/issues, release notes

<!--
**Note:** This checklist is iterative and should be reviewed and updated every time this enhancement is being considered for a milestone.
-->

[kubernetes.io]: https://kubernetes.io/
[kubernetes/enhancements]: https://git.k8s.io/enhancements
[kubernetes/kubernetes]: https://git.k8s.io/kubernetes
[kubernetes/website]: https://git.k8s.io/website

## Summary

<!--
This section is incredibly important for producing high-quality, user-focused
documentation such as release notes or a development roadmap. It should be
possible to collect this information before implementation begins, in order to
avoid requiring implementors to split their attention between writing release
notes and implementing the feature itself. KEP editors and SIG Docs
should help to ensure that the tone and content of the `Summary` section is
useful for a wide audience.

A good summary is probably at least a paragraph in length.

Both in this section and below, follow the guidelines of the [documentation
style guide]. In particular, wrap lines to a reasonable length, to make it
easier for reviewers to cite specific portions, and to minimize diff churn on
updates.

[documentation style guide]: https://github.com/kubernetes/community/blob/master/contributors/guide/style-guide.md
-->

Container security consists of many different kernel features that work together
to make containers secure. User namespaces isolate user and group IDs by
allowing processes to run with different IDs in the container and in the host.
Specially, a process running as privileged in a container can be
unprivileged in the host. This makes it possible to give more capabilities to the containers and
protects the host and other containers from malicious or compromised containers.

This KEP adds a new `userNamespaceMode` field  to `pod.Spec`. It allows users to
place pods in different user namespaces increasing the  pod-to-pod and
pod-to-host isolation. This extra isolation increases the cluster security as it
protects the host and other pods from malicious or compromised processes inside
containers that are able to break into the host. This KEP proposes three
different modes: `Host` keeps the current behaviour, `Cluster` uses the same
ID mapping for all the pods (very similar to the previous [Support Node-Level User
Namespaces
Remapping](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/node-usernamespace-remapping.md)
proposal) and `Pod` increases pod-to-pod isolation.

## Motivation

<!--
This section is for explicitly listing the motivation, goals and non-goals of
this KEP.  Describe why the change is important and the benefits to users. The
motivation section can optionally provide links to [experience reports] to
demonstrate the interest in a KEP within the wider Kubernetes community.

[experience reports]: https://github.com/golang/go/wiki/ExperienceReports
-->

From [user_namespaces(7)](https://man7.org/linux/man-pages/man7/user_namespaces.7.html):
> User namespaces isolate security-related identifiers and attributes, in
particular, user IDs and group IDs, the root directory, keys, and capabilities.
A process's user and group IDs can be different inside and outside a user
namespace. In particular, a process can have a normal unprivileged user ID
outside a user namespace while at the same time having a user ID of 0 inside
the namespace; in other words, the process has full privileges for operations
inside the user namespace, but is unprivileged for operations outside the
namespace.

The goal of supporting user namespaces in Kubernetes is to be able to run
processes in pods with a different user and group IDs than in the host.
Specifically, a privileged process in the pod runs as an unprivileged process in the
host. If such a process is able to break into the host, it'll have limited
impact as it'll be running as an unprivileged user there.

The following security vulnerabilities are mitigated with
user namespaces and it is expected that using them would mitigate against some of the future vulnerabilities.
- CVE-2016-8867: Privilege escalation inside containers
  - https://github.com/moby/moby/issues/27590
- CVE-2018-15664: TOCTOU race attack that allows to read/write files in the host
  - https://github.com/moby/moby/pull/39252
- CVE-2019-5736: Host runc binary can be overwritten from container
  - https://github.com/opencontainers/runc/commit/0a8e4117e7f715d5fbeef398405813ce8e88558b

### Goals

<!--
List the specific goals of the KEP. What is it trying to achieve? How will we
know that this has succeeded?
-->

- Increase node to pod isolation in Kubernetes by mapping user and group IDs
  inside the container to different IDs in the host. In particular, mapping root
  inside the container to unprivileged user and group IDs in the node.
- Make it possible to run workloads that need "dangerous" capabilities such as
  `CAP_SYS_ADMIN` without impacting the host.
- Benefit from the security hardening that user namespaces are expected to
  provide against some of the future unknown runtime vulnerabilities

### Non-Goals

<!--
What is out of scope for this KEP? Listing non-goals helps to focus discussion
and make progress.
-->

- Provide a way to run the kubelet process or container runtimes as an
  unprivileged process. Although initiatives like
  [usernetes](https://github.com/rootless-containers/usernetes) and this KEP
  both make use of user namespaces, it is a different implementation for a
  different purpose.
- Mounting volumes in pods with a user ID mapping. Although the authors of this
  KEP would like to have this feature in the future, this is out of scope of
  this KEP. The complexity of this would probably require to write a separate
  KEP.

## Proposal

<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation. The "Design Details" section below is for the real
nitty-gritty.
-->

This proposal aims to support user namespaces in Kubernetes by extending the pod
specification with a new `userNamespaceMode` field. This field can have 3 values:

- **Host**:
  The pods are placed in the host user namespace, this is the current Kubernetes
  behaviour. This mode is intended for pods that only work in the root (host)
  user namespace. It is the default mode when `userNamespaceMode` field is not
  set.

- **Cluster**:
  All the pods in the cluster are placed in a different user namespace but they
  use the same ID mappings. This mode doesn't provide full pod-to-pod isolation
  as all the pods with `Cluster` mode have the same effective IDs on the host.
  It provides pod-to-host isolation as the IDs are different inside the
  container and in the host. This mode is intended for pods sharing volumes as
  they will run with the same effective IDs on the host, allowing them to read
  and write files in the shared storage.

- **Pod**:
  Each pod is placed in a different user namespace and has a different and
  non-overlapping ID mapping. This mode is intended for stateless pods, i.e.
  pods using only ephemeral volumes like `configMap,` `downwardAPI`, `secret`,
  `projected` and `emptyDir`. This mode not only provides host-to-pod isolation
  but also  pod-to-pod isolation as each pod has a different range of effective
  IDs in the host.

### User Stories

<!--
Detail the things that people will be able to do if this KEP is implemented.
Include as much detail as possible so that people can understand the "how" of
the system. The goal here is to make this feel real for users without getting
bogged down.
-->

#### Story 1

As a cluster admin, I want run some pods with privileged capabilities because
the applications in the pods require it (e.g. `CAP_SYS_ADMIN` to mount a FUSE
filesystem or `CAP_NET_ADMIN` to setup a VPN) but I don't want this extra
capability to give any extra privilege on the host.

#### Story 2

As a cluster admin, I want to allow some pods to share the host user namespace
if they need a feature only available in such user namespace, such as loading a kernel module with `CAP_SYS_MODULE`.

### Notes/Constraints/Caveats

<!--
What are the caveats to the proposal?
What are some important details that didn't come across above?
Go in to as much detail as necessary here.
This might be a good place to talk about core concepts and how they relate.
-->

#### Volumes Support

The Linux kernel uses the effective user and group IDs (the ones the host) to
check the file access permissions. Since with user namespaces IDs are mapped to
a different value on the host, this causes issues accessing volumes if the
pod is run with a different mapping, i.e. the effective user and group IDs on
the host change.

This proposal supports volume without changing the user and group IDs and leaves
that problem to the user to manage. Future Linux kernel features such as shiftfs
could allow different pods to see a volume with its own IDs but it is out of
scope of this proposal. Among the possible future kernel solutions, we can list:

- [shiftfs: uid/gid shifting filesystem](https://lwn.net/Articles/757650/)
- [A new API for mounting filesystems](https://lwn.net/Articles/753473/)
- [user_namespace: introduce fsid mappings](https://lwn.net/Articles/812221/)

In regard to this proposal, volumes can be divided in ephemeral and non-ephemeral.

Ephemeral volumes are associated to a **single** pod and their lifecyle is
dependent on that pod. These are `configMap`, `secret`, `emptyDir`,
`downwardAPI`, etc. These kind of volumes can work with any of the three
different modes of `userNamespaceMode` as they are not shared by different pods
and hence all the processes accessing those volumes have the same effective user
and group IDs. Kubelet creates the files for those volumes and it can easily set
the file ownership too.

Non-ephemeral volumes are more difficult to support since they can be persistent
and shared by multiple pods. This proposal supports volumes with two different
strategies:
- The `Cluster` makes it easier for pods to share files using volumes when those
  don't have access permissions for `others` because the effective user and
  group IDs on the host are the same for all the pods.
- The semantics of semantics of `fsGroup` are respected, if it's specified it's
  assumed to be the correct GID in the host and a 1-to-1 mapping entry for the
  `fsGroup` is added to the GID mappings for the pod.

This KEP doesn't impose any restriction on the different volumes and
`userNamespaceMode` combinations and leaves it to users to chose the correct
combinations based on their specific needs.

There are some cases that require special attention from the user. For instance,
a process inside a pod will not be able to access files with mode `0700` and
owned by a user different than the effective user of that process in a volume
that doesn't support the semantics of `fsGroup` (doesn't support
[`SetVolumeOwnership`](https://github.com/kubernetes/kubernetes/blob/00da04ba23d755d02a78d5021996489ace96aa4d/pkg/volume/volume_linux.go#L42)
that updates permissions and ownership of the files to be accesible by the
`fsGroup` group ID). These pods should be run in `Host` mode.

#### Container Runtime Support

- **Docker**:
Docker only supports a [single ID
  mappings](https://docs.docker.com/engine/security/userns-remap/) shared by all
  containers running in the host. There is not support for [multiple ID
  mappings](https://github.com/moby/moby/issues/28593) yet. Dockershim runtime
  is only compatible with pods running in `Host` and `Cluster` modes. The user
  has to guarantee that the ID mappings configured in Docker through the
  `userns-remap` parameter and the cluster-wide range configured in the kubelet
  are the same. The dockershim implementation includes a check to verify that
  the IDs mapping received from the kubelet are equal to the ones configured in
  Docker, returning an error otherwise.
- **containerd**:
  It's quite straigtforward to implement the CRI changes proposed below in
  containerd/cri, we did it in
  [this](https://github.com/kinvolk/containerd-cri/commits/mauricio/userns_poc)
  PoC.
- **cri-o**:
CRI-O recently [added](https://github.com/cri-o/cri-o/pull/3944) support for
  user namespaces through a pod annotation. The extensions to make it work with
  the current CRI changes are small.
- TODO(Mauricio): gVisor, katacontainers?

containerd and cri-o will provide support for the 3 possible values of `userNamespaceMode`.

### Risks and Mitigations

<!--
What are the risks of this proposal, and how do we mitigate? Think broadly.
For example, consider both security and how this will impact the larger
Kubernetes ecosystem.

How will security be reviewed, and by whom?

How will UX be reviewed, and by whom?

Consider including folks who also work outside the SIG or subproject.
-->

#### Breaking Existing Workloads

Some features that don't work when the host user namespace is not shared are:

- **Some Capabilities**:
  The Linux kernel takes into consideration the user namespace a process is
  running in while performing the capabilities check. There are some
  capabilities that are only available in the root (host) user namespace such as
  `CAP_SYS_TIME`, `CAP_SYS_MODULE` & `CAP_MKNOD`.

  If a pod is given one of those capabilities it will still be deployed, but the
  capability will be ineffective and processes using those capabilities will
  fail. This is not impacting the implementation in Kubernetes. If users need
  the capability to be effective, they should use `userNamespaceMode=Host`.

  The list of such capabilities is likely to change from one Linux version to
  another. For example, Linux now has [time
  namespaces](https://man7.org/linux/man-pages/man7/time_namespaces.7.html) and
  there are ways to make `CAP_SYS_TIME` work inside a user namespace. There are
  also discussions to make `CAP_MKNOD` work in user namespaces.

- **Sharing Host Namespaces**:
  There are some limitations in the Linux kernel and in the runtimes that
  prevent sharing other host namespaces when the host user namespace is not
  shared.
  - Mounting `mqueue` (`/dev/mqueue`) is not allowed from a process in a user
    namespace that does not own the IPC namespace. Pods with `hostIPC=true` and
    `userNamespaceMode=Pod|Cluster` can fail.
  - Mounting `procfs` (`/proc`) is not allowed from a process in a user namespace
    that does not own the PID namespace. Pods with `hostPID=true` and
    `userNamespaceMode=Pod|Cluster` can fail.
  - Mounting `sysfs` (`/sys`) is not allowed from a process in a user namespace
    that does not own the network namespace. Impact: pods with
    `hostNetwork=true` and `userNamespaceMode=Pod|Cluster` can fail.

  If users specify `userNamespaceMode=Pod|Cluster` and one of these
  `host{IPC,PID,Network}=true` options, runc will currently fail to start the
  container. The kubelet does **not** try to prevent that combination of options,
  in case runc or the kernel make it possible in the future to use that
  combination.

In order to avoid breaking existing workloads `Host` is the default value of `userNamespaceMode`.

#### Duplicated Snapshots of Container Images

The owners of the files of a container image have to been set accordingly to the
ID mappings used for that container. For example, if the user 0 in the container
is mapped to the host user 100000, then the `/root` directory has to be owned by
user ID 100000 in the host, so it appears to belong to root in the container.
The current implementation in container runtimes is to recursively perform a
`chown` operation over the image snapshot when it's pulled. This presents a risk
as it potentially increases the time and the storage needed to handle the
container images.

There is not immediate mitigation for it, [we
talked](https://lists.linuxfoundation.org/pipermail/containers/2020-September/042230.html)
to the Linux kernel community and [they
replied](https://lists.linuxfoundation.org/pipermail/containers/2020-September/042230.html)
they are working on a solution for it. If the Linux kernel provides a solution
for this problem, that would be something that container runtimes should use. It
does not impact the kubelet nor the CRI gRPC spec.

Another risk is exausting the disk space on the nodes if pods are repeativily
started and stopped while using `Pod` mode. Since `Pod` mode is planned for
phase 2 we haven't considered a mitigation for this case.

#### Container Images with High IDs

There are container images designed to run with high user and group IDs. It's
possible that the IDs range assigned to the pod is not big enough to accommodate
these IDs, in this case they will be mapped to the `nobody` user in the host.

It's not a big problem in the `Cluster` case, the users have to be sure that
they provide a range accommodating these IDs. It's more difficult to handle in
the `Pod` case as the logic to allocate the ranges for each pod has to take this
information in consideration. It's likely that this requires some changes to the
CRI and kubelet so the runtimes can inform the kubelet what are the IDs present
on a specific container image.

## Implementation Phases

The implemenation of this KEP in a single phase is complicated as there are many
discussions to be done. We learned from previous attempts to bring this support in
that it should be done in small steps to avoid losing the focus on the
discussion. It's also true that a full plan should be agreed at the beginning to
avoid changing the implementation drastically in further phases.

This proposal implementation aims to be divided in the following phases:

### Phase 1

This first phase includes:
 - Extend the PodSpec with the `userNamespaceMode` field.
 - Extend the CRI with user and ID mappings fields.
 - Implement support for `Host` and `Cluster` user namespace modes.

The goal of this phase is to implement some initial user namespace support
providing pod-to-host isolation and supporting volumes. The implementation of
the `Pod` mode is out of scope in this phase because it requires a non
negligible amount of work and we could risk losing the focus failing to deliver
this feature.

### Future Phases

These phases aim to implement the `Pod` mode. After these phases are completed the
full advantanges of user namespaces can be used in some cases (stateless
workloads).

There are some things that have to be studied with more detail for these
phase(s) but are not needed for phase 1, hence they are not discussed in detail:

- **Pod Default Mode**:
  It's not clear yet what should be the process to make this happen as this is a
  potentially non backwards compatible change. It's specially relevant for
  workloads not compatible with user namespaces. A [host defaulting
  mechanishm](#host-defaulting-mechanishm) can help to make this transiction
  smoother.
- **Duplicated Snapshots of Container Images**:
  It's not clear when and how this support will land in the Linux Kernel.
- **ID Mappings Allocation Algorithm**
  The `Pod` mode requires to have each pod in different and non-overlapping ID mapping. It requires to implement an algorithm that performs that allocation. There are some open questions about it:
    - What should be the length of the mapping assigned to each Pod?
    - How to get the ID mapping range of a running Pod when kubelet crashes?
    - Can the user specify the ID mappings for a pod?
- **High IDs in Container Images**:
  The IDs present on the image are not available as image metadata. The runtimes
  would have to perform an image check, like analysing the `/etc/password` file,
  to discover what those IDs are. The kubelet and the CRI will require some
  changes to make this information available to the ID mappings allocator
  algorithm. It would have to be sure that the allocated mappings include those
  IDs and should have some logic to protect against special crafted images to
  perform a kind of DOS allocating too many IDs for a given container.
- **Security Considerations**:
  Once `Pod` is the default mode, it is needed to control who can use `Host` and
  `Cluster` modes. This can be done through Pod Security Policies if they are
  available at that time.

## Design Details

<!--
This section should contain enough information that the specifics of your
change are understandable. This may include API specs (though not always
required) or even code snippets. If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.
-->


This section only focuses on phase 1 as specified above.

### Summary of the Proposed Changes

- Extend the CRI to have a user namespace mode and the user and group ID mappings.
- Add a `userNamespaceMode` field to the pod spec.
- Add the cluster-wide ID mappings to the kubelet configuration file.
- Add a `UserNamespacesSupport` feature flag to enable / disable the user.
  namespaces support.
- Update owner of ephemeral volumes populated by the kubelet.

### PodSpec Changes

`v1.PodSpec` is extended with a new `UserNamesapceMode` field:

```
const (
	UserNamespaceModeHost    PodUserNamespaceMode = "Host"
	UserNamespaceModeCluster PodUserNamespaceMode = "Cluster"
	UserNamespaceModePod     PodUserNamespaceMode = "Pod"
)

type PodSpec struct {
...
  // UserNamespaceMode controls how user namespaces are used for this Pod.
  // Three modes are supported:
  // "Host": The pod shares the host user namespace. (default value).
  // "Cluster": The pod uses a cluster-wide configured ID mappings.
  // "Pod": The pod gets a non-overlapping ID mappings range.
  // +k8s:conversion-gen=false
  // +optional
  UserNamespaceMode PodUserNamespaceMode `json:"userNamespaceMode,omitempty" protobuf:"bytes,36,opt  name=userNamespaceMode"`
...
```

### CRI API Changes

The CRI is extended to (optionally) specify the user namespace mode
and the ID mappings for a pod.
[`NamespaceOption`](https://github.com/kubernetes/cri-api/blob/1eae59a7c4dee45a900f54ea2502edff7e57fd68/pkg/apis/runtime/v1alpha2/api.proto#L228)
is extended with two new fields:
- A `user` `NamespaceMode` that defines if the pod should run in an independent
  user namespace (`POD`) or if it should share the host user namespace
  (`NODE`).
- The ID mappings to be used if the user namespace mode is `POD`.

```
// LinuxIDMapping represents a single user namespace mapping in Linux.
message LinuxIDMapping {
   // container_id is the starting ID for the mapping inside the container.
   uint32 container_id = 1;
   // host_id is the starting ID for the mapping on the host.
   uint32 host_id = 2;
   // number of IDs in this mapping.
   uint32 length = 3;
}

// LinuxUserNamespaceConfig represents the user and group ID mapping in Linux.
message LinuxUserNamespaceConfig {
   // uid_mappings is an array of user ID mappings.
   repeated LinuxIDMapping uid_mappings = 1;
   // gid_mappings is an array of group ID mappings.
   repeated LinuxIDMapping gid_mappings = 2;
}

// NamespaceOption provides options for Linux namespaces.
message NamespaceOption {
    ...
    // User namespace for this container/sandbox.
    // Note: There is currently no way to set CONTAINER scoped user namespace in the Kubernetes API.
    // Namespaces currently set by the kubelet: POD, NODE
    Namespace user = 5;
    // ID mappings to use when the user namespace mode is POD.
    LinuxUserNamespaceConfig mappings = 6;
}
```

### Test Plan

<!--
**Note:** *Not required until targeted at a release.*

Consider the following in developing a test plan for this enhancement:
- Will there be e2e and integration tests, in addition to unit tests?
- How will it be tested in isolation vs with other components?

No need to outline all of the test cases, just the general strategy. Anything
that would count as tricky in the implementation, and anything particularly
challenging to test, should be called out.

All code is expected to have adequate tests (eventually with coverage
expectations). Please adhere to the [Kubernetes testing guidelines][testing-guidelines]
when drafting this test plan.

[testing-guidelines]: https://git.k8s.io/community/contributors/devel/sig-testing/testing.md
-->

TBD

### Graduation Criteria

<!--
**Note:** *Not required until targeted at a release.*

Define graduation milestones.

These may be defined in terms of API maturity, or as something else. The KEP
should keep this high-level with a focus on what signals will be looked at to
determine graduation.

Consider the following in developing the graduation criteria for this enhancement:
- [Maturity levels (`alpha`, `beta`, `stable`)][maturity-levels]
- [Deprecation policy][deprecation-policy]

Clearly define what graduation means by either linking to the [API doc
definition](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-versioning)
or by redefining what graduation means.

In general we try to use the same stages (alpha, beta, GA), regardless of how the
functionality is accessed.

[maturity-levels]: https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions
[deprecation-policy]: https://kubernetes.io/docs/reference/using-api/deprecation-policy/

Below are some examples to consider, in addition to the aforementioned [maturity levels][maturity-levels].

#### Alpha -> Beta Graduation

- Gather feedback from developers and surveys
- Complete features A, B, C
- Tests are in Testgrid and linked in KEP

#### Beta -> GA Graduation

- N examples of real-world usage
- N installs
- More rigorous forms of testing—e.g., downgrade tests and scalability tests
- Allowing time for feedback

**Note:** Generally we also wait at least two releases between beta and
GA/stable, because there's no opportunity for user feedback, or even bug reports,
in back-to-back releases.

#### Removing a Deprecated Flag

- Announce deprecation and support policy of the existing flag
- Two versions passed since introducing the functionality that deprecates the flag (to address version skew)
- Address feedback on usage/changed behavior, provided on GitHub issues
- Deprecate the flag

**For non-optional features moving to GA, the graduation criteria must include
[conformance tests].**

[conformance tests]: https://git.k8s.io/community/contributors/devel/sig-architecture/conformance-tests.md
-->

TBD
Mauricio: Should we require Pod mode to be implemented to switch to Beta?

#### Alpha -> Beta

- Future Complete:
  - `Pod` mode implemented

#### Beta -> GA

### Upgrade / Downgrade Strategy

<!--
If applicable, how will the component be upgraded and downgraded? Make sure
this is in the test plan.

Consider the following in developing an upgrade/downgrade strategy for this
enhancement:
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to maintain previous behavior?
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to make use of the enhancement?
-->

### Version Skew Strategy

<!--
If applicable, how will the component handle version skew with other
components? What are the guarantees? Make sure this is in the test plan.

Consider the following in developing a version skew strategy for this
enhancement:
- Does this enhancement involve coordinating behavior in the control plane and
  in the kubelet? How does an n-2 kubelet without this feature available behave
  when this feature is used?
- Will any other components on the node change? For example, changes to CSI,
  CRI or CNI may require updating that component before the kubelet.
-->

The container runtime will have to be updated in the nodes to support this feature.

The new `user` field in the `NamespaceOption` will be ignored by an old runtime
without user namespaces support. The container will be placed in the host user
namespace. It's a responsibility of the user to guarantee that a runtime
supporting user namespaces is used.

An old version of kubelet without user namespaces support can cause some
issues too. In this case the runtime can wrongly infer that the `user` field
is set to `POD` in the `NamespaceOption` message. To avoid this problem the
runtime should check if the `mappings` field contains any mappings, an error
should be raised otherwise.

## Production Readiness Review Questionnaire

<!--

Production readiness reviews are intended to ensure that features merging into
Kubernetes are observable, scalable and supportable; can be safely operated in
production environments, and can be disabled or rolled back in the event they
cause increased failures in production. See more in the PRR KEP at
https://git.k8s.io/enhancements/keps/sig-architecture/20190731-production-readiness-review-process.md.

The production readiness review questionnaire must be completed for features in
v1.19 or later, but is non-blocking at this time. That is, approval is not
required in order to be in the release.

In some cases, the questions below should also have answers in `kep.yaml`. This
is to enable automation to verify the presence of the review, and to reduce review
burden and latency.

The KEP must have a approver from the
[`prod-readiness-approvers`](http://git.k8s.io/enhancements/OWNERS_ALIASES)
team. Please reach out on the
[#prod-readiness](https://kubernetes.slack.com/archives/CPNHUMN74) channel if
you need any help or guidance.

-->

### Feature Enablement and Rollback

* **How can this feature be enabled / disabled in a live cluster?**
  - [X] Feature gate
    - Feature gate name: UserNamespacesSupport
    - Components depending on the feature gate: kubelet

* **Does enabling the feature change any default behavior?**
  The default mode for usernamespaces is `Host`, so the default behaviour is not changed.

* **Can the feature be disabled once it has been enabled (i.e. can we roll back
  the enablement)?**
  Yes, by disabling the `UserNamespacesSupport` feature gate.
  The effective user and group IDs of the process in the host would be different
  before and after disabling the feature for pods running in `Cluster` and `Pod`
  modes. This can cause access issues to pods accessing files saved in
  volumes.

* **What happens if we reenable the feature if it was previously rolled back?**
  The situation is very similar to the described above. The pod will be able to
  access the files written when the feature was enabled but can have issues to
  access those files written while the feature was disabled.

* **Are there any tests for feature enablement/disablement?**
  TBD

### Rollout, Upgrade and Rollback Planning

Will be added before transition to beta.

* **How can a rollout fail? Can it impact already running workloads?**

* **What specific metrics should inform a rollback?**

* **Were upgrade and rollback tested? Was the upgrade->downgrade->upgrade path tested?**

* **Is the rollout accompanied by any deprecations and/or removals of features, APIs,
fields of API types, flags, etc.?**


### Monitoring Requirements

Will be added before transition to beta.

* **How can an operator determine if the feature is in use by workloads?**

* **What are the SLIs (Service Level Indicators) an operator can use to determine
the health of the service?**

* **What are the reasonable SLOs (Service Level Objectives) for the above SLIs?**

* **Are there any missing metrics that would be useful to have to improve observability
of this feature?**

### Dependencies

* **Does this feature depend on any specific services running in the cluster?**: No.

### Scalability

* **Will enabling / using this feature result in any new API calls?** No.

* **Will enabling / using this feature result in introducing new API types?** No.

* **Will enabling / using this feature result in any new calls to the cloud
provider?** No.

* **Will enabling / using this feature result in increasing size or count of
the existing API objects?** Yes. The PodSpec will be increased. TODO(Mauricio): what is the increased size?

* **Will enabling / using this feature result in increasing time taken by any
operations covered by [existing SLIs/SLOs]?**
  Yes. The runtime has to set correct ownership for the container image
  before starting it.
  TODO(Mauricio): check what are those SLIs/SLOs and if this case actually applies.

* **Will enabling / using this feature result in non-negligible increase of
resource usage (CPU, RAM, disk, IO, ...) in any components?**: No.

### Troubleshooting

Will be added before transition to beta.

* **How does this feature react if the API server and/or etcd is unavailable?**

* **What are other known failure modes?**

* **What steps should be taken if SLOs are not being met to determine the problem?**

## Implementation History

<!--
Major milestones in the lifecycle of a KEP should be tracked in this section.
Major milestones might include:
- the `Summary` and `Motivation` sections being merged, signaling SIG acceptance
- the `Proposal` section being merged, signaling agreement on a proposed design
- the date implementation started
- the first Kubernetes release where an initial version of the KEP was available
- the version of Kubernetes where the KEP graduated to general availability
- when the KEP was retired or superseded
-->

## Drawbacks

<!--
Why should this KEP _not_ be implemented?
-->

TBD:
Some ideas
- another configuration knob is added
- user namespaces could make troubleshooting difficult
- volumes are really trickly to handle
- any performance issues?

## Alternatives

<!--
What other approaches did you consider, and why did you rule them out? These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->

### Differences with Previous Proposal
Even if this KEP is heavily based on the previous [Support Node-Level User
Namespaces
Remapping](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/node-usernamespace-remapping.md)
proposal there are some big differences:
- The previous proposal aimed to configure the ID mappings in the container
  runtime instead of the kubelet. In this proposal this decision is made in the
  kubelet because:
  - It has knowledge of Kubernetes elements like volumes, pods, etc.
  - Runtimes will be more simple as they don't have to implement logic to
    allocate non-overlapping ID mappings.
  - We keep the behaviour consistent among runtimes as kubelet will be the one
    ordering what to do.
- That proposal didn't consider having different ID mappings for each pod. Even
  if it's not planned for the first phase, this KEP takes that into
  consideration performing the needed changes in the CRI from the beginning.

### Default Value for userNamespaceMode

This proposal intends to have `Host` instead of `Pod` as default value for the
user namespace mode. The rationale behind this decision is that it avoids
breaking existing workloads that don't work with user namespaces. We are aware
that this decision has the drawback that pods that have the `userNamespaceMode`
set will not have the security advantages of user namespaces but we consider
it's more important to keep compatibility with previous workloads.

### Host Defaulting Mechanishm

Previous proposals like [Node-Level UserNamespace
implementation](https://github.com/kubernetes/kubernetes/pull/64005) had a
mechanism to default to the host user namespace when the pod specification includes
features that could be not compatible with user namespaces (similar to [Default host user
namespace via experimental
flag](https://github.com/kubernetes/kubernetes/pull/31169)).

This proposal doesn't require a similar mechanishm given that the default mode
is `Host` that works with all current existing workloads.

## References

- Support Node-Level User Namespaces Remapping design proposal document.
  - https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/node-usernamespace-remapping.md
- Node-Level UserNamespace implementation
  - https://github.com/kubernetes/kubernetes/pull/64005
- Support node-level user namespace remapping
  - https://github.com/kubernetes/enhancements/issues/127
- Default host user namespace via experimental flag
  - https://github.com/kubernetes/kubernetes/pull/31169
- Add support for experimental-userns-remap-root-uid and
  experimental-userns-remap-root-gid options to match the remapping used by the
  container runtime
  - https://github.com/kubernetes/kubernetes/pull/55707
- Track Linux User Namespaces in the pod Security Policy
  - https://github.com/kubernetes/kubernetes/issues/59152
