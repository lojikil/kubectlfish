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
 -->
- go static analysis tools are a mixed bag
- minimal tooling specifically for k8s **at a code level**
- enormous code base
- with lots of components, styles, companies...

---

# the how

- mainly manual: lots of `ack`, some code indexing
- 
