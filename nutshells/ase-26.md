---
layout: post
title: "ASE'26: research paper"
categories: new publication
paper-title: 'Not In My Git Yard: Catching Backdoors at Commit and Release Time'
topic: fuzzing, dynamic analysis, backdoors, continuous integration
pdf: /assets/publications/papers/2026-ase.pdf
date: 2026-09-01
---

## Motivation

Recent attacks against the **software supply chain**, such as the infamous
[XZ Utils backdoor](https://research.swtch.com/xz-timeline), have shown that it is possible to
inject subtle backdoors into projects at the core of modern dependency networks, potentially
affecting a massive number of users in one fell swoop.

Other documented attacks against the [ProFTPD](https://nvd.nist.gov/vuln/detail/CVE-2010-2010),
[vsFTPd](https://nvd.nist.gov/vuln/detail/CVE-2011-2523), and
[PHP](https://news-web.php.net/php.internals/113838) projects demonstrate that two major modes of
backdoor injection exist: malicious code changes **during development** (i.e., via a malicious
commit) and during the **software release process** (e.g., via a compromised release distribution
server or bundled external dependency).

Historically, such attacks have been detected **by chance, often only after the backdoor has made
its way into a software release**, at which point it may already have affected end users.
Systematically preventing such malicious code changes from affecting a project's codebase and
released binaries remains largely dependent on **manual code review**. However, as popular
open-source projects receive ever-increasing numbers of contributions (e.g., _pull requests_), it is
becoming increasingly difficult to reliably vet commits at scale. Downstream, Linux distributions
that package software projects for release may inherit **hundreds of thousands of lines** of code
changes per new release, which are similarly challenging to review.

Automatic techniques for detecting and blocking software revisions that introduce new
vulnerabilities already exist, for example through fuzzing integrated into Continuous Integration
(CI) pipelines. In practice, short fuzzing campaigns, such as the 10-minute runs supported by the
widely used
[CIFuzz framework](https://google.github.io/oss-fuzz/getting-started/continuous-integration), can
identify crash-inducing vulnerabilities in new commits and thereby help prevent exploitable bugs
from being integrated into the codebase. However, **backdoors typically do not cause crashes or
other readily observable failures, making them effectively undetectable using conventional
fuzzing-based approaches.**

At the other end of the software development toolchain, several tools exist for vetting binaries
(such as IoT firmware) for potential backdoors. **However, they are not compatible with the speed
and level of automation required by CI pipelines and release-time vetting.** These tools are
typically designed to assist, rather than fully replace, manual binary reverse-engineering efforts.
Even the most automated approaches, including our own fuzzing-based [ROSA](/nutshells/icse-25.html)
tool, typically generate **false positives** that must be reviewed and dismissed manually. As a
result, adopting ROSA within CI pipelines would introduce systematic and substantial development
delays, likely causing developers to ignore warnings or abandon the tool altogether.

## Proposal

In this work, we introduce **the first fuzzing-based backdoor detection approach compatible with the
speed and level of automation required for vetting code changes in CI pipelines and at release
time.** Compared to our previous [ROSA](/nutshells/icse-25.html) fuzzer for backdoor detection, the
key intuition behind Lily is to leverage **historical data** from the project's development process
to detect _backdoor injections_, prune false positives, and precisely locate suspicious backdoor
code. Specifically, Lily detects **backdoor injections** between **version pairs of a given
program**: it analyzes the project's codebase before and after a proposed code patch, ranging from a
single commit to an entirely new release, and blocks the change from being applied if it is deemed
suspicious.

Lily operates through the following successive steps:

1. **Constructing the standard behavior corpus**: Using previously generated fuzzing inputs or a
   regression test suite, Lily constructs a corpus of _standard behaviors_, approximated as sets of
   emitted system call types, referred to as _system call profiles_. This corpus characterizes the
   expected behavior of the patched program. In a "rolling" deployment setting, where Lily is
   executed for every new commit or release, the fuzzer-generated inputs from the previous run can
   be reused. By executing the patched program on these inputs and collecting the resulting system
   call profiles, Lily efficiently maintains an up-to-date corpus of standard behaviors.
2. **Vetting atypical behaviors**: Lily uses a fuzzer to explore the _patched_ version of the
   program, collecting the _system call profile_ associated with each generated input. If there is
   an exact match in the standard behavior corpus, the input is considered _safe_; otherwise, it is
   classified as _atypical_.
3. **Vetting novel behaviors**: For each _atypical_ input, Lily executes the _unpatched_ version of
   the program and collects its system call profile. If the system call profile remains unchanged,
   the input is considered safe: although its behavior is _atypical_, it appears to have
   historically occurred in the program and is therefore likely expected. If the system call profile
   changes, the input is both _atypical_ and _novel_. Most likely, this reflects an unexpected
   behavioral change introduced by the code modification, potentially indicating a backdoor.
4. **Locating suspicious code changes**: To assist human reviewers spot the suspicious code, Lily
   starts from the "raw diff" between the two program versions and refines it to the lines
   responsible for each suspicious (atypical and novel) behavior. Specifically, Lily intersects the
   "raw diff" with the _Source Line of Code (SLOC) coverage_ of the suspicious input and with the
   _system call emission sites_ corresponding to each suspicious system call type. In practice, the
   final report typically contains only a handful of lines of code to examine, even for diffs
   spanning millions of SLOCs across releases, together with the list of suspicious system call
   types.

## Experiments and results

We implemented Lily on top of the AFL++ and
[ROSA toolchain](https://github.com/binsec/rosa/tree/lily). We evaluate Lily on **545 version pairs
drawn from 13 popular open-source projects**, including both commit-level and release-level patches,
with and without backdoor injections. The injected backdoors consist of real-world and synthetic
samples from the [ROSARUM backdoor detection benchmark](https://github.com/binsec/rosarum). For each
version pair, we perform 20 independent runs with only 10 minutes of fuzzing, emulating the resource
constraints of a typical CI runner. We measure the **backdoor detection rate** on version pairs
containing injected backdoors, the **false alarm rate** on benign version pairs, and the **precision
and conciseness** of the generated **suspicious code change reports**.

Our evaluation shows that Lily reliably detects backdoors at both the commit level (**90% detection
rate**) and the release level (**83% detection rate**) while maintaining **very low false alarm
rates**, namely 0.2% for commit pairs and 4.3% for release pairs. The few false alarms that do occur
arise primarily in predictable situations, such as large merge commits or modifications to the
fuzzing harness. Furthermore, Lily's localization reports consistently enable rapid validation or
escalation to project maintainers by **reducing the manual review effort from thousands or even
millions of SLOCs to fewer than a dozen lines of code**, together with the corresponding suspicious
system call types.

Finally, we evaluate a range of adversarial strategies specifically designed to circumvent Lily's
detection mechanisms. Through both quantitative and qualitative analyses, we identify
countermeasures that either fully mitigate these attacks or substantially reduce their reliability,
thereby raising the cost and complexity of successfully evading detection.

## Further information

- Read the [paper](/assets/publications/papers/2026-ase.pdf), download the
  [artifact](https://zenodo.org/records/19337349), try out
  [Lily](https://github.com/binsec/rosa/tree/lily).
- Published in the
  [41st International Conference on Automated Software Engineering](https://conf.researchr.org/details/ase-2026/ase-2026-research-track/22/Not-In-My-Git-Yard-Catching-Backdoors-at-Commit-and-Release-Time).
