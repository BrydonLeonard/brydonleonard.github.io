---
layout: post
title:  "Exact cover, Algorithm X, and Lego"
date:   2023-07-02
categories: learning
---

# Exact covers, Algorithm X, and Lego

## No dice

You may have come across examples of people online making pictures with dice, like the ones below. I had, and I was interested in trying something similar myself. 

<iframe id="reddit-embed" src="https://www.redditmedia.com/r/Damnthatsinteresting/comments/rxdh0b/dice_art/?ref_source=embed&amp;ref=share&amp;embed=true" sandbox="allow-scripts allow-same-origin allow-popups" style="border: none;" height="620" width="640" scrolling="no"></iframe>

<iframe id="reddit-embed" src="https://www.redditmedia.com/r/nextfuckinglevel/comments/jhuq2e/using_dice_to_create_art_is_extraordinary/?ref_source=embed&amp;ref=share&amp;embed=true" sandbox="allow-scripts allow-same-origin allow-popups" style="border: none;" height="620" width="640" scrolling="no"></iframe>

Being a software developer, half of the fun for me lay in writing a program to turn pictures into solutions _for_ me. Making pictures with dice is actually fairly straightfoward, since each die is effectively a single pixel. Finding a "solution" (what I'm going to call the output of these programs that tell me where to put things) is a case of:
1. Finding a picture
2. Scaling it to a resolution that you have enough dice to cover (a resolution of 4x4 would require 16 dice)
3. Switching to grayscale (you could leave it in colour if you had a big bag of coloured dice, but lets assume regular black and white dice for now)
4. Decreasing the colour *depth* to 6. Colour depth is the number of different colours in the image, so by reducing a grayscale image's colour depth to 6, we end up with 6 shades of white, black, and gray.
5. Map each of the 6 colours to one of the numbers on the dice
6. Done!

Here's an example with a stock photo:

| | |
| :---: | :---: |
| The original image:<br/>![original image](/assets/images/algorithm-x/roll-1.png) | Scaled to a lower resolution:<br/>![scaled](/assets/images/algorithm-x/roll-2.png) |
| Switched to grayscale:<br/>![grayscale](/assets/images/algorithm-x/roll-3.png) | Drop the colour depth:<br/>![colour depth decreased](/assets/images/algorithm-x/roll-4.png) |

The solution output would then look something like:
```
1	1	1	1	1	1	5	1	1	1	...
1	1	1	1	1	1	1	1	1	1	...
1	1	1	5	1	1	5	5	1	1	...
1	1	1	1	1	5	1	1	1	5	...
5	1	1	1	5	5	1	5	1	5	...
1	5	5	5	1	5	5	1	1	5	...
...
```

As someone building one of these images, you'd then take the grid of numbers, place each die with the correct value facing upwards and you'd be done. The pieces of art that people have created with dice are really impressive, but as someone looking for a fun programming project (and someone who seems to enjoy making life hard for himself), dice felt too easy. I tossed the dice idea and went looking for something more interesting. I eventually landed on Lego.

Making pixel art with Lego is more *interesting* than dice for a few reasons:
- The pieces are all different sizes
- Each piece can only be a single colour, unlike dice, where a single die can be changed to any of the six colours by rotating it
- The *orientation* of the pieces matters. Placing bricks vertically and horizontally changes how they cover the image

## How exactly do we cover the picture?

### Exact covers!

I rushed headfirst into my implementation, only to realise very quickly that placing the Lego bricks was a more complex task that I'd initially anticipated. I had, however, already gone out and bought a 1kg bag of "Lego" bricks, so I was committed. I haven't come across examples online of people solving the Lego pixel art problem, but the problem didn't seem unique enough that some abstraction of the same problem wouldn't have been solved. My first clue as to what problem I was actually solving came when I encountered [polyomino](https://en.wikipedia.org/wiki/Polyomino) puzzles, which involve placing a set of polyominos in a rectangle:

![polyomino puzzle](/assets/images/algorithm-x/polyomino-puzzle.png)
*Donald Knuth [arXiv:cs/0011047](https://arxiv.org/abs/cs/0011047) [cs.DS]*

> If this looks a little like Tetris, that's because the pieces in Tetris are [*tetrominoes*](https://en.wikipedia.org/wiki/Tetromino), which are polyominos, but only made up of four squares. Another type of polyomino you'll likely be familiar with are dominoes, which are made up of two squares.

The polyomino puzzle can be generalized to an [exact cover problem](https://en.wikipedia.org/wiki/Exact_cover). The Wikipedia definition of an exact cover is fairly dense:

> Given a collection S of subsets of a set X, an exact cover is a subcollection S* of S such that each element in X is contained in exactly one subset in S*. One says that each element in X is covered by exactly one subset in S*.

Let's take a look at a specific example to make sense of the definition. Take the matrix below (I've numbered the rows to make them easier to refer to):

```
1 -> | 0 | 0 | 0 | 1 |
2 -> | 0 | 1 | 1 | 1 |
3 -> | 0 | 1 | 0 | 1 |
4 -> | 1 | 0 | 1 | 0 |
```

To find an *exact cover*, we need to select some of the rows from the matrix so that in our selection of rows, there is exactly one 1 in each column. In this case, we can take rows 3 and 4:

```
3 -> | 0 | 1 | 0 | 1 |
4 -> | 1 | 0 | 1 | 0 |
```

Tying this back into the definition from Wikipedia,
- `S` is the set of all four rows: `{| 0 | 0 | 0 | 1 |, | 0 | 1 | 1 | 1 |, | 0 | 1 | 0 | 1 |, | 1 | 0 | 1 | 0 |}`
- `X` is our desired end state: `| 1 | 1 | 1 | 1 |`
- `S*` is the set containing rows 3 and 4: `{| 0 | 1 | 0 | 1 |, | 1 | 0 | 1 | 0 |}`

### Generalized tetrominos

Now that we've covered exactly what an exact cover is, let's take a look at how we can generalize other types of puzzles to the exact same "pick some rows" problem that we discussed above. Let's say we have a 4x4 grid and we want to fill it with *monominoes* (1x1 polyominoes):

![monomino puzzle](/assets/images/algorithm-x/puzzle-generalisation-1.png)

Let's give each row in the grid a letter and each column a number. We'll also put a `1` in every grid cell that we want covered (that's all of them):
```
    1   2
A | 1 | 1 |
B | 1 | 1 |
```

Now for the interesting bit: We lay out the grid's rows side-by-side to get a new one-dimensional matrix that describes the solution we want. Each column of this new matrix corresonds to one cell in the original grid (I've given the columns headings to make it clearer which cells they represent):

```
  A1   A2   B1   B2 
| 1  | 1  | 1  | 1  |
```

The last step to get back to our "pick some rows" problem is to take *every possible position* that we could place a monomino and make it a row in our new matrix (which I'll now call the *solution matrix*). If a given row corresponds to, say, a monomino being placed in cell A1 (like in the image below), that row will have a one in column A1 and 0 everywhere else. (We also remove the row of all ones, since that's our desired solution and not actually part of the puzzle)

![monomino single placement example](/assets/images/algorithm-x/puzzle-generalisation-1-row-example.png)

```
  A1   A2   B1   B2 
| 1  | 0  | 0  | 0  | <-- A monomino in cell A1
| 0  | 1  | 0  | 0  | <-- A monomino in cell A2
| 0  | 0  | 1  | 0  | <-- A monomino in cell B1
| 0  | 0  | 0  | 1  | <-- A monomino in cell B2
```

Now, we're back to the same problem as earlier! In this specific example, we would actually pick *every* row to solve the problem. That makes sense, since we'll want to place a monomino in every possible position to completely cover the grid:

![solved monomino](/assets/images/algorithm-x/puzzle-generalisation-1-solution.png)

To solidify the process of creating new solution matrices, let's look at another 2x2 grid that we want to cover with 2x1 dominos (placed horizontally or vertically):

![domino puzzle](/assets/images/algorithm-x/puzzle-generalisation-2.png)

In this case, each row in the solution matrix will have *two* `1`s because each domino covers two cells in the grid. The two example piece placements `A` and `B` below, correspond to the solution matrix rows below:

![domino puzzle row examples](/assets/images/algorithm-x/puzzle-generalisation-2-row-example.png)

```
  A1   A2   B1   B2 
| 1  | 0  | 1  | 0  | <-- Placement A
| 1  | 1  | 0  | 0  | <-- Placement B
```

If you squint at them, Lego pieces kinda look like polyominoes. To solve the lego placement problem, we generate the solution matrix in exactly the same way as for the example problems in this section. Actually iterating through all of the positions that Lego bricks could be placed takes some work, but we'll look at how I did that later.

## The secret ingredient: Algorithm X

### What is?

Now that we've covered how we can convert the problem of placing lego bricks on a grid into a full cover problem, let's take a look at the algorithm, *Algorithm X* that I used to solve that problem. Throughout this discussion, I find it useful to remember that all we're solving is that problem of picking a combination of rows that gets us a 1 in each column. If you're wondering about the algorithm's name:

> The following nondeterministic algorithm, which I will call algorithm X for lack of a better name, finds all solutions to the exact cover problem defined by any given matrix A of 0s and 1s.
> <p style="margin-left: 50px;">Donald Knuth</p>

Algorithm X is described as a recursive, nondeterministic, depth-first, backtracking algorithm:
- Recursive and depth first because we're going to pick a row to include in our solution, remove the other rows in the matrix that couldn't go along with it, and repeat the process on the new, smaller matrix.
- Nondeterministic because we pick the row randomly (though the set of options from which we pick is deterministic).
- Backtracking because when our depth-first attempt to find a solution bottoms out at an invalid one, we'll reverse our steps to the last valid solution and try again.

Here's a very wordy explanation of the steps involved in the algorithm:
1. Find the column `C` that contains the fewest `1`s (if there are multiple with the same number, pick the first). We could really do this with any of the columns, but using the column with the fewest ones tends to find solutions faster. 
  - If you find that the matrix has no columns at all, whatever is in the partial solution is your final solution!
  - On the other hand, if you find a column that has no `1`s, there's no valid solution with the matrix. Backtrack from here by reversing the steps you ran before this.
2. Column `C` with have one or more `1`s. Pick a row `R` that has one of those `1`s in `C`.
3. Now, we're going to see if we can find a solution that includes that includes row `R`, so add it to the *partial solution*. If we don't manage to find a solution that includes `R`, the algorithm will backtrack to here and we can try a different column.
4. If any rows other than `R` have `1`s values in the same columns as `R`, those rows can't be in a solution along with `R` because that solution would have two values for one of its columns. Remove all of those clashing rows.
5. Now that any column in which `R` has a `1`s has been removed, we don't need to check those columns again (there's nothing left to clash). Remove all columns in which `R` has a `1`s.
6. We now have a smaller matrix with one additional row in the partial solution. Repeat from step 1 with the new matrix.

Implement that and you've got a solver for the exact cover problem that we discussed earleir. If you want to get a little more in-depth, here's the pseudocode lifted from Knuth's Paper:

```
// For a matrix A

If A is empty, the problem is solved; terminate successfully.
Otherwise choose a column, c (deterministically).
Choose a row, r, such that A[r, c] = 1 (nondeterministically).
Include r in the partial solution.
For each j such that A[r, j] = 1,
  delete column j from matrix A;
  for each i such that A[i, j] = 1,
    delete row i from matrix A.
    Repeat this algorithm recursively on the reduced matrix A.
```

There are examples all over the internet of what this looks like in practice, but I'll include one here too to make the description a little more concrete. The diagram below shows the process of finding an exact cover for a 7x7 matrix. Each sub-graph (each time the matrix shrinks) is a recursive call to the solver.

<!-- Mermaid diagram: /mermaid/algXExample.md -->

<svg aria-roledescription="flowchart-v2" role="graphics-document document" viewBox="-8 -8 398.96875 1563.671875" style="max-width: 398.96875px;" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns="http://www.w3.org/2000/svg" width="100%" id="mermaid-1688330326421"><style>#mermaid-1688330326421{font-family:monospace;font-size:13px;fill:#333;}#mermaid-1688330326421 .error-icon{fill:hsl(180, 0%, 100%);}#mermaid-1688330326421 .error-text{fill:rgb(0, 0, 0);stroke:rgb(0, 0, 0);}#mermaid-1688330326421 .edge-thickness-normal{stroke-width:2px;}#mermaid-1688330326421 .edge-thickness-thick{stroke-width:3.5px;}#mermaid-1688330326421 .edge-pattern-solid{stroke-dasharray:0;}#mermaid-1688330326421 .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-1688330326421 .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-1688330326421 .marker{fill:#0b0b0b;stroke:#0b0b0b;}#mermaid-1688330326421 .marker.cross{stroke:#0b0b0b;}#mermaid-1688330326421 svg{font-family:monospace;font-size:13px;}#mermaid-1688330326421 .label{font-family:monospace;color:#333;}#mermaid-1688330326421 .cluster-label text{fill:rgb(0, 0, 0);}#mermaid-1688330326421 .cluster-label span{color:rgb(0, 0, 0);}#mermaid-1688330326421 .label text,#mermaid-1688330326421 span{fill:#333;color:#333;}#mermaid-1688330326421 .node rect,#mermaid-1688330326421 .node circle,#mermaid-1688330326421 .node ellipse,#mermaid-1688330326421 .node polygon,#mermaid-1688330326421 .node path{fill:#fdfdfd;stroke:hsl(0, 0%, 89.2156862745%);stroke-width:1px;}#mermaid-1688330326421 .node .label{text-align:center;}#mermaid-1688330326421 .node.clickable{cursor:pointer;}#mermaid-1688330326421 .arrowheadPath{fill:undefined;}#mermaid-1688330326421 .edgePath .path{stroke:#0b0b0b;stroke-width:2.0px;}#mermaid-1688330326421 .flowchart-link{stroke:#0b0b0b;fill:none;}#mermaid-1688330326421 .edgeLabel{background-color:hsl(-120, 0%, 99.2156862745%);text-align:center;}#mermaid-1688330326421 .edgeLabel rect{opacity:0.5;background-color:hsl(-120, 0%, 99.2156862745%);fill:hsl(-120, 0%, 99.2156862745%);}#mermaid-1688330326421 .cluster rect{fill:hsl(180, 0%, 100%);stroke:hsl(180, 0%, 90%);stroke-width:1px;}#mermaid-1688330326421 .cluster text{fill:rgb(0, 0, 0);}#mermaid-1688330326421 .cluster span{color:rgb(0, 0, 0);}#mermaid-1688330326421 div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:monospace;font-size:12px;background:hsl(180, 0%, 100%);border:1px solid undefined;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-1688330326421 .flowchartTitleText{text-anchor:middle;font-size:18px;fill:#333;}#mermaid-1688330326421 :root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}</style><g><marker orient="auto" markerHeight="12" markerWidth="12" markerUnits="userSpaceOnUse" refY="5" refX="10" viewBox="0 0 12 20" class="marker flowchart" id="flowchart-pointEnd"><path style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 0 0 L 10 5 L 0 10 z"></path></marker><marker orient="auto" markerHeight="12" markerWidth="12" markerUnits="userSpaceOnUse" refY="5" refX="0" viewBox="0 0 10 10" class="marker flowchart" id="flowchart-pointStart"><path style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 0 5 L 10 10 L 10 0 z"></path></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5" refX="11" viewBox="0 0 10 10" class="marker flowchart" id="flowchart-circleEnd"><circle style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" r="5" cy="5" cx="5"></circle></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5" refX="-1" viewBox="0 0 10 10" class="marker flowchart" id="flowchart-circleStart"><circle style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" r="5" cy="5" cx="5"></circle></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5.2" refX="12" viewBox="0 0 11 11" class="marker cross flowchart" id="flowchart-crossEnd"><path style="stroke-width: 2; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 1,1 l 9,9 M 10,1 l -9,9"></path></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5.2" refX="-1" viewBox="0 0 11 11" class="marker cross flowchart" id="flowchart-crossStart"><path style="stroke-width: 2; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 1,1 l 9,9 M 10,1 l -9,9"></path></marker><g class="root"><g class="clusters"><g id="subGraph4" class="cluster default"><rect height="1316.296875" width="382.96875" y="231.375" x="0" ry="0" rx="0" style=""></rect><g transform="translate(166.4609375, 231.375)" class="cluster-label"><foreignObject height="20.796875" width="50.046875"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">frame 1</span></div></foreignObject></g></g><g id="subGraph0" class="cluster default"><rect height="148.1875" width="149.34375" y="719.125" x="21.46875" ry="0" rx="0" style=""></rect><g transform="translate(63.9765625, 719.125)" class="cluster-label"><foreignObject height="20.796875" width="64.328125"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">frame 2-1</span></div></foreignObject></g></g><g id="subGraph3" class="cluster default"><rect height="803.546875" width="172.15625" y="719.125" x="190.8125" ry="0" rx="0" style=""></rect><g transform="translate(244.7265625, 719.125)" class="cluster-label"><foreignObject height="20.796875" width="64.328125"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">frame 2-2</span></div></foreignObject></g></g><g id="subGraph2" class="cluster default"><rect height="390.578125" width="132.15625" y="1107.09375" x="210.8125" ry="0" rx="0" style=""></rect><g transform="translate(237.578125, 1107.09375)" class="cluster-label"><foreignObject height="20.796875" width="78.625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">frame 2-2-1</span></div></foreignObject></g></g><g id="subGraph1" class="cluster default"><rect height="85.796875" width="92.921875" y="1386.875" x="230.4296875" ry="0" rx="0" style=""></rect><g transform="translate(230.4296875, 1386.875)" class="cluster-label"><foreignObject height="20.796875" width="92.921875"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">frame 2-2-1-1</span></div></foreignObject></g></g></g><g class="edgePaths"><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-start LE-1_1" id="L-start-1_1-0" d="M172.9375,160.578125L172.9375,166.47786458333334C172.9375,172.37760416666666,172.9375,184.17708333333334,172.9375,195.9765625C172.9375,207.77604166666666,172.9375,219.57552083333334,172.9375,229.64192708333334C172.9375,239.70833333333334,172.9375,248.04166666666666,172.9375,252.20833333333334L172.9375,256.375"></path><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-1_1 LE-1_2" id="L-1_1-1_2-0" d="M111.8203125,399.191581672309L103.16015625,408.05157847692413C94.5,416.91157528153934,77.1796875,434.63156889076964,70.36977372872771,449.39130527871816C63.55985995745542,464.1510416666667,67.26034491491086,475.9505208333333,69.11058739363857,481.8502604166667L70.96082987236629,487.75"></path><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-1_2 LE-1_3" id="L-1_2-1_3-0" d="M84.10919403785115,648.328125L83.22510961487596,654.2278645833334C82.34102519190077,660.1276041666666,80.57285634595038,671.9270833333334,79.68877192297519,683.7265625C78.8046875,695.5260416666666,78.8046875,707.3255208333334,80.59055959378954,720.8580729166666C82.37643168757909,734.390625,85.94817587515816,749.65625,87.7340479689477,757.2890625L89.51992006273724,764.921875"></path><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-1_3 LE-1_2" id="L-1_3-1_2-0" d="M106.58039362347111,764.921875L109.39642176955925,757.2890625C112.21244991564741,749.65625,117.8445062078237,734.390625,120.66053435391184,720.8580729166666C123.4765625,707.3255208333334,123.4765625,695.5260416666666,122.08250598986471,683.7265625C120.68844947972941,671.9270833333334,117.90033645945886,660.1276041666666,116.50627994932357,654.2278645833334L115.11222343918828,648.328125"></path><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-1_2 LE-1_1" id="L-1_2-1_1-0" d="M149.43894242132632,487.75L153.35536868443862,481.8502604166667C157.2717949475509,475.9505208333333,165.10464747377543,464.1510416666667,169.02107373688773,452.3515625C172.9375,440.5520833333333,172.9375,428.7526041666667,172.9375,422.8528645833333L172.9375,416.953125"></path><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-1_1 LE-2_1" id="L-1_1-2_1-0" d="M234.0546875,404.68024410980007L241.19401041666666,412.62546384150005C248.33333333333334,420.57068357320003,262.6119791666667,436.4611230366,269.7513020833333,450.3060823516334C276.890625,464.1510416666667,276.890625,475.9505208333333,276.890625,481.8502604166667L276.890625,487.75"></path><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-2_1 LE-2_2" id="L-2_1-2_2-0" d="M276.890625,648.328125L276.890625,654.2278645833334C276.890625,660.1276041666666,276.890625,671.9270833333334,276.890625,683.7265625C276.890625,695.5260416666666,276.890625,707.3255208333334,276.890625,717.3919270833334C276.890625,727.4583333333334,276.890625,735.7916666666666,276.890625,739.9583333333334L276.890625,744.125"></path><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-2_2 LE-2_3" id="L-2_2-2_3-0" d="M276.890625,842.3125L276.890625,846.4791666666666C276.890625,850.6458333333334,276.890625,858.9791666666666,276.890625,869.0455729166666C276.890625,879.1119791666666,276.890625,890.9114583333334,276.890625,902.7109375C276.890625,914.5104166666666,276.890625,926.3098958333334,276.890625,932.2096354166666L276.890625,938.109375"></path><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-2_3 LE-2_4" id="L-2_3-2_4-0" d="M276.890625,1036.296875L276.890625,1042.1966145833333C276.890625,1048.0963541666667,276.890625,1059.8958333333333,276.890625,1071.6953125C276.890625,1083.4947916666667,276.890625,1095.2942708333333,276.890625,1105.3606770833333C276.890625,1115.4270833333333,276.890625,1123.7604166666667,276.890625,1127.9270833333333L276.890625,1132.09375"></path><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-2_4 LE-2_5" id="L-2_4-2_5-0" d="M276.890625,1188.6875L276.890625,1194.5872395833333C276.890625,1200.4869791666667,276.890625,1212.2864583333333,276.890625,1224.0859375C276.890625,1235.8854166666667,276.890625,1247.6848958333333,276.890625,1253.5846354166667L276.890625,1259.484375"></path><path marker-end="url(#flowchart-pointEnd)" style="fill:none;" class="edge-thickness-normal edge-pattern-solid flowchart-link LS-2_5 LE-2_6" id="L-2_5-2_6-0" d="M276.890625,1316.078125L276.890625,1321.9778645833333C276.890625,1327.8776041666667,276.890625,1339.6770833333333,276.890625,1351.4765625C276.890625,1363.2760416666667,276.890625,1375.0755208333333,276.890625,1385.1419270833333C276.890625,1395.2083333333333,276.890625,1403.5416666666667,276.890625,1407.7083333333333L276.890625,1411.875"></path></g><g class="edgeLabels"><g transform="translate(172.9375, 195.9765625)" class="edgeLabel"><g transform="translate(-3.578125, -10.3984375)" class="label"><foreignObject height="20.796875" width="7.15625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">1</span></div></foreignObject></g></g><g transform="translate(59.859375, 452.3515625)" class="edgeLabel"><g transform="translate(-3.578125, -10.3984375)" class="label"><foreignObject height="20.796875" width="7.15625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">2</span></div></foreignObject></g></g><g transform="translate(78.8046875, 683.7265625)" class="edgeLabel"><g transform="translate(-3.578125, -10.3984375)" class="label"><foreignObject height="20.796875" width="7.15625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">3</span></div></foreignObject></g></g><g transform="translate(123.4765625, 683.7265625)" class="edgeLabel"><g transform="translate(-3.578125, -10.3984375)" class="label"><foreignObject height="20.796875" width="7.15625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">4</span></div></foreignObject></g></g><g transform="translate(172.9375, 452.3515625)" class="edgeLabel"><g transform="translate(-3.578125, -10.3984375)" class="label"><foreignObject height="20.796875" width="7.15625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">5</span></div></foreignObject></g></g><g transform="translate(276.890625, 452.3515625)" class="edgeLabel"><g transform="translate(-3.578125, -10.3984375)" class="label"><foreignObject height="20.796875" width="7.15625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">6</span></div></foreignObject></g></g><g transform="translate(276.890625, 683.7265625)" class="edgeLabel"><g transform="translate(-3.578125, -10.3984375)" class="label"><foreignObject height="20.796875" width="7.15625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">7</span></div></foreignObject></g></g><g transform="translate(276.890625, 902.7109375)" class="edgeLabel"><g transform="translate(-3.578125, -10.3984375)" class="label"><foreignObject height="20.796875" width="7.15625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">8</span></div></foreignObject></g></g><g transform="translate(276.890625, 1071.6953125)" class="edgeLabel"><g transform="translate(-3.578125, -10.3984375)" class="label"><foreignObject height="20.796875" width="7.15625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">9</span></div></foreignObject></g></g><g transform="translate(276.890625, 1224.0859375)" class="edgeLabel"><g transform="translate(-7.1484375, -10.3984375)" class="label"><foreignObject height="20.796875" width="14.296875"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">10</span></div></foreignObject></g></g><g transform="translate(276.890625, 1351.4765625)" class="edgeLabel"><g transform="translate(-7.1484375, -10.3984375)" class="label"><foreignObject height="20.796875" width="14.296875"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel">11</span></div></foreignObject></g></g></g><g class="nodes"><g transform="translate(172.9375, 336.6640625)" id="flowchart-1_1-70" class="node default"><rect height="160.578125" width="122.234375" y="-80.2890625" x="-61.1171875" ry="0" rx="0" style="" class="basic label-container"></rect><g transform="translate(-53.6171875, -72.7890625)" style="" class="label"><foreignObject height="145.578125" width="107.234375"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">-  <b style="color:limegreen">1</b> 2 3 4 5 6 7<br>A <b style="color:limegreen">1</b> 0 0 1 0 0 1<br>B <b style="color:limegreen">1</b> 0 0 1 0 0 0<br>C 0 0 0 1 1 0 1<br>D 0 0 1 0 1 1 0<br>E 0 1 1 0 0 1 1<br>F 0 1 0 0 0 0 1</span></div></foreignObject></g></g><g transform="translate(96.140625, 568.0390625)" id="flowchart-1_2-72" class="node default"><rect height="160.578125" width="122.28125" y="-80.2890625" x="-61.140625" ry="0" rx="0" style="" class="basic label-container"></rect><g transform="translate(-53.640625, -72.7890625)" style="" class="label"><foreignObject height="145.578125" width="107.28125"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">-  <b style="color:red">1</b> 2 3 <b style="color:red">1</b> 5 6 <b style="color:red">1</b><br><b style="color:limegreen">A</b> <b style="color:limegreen">1</b> 0 0 <b style="color:limegreen">1</b> 0 0 <b style="color:limegreen">1</b><br>B <b style="color:red">1</b> 0 0 <b style="color:red">1</b> 0 0 0<br>C 0 0 0 <b style="color:red">1</b> 1 0 <b style="color:red">1</b><br>D 0 0 1 0 1 1 0<br>E 0 1 1 0 0 1 <b style="color:red">1</b><br>F 0 1 0 0 0 0 <b style="color:red">1</b></span></div></foreignObject></g></g><g transform="translate(276.890625, 568.0390625)" id="flowchart-2_1-84" class="node default"><rect height="160.578125" width="122.265625" y="-80.2890625" x="-61.1328125" ry="0" rx="0" style="" class="basic label-container"></rect><g transform="translate(-53.6328125, -72.7890625)" style="" class="label"><foreignObject height="145.578125" width="107.265625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">-  <b style="color:red">1</b> 2 3 <b style="color:red">1</b> 5 6 1<br>A <b style="color:red">1</b> 0 0 <b style="color:red">1</b> 0 0 1<br><b style="color:limegreen">B</b> <b style="color:limegreen">1</b> 0 0 <b style="color:limegreen">1</b> 0 0 0<br>C 0 0 0 <b style="color:red">1</b> 1 0 1<br>D 0 0 1 0 1 1 0<br>E 0 1 1 0 0 1 1<br>F 0 1 0 0 0 0 1</span></div></foreignObject></g></g><g transform="translate(276.890625, 793.21875)" id="flowchart-2_2-87" class="node default default"><rect height="98.1875" width="93.640625" y="-49.09375" x="-46.8203125" ry="0" rx="0" style="" class="basic label-container"></rect><g transform="translate(-39.3203125, -41.59375)" style="" class="label"><foreignObject height="83.1875" width="78.640625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">- 2 3 <b style="color:limegreen">5</b> 6 7<br>D 0 1 <b style="color:limegreen">1</b> 1 0<br>E 1 1 0 1 1<br>F 1 0 0 0 1<br></span></div></foreignObject></g></g><g transform="translate(276.890625, 987.203125)" id="flowchart-2_3-89" class="node default default"><rect height="98.1875" width="93.6875" y="-49.09375" x="-46.84375" ry="0" rx="0" style="" class="basic label-container"></rect><g transform="translate(-39.34375, -41.59375)" style="" class="label"><foreignObject height="83.1875" width="78.6875"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">- 2 <b style="color:red">3</b> <b style="color:red">5</b> <b style="color:red">6</b> 7<br><b style="color:limegreen">D</b> 0 <b style="color:limegreen">1</b> <b style="color:limegreen">1</b> <b style="color:limegreen">1</b> 0<br>E 1 <b style="color:red">1</b> 0 <b style="color:red">1</b> 1<br>F 1 0 0 0 1<br></span></div></foreignObject></g></g><g transform="translate(276.890625, 1160.390625)" id="flowchart-2_4-93" class="node default default"><rect height="56.59375" width="50.75" y="-28.296875" x="-25.375" ry="0" rx="0" style="" class="basic label-container"></rect><g transform="translate(-17.875, -20.796875)" style="" class="label"><foreignObject height="41.59375" width="35.75"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">- <b style="color:limegreen">2</b> 7<br>F <b style="color:limegreen">1</b> 1</span></div></foreignObject></g></g><g transform="translate(276.890625, 1287.78125)" id="flowchart-2_5-95" class="node default default"><rect height="56.59375" width="50.78125" y="-28.296875" x="-25.390625" ry="0" rx="0" style="" class="basic label-container"></rect><g transform="translate(-17.890625, -20.796875)" style="" class="label"><foreignObject height="41.59375" width="35.78125"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">- <b style="color:red">2</b> <b style="color:red">7</b><br><b style="color:limegreen">F</b> <b style="color:red">1</b> <b style="color:red">1</b></span></div></foreignObject></g></g><g transform="translate(276.890625, 1429.7734375)" id="flowchart-2_6-99" class="node default default"><rect height="35.796875" width="22.15625" y="-17.8984375" x="-11.078125" ry="0" rx="0" style="fill:limegreen;" class="basic label-container"></rect><g transform="translate(-3.578125, -10.3984375)" style="" class="label"><foreignObject height="20.796875" width="7.15625"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">-</span></div></foreignObject></g></g><g transform="translate(96.140625, 793.21875)" id="flowchart-1_3-76" class="node default"><rect height="56.59375" width="79.34375" y="-28.296875" x="-39.671875" ry="0" rx="0" style="fill:#ff8888;" class="basic label-container"></rect><g transform="translate(-32.171875, -20.796875)" style="" class="label"><foreignObject height="41.59375" width="64.34375"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">- <b style="color:limegreen">2</b> 3 5 6<br>D 0 1 1 1</span></div></foreignObject></g></g><g transform="translate(172.9375, 80.2890625)" id="flowchart-start-68" class="node default"><rect height="160.578125" width="122.21875" y="-80.2890625" x="-61.109375" ry="0" rx="0" style="" class="basic label-container"></rect><g transform="translate(-53.609375, -72.7890625)" style="" class="label"><foreignObject height="145.578125" width="107.21875"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">-  1 2 3 4 5 6 7<br>A 1 0 0 1 0 0 1<br>B 1 0 0 1 0 0 0<br>C 0 0 0 1 1 0 1<br>D 0 0 1 0 1 1 0<br>E 0 1 1 0 0 1 1<br>F 0 1 0 0 0 0 1</span></div></foreignObject></g></g></g></g></g></svg>)

1. The smallest number of `1`s in any column is 2. Column 1 is the first column with 2 `1`s, so choose it. 
2. We then pick one of the `1`s in column 1 randomly. In this case, we choose A. **We add A to the partial solution**. We then find any rows that clash with A and remove them by checking for 1s in the same columns as those in row A. Clashing `1`s are shown in red.
3. Remove all clashing rows and remove all columns in which A has a `1` to get a smaller solution matrix. Once again, select the column with the fewest `1`s. It's column 2, which has no `1`s at all: this branch has no valid solutions. It's time to backtrack.
4. Add back all rows and columns in the reverse order that they were removed to get back to the full matrix.
5. Continue stepping back up to the point at which we picked one of column 1's `1`s. **A is removed from the partial solution.**
6. We now pick another of the `1`s in column 1. This time, it's row B. **B is added to the partial solution.**
7. As before, find any clashing rows and columns and removed them.
8. Column 5 has the fewest `1`s, so pick it. It only has a single `1`, so we choose row D. **Row D is added to the partial solution.**
9. Find any clashing rows and columns and removed them.
10. Rows 2 and 7 both have a single `1`, so pick 2 since it's first. We then choose the only `1`, which is in row F. **Row F is added to the partial solution.**
11. Remove clashing rows.

At the end of that process, we ended up with a matrix that has no columns left, which means that our partial solution is a full cover! Our solution has rows B, D, and F and looks like:


<!-- Mermaid diagram: /mermaid/algXFinal.md -->
<svg aria-roledescription="flowchart-v2" role="graphics-document document" viewBox="-8 -8 138.21875 114.1875" style="max-width: 138.21875px;" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns="http://www.w3.org/2000/svg" width="100%" id="mermaid-1688330551473"><style>#mermaid-1688330551473{font-family:monospace;font-size:13px;fill:#333;}#mermaid-1688330551473 .error-icon{fill:hsl(180, 0%, 100%);}#mermaid-1688330551473 .error-text{fill:rgb(0, 0, 0);stroke:rgb(0, 0, 0);}#mermaid-1688330551473 .edge-thickness-normal{stroke-width:2px;}#mermaid-1688330551473 .edge-thickness-thick{stroke-width:3.5px;}#mermaid-1688330551473 .edge-pattern-solid{stroke-dasharray:0;}#mermaid-1688330551473 .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-1688330551473 .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-1688330551473 .marker{fill:#0b0b0b;stroke:#0b0b0b;}#mermaid-1688330551473 .marker.cross{stroke:#0b0b0b;}#mermaid-1688330551473 svg{font-family:monospace;font-size:13px;}#mermaid-1688330551473 .label{font-family:monospace;color:#333;}#mermaid-1688330551473 .cluster-label text{fill:rgb(0, 0, 0);}#mermaid-1688330551473 .cluster-label span{color:rgb(0, 0, 0);}#mermaid-1688330551473 .label text,#mermaid-1688330551473 span{fill:#333;color:#333;}#mermaid-1688330551473 .node rect,#mermaid-1688330551473 .node circle,#mermaid-1688330551473 .node ellipse,#mermaid-1688330551473 .node polygon,#mermaid-1688330551473 .node path{fill:#fdfdfd;stroke:hsl(0, 0%, 89.2156862745%);stroke-width:1px;}#mermaid-1688330551473 .node .label{text-align:center;}#mermaid-1688330551473 .node.clickable{cursor:pointer;}#mermaid-1688330551473 .arrowheadPath{fill:undefined;}#mermaid-1688330551473 .edgePath .path{stroke:#0b0b0b;stroke-width:2.0px;}#mermaid-1688330551473 .flowchart-link{stroke:#0b0b0b;fill:none;}#mermaid-1688330551473 .edgeLabel{background-color:hsl(-120, 0%, 99.2156862745%);text-align:center;}#mermaid-1688330551473 .edgeLabel rect{opacity:0.5;background-color:hsl(-120, 0%, 99.2156862745%);fill:hsl(-120, 0%, 99.2156862745%);}#mermaid-1688330551473 .cluster rect{fill:hsl(180, 0%, 100%);stroke:hsl(180, 0%, 90%);stroke-width:1px;}#mermaid-1688330551473 .cluster text{fill:rgb(0, 0, 0);}#mermaid-1688330551473 .cluster span{color:rgb(0, 0, 0);}#mermaid-1688330551473 div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:monospace;font-size:12px;background:hsl(180, 0%, 100%);border:1px solid undefined;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-1688330551473 .flowchartTitleText{text-anchor:middle;font-size:18px;fill:#333;}#mermaid-1688330551473 :root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}</style><g><marker orient="auto" markerHeight="12" markerWidth="12" markerUnits="userSpaceOnUse" refY="5" refX="10" viewBox="0 0 12 20" class="marker flowchart" id="flowchart-pointEnd"><path style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 0 0 L 10 5 L 0 10 z"></path></marker><marker orient="auto" markerHeight="12" markerWidth="12" markerUnits="userSpaceOnUse" refY="5" refX="0" viewBox="0 0 10 10" class="marker flowchart" id="flowchart-pointStart"><path style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 0 5 L 10 10 L 10 0 z"></path></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5" refX="11" viewBox="0 0 10 10" class="marker flowchart" id="flowchart-circleEnd"><circle style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" r="5" cy="5" cx="5"></circle></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5" refX="-1" viewBox="0 0 10 10" class="marker flowchart" id="flowchart-circleStart"><circle style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" r="5" cy="5" cx="5"></circle></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5.2" refX="12" viewBox="0 0 11 11" class="marker cross flowchart" id="flowchart-crossEnd"><path style="stroke-width: 2; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 1,1 l 9,9 M 10,1 l -9,9"></path></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5.2" refX="-1" viewBox="0 0 11 11" class="marker cross flowchart" id="flowchart-crossStart"><path style="stroke-width: 2; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 1,1 l 9,9 M 10,1 l -9,9"></path></marker><g class="root"><g class="clusters"></g><g class="edgePaths"></g><g class="edgeLabels"></g><g class="nodes"><g transform="translate(61.109375, 49.09375)" id="flowchart-start-2" class="node default"><rect height="98.1875" width="122.21875" y="-49.09375" x="-61.109375" ry="0" rx="0" style="" class="basic label-container"></rect><g transform="translate(-53.609375, -41.59375)" style="" class="label"><foreignObject height="83.1875" width="107.21875"><div style="display: inline-block; white-space: nowrap;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel">-  1 2 3 4 5 6 7<br>B 1 0 0 1 0 0 0<br>D 0 0 1 0 1 1 0<br>F 0 1 0 0 0 0 1</span></div></foreignObject></g></g></g></g></g></svg>

### Dancing towards a solution

Once you've worked through Algorithm X, it seems like a fairly straightforward algorithm. If you looked no further and wrote a solution for it, you'd have a great solver for toy exact cover problems. The issue is that Algorithm X has exponential time complexity. We could give up and go find something else, but Knuth proposed a nice way to improve the algorithm's performance so that while it's theoretical time complexity is still abysmal, you might actually live to see a solution. That improvement is a sparse matrix implementation based on *Dancing Links* (DLX).

The premise of dancing links is that in a doubly-linked list, you can *cover* a node by removing references to it from its neighbouring nodes (effectively removing it from the list) while keeping the covered node's pointers intact. To replace the node, you simply reverse the process. The pseudocode looks like:

```
cover(node):
  node.prev.next = node.next
  node.next.prev = node.prev

uncover(node):
  node.next.prev = node
  node.prev.next = node
```

And here's a more visual representation over the cover operation on node B in an example doubly linked list:

![](http://www.plantuml.com/plantuml/svg/TPAnQiGm38PtFOMuReKizjP2ELkFKJeOOYu1DmSvbr9Atxso948CIIP_looKJnYkZvhM-lLPW0yrutt9-0l8tvJJYCCwl64G3WfH82gG0E5GzfMozTGq5sM2Fu3tvmySHYQU0ZQmVjzjeF8bN30zCBXz5YMB3fzkQ_fvBVEINyfycwhW_YR9JUgwcud4xT1Lslz9fHtSfktCMvDWI8gnHt8cRDKTsW8iuViIjKMmlow2xRT5J6lEXwlm1wlm1zB-LDgVIljDsfkq9i7G1aL3cq6CfQyMwpLfIyWtNROqtUO8QYe1lmYuvcNa1_WF)

A key observation in why DLX makes sense when implementing Algorithm X is that our solution matrix is mostly empty. Especially as the width of the solution matrix grows, we end up with rows that have a handful of 1s scattered through their thousands of columns. That makes Algorithm X super inefficient on a regular two-dimensional list because:
- The time complexity of counting 1s in a column is relative to the number of rows (and not the number of 1s) because we have to iterate through every row to check if the value is a 1
- If we *actually* remove rows and columns from the list, we have to shift every element after it forward one space. We then have to reverse that operation when adding the rows and columns back. In algorithm X, we spend a lot of time adding and removing rows and columns, so that really adds up, especially as we move towards much larger solution spaces.
- When we cover a row, we have to iterate through each column and check whether it has a one in the row being covered to figure out whether it needs to be covered in the first place. Similarly to counting 1s, having to iterate through the columns make the complexity scale with the size of the problem space size, instead of the number of clashing columns.
- The matrices just start to take up inordinate amounts of space. In one of the problems I attempted to solve, I had a solution matrix slightly larger than 1000x10000. If you want to copy the matrix to parallelise the problem (which you do want to do), you're going to run into memory issues pretty quick.

There are undoubtedly other ways to address the issues above, but DLX is one way to do so. The DLX solution to the exact cover problem uses a two-dimensional circular doubly-linked list. Here's an example of one of these matrices; note how we're not storing any `0`s. Just the ones:

![](/assets/images/algorithm-x/sparse-matrix.svg)

With this type of matrix, if we want to find the next `1` to the right of the current node, instead of having to work our way through columns, checking each of them for a `1` in the right row, we can go straight there! Covering rows and colums is now also a lot more efficient, since we're just rearranging some pointers instead of shifting all of those rows. We also somewhat solve our space problem, since we're not longer storing all those useless `0`s. Counting is also more efficient with this approach, since it now has time complexity that's proportional to the number of `1`s in a column. 

Some extra bits:
- In the canonical implementation, the headers themselves are just nodes in the matrix that have a special flag set. I would highly recommend using that implementation. I learned the hard way that attempts to tidy it up will make one's life hard.
  - The primary advantage of the original approach is that you can re-use the exact same methods to work with headers as any other node.
- In addition to the pointers in the diagram above, all nodes also:
  - keep track of their row
  - have a pointer to their column header
- Counting `1`s in a column is faster with the sparse matrix, but it's *even* faster to just maintain the count in the header. 
- I found it interesting that, to get a value equivalent to the "height" of a regular matrix, you don't look for the column with the most nodes. Instead, you scan all columns for the one that has a node with the largest "row" value.

### How'd it go?

tl;dr not great. 

The algorithm works! It generates valid solutions. Unfortunately, however, there seem to be two hurdles to overcome if I want to actually get _good_ results. 

Firstly, there are many possible ways to place the lego bricks. Far too many for me to evaluate them manually. That means that I need some way of telling the algorithm which solutions are _better_ than others. This problem is akin to selecting a [fitness function](https://en.wikipedia.org/wiki/Fitness_function#:~:text=A%20fitness%20function%20is%20a,to%20achieving%20the%20set%20aims.) for a genetic algorithm and is particularly difficult. I worked through a few options, each of which used various weightings and combinations of the difference between the desired and actual colour for a given pixel and the distance of those pixels from the centre of the image. The error functions were all fairly naive and didn't give great results. Centre-weighting errors in particular was a poor-mans attempt at favouring correct colours in "more important" areas of the image. I suspect that outcomes could be significantly improved by using edge detection and minimizing contrast errors around those edges. 

Secondly, some images will just be better-suited to this lego brick tiling. If I've only got 90 studs worth of white bricks, an image that has more than 90 white pixels _cannot_ have all of the white pixels correct. There's an art to downscaling and reducing the colour depth of images. Investing more time in finding approaches that suit the final use-case could result in images that are fundamentally _easier_ to tile. 

Either way, here's one solution:


| | | |
| :---: | :---: | :---: |
| The original image:<br/>![original image](/assets/images/algorithm-x/Poo.png) | The version that the algorithm tried to tile:<br/>![scaled](/assets/images/algorithm-x/LowResPoo.png) |  The result:<br/>![scaled](/assets/images/algorithm-x/TiledPoo.png) |

### Closing out

For now, I've pushed the mess of code that gave us the work of art above to GitHub. One day, when I'm motivated, I'll come back, clean it up, and hopefully implement some of the improvements that I think will get the results to a level that I'm happy with.