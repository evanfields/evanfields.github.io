---
layout: post
title: The diet problem, personalized
---

## Overview

Recently I've been curious about applying a variation of the [diet problem](https://en.wikipedia.org/wiki/Stigler_diet) to my own life. 
The diet problem is a classic linear programming problem - predating the Simplex algorithm! - which attempts to find the cheapest diet (consisting of a pre-specified list of foods) subject to various minimum nutrition constraints. 

This post describes some of my experiments in exploring a modified version of the classic diet problem: instead of looking for the cheapest diet subject to nutrition constraints, I want to try to find the tastiest diet subject to nutrition constraints. I intend to show my process in this post, rather than just giving a final formulation and solution. That means along the way we'll encounter some mistakes I made.

First, the motivation: why the tastiest diet, rather than the cheapeast? Many (most?) of my hobbies revolve around food, and a solid portion of my discretionary spending goes toward interesting food and drink. This suggests that finding the cheapest diet isn't that useful for me: even if I were to implement it, I'd then have leftover money that I wanted to spend on...food. Further, food is really cheap in the United States these days. For example, enriched whole wheat flour can pretty easily be found on the order of \\$1/pound. That means daily caloric needs can be met easily - sans optimization or deal hunting - for roughly a dollar a day. Throw in half a head of cabbage and for \\$2 a day you can have plenty of servings of vegetables, all the calories and protein you need, as well as somewhere north of 200% of your daily recommended fiber intake. [1]

So it seems given my personal preferences and the availability of cheap food, merely optimizing price is the wrong way to go. We'll start with optimizing tastiness, and later we can include price as necessary, perhaps as a constraint or additional term in the objective function.

## General formulation, high level

We'll gather data on eligible foods and index these foods `1...n`. For each food `i` let `t_i` be the tastiness of food `i` and `x_i >= 0` be a decision variable indicating how much of food `i` I should eat per week. Why per week? Ease of interpretability; a solution that says to eat .14 slices of cake per day seems less useful than one suggesting 1.0 slices per week. Then our objective should be `Problem 1: max t^T x such that x >= 0`, i.e. the sum across all foods of tastiness times amount eaten.

There's an immediate problem: this formulation is unbounded; we need some constraints preventing me from eating [malted chocolate chip cookies](http://www.seriouseats.com/2017/02/how-to-use-barley-malt-syrup-to-make-cookies.html) all day every day. The most obvious constraint is a limit on total calories [2] consumed. My current caloric goal is 1900 calories per day, so 13300 calories per week. Let `c_i` be the caloric content of food `i`. This modifies the problem to
```
Problem 2:
max t^T x
st  c^T x <= 13300
    x >= 0
```
Now the problem is well-posed. Here is the very limited set of foods I started with:
```
| Name                            | IsFood | IsIngredient | Size                    | FruitVegServings | FatG        | CarbG       | ProteinG    | FiberG      | Calories    | Tastiness |
|---------------------------------|--------|--------------|-------------------------|------------------|-------------|-------------|-------------|-------------|-------------|-----------|
| Egg                             | 1      | 1            | 1 egg                   | 0                | 6           | 0.5         | 6           | 0           | 80          | 10        |
| Milk (skim)                     | 1      | 1            | 240g                    | 0                | 0           | 12          | 8           | 9           | 86          | 4         |
| AP flour                        | 0      | 1            | 30g                     | 0                | 0.3         | 23          | 3           | 0.8         | 105         | 8         |
| WWW flour                       | 0      | 1            | 30g                     | 0                | 0.5         | 21          | 4           | 3           | 100         | 6         |
| butter                          | 1      | 1            | 14g                     | 0                | 11.11111111 | 0           | 0           | 0           | 100         | 15        |
| olive oil                       | 1      | 1            | 14g                     | 0                | 13.33333333 | 0           | 0           | 0           | 120         | 12        |
| chicken thigh                   | 1      | 1            | 1 thigh, 240g with bone | 0                | 29          | 0           | 33          | 0           | 400         | 12        |
| apple, granny smith             | 1      | 1            | 1 medium, 200g          | 1                | 0           | 24          | 1           | 5           | 103         | 8         |
| banana                          | 1      | 1            | 1 medium, 185g          | 1                | 0.4         | 27          | 1.3         | 3.1         | 105         | 6         |
| baby carrots                    | 1      | 1            | 100g                    | 0.75             | 0           | 8.2         | 0.6         | 1.8         | 35          | 6         |
| pancake, www                    | 1      | 0            | 1 regular               | 0                | 22.6337037  | 49.475      | 16.69333333 | 6.18        | 446.6166667 | 15        |
| buttermilk                      | 0      | 1            | 100g                    | 0                | 0.9         | 5           | 3.4         | 0           | 41          | 11        |
| chocolate chip (G60%)           | 1      | 1            | 20g                     | 0                | 7.5         | 10          | 1.3         | 1.3         | 100         | 14        |
| peach                           | 1      | 1            | 1 large, 180g           | 1                | 0.4         | 17.3        | 1.6         | 2.6         | 68          | 14        |
| raw fruit, general              | 1      | 1            | 1 piece                 | 1                | 0.266666667 | 22.76666667 | 1.3         | 3.566666667 | 92          | 9         |
```
Solving [3] Problem 2 gives a solution of 195.59 peaches per week. I like peaches a whole lot - I'm from Georgia - but that doesn't seem right! Peaches aren't even the tastiest food in the list; e.g. butter is tastier. However, of the above foods, peaches do have the most tastiness per calorie. The subjective tastiness ratings I gave each food reflect how enjoyable a given food is to eat, not how long it takes to eat (how long the enjoyment lasts) or how full the food makes me feel (how long until I need to consume more of the caloric budget).

An easy adjustment is to weight each food's tastiness by its caloric content. This captures the intuition that more caloric foods are more filling and take longer to eat than less caloric foods, so the tastiness contained in more caloric foods can be enjoyed for longer. It isn't perfect - how fast a food is eaten and how filling it is depend on more than just that food's calorie content - but it's a good start. This gives us an updated problem:
```
Problem 3:
max (c*t)^T x / 13300
st  c^T x <= 13300
    x >= 0
```
where `(c*t)` is element-wise multiplication of `c` and `t`. Now the objective is not total tastiness but the average tastiness per calorie, so that eating 100cal of a tastiness 10 food is better than eating two servings of a 50cal tastiness 5 food.

With this modification, the optimal solution is to eat just shy of 30 (large) pancakes per week. Getting there - pancakes are delicious - but still not a reasonable diet.

## Additional health constraints

Health is complicated [citation needed]; presumably simple calorie regulation is not sufficient. Based off my total layman's understanding of nutrition science, I've added the following constraints:

- At least 25g of fiber per day. I increasingly read that fiber is extremely important, especially for microbiome health. I don't think that's a controversial claim, but see e.g. "If Our Bodies Could Talk" by [James Hamblin](https://www.theatlantic.com/author/james-hamblin/) for a recent and humorous take on this and other health topics.
- At least 4 servings of fruits or vegetables per day. Probably this should be higher.
- At least 60g of protein per day. Some sources - particularly those targetting males looking to increase their muscle mass -ecommend way more, but this is a pretty middle of the road mainstream recommendation.

Note that I haven't added any constraints on minimum fat and carbohydrates. See e.g. [here](http://www.stephanguyenet.com/meta-analysis-impact-of-carbohydrate-vs-fat-calories-on-energy-expenditure-and-body-fatness/) for a writeup of a meta-analysis comparing fat-heavy and carbohydrate-heavy diets. Anyway, given my dietary preferences for cookies and ice cream, neither should be a problem. Let `fiber_i`, `protein_i`, and `fruitveg_i` indicate the respective fiber, protein, and fruit/vegetable contents of each food.
```
Problem 4:
max (c*t)^T x / 13300
st  c^T x <= 13300
    fiber^T x >= 175
    protein^T x >= 420
    fruitveg^T x >= 28
    x >= 0
```
Solving problem 4 gives a solution of 28 peaches and 25.5 pancakes per week. We're approaching a diet both reasonable and delicious, but we're not there yet. 

## Diversity of foods

One problem with eating only peaches and pancakes: it gets boring, even though both foods are great. Variety is the spice of life, etc. The formulation in problem 4 pushes the solution towards only the most efficient foods. For example, peaches provide more protein and fiber per calorie than do bananas _and_ are tastier. So an optimal solution to problem 4 can never contain any bananas.

We can partially address this by introducing a binary indicator `z_i` for each food `i`. `z_i` will be 1 if and only if the solution includes eating at least one serving of food `i` per week. So the more foods I eat at least one serving of per week, the greater `sum(z)` will be. We can add a multiple of this sum to the objective value to serve as an encouragement towards diversity. Let's also make `x` integer to avoid fractional servings.
```
Problem 5:
max (c*t)^T x / 13300 + L * sum(z)
st  c^T x <= 13300
    fiber^T x >= 175
    protein^T x >= 420
    fruitveg^T x >= 28
    z <= x
    x >= 0 integer
    z binary
```
Note that there's a natural intepretation of the diversity factor `L`: it's the tastiness "bump" we give to a food which hasn't yet been included in the diet yet. Solving problem 5 with `L = 1` gives a really interesting solution: per week, 25 peaches, 21 pancakes, 11 tablespoons of butter, and one serving each of egg, milk, olive oil, chicken thigh, granny smith apple, banana, baby carrots, chocolate chips, and other raw fruit.

What's happening here? Lots of foods are worth including just enough of to get the diversity bonus, but for most of these foods a second serving per week doesn't make sense: the second serving doesn't increase the diversity score, so those calories should be spent on one of the most delicious foods.

This is the most reasonable diet yet, but still not practical. I think there are two main problems. Firstly, I just haven't provided data on enough foods. Where are the roasted vegetables, the salads, the potatoes, the brownies, the cakes, the macarons, etc? Secondly, the notion of diversity in problem 5 doesn't quite capture intuition correctly. In problem 5, any food eaten contributes to the diveristy of the diet, no matter how much is eaten. So eating 7 apples, 7 bananas, and 7 peaches per week would count as no more diverse than eating 1 apple, 1 banana, and 19 peaches per week.

A future post will try to fix these problems as well as explore the impacts of adding some noise to the data.

[1] Gastronomically and practically, I do not recommend this diet.

[2] More properly, kilocalories, sometimes written Calories. Apparently calories are a deprecated unit, but the kilocalorie is commonly used in the States as a measure of food energy content. For readability and familiarity, I'll stick with writing "calories."

[3] I use [JuMP](https://github.com/JuliaOpt/JuMP.jl) to specify the problem. The solver is Gurobi, which I use with an academic license.
