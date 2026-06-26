# Algorithm Project: shortest paths on social networks

We tried six shortest-path algorithms on five real social networks and watched how each one
behaves: how long it takes to answer, how much memory it uses, and how that cost changes as the
graphs get bigger. The question we kept coming back to was a practical one. If you need the
shortest path between two people in a network, which algorithm should you reach for, and does the
answer change once the network gets large?

## datasets

The five graphs are all social or trust networks. The smallest has under 4,000 nodes and the
largest has more than 100,000.

- `soc-sign-bitcoinalpha`: 3,783 nodes, trust ratings from the Bitcoin Alpha market
- `soc-sign-bitcoinotc`: 5,881 nodes, trust ratings from the Bitcoin OTC market
- `soc-advogato`: 6,551 nodes, weighted trust links between developers
- `soc-epinions`: 26,588 nodes, who trusts whom on Epinions
- `soc-LiveMocha`: 104,103 nodes, the LiveMocha language-learning network

There was one thing to sort out before any of this would work. The Bitcoin ratings run from -10
to 10, and shortest-path algorithms can't handle negative weights. We turned each rating into a
cost of `11 - rating`, which keeps every edge between 1 and 21.

## algorithms and who wrote them

- Dijkstra with a binary heap: Aya and Mariam
- ALT (A* with landmark heuristics) and PLL (pruned landmark labeling): Ahmad and Layan
- Bidirectional Dijkstra and a landmark distance oracle: Laila, Ahmed Ismail, Zeina, and Yehya
- Weighted single-source shortest paths (the Thorup notebook): Jana and Mohamed

## notebooks

- `dijkstra_analysis.ipynb`: Dijkstra, with timing, memory, and a growth curve.
- `alt_pll_analysis.ipynb`: ALT and PLL on the same datasets.
- `Algorithms_project_Bidirectional_Dijkstra_&_Distance_oracles.ipynb`: bidirectional search
  and the distance oracle.
- `algorithms_project_MandJ_2.ipynb`: weighted single-source shortest paths. It adds the real
  weight on each edge instead of counting hops, and includes a growth curve.

## how each algorithm finds the path

Each clip traces a real run of one algorithm on a small graph, so you can watch it work instead
of just seeing the answer. The colours mean the same thing every time:

- green is the start (S), gold is the goal (T)
- grey nodes haven't been touched yet
- the coloured nodes are the ones the algorithm is finished with
- yellow is the frontier, the nodes still sitting in the queue
- red is the final shortest path

### Dijkstra (Aya and Mariam)

![Dijkstra](media/videos/1_dijkstra.gif)

Dijkstra grabs the closest unfinished node, marks it done, and drops its neighbours into the
queue. It has no idea where T is, so the yellow frontier just spreads outward in every direction.
By the time it reaches T it has already settled most of the graph. The answer is always right, but
it does plenty of work to get there.

### Thorup, weighted version (Jana and Mohamed)

![Thorup](media/videos/2_thorup.gif)

Instead of pulling one node at a time, this sorts nodes into buckets by distance and clears a
whole bucket in one step. The buckets are what let it skip the sorting that slows Dijkstra down.
This is the fixed version. The first draft added 1 per hop and ignored the edge weights, so it was
really just counting steps. It now adds the real weight on each edge.

### ALT, goal-directed A* (Ahmad and Layan)

![ALT](media/videos/3_alt.gif)

ALT runs the same queue as Dijkstra but with a hint. A few landmark nodes give a lower bound on
how far each node still is from T, so the search leans toward nodes that look closer to the goal.
The explored area stretches toward T rather than fanning out in a circle, and far fewer nodes
change colour than in the Dijkstra clip. The answer is still exact.

### PLL, hub labeling (Ahmad and Layan)

![PLL](media/videos/4_pll.gif)

PLL front-loads the work. For every node it keeps a short list of hub nodes and distances. When
you ask a question it doesn't search the graph at all. It lines up S's hubs and T's hubs, picks
the best one they share, and adds the two stored distances. That's why the clip jumps straight to
the hubs and the path. The cost is building those labels once.

### Bidirectional Dijkstra (Laila, Ahmed Ismail, Zeina and Yehya)

![Bidirectional](media/videos/5_bidirectional.gif)

Two searches run at once, one forward from S in blue and one backward from T in orange, taking
turns. They grow toward each other and stop the moment they touch. Two small circles cover the
distance instead of one big one, so it settles far fewer nodes than plain Dijkstra, with no setup
and still an exact answer.

### Distance oracle (Laila, Ahmed Ismail, Zeina and Yehya)

![Distance oracle](media/videos/6_distance_oracle.gif)

The oracle keeps a few landmarks and answers by routing through the best one, S to landmark to T.
It's the quickest to respond and barely cares how big the graph is. The catch is that its answer
is an estimate, not always the exact shortest distance.

## the benchmark graphs

We timed every algorithm on every dataset over sub-graphs of growing size. The first figure shows
all 30 combinations together, with the algorithms down the rows and the datasets across the
columns. Each small panel is one algorithm on one dataset, time against number of nodes.

![all 30 plots](media/figures/30_grid_time_vs_size.png)

The next five figures put all six algorithms on one chart, one per dataset, so you can see how
each one grows. The `b` in each legend is the fitted growth rate: near 1 means the time grows
roughly in step with the graph, near 2 means it grows much faster. PLL and the oracle stay almost
flat because their answer time barely depends on size.

![Bitcoin Alpha](media/figures/growth_bitcoin_alpha.png)

![Bitcoin OTC](media/figures/growth_bitcoin_otc.png)

![Advogato](media/figures/growth_advogato.png)

![Epinions](media/figures/growth_epinions.png)

![LiveMocha](media/figures/growth_livemocha.png)

## summary and which algorithm wins

The six algorithms sorted themselves into three groups.

**The full-scan methods, Dijkstra and Thorup, are the slowest.** They build the entire
shortest-path tree out from the source, so their time climbs steeply as the graph grows. They're
simple and always right, but they do the most work per question.

**Bidirectional Dijkstra is the best choice when you want one exact answer with no setup.**
Meeting in the middle means it settles far fewer nodes than plain Dijkstra. It stays low on every
chart, needs no preparation, and is still exact. If you only ask a question here and there, this
is the one to use.

**PLL and the distance oracle win once you ask many questions.** Both spend a one-time cost
building an index and then answer almost instantly, flat across every dataset. The difference is
accuracy. PLL stays exact, while the oracle trades that for a fast estimate.

**So is there a single best? It depends on how often you ask.**

- If you query the same graph over and over, **PLL is the winner.** Its answers come back in well
  under a millisecond, the time barely grows with the graph, and unlike the oracle they're exact.
  The only price is building the labels once up front.
- If you only need one answer and haven't prepared anything, **Bidirectional Dijkstra is the
  winner**, since it's exact, needs no setup, and still beats the full-scan methods.

Forced to pick one overall, we'd say **PLL**. On these social networks it gave the fastest exact
answers and stayed the flattest as the graphs grew, which is what you want when the graph stays
put and the questions keep coming.
