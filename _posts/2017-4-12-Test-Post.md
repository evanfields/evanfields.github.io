---
layout: post
title: The diet problem, personalized.
---

## Overview

Recently I've been curious about applying a variation of the [diet problem](https://en.wikipedia.org/wiki/Stigler_diet) to my own life. 
The diet problem is a classic linear programming problem - predating even the Simplex algorithm! - which attempts to find the cheapest diet (consisting of a pre-specified list of foods) subject to various minimum nutrition constraints. 

This post describes some of my experiments in exploring a modified version of the classic diet problem: instead of looking for the cheapest diet subject to nutrition constraints, I want to try to find the tastiest diet subject to nutrition constraints. I intend to show my process in this post, rather than just giving a final formulation and solution. That means along the way we'll encounter some mistakes I made.

First, the motivation: why the tastiest diet, rather than the cheapeast? Many (most?) of my hobbies revolve around food, and a solid portion of my discretionary spending goes toward interesting food and drink. This suggests that finding the cheapest diet isn't that useful for me: even if I were to implement it, I'd then have leftover money that I wanted to spend on...food. Further, food is really cheap in the United States these days. For example, enriched whole wheat flour can pretty easily be found on the order of $1/pound. That means daily caloric needs can be met easily - sans optimization or deal hunting - for roughly a dollar a day. Throw in half a head of cabbage and for $2 a day you can have plenty of servings of vegetables, all the calories and protein you need, as well as somewhere north of 200% of your daily recommended fiber intake. [1]

So it seems given my personal preferences and the availability of cheap food, merely optimizing price is the wrong way to go. We'll start with optimizing tastiness, and later we can include price as necessary, perhaps as a constraint or additional term in the objective function.


[1] Gastronomically and practically, I do not recommend this diet.
