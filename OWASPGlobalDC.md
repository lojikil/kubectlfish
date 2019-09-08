# swimming with the kubectl fish
## the why, the how, the what of the CNCF Kubernetes Assessment

---
<!--
page_number: true
footer: @lojikil//Trail of Bits//swimming with the kubectl fish
-->

# `git blame` 

```
[lojikil.com]
Stefan Edwards (lojikil) is not presently logged in.

- Principal Security Consultant at Trail of Bits
- Twitter/Github/Lobste.rs: lojikil
- Works in: Defense, FinTech, Blockchain, IoT, compilers,
vCISO services
- Previous: net, web, adversary sim, &c.
- Infosec philosopher, amateur^wprofessional programming
language theorist, everyday agronomer, father.

WARNING: DEAF
WARNING: Noo Yawk
```

---

# outline
1. intro
1. outline
1. takeaways
1. the why: audit objectives
1. the how: modeling & assessing large FLOSS
1. the what: selected vulns

---

# takeaways

1. large, heterogeneous codebase 
2. large, heterogeneous devbase
3. lurking assumptions, threats, and vulnerabilities 

## github.com/trailofbits/audit-kubernetes

---

# the why: audit objectives

- first public audit of Kubernetes from the CNCF
- meant to find new bugs
- desired result? low hanging fruit & threat model

---

# the why: anti-objectives

- "red team" (adversary) lateral movement w/o novel bug
- Helm/Brigade/Whatever bugs
- cloud-provider specific bugs
- previously-known issues

---

# the why: results

- a security review: 37 findings
- a threat model: 17 findings + control analysis 
- a white paper: ergonomics of k8s

---

# the why: empower other researchers

- more-complete data flow, control analysis, & threats
  - CNCF's SIGSEC
- areas where more bugs lurk 
  - Nico Waisman's integer-width bug hunt with Semmle
- direction for k8s devs
  - Craig Ingram mapping all findings to GH issues
  - GKE mapping the findings to GKE's own controls
- future audits

---

# the how 

- easily a more interesting part
- unique tooling & ideas
- lots and lots of ~~RipIt~~ Code Reading

---

# the how: interesting stats

- subcomponents: kube-apiserver, etcd, KCM/CCM, kube-scheduler, kubelet, kube-proxy, CRI
- control families: authentication, authorization, cryptography, secrets management, networking, "multi-tenancy"
  - last one was a hard one
- against k8s 1.13.4
- All told? 2.7+ mSLoC Go, 126+ kSLoC C, &c.

---

# the how: the problems

<!-- NOTES:
 - Govet broke
 - gosec is mixed
 - Go's package management is weird...
 - it's hard to do, unlike say npm audit or cargo-audit
 - show of hands, how many have audited large code bases without tooling help?
 -->
- go static analysis tools are a mixed bag
- minimal tooling specifically for k8s **at a code level**
- enormous code base
- with lots of components, styles, companies...
- minimal dataflow, connection analysis, &c.

---

# the how: top level approach

1. Kubernetes in Action + Running Clusters
1. container sec class at SteelCon
1. "Kubernetes Clusters I have Known and Loved"
1. prep the code into `audit-kubernetes`     

---

# the how: code

- mainly manual: lots of `ack` (me) or Goland (normal people), some code indexing
- internal checklist of golang problems
- minimal: govet, gosec, errcheck 
  - actually did help to kickstart

---

# the how: code (part deux)

sidebar if I had to again:

- ~~use signal, use tor~~ use Semmle, use Custom Golang tooling
- Nico has found **hundreds** of bugs with simple searches in Semmle
- More krf

---

# the how: threat modeling

- I use a slightly-different approach 
  - Brian Glas (@infosecdad) & I have worked on for ~10 years
- Controls, Data, Gaps, Inherited Controls... all par for the course
- **NEW** Mozilla Rapid Risk Assessment (RRA) docs 

---

# the how: threat modeling FLOSS?

- what do we need in threat modeling?
  - ~~defined users~~ distributed user base
  - ~~responsible devs~~ distributed, unpaid dev base
  - ~~business direction~~ minimal overarching purpose

---

# the how: threat modeling FLOSS

1. design rough dataflow
    1. k8s docs aren't fun
    1. K8s in Action, but slightly out of date  
1. talk with developers
1. write up notes
1. report threats, control analysis, &c. 
 
---

# the how: talk with developers 

1. Pre-fill RRAs
2. Email SIGs for meetings
3. Meeting with Devs
4. Send RRA documents back to SIGs for PR
5. Iterate
6. Write Report

### https://github.com/trailofbits/audit-kubernetes/commits/master/notes/stefan.edwards/rra

---

# the what: 3 things

1. devs have widely varied backgrounds
2. Linux interfaces are non-trivial to code against
3. preponderance of (policy) choice 

---

# the what: devs

- devs come from a wide background
- what does this do?

```
        v, err := strconv.Atoi("4294967377")
        g := int32(v)
        fmt.Printf("v: %v, g: %v\n", v, g);
```

---

# the what: devs

- Issues?
  - Unsigned to Signed
  - Wrong Width: `$GO_ARCH`-specific width to 16/32/64 bits
  - Both: many flows of `strconv.Atoi` => `int16`
- devs may not have background on machine-width ints 

---

# the what: devs 

- We found a number of flows from Incorrect Widths/Sign
- Nico Waisman (of Semmle) found _many_

```
if len(s) > 1 && (s[0] == 'E' || s[0] == 'e') {
parsed, err := strconv.ParseInt(string(s[1:]), 10, 64)
	if err != nil {
		return 0, 0, DecimalExponent, false
	}
	return 10, int32(parsed), DecimalExponent, true
}
```

---

# the what: devs

logging things that shouldn't be logged, missing logging, log rotation

---

# the what: devs

file permissions 
easy to mess up, harder to fix wholistically 

---

# the what: linux

cgroups

---

# the what: linux

pid checks are not what you expect 

---

# the what: linux

seccomp is actually hard

---

# the what: policy

PSP not on by default

---

# the what: policy

NetworkPolicy is only applied if you choose a CNI that applies it

---

