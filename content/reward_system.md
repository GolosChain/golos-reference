Golos reward system
--------------------

As discussed in the [Steem whitepaper](https://steem.io/SteemWhitePaper.pdf), content and curation rewards are 1 GOLOS per block, or 3.875% per year, whichever is greater. The current implementation rolls both content and curation rewards into a single accumulator called total_reward_fund_steem(available with the cli_wallet command info, the get_dynamic_global_properties RPC, or at the golosd.com alternate UI).

This article is about to consider reward systems applied to different hardforks.

So total_reward_fund_steem emplaced in dynamic_global_properties_object is the total reward available for all posts as of the current block. For the hardfork 17-th this was changed to the separate reward pools for comments and posts, split to 10% and 90% from total_reward_fund_steem. As of this writing (block 13677875 on 2017-02-07), total_reward_fund_steem is 38503.172 GOLOS. This is distributed according to a post's V-shares (which are called rshares2 in the code). There are currently about 8,393,667,330 septillion (one septillion is 10^24 = 1,000,000,000,000,000,000,000,000) V-shares in existence, so one septillion V-shares is worth approximately 0.0213 GOLOS. As described in the "Allocation and Supply" section of the whitepaper, Golos are added to total_reward_fund_steem every block. When a post is paid, its V-shares and rewards are simultaneously removed from the pool, resulting in no net change of the current Golos per V-share.

Each upvote "prints" V-shares, causing the upvoted post to get an increased percentage of total_reward_fund_steem and slightly decreasing all other posts' percentage.
The network "prints" a little bit of GOLOS every block and adds it to the reward without regard for the number of V-shares that have been issued, slightly increasing each post's payout every block.

As discussed in the whitepaper, for every 1 GOLOS printed to pay post rewards, 9 GOLOS are printed to be divided among all holders of Golos Power, meaning that users who make a long-term commitment to the network by holding Golos Power are protected to some extent from the inflationary effects of the "printing press". This was required at the start of the network to create most of the initial supply, but as for hardfork 16-th this was changed to the inflation-based issuance. At the introduction block 3860400 inflation rate was set to 15% with reduction by 0.01% every 250k blocks to 0.95%.

> The payout per user will be unequal; there will be many users who get much less than $2000, or even nothing at all. This hypothetical situation is further based on the totally unrealistic assumption that the price of GOLOS will remain the same over time. Remember my disclaimer.

Each upvote is assigned a number of R-shares by the system based on the account's Golos Power and remaning voting power. Exactly how the R-shares are computed will be discussed in a future post in this series. The purpose of this post is to describe how V-shares are obtained from R-shares. Later we will discuss a different curve.

The "how" is quite simple; V-shares are equal to R-shares squared. So if Bob's post has twice as many R-shares as Alice's post, Bob gets 4x Alice's V-shares reward; if Bob's post has ten times as many R-shares as Alice's post, Bob's post gets one hundred times Alice's V-shares reward.

This O(R^2) behavior biases the system toward "winner-take-all", concentrating the payouts in the form of fewer, larger rewards for the big winners. This is a deliberate design decision to aid in attracting and retaining users: Humans are notoriously bad at thinking about probabilities, and users will instead judge the system based on the sizes of the largest rewards. The O(R^2) part of the system is designed to attract risk-seeking users.

> Users with a small balance ("minnows") can be considered to be risk-seeking. "You can earn approximately X% per year on your $5 worth of GOLOS" is much less attractive than "if your post gets a ton of upvotes, you might be looking at a $100-$10,000 payday," even if the EV of both propositions are the same. The minnow user class is very important, as the vast majority of users in a virally growing Golos network will be minnows.

### What is the V-shares curve?

We can define V-shares to be a function of R-shares, call it v-bar; the current implementation of v-bar is simply:

```\bar{v}(R) = R^2```

This function has several desirable properties:

1. v-bar is O(R^2) in the long run (to strike just the right amount of balance between "winner-take-all" and "everybody has a chance")
2. v-bar goes through the origin (no R-shares gives you no rewards)
3. v-bar is an increasing function (more R-shares gives you more rewards)
4. v-bar has an increasing slope (more R-shares gives you more rewards faster)
5. v-bar is a low-degree polynomial function (for efficiency of implementation)
6. v-bar has a leading term with a coefficient of 1
Unfortunately, it has one undesirable property: The function is flat (in calculus terms, it has a zero derivative) at R = 0. Our undesirable property can be stated as:

    1. v-bar has zero slope at the origin (the first R-shares get almost no V-shares)
That is, our list of desirable properties should expand to include:

    2. v-bar has positive slope at the origin (there is a minimum exchange rate of R-shares to V-shares)
Redefining the V-shares curve
Fortunately, it is fairly simple to redefine v-bar by keeping the same curve, but shifting the point we consider the origin up and to the right until we get to a place with a positive slope. We can write a new definition of v-bar based on shifting s units (this reduces to the old definition when s = 0):

``` \bar{v}(R) = (R+s)^2 - s^2 ```

This v-bar retains desirable properties 1-6, but adds the additional desirable property P7' when s > 0. Of course V-shares are no longer equal to R-shares squared if we have s > 0, which explains why I'm calling them V-shares in this series of posts, instead of rshares2: The rshares2 name implies the s = 0 curve, which will no longer be used if my hardfork is adopted.

V-shares are equal to R-shares squared (a parabolic curve starting at the vertex where the curve is flat)
The curve's increasing slope means more V-shares per R-share for posts with more total R-shares
I propose a hardfork modification to the parabolic curve: Start to the right, so you still get increasing slope effect, but avoid the flat part
This post's main purpose has been to describe the current behavior and the proposed hardfork modification. There is a much more powerful theoretical justification for the proposed hardfork modification, but that justification will wait until a later post.

### The principles

We'll start out with a list of principles. Principles are properties that are considered desirable [1] by system designers. The algorithm itself is a direct consequence of these principles.

The SBS [2] princple. The SBS principle is that R-shares are fungible, homogenous, and arbitrarily finely divisible. If Alice and Bob both upvote the same post, neither Alice nor Bob's payout should change regardless of whether Alice keeps her funds in a single account or splits her funds up into accounts Alice1 and Alice2 (all else being equal).

The proportionality principle. Upvoters' curation rewards change proportionally for subsequent upvotes. For example, suppose Alice and Bob vote on the same post, Alice's reward is 200 V-shares, and Bob's reward is 100 V-shares. The proportionality principle states that if subsequent votes increase Bob's reward to 500 V-shares, then Alice's reward must simultaneously increase to 1000 V-shares. [3] [4]

The ultimate indifference principle. The ultimate indifference principle relates to the concept of an ultimate upvoter (UU), a (theoretical) upvoter who has a small balance and is always the last person to upvote a post. The ultimate indifference principle states that the UU is indifferent about which post she will upvote (assuming she only cares about how many V-shares she personally receives from curation rewards).

The quadratic reward principle. The total curation reward of a post in V-shares is equal to some quadratic function v(R) [5] of the total R-shares upvoting the post, with a few minor technical restrictions [6].

The monotonic supply schedule principle. The weight distributed per R-share decreases as the total upvoting R-shares increases.

### Meta-discussion issues
Any discussion of changes to the reward system may get contentious due to the non-trivial amounts of GOLOS involved. I mostly prefer to stay away from blockchain politics and focus strictly on technical matters. Here are a few thoughts which I hope will clarify the boundary of the topic under discussion and hopefully keep the comments from degenerating too far:

The CEO of GolosFund. is the top curation rewards recipient according to the old code. If the developers' objective was to use the curation reward system to allow whales / insiders to more easily line their pockets, we wouldn't be pushing for changes.

There are technical costs to maintaining multiple reward systems. If we can get a new system that's demonstrably better than the old system implemented and adequately tested before July 4, it will make me very happy as a backend developer if we can then delete the old system code.

There's a single unique algorithm which simultaneously follows all of the principles in this post. If you accept the principles, then you must accept my algorithm as the One True Curation Reward Algorithm, since it logically follows from the principles.

Selecting a set of principles is an important problem, but it is a problem of a social, political or philosophical nature. Such questions are out of scope for this series of posts. I want to focus on the strictly technical problem of deriving an algorithm from this particular set of principles.

If you object to any of these principles, it is up to you to lay out a viable alternative set of principles, construct a convincing social/political/philosophical argument for your principles which is accepted by the community, and solve the same technical problem of how to derive an algorithm from your principles.

[1] Rewards should be fair, incentivize behavior which enhances the community, and have a simple and efficient implementation.

[2] SBS stands for "satoshi by satoshi".

[3] The proportionality principle is equivalent to stating that each upvote is assigned some weight, and the post's total V-shares curation reward is divided proportionally to the weights.

[4] The astute reader may notice that the SBS principle and proportionality principle sound similar, and ask whether one of these principles is a special case of the other. The SBS principle and the proportionality principle both describe forms of homogeneity. However, stake may cease to be homogenous due to the time order in which it votes. The SBS principle says R-shares (the "stuff" used to vote) are homogenous, the proportionality principle says weight (the "stuff" used to assign rewards) is homogenous. The weight per R-share may change as the post is upvoted without violating the proportionality principle or the SBS principle (as long as this change only depends on the amount of R-shares seen by the post). So it is possible to have a system where (Alice votes, then Bob votes) does not have the same consequence as (Bob votes, then Alice votes) while respecting both the SBS principle and the proportionality principle.

[5] I used the notation v-bar in the internal Golos notes and my previous post. But I decided to drop the v-bar due to Unicode issues.

[6] These minor technical restrictions are as follows:

v(R) has a leading coefficient of 1
v(0) = 0
v'(R) >= 0 when R >= 0
These restrictions mean that v is the parabola y = x^2 with the origin shifted to the right by some amount s. The general form of such v is v(R) = (R+s)^2 - s^2 with s >= 0.

This post is a mathematically intense post. In order to follow everything in this post, your mathematical toolbox will need to include sharp algebra skills. Non-mathematically-inclined readers may wish to skip this post, or only read the conclusion.

I will frequently reference equations and principles discussed earlier in the series. If you have questions about an equation or principle, try reading the previous posts in this series.

For now, I will not discuss how to treat downvotes or changing of votes. It's much easier to develop a basic algorithm in a simplified upvotes-only world and then extend it to a world where downvotes and changing of votes is possible.

### Thinking in totals
The proportionality principle says we can think of upvotes as receiving "weight" which will be proportional to their ultimate payout. The total weight Wₚ that's been handed out to post P at some point in time t is some function of P and t. The SBS principle, however, says that W cannot make reference to any information outside the number of R-shares that exist on post P. So if Rₚ(t) is the number of R-shares which have been paid to post P at time t, we may consider Wₚ to be some function of Rₚ(t); i.e. Wₚ = w(Rₚ(t)). We may simplify the notation if we allow Rₚ to be the "time variable" and drop the separate time variable t, giving Wₚ = w(Rₚ).

Similarly, we can consider post P's total V-shares Vₚ = v(Rₚ). The quadratic reward principle lets us go further and say v(R) = (R+s)^2 - s^2.

To review, the total weight W and total V-shares V given to all upvoters of a post are both functions of the R-shares on the post.

### Thinking in margins
Starting from this section, I'll be focusing on the world of a single post P, so I'll stop using the P subscript (this will simplify the notation).

Let's imagine an ultimate upvoter named Alice (she is the last person to upvote a post and she has a small balance). Alice's upvote is worth dR R-shares. She upvotes a post which has already been upvoted by Bob, who has a much larger balance. A little bit of weight dW will be created and given to Alice, and the post's V-shares will increase by dV, which will be split among Alice and Bob according to their weight.

The price Alice pays for dW weight (denominated in R-shares "paid" per weight received) is dR / dW; let us call this price ρ (Greek letter rho).

Clearly, the post's ratio of weight to V-shares before Alice acts is σ = W / V (Greek letter sigma). Alice's balance is so small that her actions did not appreciably change the amount of V-shares and weight on the post; so after Alice's actions the weight to V-shares ratio is still σ. So σ is the "price" Alice and Bob both "pay" when their weight is "traded" for V-shares when the post's curation rewards are distributed.

The overall price Alice pays in this two-step trade (R-shares paid per V-share received) is, of course, the product of these prices σ·ρ.

### Thinking in principles
The price Alice the ultimate upvoter pays for V-shares is σ·ρ. The ultimate indifference principle says that this price must be equal to some global constant value A. 

We can view the state of the system (R, W) as a point on the plane. It starts out at the origin (R, W) = (0, 0), and we can see how it evolves as upvotes accumulate by setting dR to some small fixed constant value (e.g. one satoshi worth of R-shares), then repeatedly updating the point to (R + dR, W + dW) where dW is calculated by the above equation. Each update's calculation of a new W value incorporates the old R and W values; while R simply marches forward at a constant rate of dR, the value of W evolves in a non-trivial way. 

I will frequently reference equations and principles discussed earlier in the series. If you have questions about an equation or principle, try reading the previous posts in this series.

### Solving the equation
In the previous part, we left off with the equation:

{dW \over dR} = {W \over A ((R+s)^2 - s^2)}

Readers familiar with calculus should immediately recognize this as a differential equation. Specifically, it is a first-order separable differential equation involving a low-degree rational function, which leads us to suspect that an exact symbolic solution may be possible.

Solving the equation, s = 0 solution
When s = 0 the solution is:

```W = B e^{-1 \over A R}```

It's clear that the solution rises quickly early on, but slows as R becomes larger. We can clearly see that the total weight issued, W(R), goes toward B = 1 as W becomes large. Interestingly, B represents the maximum amount of weight the algorithm is ever "allowed" to issue to a post's upvoters, regardless of how many upvoters there are -- it's a "weight ceiling". The fact that the weight does have a ceiling was not one of our starting assumptions; rather it was a consequence of them. I was actually quite astonished when I discovered that the existence of a ceiling actually follows from the principles!

There is an interesting little corner of the graph, which I've outlined with a small box near the origin. 

So actually W starts off flat, starts to rise at an increasing rate, then (by eyeball, at about R = 0.5) its rate of increase starts to fall. Where the function changes from rising rate of increase to a falling rate of increase is called an inflection point.

We can determine the inflection point exactly from this expression. The inflection point occurs when W'' is zero, at R = 1 / (2A). The "eyeball estimate" of R = 0.5 was spot-on!

The monotonic supply schedule principle requires the W curve to have a decreasing slope everywhere. The region before the inflection point, where the slope increases, violates the monotonic supply schedule principle. As a result, the s = 0 solution isn't compatible with the principles listed earlier in this series. In the next post in this series we will examine the s > 0 solution.

### Conclusion
We started out with some principles, which are mostly different specific mathematical ways to quantify the idea of a "fair" reward system which accelerates [1] the reward on a single post as upvotes accumulate. The most basic accelerating scheme simply determines the total curation reward for a post as the square of the number of upvoting R-shares, V(R) = R^2; this is the "curation reward curve".

In today's post, we discovered a problem: The mathematical consequences of the first four principles, applied to the curation reward curve V(R) = R^2, result in a system where sometimes later upvoters get more weight per R-share than earlier voters! In the next post we'll examine a different curation reward curve and show that it doesn't suffer from this problem.

[1] The reward (as measured in V-shares) actually does continue to accelerate forever. But since V-shares are claims on a pool of GOLOS whose size is bounded by the allowed inflation, a post accumulating truly huge numbers of upvotes would see the increases in its reward (as measured in GOLOS) eventually start to slow due to its V-shares beginning to represent a large majority of the pool.

### The story so far
Since we've developed some principles, which are mostly different specific mathematical ways to quantify the idea of a "fair" reward system. We found a differential equation which follows as a consequence of the principles:

{dW \over dR} = {W \over A ((R+s)^2 - s^2)}

In the previous post, we showed that when s = 0, the solution of this differential equation is unsatisfactory and inconsistent with the monotonic supply schedule principle. The current post's main task is to examine the s > 0 case.

Solving the equation, s > 0 solution
The s > 0 case is actually fairly straightforward with a quick side trip to Wikipedia's table of integrals:

curation-derivation-calculus-s-nonzero.svg

This is a formal result, but we should do some sanity checks: Check that the derivative matches, plot the function, and determine whether inflection points are possible, to be sure that this solution does not suffer from the same flaw as the s = 0 solution.

Checking the equation
To start off, we'll algebraically verify the expression for W satisfies the original differential equation. A matter of some algebra shows that it does:

curation-derivation-calculus-s-nonzero-check-first-derivative.svg

Building intuition
Let's think about the sub-expression R / (R+2s), which I'll call q. Initially q starts out at zero when R = 0, and q grows to approach 1 as R gets much larger that 2s. Raising to the power 1 / (2As), which I'll call p, maps the interval [0, 1] to itself in a monotonically increasing way, meaning the W curve as a whole should increase monotonically from 0 to B. Higher values of p means that for small q, the response of W to increasing q is flatter, but also accelerates more for moderate values of q. For a fixed value of s, higher values of p correspond to lower values of A. Plotting the curves gives a clear picture:

weight.png

Again like the s = 0 case, the parameter B (originally arising from a constant of integration) is the maximum weight that will ever be issued as R-shares goes to infinity. As emphasized by the tick labels on this plot, the parameters s and B can be viewed as establishing horizontal and vertical scales: Changing either simply gives the same picture stretched or squashed.

Inflection point analysis
One of these curves is not like the others. The blue A = 0.5 / 2s curve in the picture actually accelerates for small values of R. This curve violates the monotonic supply schedule principle and should be ruled out. Similarly to the previous post, we can find the second derivative and check for inflection points:

curation-derivation-calculus-s-nonzero-second-derivative.svg

An inflection point occurs when W'' = 0. Only the middle term of the three-term product for W'' may be zero, which occurs when:

curation-derivation-calculus-s-nonzero-second-derivative-zero.svg

Thus, as long as we set s >= 1 / (2A), the inflection point occurs when R <= 0 and we no longer violate the monotonic supply schedule principle!

Conclusion
We started out with some principles, which are mostly different specific mathematical ways to quantify the idea of a "fair" reward system which accelerates [1] the reward on a single post as upvotes accumulate. In the previous post, we decided V(R) = (R+s)^2 - s^2 (or equivalently, V(R) = R^2 + 2Rs) is the simplest curation reward curve [2] which might be compatible with the principles.

In this post, we took this curation reward curve as a given, and found that our principles imply a particular weight curve [4] given by:

curation-result.svg

This curve is actually a family of curves; we must choose a member of this family by setting the values of the parameters A, B and s. All three parameters must be positive; in addition we have the constraint s >= 1 / (2A). The values of A and B are fixed by certain practical implementation concerns [5], leaving s as the only "free" parameter. In a future post I will discuss some considerations of how the value of s should be set.

[1] The reward (as measured in V-shares) actually does continue to accelerate forever. But since V-shares are claims on a pool of GOLOS whose size is bounded by the allowed inflation, a post accumulating truly huge numbers of upvotes would see the increases in its reward (as measured in GOLOS) eventually start to slow due to its V-shares beginning to represent a large majority of the pool.

[2] The curation reward curve V(R) is the formula which determines "the size of the pie": How many V-shares will be assigned to the post's total curation rewards as a function of the number [3] of upvotes.

[3] Upvotes are measured in R-shares, which are determined by Golos Power times voting power. Voting power is usually near 100% and regenerates over time, but a little bit is consumed with each upvote, meaning that curation rewards are subject to diminishing returns if you use a very large number of upvotes in a short period of time.

[4] The weight curve W(R) is the formula which determines "the slicing of the pie": How much weight is given out based on the number of R-shares. Specifically, if your upvote increases the R-shares on the post from R0 to R1, the weight will increase from W(R0) to W(R1); your curation reward weight will be W(R1) - W(R0). The total number of V-shares for the post's curation reward is calculated by the curation reward curve, then divided among all upvoters proportional to their weights.

[5] The choice A = 1 / (2s) is the only principles-compliant choice for which computing W does not require computing a non-integer power. The choice of B is completely inconsequential as long as it's large enough to avoid rounding errors.