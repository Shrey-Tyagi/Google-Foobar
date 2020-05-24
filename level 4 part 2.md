Escape Pods
===========

You've blown up the LAMBCHOP doomsday device and broken the bunnies out of Lambda's prison - and now you need to escape from the space station as quickly and as orderly as possible! The bunnies have all gathered in various locations throughout the station, and need to make their way towards the seemingly endless amount of escape pods positioned in other parts of the station. You need to get the numerous bunnies through the various rooms to the escape pods. Unfortunately, the corridors between the rooms can only fit so many bunnies at a time. What's more, many of the corridors were resized to accommodate the LAMBCHOP, so they vary in how many bunnies can move through them at a time. 

Given the starting room numbers of the groups of bunnies, the room numbers of the escape pods, and how many bunnies can fit through at a time in each direction of every corridor in between, figure out how many bunnies can safely make it to the escape pods at a time at peak.

Write a function solution(entrances, exits, path) that takes an array of integers denoting where the groups of gathered bunnies are, an array of integers denoting where the escape pods are located, and an array of an array of integers of the corridors, returning the total number of bunnies that can get through at each time step as an int. The entrances and exits are disjoint and thus will never overlap. The path element path[A][B] = C describes that the corridor going from A to B can fit C bunnies at each time step.  There are at most 50 rooms connected by the corridors and at most 2000000 bunnies that will fit at a time.

For example, if you have:
entrances = [0, 1]
exits = [4, 5]
path = [
  [0, 0, 4, 6, 0, 0],  # Room 0: Bunnies
  [0, 0, 5, 2, 0, 0],  # Room 1: Bunnies
  [0, 0, 0, 0, 4, 4],  # Room 2: Intermediate room
  [0, 0, 0, 0, 6, 6],  # Room 3: Intermediate room
  [0, 0, 0, 0, 0, 0],  # Room 4: Escape pods
  [0, 0, 0, 0, 0, 0],  # Room 5: Escape pods
]

Then in each time step, the following might happen:
0 sends 4/4 bunnies to 2 and 6/6 bunnies to 3
1 sends 4/5 bunnies to 2 and 2/2 bunnies to 3
2 sends 4/4 bunnies to 4 and 4/4 bunnies to 5
3 sends 4/6 bunnies to 4 and 4/6 bunnies to 5

So, in total, 16 bunnies could make it to the escape pods at 4 and 5 at each time step.  (Note that in this example, room 3 could have sent any variation of 8 bunnies to 4 and 5, such as 2/6 and 6/6, but the final solution remains the same.)

Languages
=========

To provide a Java solution, edit Solution.java
To provide a Python solution, edit solution.py

Test cases
==========
Your code should pass the following test cases.
Note that it may also be run against hidden test cases not shown here.

```
-- Java cases --
Input:
Solution.solution({0, 1}, {4, 5}, {{0, 0, 4, 6, 0, 0}, {0, 0, 5, 2, 0, 0}, {0, 0, 0, 0, 4, 4}, {0, 0, 0, 0, 6, 6}, {0, 0, 0, 0, 0, 0}, {0, 0, 0, 0, 0, 0}})
Output:
    16

Input:
Solution.solution({0}, {3}, {{0, 7, 0, 0}, {0, 0, 6, 0}, {0, 0, 0, 8}, {9, 0, 0, 0}})
Output:
    6

-- Python cases --
Input:
solution.solution([0], [3], [[0, 7, 0, 0], [0, 0, 6, 0], [0, 0, 0, 8], [9, 0, 0, 0]])
Output:
    6

Input:
solution.solution([0, 1], [4, 5], [[0, 0, 4, 6, 0, 0], [0, 0, 5, 2, 0, 0], [0, 0, 0, 0, 4, 4], [0, 0, 0, 0, 6, 6], [0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0]])
Output:
    16
```

```python
import decimal
inf = decimal.Decimal("Infinity")

def solution(entrances, exits, path):
    transform(entrances, exits, path)
    flow = 0
    length = len(path)
    flows = [[0 for i in range(length)] for j in range(length)]
    start = 0
    end = length - 1
    while True:
        ap_flow, parents = bfs(start, end, path, flows)
        if ap_flow == 0:
            break
        flow += ap_flow
        v = end
        while v != start:
            u = parents[v]
            flows[u][v] += ap_flow
            flows[v][u] -= ap_flow
            v = u
    return flow

def bfs(start, end, path, flows):
    global inf
    length = len(path)
    parents = [-1] * length     # parent table for reverse tracing path
    parents[start] = -2         # differentiate start point from others

    queue = []
    queue.append(start)
    while queue and parents[end] == -1:
        u = queue.pop(0)
        for v in range(length):
            cf = path[u][v] - flows[u][v]
            if cf > 0 and parents[v] == -1:
                queue.append(v)
                parents[v] = u

    if parents[end] == -1:      # if t can not been reached
        return 0, parents

    v = end
    delta = inf
    while v != start:
        u = parents[v]
        cf = path[u][v] - flows[u][v]
        delta = min(delta, cf)
        v = u

    return delta, parents

def transform(entrances, exits, path):
    # transform multi-source, multi-sink flow network to single-source,
    # single-sink flow network
    global inf
    length = len(path)
    entrances = [i + 1 for i in entrances]
    exits = [i + 1 for i in exits]
    for row in path:
        row.insert(0, 0)
        row.append(0)
    start_row = [0] * (length + 2)
    for i in entrances:
        start_row[i] = inf
    path.insert(0, start_row)
    end_row = [0] * (length + 2)
    for i in exits:
        path[i][-1] = inf
    path.append(end_row)
    return entrances, exits, path
```