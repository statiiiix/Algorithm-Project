# Visual results — videos & graphs

This folder holds the rendered output of `finalsub5.ipynb`: an animated demonstration of how
each algorithm finds a shortest path, and the benchmark graphs comparing all six algorithms on
all five datasets. The videos are GIFs (they play automatically on GitHub).

Algorithms and authors:

| Algorithm | Authors |
|---|---|
| Dijkstra (binary heap) | Aya & Mariam |
| ALT (A* + landmarks) and PLL (pruned landmark labeling) | Ahmad & Layan |
| Bidirectional Dijkstra and Distance Oracle | Laila, Ahmed Ismail, Zeina & Yehya |
| Thorup / weighted SSSP (corrected) | Jana & Mohamed |

In every video: **S** (green) = source, **T** (gold) = target, grey = unvisited, the algorithm's
own colour = settled/explored, yellow = the live frontier (the priority queue), red = the final
shortest path. Each video traces the algorithm's *actual* execution, so the mechanisms look
genuinely different from one another.

---

## Videos — how each algorithm finds the path

### 1. Dijkstra (Aya & Mariam)
![Dijkstra](videos/1_dijkstra.gif)

Pops the **closest** unsettled node each step and pushes its neighbours into the queue. The
yellow frontier expands **uniformly in every direction**, so Dijkstra settles *many* nodes
before it reaches T.

### 2. Thorup / weighted SSSP — bucket processing (Jana & Mohamed, corrected)
![Thorup](videos/2_thorup.gif)

Instead of a single creeping frontier, vertices are grouped into **distance buckets** and a
whole bucket (an entire distance level) is emptied at once — the mechanism that lets Thorup
avoid the priority-queue sorting bottleneck. (This is the corrected, weight-respecting version;
the original submission ignored edge weights.)

### 3. ALT — goal-directed A* (Ahmad & Layan)
![ALT](videos/3_alt.gif)

The same priority queue as Dijkstra, but ordered by `f = g + h` where `h` is a **landmark
lower-bound** on the distance to T. The search is pulled toward T and settles **far fewer**
nodes — compare its small, directed footprint with Dijkstra's broad spread.

### 4. PLL — hub labeling (Ahmad & Layan)
![PLL](videos/4_pll.gif)

No graph search at query time. It highlights the **hubs** stored in S's label and T's label,
finds the best **shared hub**, and reads the distance straight off — answering in near-constant
time.

### 5. Bidirectional Dijkstra — two frontiers meeting (Laila, Ahmed Ismail, Zeina & Yehya)
![Bidirectional](videos/5_bidirectional.gif)

Runs **two** searches at once — forward from S (blue) and backward from T (orange) — that grow
toward each other and **stop the moment they collide**. Two small balls cover the gap instead of
one big one, so fewer nodes are settled.

### 6. Distance Oracle — landmark routing (Laila, Ahmed Ismail, Zeina & Yehya)
![Distance Oracle](videos/6_distance_oracle.gif)

Picks a few **landmarks**, then estimates the S→T distance by routing through the best one — an
approximate answer with almost no per-query work.

---

## Graphs — the benchmark (35 plots)

### The 30 plots: execution time vs graph size, every algorithm × dataset
![30-plot grid](figures/30_grid_time_vs_size.png)

A 6×5 grid — **rows = algorithms, columns = datasets**. Each of the 30 panels shows one
algorithm's query time as the graph grows. Reading down a column compares the six algorithms on
the same dataset; reading across a row shows how one algorithm behaves across datasets.

### The 5 plots: per-dataset growth-rate comparison (all six algorithms)

One plot per dataset, overlaying all six growth curves on a log scale. The legend lists each
algorithm's fitted growth exponent `b` from `t ≈ a·V^b` (`b≈1` linear, `b≈1.1` linearithmic,
`b≈2` quadratic; PLL and the Oracle are near-flat because their query time barely depends on
graph size).

| Dataset | Plot |
|---|---|
| Bitcoin Alpha | ![Bitcoin Alpha](figures/growth_bitcoin_alpha.png) |
| Bitcoin OTC | ![Bitcoin OTC](figures/growth_bitcoin_otc.png) |
| Advogato | ![Advogato](figures/growth_advogato.png) |
| Epinions | ![Epinions](figures/growth_epinions.png) |
| LiveMocha | ![LiveMocha](figures/growth_livemocha.png) |

**What the curves show:** Dijkstra and Thorup compute a full single-source tree, so they climb
steepest (`b` around linearithmic and above). Bidirectional settles fewer nodes. ALT is
goal-directed but, on these small-world social graphs, the heuristic only helps modestly. PLL and
the Distance Oracle sit flat at the bottom — near-constant query time after a one-time
preprocessing step (PLL exact, Oracle approximate).
