---
title: "Why this portfolio is shaped the way it is"
date: 2026-09-09
summary: >
  The business problem behind chamberlain, and why the portfolio composes existing
  tools instead of inventing new ones.
---

I've spent 14 years solving the same problem in different clothes: a self-service
report builder at Nielsen, connector APIs at Voyager, channel provisioning and
session vending at Prime Video, a self-service deploy API at Cotiviti. Different
domains, same shape — **remove the human gatekeeper by turning toil into an API.**

This portfolio is that problem, stated once, on purpose: *a team wants to ship a
workload; today that takes a platform engineer, an SRE, a security review, and a
week.* `chamberlain` is a control plane that does it safely instead.

## What this portfolio is not

It composes OPA, OpenBao, Temporal, and Postgres rather than reinventing any of
them. The work is the boundary — the guarantee the whole system makes that none of
its components make alone.

## What's next

Nothing is built yet. This post exists before the code does, deliberately.
