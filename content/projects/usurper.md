---
title: "usurper"
date: 2026-09-09
order: 2
params:
  repo: "https://github.com/tchaturvedi/usurper"
  status: "in-progress"
summary: >
  A deterministic fault-injection harness. Kills processes mid-transaction,
  partitions the network, skews clocks, and attempts confused-deputy attacks
  against chamberlain — then checks the result against declared invariants.
---

## What this is

`usurper` plays the adversary against chamberlain: every fault it injects is framed
as an attempt to seize authority the system didn't grant — a killed leader is a
chance to claim leadership illegitimately, a network partition is a chance to act
on stale authority, a forged request is an attempt to reach another tenant's
resources. The harness checks whether each attempt actually succeeds.

## Honest status, as of this page

Nothing is running yet — `usurper` starts once chamberlain has real behavior to
break. This page will fill in with the count of real bugs it found, which is the
actual headline number for this project, once there's a result to report.

Check the [repo](https://github.com/tchaturvedi/usurper) for current state.
