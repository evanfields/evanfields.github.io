---
layout: post
title: Allocating Electoral College Seats
---

I was thinking about the 2016 U.S. Presidential election - unfortunately, that happens a lot these days. The number of electors allocated to each state depends on that state's population, but the allocation is only updated after a census. I was curious how the allocation of electoral seats would change and whether that would have made the Electoral College result closer (spoilers: yes, but not by much).

Apparently U.s. House seats (and by extension Electoral College seats) are allocated using the [Huntington-Hill method](https://en.wikipedia.org/wiki/Huntington–Hill_method). This isn't something I know anything about. For this thought experiment, I did the updated allocation using a small mixed integer program.

Deciding how to allocate seats is a problem more philosophical than mathematical. Is the goal for each elector to represent about the same number of people? To limit under and/or over representation? How do you balance a few people being underrepresented by a lot versus a lot of people being underrepresented by a little? I don't have more than a rough intuitive answer to these questions.

A particular allocation of seats would seem to be bad to the extent that some people are denied political agency. If, for example, Colorado were given no seats in the Electoral College, that would seem unfair to the people of Colorado. So for this problem I set the objective as minimizing underrepresentation. 

The population data I used is also from [Wikipedia](https://en.wikipedia.org/wiki/List_of_U.S._states_and_territories_by_population) (on-a-whim experiments = easiest data available) and represents state-level population as of July 1st, 2016. For each state [1] $i$ let $p_i$ be the state's population and $f_i$ be the proportion of the population living in that state, i.e. $f_i = p_i / \sum_j p_j$. For each state $i$, I denote by $c_i$ the number of seats allocated in the Electoral College. Ideally, $\frac{c_i}{538} = f_i$ for every state, but of course this isn't possible. For each state, the underrepresentation is $\left(f_i - \frac{c_i}{538}\right)_+$, ie 0 if the state has at least as many seats as its population would suggest, and the difference between its population share and its electoral seat share if the former is larger.

Because some states have much larger populations than others, it's appropriate to weight each state's underrepresentation by its population. Intuitively, it's better for 100,000 people to be slightly underrepresented than for 1,000,000 people to be underrepresented by the same amount. So the objective will be to minimize $$\sum\limits_{i=1}^{51} p_i \left(f_i - \frac{c_i}{538}\right)_+$$.

That last term is nonlinear in the decision variables $c$, but we can use a standard modeling trick to make everything linear. The formulation is actually quite small:

$$
\begin{align*}
\underset{c,u}{\text{minimize}}\quad & \sum_{i=1}^{51} p_i u_i \\
\text{such that } & u_i \geq f_i - \frac{c_i}{538} \quad \forall i \\
& c_i \geq 0 \\
& c_i \text{ integer} \\
& u_i \geq 0
\end{align*}
$$

[1] Counting D.C. as a state, which is why $i$ goes from 1 to 51
