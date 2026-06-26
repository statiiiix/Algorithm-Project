# Algorithm Project: shortest paths on social networks

This project measures how different shortest-path algorithms behave on real social-network
graphs. We took five public datasets, ran each algorithm on them, and compared running time,
memory use, and how the cost grows as the graphs get bigger.

## datasets

All five are social or trust networks. They range from about 3,800 nodes up to 104,000.

- `soc-sign-bitcoinalpha`: 3,783 nodes, Bitcoin Alpha trust ratings
- `soc-sign-bitcoinotc`: 5,881 nodes, Bitcoin OTC trust ratings
- `soc-advogato`: 6,551 nodes, weighted trust links between developers
- `soc-epinions`: 26,588 nodes, who-trusts-whom on Epinions
- `soc-LiveMocha`: 104,103 nodes, the LiveMocha language-learning network

The Bitcoin ratings run from -10 to 10. Shortest-path algorithms need non-negative weights,
so we map each rating to a cost of `11 - rating`, which keeps every weight between 1 and 21.

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
- `algorithms_project_MandJ_2.ipynb`: weighted single-source shortest paths. It sums the real
  weight along each edge instead of counting hops, and includes a growth curve.

## how each algorithm finds the path

Each clip below traces the real run of one algorithm on a small graph, so you can watch how it
works rather than just see the answer. The colours are the same in every clip:

- **S** (green) is the start, **T** (gold) is the goal
- grey nodes have not been touched yet
- the coloured nodes are the ones the algorithm has finished with (settled)
- yellow is the live frontier, the nodes waiting in the queue
- the red trail is the final shortest path

### Dijkstra (Aya and Mariam)

![Dijkstra](media/videos/1_dijkstra.gif)

Dijkstra always takes the closest unfinished node, marks it done, and adds its neighbours to the
queue. Because it has no idea where T is, the yellow frontier grows outward in a circle in every
direction at once. By the time it reaches T it has already settled most of the graph. It always
gives the correct answer, but it does a lot of work to get there.

### Thorup, weighted version (Jana and Mohamed)

![Thorup](media/videos/2_thorup.gif)

Instead of pulling out one node at a time, this groups nodes into buckets by distance and clears
a whole bucket, one distance level, in a single step. That bucket idea is what lets it skip the
sorting that Dijkstra spends time on. This is the corrected version: the original code added 1
per hop and ignored the edge weights, so it was really just counting hops. It now adds the real
weight on each edge.

### ALT, goal-directed A* (Ahmad and Layan)

![ALT](media/videos/3_alt.gif)

ALT uses the same queue as Dijkstra but adds a hint: a few landmark nodes give a lower bound on
how far each node still is from T. The search prefers nodes that look closer to T, so the
explored area stretches toward the goal instead of spreading out as a circle. Watch how many
fewer nodes turn colour compared to the Dijkstra clip. The answer is still exact.

### PLL, hub labeling (Ahmad and Layan)

![PLL](media/videos/4_pll.gif)

PLL does its heavy work ahead of time. For every node it stores a small list of hub nodes with
distances. At question time it does not search the graph at all: it lines up S's hubs and T's
hubs, finds the best hub they share, and adds the two stored distances. That is why the clip
jumps straight to the hubs and the path. The trade-off is the one-time cost of building the
labels.

### Bidirectional Dijkstra (Laila, Ahmed Ismail, Zeina and Yehya)

![Bidirectional](media/videos/5_bidirectional.gif)

This runs two searches at the same time, one forward from S (blue) and one backward from T
(orange), and alternates between them. They grow toward each other and stop the moment they
touch. Two small circles cover the gap instead of one big one, so it settles far fewer nodes
than plain Dijkstra while still returning the exact answer, with no preprocessing needed.

### Distance oracle (Laila, Ahmed Ismail, Zeina and Yehya)

![Distance oracle](media/videos/6_distance_oracle.gif)

The oracle keeps a handful of landmarks and answers a question by routing through the best one,
S to landmark to T. It is the fastest to answer and barely cares about graph size, but the
number it gives is an estimate, not always the exact shortest distance.

## the benchmark graphs

We timed every algorithm on every dataset over sub-graphs of growing size. The first figure is
all 30 combinations at once, with the algorithms down the rows and the datasets across the
columns. Each small panel is one algorithm on one dataset, time against number of nodes.

![all 30 plots](media/figures/30_grid_time_vs_size.png)

The next five figures put all six algorithms on one chart, one chart per dataset, so you can
compare how fast each one grows. The number `b` in each legend is the fitted growth rate: near 1
means the time grows in line with the graph, near 2 means it grows much faster. PLL and the
oracle stay almost flat because their answer time hardly depends on size.

![Bitcoin Alpha](media/figures/growth_bitcoin_alpha.png)

![Bitcoin OTC](media/figures/growth_bitcoin_otc.png)

![Advogato](media/figures/growth_advogato.png)

![Epinions](media/figures/growth_epinions.png)

![LiveMocha](media/figures/growth_livemocha.png)

## summary and which algorithm wins

Across all five datasets the six algorithms split into three groups.

**The full-scan methods, Dijkstra and Thorup, are the slowest.** They build the whole
shortest-path tree from the source, so their time climbs steeply as the graph grows (the fitted
growth rate sits near linear-times-log and above). They are simple and always correct, but they
do the most work per question.

**Bidirectional Dijkstra is the best choice when you want one exact answer with no setup.** By
meeting in the middle it settles far fewer nodes than plain Dijkstra, stays low on every chart,
needs no preprocessing, and is still exact. If you only ask a question now and then, this is the
one to use.

**PLL and the distance oracle win once you ask many questions.** Both pay a one-time cost to
build an index and then answer almost instantly, with a flat line on every dataset. The
difference is accuracy: PLL gives the exact distance, the oracle gives a fast estimate.

**So is there a single best? It depends on how often you ask.**

- For repeated questions on the same graph, **PLL is the winner.** It answers in well under a
  millisecond, the time barely grows with the graph, and unlike the oracle the answers are
  exact. The only price is building the labels once up front.
- For a single question with nothing prepared, **Bidirectional Dijkstra is the winner**, since
  it is exact, needs no setup, and still beats the full-scan methods.

If we have to name one overall, it is **PLL**: on these social networks it gave the fastest exact
answers and scaled the flattest, which is exactly what you want when the graph is fixed and the
questions keep coming.
