---
layout: post
title: Allocating Electoral College Seats
---

I was thinking about the 2016 U.S. Presidential election - unfortunately, that happens a lot these days. The number of electors allocated to each state depends on that state's population, but the allocation is only updated after a census. I was curious how the allocation of electoral seats would change if the census were performed just before the 2016 election and whether that would have made the Electoral College result closer (spoilers: yes, but not by much).

For this thought experiment, I did the updated allocation using a small mixed integer program. Integer programming is a powerful (and fun) technique, and it's well suited to modeling different notions of fairness. It's not, however, the tool used for the real allocation: apparently U.S. House seats (and by extension Electoral College seats) are allocated using the [Huntington-Hill method](https://en.wikipedia.org/wiki/Huntington–Hill_method). I know approximately nothing about fair allocation in general or the Huntington-Hill method in particular; feel free to educate me.

Deciding how to allocate seats is a problem more philosophical than mathematical. Is the goal for each elector to represent about the same number of people? To limit under and/or over representation? How do you balance a few people being underrepresented by a lot versus a lot of people being underrepresented by a little? I don't have more than a rough intuitive answer to these questions, but I think the following is at least defensible:

A particular allocation of seats would seem to be bad to the extent that some people are denied political agency. If, for example, Colorado were given only one seat in the Electoral College, that would seem unfair to the people of Colorado because they have disproportionately little political voice. So for this problem I set the objective as minimizing underrepresentation. 

The population data I used is from [Wikipedia](https://en.wikipedia.org/wiki/List_of_U.S._states_and_territories_by_population) (on-a-whim experiments = easiest data available) and represents state-level population as of July 1st, 2016. For each state[^1] $i$ let $p_i$ be the state's population and $f_i$ be the proportion of the nation's population living in that state, i.e. $f_i = p_i / \sum_j p_j$. For each state $i$, I denote by $c_i$ the number of seats allocated in the Electoral College. Ideally, $\frac{c_i}{538} = f_i$ for every state, but of course this isn't possible. For each state, the underrepresentation is $\left(f_i - \frac{c_i}{538}\right)_+$, i.e. 0 if the state has at least as many seats as its population would suggest, otherwise the difference between its population share and its electoral seat share.

Because some states have much larger populations than others, it's appropriate to weight each state's underrepresentation by its population. Intuitively, it's better for 100,000 people to be slightly underrepresented than for 1,000,000 people to be underrepresented by the same amount. So the objective will be to minimize $$\sum\limits_{i=1}^{51} p_i \left(f_i - \frac{c_i}{538}\right)_+$$.

That last term is nonlinear in the decision variables $c$, but we can use a standard modeling trick to make everything linear. The formulation is actually quite small:

$$
\begin{align*}
\underset{c,u}{\text{minimize}}\quad & \sum_{i=1}^{51} p_i u_i \\
\text{such that}\quad & u_i \geq f_i - \frac{c_i}{538} \quad \forall i \\
& c_i \geq 3 \quad \forall i\\
& c_i \text{ integer} \\
& u_i \geq 0
\end{align*}
$$

Notice that variable $u_i$ is greater than both 0 and the misrepresentation of state $i$. If state $i$ is overrepresented, then $u_i$ will be 0 (since we minimize with a positive coefficient on $u_i$), and if state $i$ is underrepresented, then $u_i$ will be the underrepresentation. This problem is pretty compact; on my aging computer, Gurobi solves it in 0.14 seconds. Nice!

Here's the comparison between the current and suggested allocation:
```
|  State                 |  Population  |  Current seats  |  Proposed seats  | 
|------------------------|--------------|-----------------|------------------| 
|  Alabama               | 4863300      | 9               | 8                | 
|  Alaska                | 741894       | 3               | 3                | 
|  Arizona               | 6931071      | 11              | 11               | 
|  Arkansas              | 2988248      | 6               | 3                | 
|  California            | 39250017     | 55              | 66               | 
|  Colorado              | 5540545      | 9               | 9                | 
|  Connecticut           | 3576452      | 7               | 3                | 
|  Delaware              | 952065       | 3               | 3                | 
|  District of Columbia  | 681170       | 3               | 3                | 
|  Florida               | 20612439     | 29              | 35               | 
|  Georgia               | 10310371     | 16              | 17               | 
|  Hawaii                | 1428557      | 4               | 3                | 
|  Idaho                 | 1683140      | 4               | 3                | 
|  Illinois              | 12801539     | 20              | 22               | 
|  Indiana               | 6633053      | 11              | 11               | 
|  Iowa                  | 3134693      | 6               | 3                | 
|  Kansas                | 2907289      | 6               | 3                | 
|  Kentucky              | 4436974      | 8               | 7                | 
|  Louisiana             | 4681666      | 8               | 7                | 
|  Maine                 | 1331479      | 4               | 3                | 
|  Maryland              | 6016447      | 10              | 10               | 
|  Massachusetts         | 6811779      | 11              | 11               | 
|  Michigan              | 9928301      | 16              | 17               | 
|  Minnesota             | 5519952      | 10              | 9                | 
|  Mississippi           | 2988726      | 6               | 3                | 
|  Missouri              | 6093000      | 10              | 10               | 
|  Montana               | 1042520      | 3               | 3                | 
|  Nebraska              | 1907116      | 5               | 3                | 
|  Nevada                | 2940058      | 6               | 3                | 
|  New Hampshire         | 1334795      | 4               | 3                | 
|  New Jersey            | 8944469      | 14              | 15               | 
|  New Mexico            | 2081015      | 5               | 3                | 
|  New York              | 19745289     | 29              | 33               | 
|  North Carolina        | 10146788     | 15              | 17               | 
|  North Dakota          | 757952       | 3               | 3                | 
|  Ohio                  | 11614373     | 18              | 19               | 
|  Oklahoma              | 3923561      | 7               | 6                | 
|  Oregon                | 4093465      | 7               | 6                | 
|  Pennsylvania          | 12802503     | 20              | 22               | 
|  Rhode Island          | 1056426      | 4               | 3                | 
|  South Carolina        | 4961119      | 9               | 8                | 
|  South Dakota          | 865454       | 3               | 3                | 
|  Tennessee             | 6651194      | 11              | 11               | 
|  Texas                 | 27862596     | 38              | 47               | 
|  Utah                  | 3051217      | 6               | 3                | 
|  Vermont               | 624594       | 3               | 3                | 
|  Virginia              | 8411808      | 13              | 14               | 
|  Washington            | 7288000      | 12              | 12               | 
|  West Virginia         | 1831102      | 5               | 3                | 
|  Wisconsin             | 5778708      | 10              | 9                | 
|  Wyoming               | 585501       | 3               | 3                | 
```
Or, in graphical form:

![state-level EV changes]({{ site.baseurl }}/images/states_delta.svg "Changes in electoral seats")

Roughly, it seems like big states get more seats and medium-sized states get fewer. Small states are already held up by the floor of 3 seats (two senators and a representative), so they don't lose any seats.

Under this electoral assignment (and assuming no faithless electors), Clinton would have received 237 electoral votes - actually pretty close to the 232 (again, ignoring faithless electors) she actually received. Like in 2000, Florida would have been enough to swing the election, and instead of talking about ~50,000 people in Wisconsin, Michigan, and Pennsylvania, we'd be talking about ~40,000 people in just Michigan and Pennsylvania.

This result was actually a bit of a surprise to me: I thought with people generally moving towards cities in big states and cities generally voting blue, the difference would have been larger (though probably not large enough to change the overall election result). But in fact while this assignment of votes would have helped Clinton in places like California and New York, it would have hurt her in places like Connecticut and Texas.

I'm not sure there are meaningful lessons to take from all this. That electoral seats should be reallocated more frequently? Maybe, but the census is already underfunded. Perhaps this further emphasizes the terrible importance of voter suppression, which appears to have been quite successful in Florida in 2016. Mainly, integer programming is cool.

[^1]: Counting D.C. as a state, which is why $i$ goes from 1 to 51.
