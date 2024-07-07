---
title: "Parallel Scans"
author: "Sean Liu"
date: 2022-08-12T03:13:37-07:00
categories: ["compsci", "scans", "competitiveprogramming"]
draft: true
---

# Introduction 
The *scan* problem, also known as the *prefix sum* problem, isn't very hard to state: 

**Scan Problem**: Given an array of $N$ integers, $a_1, a_2, a_3, \dots, a_N$, find an array $b$ of length $N$ such that $b_k = \sum_{i = 1}^k a_i$. In other words, $b_k$ is the sum of the first $k$ values of $a$. 