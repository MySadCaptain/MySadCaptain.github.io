---
title: MC Outline
comments: true
date: 2019-08-28
author: Matt
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Movie
    - Music
    - Gadget
layout: post
---

## 粗略的描述下MC程序做了什么
1. a summary about the whole relaxation process
2. how many scattering type we are considering
3. how to choose scattering based on scattering rate (self-scattering)
4. Band 

## 如何得到各个scattering rate

1. 通过SVM得到波函数
2. 一点关于Matrix element的介绍
3. 计算得到scattering rate
4. 给出direct recombination rate
5. 给出 non-radiative recombination rate
![](https://i.imgur.com/UTvXwRW.jpg)
## 如何计算state after scattering
step by step
> 1. choose the q of phonon based on the possibility distribution of q (which is the integral of q in $frac1tau$)
> 2. use energy conservation and momentum conservation to calculate k'
> 3. use q, k and k' to determine the angel between k and k'
> 4. use a random number to determine whether the angel is plus or minus
![](https://i.imgur.com/gIwCgzX.jpg)
## 整个程序构建
1. give initial condition
2. got free flight time
3. scatter
4. track all information
5. go back to step 2
6. until the end(500ps or exciton die)
![](https://i.imgur.com/rjEaKUG.jpg)


