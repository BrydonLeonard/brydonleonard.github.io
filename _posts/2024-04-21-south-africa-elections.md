---
layout: post
title: "The South African Elections - How they're run"
date: 2024-04-21
categories: voting_systems
tags: elections
---

With the South African general elections coming up next month, my wife and I wanted to spend some time getting more familiar with exactly how they work. We'd initially planned to just throw all of the parties' manifestos at ChatGPT and ask it for summaries, but along the way, I got distracted by the electoral system itself. That's what this week's blog post is about!

## Parliament

South Africa has [a _Bicameral_ Parliament](https://en.wikipedia.org/wiki/National_Assembly_of_South_Africa)[^1], which means that it has two *houses*. Each of the houses has some number of _seats_, which are the positions in those houses available to be held by _members of parliament_ (MPs). The houses also each have their own responsibilities, but at a high level, parliament as a whole is responsible for representing the people who voted for it (the electorate), making laws[^2], and overseeing the government.

The first of the two houses is the National Council of Provinces (NCOP) and represents the country's 9 provinces, each of which gets 10 of the NCOP's 90 seats. The other house is the [National Assembly](https://en.wikipedia.org/wiki/National_Assembly_of_South_Africa), which has 400 seats and is the one that directly elects the president[^3]. 

There's a lot more detail we could get into, but that's enough context to start discussing how the seats in those houses are allocated!

## The ballots

In South Africa, our parliament is elected through a _single-vote closed-list proportional representation_ system. That's a lot of terms smushed together, so let's look at what each of them means:
- _**Single-vote**_ because we each only get one vote per ballot. 
- It's a ***list*** system because when each of us casts our vote, it's not for a specific candidate, but for a party that has [a whole ordered list of candidates](https://www.elections.org.za/pw/Parties-And-Candidates/Candidates-List). 
- The list is **_closed_** because we (as the public) don't have any say in what candidates the parties put on their lists; we can only choose which party we want to vote for. 
- ***Proportional representation*** means that the seats in parliament are allocated to parties in the same proportion as the public's votes for those parties.

Even though we only get a single vote per ballot, when we go to vote, we'll actually get [three separate ballots](https://dailyfriend.co.za/2024/03/30/understanding-the-party-lists-and-how-our-mps-are-elected/). Two of them (the ***national ballot*** and ***regional ballot***) are used to elect the National Assembly. The third (the ***provincial ballot***) is used to elect the provincial government in the province where you're voting. I'll come back to the provincial elections a little later. For now, let's look at just the elections for the national assembly. 

## Electing the National Assembly

In the period leading up to the elections, each party publishes a _regional_ list of candidates in each province that they want to stand for election and an optional _national_ list of candidates. When you vote for a given party on the national or regional ballot, those lists are actually what you're voting for.

200 of the National Assembly's 400 seats are filled by candidate from the parties' regional lists, while the other 200 come from the national lists. That process to determine specifically which candidates get seats takes place over three steps [1](https://www.elections.org.za/ieconline/Documents/NPE_SeatCalculationGraphic.pdf) [2](http://electionresources.org/za/system/#ASSEMBLY) [3](https://www.sanef.org.za/wp-content/uploads/2019/02/Helen-Suzman-Foundation-The-South-African-electoral-system-undated.pdf):
1. use the national ballot to determine how many _total_ seats each party should have
2. use the regional ballots to determine how many of those seats should be filled by candidates from parties' regional lists
3. for any parties that haven't already had all of their seats filled in step 2, fill the rest of their seats with candidates from their national lists

Let's look at each of those in more detail

### 1 - National ballot

This step doesn't yet assign seats to any candidates. Rather, it's purely to determine the total number of seats in the National Assembly that each party should have. Choosing the candidates comes later. 

#### 1.1 - Whole seat allocation (allocation 1)

**The first step is to allocate all of the seats that each party has won outright.** To calculate that number, South Africa uses a modified form of a formula called [the Droop quota](https://en.wikipedia.org/wiki/Droop_quota)[^4] which tells us how many votes each seat is "worth" (the weird looking brackets mean round _down_):

$$
votes\ for\ one\ seat=\left \lfloor{\frac{total\ votes}{total\ seats + 1} + 1}\right \rfloor
$$

Each party's total vote count can then be divided by that per-seat value to determine how many seats they should get. There's likely going to be a remainder after the division is done for each party; we note it down, but otherwise ignore it for now and use the whole number parts of the result to allocate seats to each party.

For example, if party A has 23 total votes and `votes for one seat = 5`, we have `23/5 = 4.6`. The party is allocated 4 seats now and we keep track of the 0.6 for later.

#### 2.2 Remainder step (allocation 2)

**Next, if all 400 seats haven't yet been allocated, we use the remainders from earlier to allocate up to 5 more seats.** We put the remainders in descending order and keep allocating seats to the parties with the next largest remainders until 5 more seats have been allocated or all 400 seats are filled. 

It's possible that the party with the highest remainder didn't actually receive any seats in allocation 1. For example, if party B received 4 total votes and `votes for one seat = 5` (like in the example above), party B would have a remainder of 0.8. It's entirely possible that that's the highest remainder of any party. As such, up to 5 parties can receive seats in the elections despite not actually receiving a full seat's worth of votes.

For example, we noted down 0.6 for party A's remainder in the previous step and just calculated party B's to be 0.8. Say there are 398 seats already allocated; if parties A, B, and C have the remainders from the table below, both A and B would get one additional seat each. 

| Party | Remainder |
| ----- | --------- |
| A     | 0.6       |
| B     | 0.8       |
| C     | 0.4       |

#### 2.3 Allocation 3

**Finally, if there are still seats empty after allocation 2, the rest are allocated to the parties with the most votes per seat that they've already received.** This effectively allocates the remaining seats to the parties that have "paid" the most votes for their seats. Only parties that already have seats are considered in this allocation.

For example, if we have the following situation:

| Party | Votes | Seats | Votes per Seat |
| ----- | ----- | ----- | -------------- |
| A     | 24    | 5     | 4.8            |
| B     | 4     | 1     | 4              |
| C     | 2     | 0     | N/A            |

Party A has the highest average votes per seat, so if there's an additional seat to be allocated, they'll receive it.

#### An example

Here's a simplified example of an election that follows this process, but only has 20 seats, 5 parties, and 150 voters. Allocation 3 is unique to the national ballot (and isn't present in the other ballots that we'll discuss later), so it's not included here:

<video width="640" controls="controls">
  <source src="/assets/images/2024-04-21-south-africa-elections/DroopQuota.mp4" type="video/mp4">
</video>

### Regional ballot

It's worth being clear at this point that the "regions" and "provinces" in the regional and provincial elections are actually the same things. The reason we use the two terms is to clearly distinguish the two different elections that happen at the same time. The *regional elections* form part of the national elections (and they're what this section will cover) whereas the *provincial elections* elect provincial legislatures (and are discussed later).

The calculation of the seat allocation from the regional ballots is very similar to that for the national ballot. Each region has its own list of parties, each of which has its own closed list of candidates. The seats allocated to the parties in each region are calculated independetly using the same Droop formula and remainder method as with the national ballot; the only difference in the process within each region is that there's no limit to the number of parties that can receive seats during the _remainder_ step (allocation 2); it continues until all seats are filled.

A key difference in the overall process, however, is that the number of seats in each region varies based on that region's population. For example, if region A has a population twice the size of region B, region A will have twice as many seats in the national assembly. That's done so that a single person's vote in roughly equivalent between the provinces when it comes to electing parties in the regional ballot. 

### Filling seats

At this point, we know how many seats each party should have in the National Assembly, as well as how many seats they received in each region. All that's left is to actually allocate those seats to candidates. 

First, we allocate the regional seats; if a party was allocated `S` seats in region `R`, the top `S` candidates in that party's regional list for region `R` become MPs. Once that process is repeated for all 9 regions, if the number of seats that the party has filled with MPs is less than the number calculated from the _national_ ballot, then those left over seats are filled by members from the party's national list. Once that process is complete for all parties that received seats, all 400 seats of the National Assembly are filled![^5]

To give an example, imagine a party that won 50 seats in the national ballot:

![Seat assignment step 1](/assets/images/2024-04-21-south-africa-elections/election_seat_filling_1.png)

Now imagine that that party won 15 seats in region A, 10 in province B, and 13 in province C. The candidates who would become MPs for the party are chosen by first taking 15 candidates from the party's regional list in A:

![Seat assignment step 2](/assets/images/2024-04-21-south-africa-elections/election_seat_filling_2.png)

Then by taking 10 from the regional list in B:

![Seat assignment step 3](/assets/images/2024-04-21-south-africa-elections/election_seat_filling_3.png)

and finally 13 from the regional list in C:

![Seat assignment step 4](/assets/images/2024-04-21-south-africa-elections/election_seat_filling_4.png)

Once those seats are all filled, the party still has 12 seats left over. The 12 candidates to fill those seats are taken from the party's national list:

![Seat assignment step 5](/assets/images/2024-04-21-south-africa-elections/election_seat_filling_5.png)

One interesting side-effect of this approach is that a party winning extra seats from the regional ballot could result in *fewer* of the party's national candidates receiving seats because they went to regional candidates instead.

## Provincial elections and the NCOP

Now that we've covered the national elections and the seats in the National Assembly are filled, we can come back to the last ballot (the provincial one) and talk about the provincial elections. Like the national elections, the provincial elections use the Droop quota with the remainder method to determine the number of seats that each party gets in their legislatures. However, rather than the 400 seats in the national government, the provincial governments each have their own numbers of seats:[^6]

| Province      | Seats |
| ------------- | ----- |
| KwaZulu-Natal | 80    |
| Gauteng       | 73    |
| Eastern Cape  | 63    |
| Limpopo       | 49    |
| Western Cape  | 42    |
| Free State    | 30    |
| Mpumalanga    | 30    |
| North West    | 30    |
| Northern Cape | 30    |

Once the seats are allocated, the seats in the NCOP can be awarded. The public doesn't vote for those seats directly; rather, each province's 10 seats are allocated proportionally to each party's representation in a given province. For example, if party A received 15 of Mpumalanga's 30 seats (50%), that party would receive 5 of the province's 10 seats on the NCOP.

## Next up

Now that I've gotten sufficiently distracted by elections processes, I want to go back to my original goal of summarizing the manifestos of each party. If the summaries come out at a reasonable length, I may post them as a supplemental post to this one.

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbrydonleonard.github.io%2Fvoting_systems%2F2024%2F04%2F21%2Fsouth-africa-elections.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

## Footnotes

[^1]: Parliament is the _legislative_ branch of the government.
[^2]: Which are enforced by the _judicial_ branch of government, which consists of (amongst others) all of the country's courts.
[^3]: Who is the head of the _executive_ branch of government.
[^4]: Interestingly, the Droop quota is a generalisation of majority rule systems (where you need more than 50% of the vote to win). You can see that by reframing a winner-takes-all vote as having a single _seat_ to be won. With that framing, you can substitute in 1 for the seat count in [the (original) Droop formula](https://en.wikipedia.org/wiki/Droop_quota) and come out with `votes/2 = 50%` , which is the proportion of votes necessary to that "seat".
[^5]: None of the resources I used for this post really went into *why* the regional and national ballots are separate. One can imagine, though, that it's because the candidates on each region's lists come from that region. That puts the candidates that get elected closer (literally) to their electorate.
[^6]: Each province picks their own seat counts here and the MPs are only responsible for governing the province itself, so the fact that (for example) KZN has so many seats has little bearing on anyone outside of the province. Each province still only gets 10 seats on the NCOP.