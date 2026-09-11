# EECE 6399 – Project 1: Search

**Course:** EECE 6399 – AI for Autonomous Systems  
**University:** The University of Texas Rio Grande Valley (UTRGV)  
**Term:** Fall 2026  
**Instructor:** Dr. Lei Cheng  

> **Due Date:** See UTRGV Brightspace for the official deadline.

---

## Project Overview

In this project, you will build general-purpose search algorithms and use them to control Pacman in a maze.

The project begins with basic path finding and gradually moves to more challenging problems in which Pacman must visit multiple locations or collect all food efficiently. By the end of the project, you should understand not only how DFS, BFS, UCS, and A* work, but also how the choice of **state representation**, **cost function**, and **heuristic** changes the behavior of a search agent.

You will work primarily in two files:

- `search.py` — general search algorithms
- `searchAgents.py` — search problems, search agents, and heuristics

The project contains eight graded questions worth **100 points total**.

---

## Learning Objectives

After completing this project, you should be able to:

- Represent a planning problem as a state-space search problem.
- Distinguish between a **state**, a **search node**, and a **path**.
- Implement graph-search versions of DFS, BFS, UCS, and A*.
- Choose an appropriate frontier data structure for each algorithm.
- Use path costs to influence search behavior.
- Design a search state that contains all necessary information without including irrelevant information.
- Explain admissibility and consistency of heuristics.
- Design useful heuristics for larger search problems.
- Recognize the tradeoff between optimality and speed in greedy search strategies.

---

# 1. Getting Started

## 1.1 Run Pacman

Open a terminal in your Project 1 repository and run:

```bash
python pacman.py
```

You can view all available command-line options with:

```bash
python pacman.py -h
```

If Pacman becomes stuck or you need to terminate the program, use:

```text
Ctrl+C
```

---

## 1.2 Verify the Provided Search Agent

Before implementing anything, make sure the provided project environment works.

Run:

```bash
python pacman.py -l tinyMaze -p SearchAgent -a fn=tinyMazeSearch
```

The provided `tinyMazeSearch` function contains a hard-coded solution for `tinyMaze`. Pacman should successfully reach the goal.

This test is important because it confirms that:

- Python is working correctly.
- The Pacman environment can launch.
- `SearchAgent` can call a search function.
- A search function is expected to return a sequence of legal actions.

---

# 2. Understanding the Project Structure

## Files You Will Edit

### `search.py`

This file contains the general search algorithms.

You will implement:

- `depthFirstSearch`
- `breadthFirstSearch`
- `uniformCostSearch`
- `aStarSearch`

These functions should be written generally. They should not contain Pacman-specific logic.

---

### `searchAgents.py`

This file contains search problems and Pacman agents that use the algorithms from `search.py`.

You will work with:

- `CornersProblem`
- `cornersHeuristic`
- `foodHeuristic`
- `findPathToClosestDot`
- `AnyFoodSearchProblem`

---

## Important Supporting Files

| File | Purpose |
|---|---|
| `pacman.py` | Runs the Pacman environment and defines the main game state |
| `game.py` | Contains the game logic and supporting classes |
| `util.py` | Provides `Stack`, `Queue`, `PriorityQueue`, and other utilities |
| `layout.py` | Reads and stores maze layouts |
| `autograder.py` | Runs the local tests |
| `searchTestClasses.py` | Search-specific testing utilities |
| `testClasses.py` | General testing utilities |
| `test_cases/` | Local test cases |

You should normally **not modify these supporting files**.

---

# 3. Important Search Concepts Before You Begin

Understanding the following ideas will make the implementation much easier.

## 3.1 State vs. Search Node

A **state** represents a situation in the search problem.

For a simple maze problem, a state might be only Pacman's current position:

```text
(x, y)
```

A **search node** usually needs more information than the state itself. For example, while searching you may need to remember:

- the current state;
- the sequence of actions used to reach it;
- the accumulated path cost.

Your search algorithm ultimately needs to return the **action sequence**, not the states that were explored.

For example:

```python
[Directions.NORTH, Directions.NORTH, Directions.EAST]
```

---

## 3.2 Tree Search vs. Graph Search

For this project, you should implement **graph search**.

A maze can contain cycles. Without remembering previously explored states, the algorithm may repeatedly visit the same locations and perform unnecessary work.

Your implementations therefore need a way to keep track of states that have already been explored.

Be careful about **when** you mark a state as explored. This detail can affect correctness and efficiency.

---

## 3.3 The Frontier

The main difference among DFS, BFS, UCS, and A* is how they decide which node to explore next.

| Algorithm | Frontier behavior |
|---|---|
| DFS | Last-in, first-out |
| BFS | First-in, first-out |
| UCS | Lowest path cost first |
| A* | Lowest `g(n) + h(n)` first |

Use the data structures provided in `util.py`. Their behavior is expected by the project tests.

---

## 3.4 Type Annotations

Some functions in the project contain Python type annotations such as:

```python
def example(position: tuple[int, int], cost: float = 1.0):
```

These annotations describe the expected types of arguments and return values.

They are useful documentation, but Python generally does not enforce them at runtime. Do not be concerned if type annotation syntax is unfamiliar; focus on the function's inputs, outputs, and docstring.

---

# 4. Local Testing

You should test frequently while working.

Run the entire local autograder:

```bash
python autograder.py --no-graphics
```

Run one question:

```bash
python autograder.py -q q2
```

Run one individual test:

```bash
python autograder.py -t test_cases/q7/food_heuristic_1
```

Force graphics:

```bash
python autograder.py --graphics
```

Disable graphics:

```bash
python autograder.py --no-graphics
```

A good development strategy is to complete and test **one question at a time** rather than writing the entire project before testing.

---

# Question 1 – Depth-First Search

**Points: 12**

## Your Task

Implement:

```python
depthFirstSearch
```

in:

```text
search.py
```

Your function must return a list of legal actions that takes the agent from the start state to a goal state.

Implement the **graph-search** version of DFS so that already explored states are not repeatedly expanded.

---

## How to Think About the Implementation

DFS explores one branch of the search tree deeply before returning to consider alternatives.

The provided `Stack` in `util.py` gives you the required last-in, first-out behavior.

A useful search node should contain enough information to answer two questions:

1. What state am I currently at?
2. What actions did I take to get here?

Depending on your design, you may also store a path cost, although DFS does not use path cost to decide what to expand.

A typical high-level process is:

```text
Create the frontier
Add the start node
Repeat:
    Remove the next node from the frontier
    Check whether it should be explored
    Check whether it is a goal
    Generate its successors
    Add appropriate successors to the frontier
```

Do not copy this directly as code. You still need to decide exactly how to represent nodes and when to update the explored-state set.

---

## Test Your Implementation

```bash
python pacman.py -l tinyMaze -p SearchAgent
python pacman.py -l mediumMaze -p SearchAgent
python pacman.py -l bigMaze -z .5 -p SearchAgent
```

Then run:

```bash
python autograder.py -q q1
```

---

## Sanity Checks

DFS is **not guaranteed to find the shortest path**.

Depending on the order in which successors are placed on the stack, the exact path may differ even when your algorithm is correct.

On `mediumMaze`, a correct implementation using the provided successor order commonly produces a path length around 130. Reversing the order can produce a substantially different DFS path. The important point is that DFS behavior depends on the order in which branches are explored.

When looking at the Pacman display, notice that the algorithm may explore locations that are not part of the final path. Exploration and the final executed path are not the same thing.

---

## Common Mistakes

- Returning states instead of actions.
- Forgetting to keep the path associated with each frontier node.
- Expanding the same state repeatedly.
- Marking only one global "current path" instead of storing an independent path for each frontier node.
- Using Python's own list behavior instead of the provided `Stack` in a way that changes expected behavior.

---

# Question 2 – Breadth-First Search

**Points: 12**

## Your Task

Implement:

```python
breadthFirstSearch
```

in:

```text
search.py
```

Again, implement **graph search**.

Use the provided `Queue` in `util.py`.

---

## How BFS Differs from DFS

DFS and BFS can use almost the same overall search structure.

The major difference is the frontier:

```text
DFS → Stack
BFS → Queue
```

BFS explores shallower paths before deeper ones.

Because every movement step in the basic maze problem has the same cost, BFS should find a path using the fewest actions.

If BFS returns a clearly longer path than expected, inspect your handling of:

- the queue;
- duplicate states;
- explored states;
- when successors are added to the frontier.

---

## Test Your Implementation

```bash
python pacman.py -l mediumMaze -p SearchAgent -a fn=bfs
python pacman.py -l bigMaze -p SearchAgent -a fn=bfs -z .5
```

If you want the animation to run faster:

```bash
python pacman.py -l bigMaze -p SearchAgent -a fn=bfs -z .5 --frameTime 0
```

You can also test whether your search algorithm is truly general:

```bash
python eightpuzzle.py
```

A correctly written generic BFS should work on the eight-puzzle without Pacman-specific changes.

Run:

```bash
python autograder.py -q q2
```

---

## Common Mistakes

- Accidentally using stack behavior.
- Treating BFS as Pacman-specific instead of using the methods of the supplied search problem.
- Returning the first successor generated instead of the path to a goal.
- Allowing many duplicate copies of the same state into the frontier.

---

# Question 3 – Uniform-Cost Search

**Points: 12**

## Your Task

Implement:

```python
uniformCostSearch
```

in:

```text
search.py
```

Use the provided priority queue.

---

## Why UCS Is Needed

BFS finds a path with the fewest actions when all actions have equal cost.

However, the fewest actions are not always the same as the **lowest-cost path**.

Imagine that some locations are expensive or dangerous and other locations are inexpensive or desirable. A rational agent should consider those costs even if that means taking more individual steps.

Uniform-Cost Search chooses the frontier node with the smallest accumulated path cost:

```text
g(n)
```

where `g(n)` is the cost of the complete path from the start state to node `n`.

---

## What You Need to Track

For UCS, your frontier entries need enough information to recover:

- the current state;
- the action sequence;
- the accumulated path cost.

When you generate a successor, its total path cost depends on:

```text
cost to reach the current node
+
cost of the new step
```

Do not prioritize a node using only the cost of its most recent action.

---

## Test Your Implementation

```bash
python pacman.py -l mediumMaze -p SearchAgent -a fn=ucs
python pacman.py -l mediumDottedMaze -p StayEastSearchAgent
python pacman.py -l mediumScaryMaze -p StayWestSearchAgent
```

These agents use different cost functions. Therefore, they may choose very different paths through similar mazes.

That is intentional.

Run:

```bash
python autograder.py -q q3
```

---

## What You Should Observe

A search algorithm does not decide what "good" means by itself.

The **cost function** defines what should be preferred.

Changing the cost function can cause the same UCS algorithm to choose a very different path.

---

## Common Mistakes

- Prioritizing only by the latest step cost.
- Treating UCS as BFS with a different container but not tracking costs.
- Ignoring the possibility that the same state can be reached through paths with different costs.
- Using a heuristic in UCS. UCS uses only `g(n)`.

---

# Question 4 – A* Search

**Points: 12**

## Your Task

Implement:

```python
aStarSearch
```

in:

```text
search.py
```

A* uses both the cost already incurred and an estimate of the cost still remaining.

The priority is:

```text
f(n) = g(n) + h(n)
```

where:

- `g(n)` = cost from the start to the current node;
- `h(n)` = heuristic estimate from the current node to a goal.

---

## Understanding the Heuristic Argument

Your A* function receives a heuristic function.

Conceptually, a heuristic is called using information similar to:

```python
heuristic(state, problem)
```

The current state is the main input. The search problem is also available so the heuristic can access information about the problem if needed.

A `nullHeuristic` is already provided. It returns no useful estimate, so A* with a null heuristic behaves like UCS.

---

## Test Your Implementation

Use the provided Manhattan-distance heuristic:

```bash
python pacman.py -l bigMaze -z .5 -p SearchAgent -a fn=astar,heuristic=manhattanHeuristic
```

Then:

```bash
python autograder.py -q q4
```

---

## Sanity Check

A correct A* implementation with a useful heuristic should usually expand fewer nodes than UCS while still returning an optimal path when the heuristic is appropriate.

Exact expansion counts can differ slightly because equal-priority nodes may be handled differently.

If A* and UCS return different optimal path costs when using a heuristic that should be consistent, carefully check:

- how you calculate `g(n)`;
- how you calculate the priority;
- whether your graph-search handling is correct.

---

## Common Mistakes

- Using only `h(n)` as the priority. That would be greedy best-first search, not A*.
- Forgetting the accumulated path cost.
- Calling the heuristic on the wrong state.
- Hard-coding one particular heuristic into the generic A* function.

---

# Question 5 – Finding All Corners

**Points: 12**

## Your Task

Implement:

```python
CornersProblem
```

in:

```text
searchAgents.py
```

The goal is now more complicated.

Pacman must find a shortest path that visits **all four corners** of the maze.

Reaching one corner is not enough.

---

## The Main Challenge: State Representation

For earlier questions, Pacman's position was enough to describe a search state.

That is no longer true.

Consider two search nodes in which Pacman is standing on the same square:

```text
Node A: Pacman has already visited three corners
Node B: Pacman has visited no corners
```

The positions are identical, but the situations are clearly not equivalent.

Therefore, the state must include both:

- Pacman's current position;
- information about which corners have already been visited.

Your state representation should contain **all information needed to determine future behavior**, but it should not include unrelated information.

Do not use the complete Pacman `GameState` as the search state.

Information such as ghost locations or unrelated game details is not needed for this problem and would create an unnecessarily large search space.

---

## Important Design Requirement

A `CornersProblem` object describes the overall search problem.

It is **not itself one changing search state**.

During BFS, many different states will exist simultaneously in the frontier. Each state must independently represent its own position and visited-corner information.

This distinction is very important.

---

## Functions to Think Through Carefully

When implementing the problem, make sure you understand what each search-problem method should do:

### Start state

The start state must represent:

- the starting Pacman position;
- which corners, if any, should already count as visited.

### Goal test

A state is a goal only when all four corners have been visited.

### Successors

For each legal movement:

- determine the next position;
- update the visited-corner information appropriately;
- create a new successor state;
- use a step cost of `1`.

Do not accidentally modify a single shared visited-corners object for every search branch.

---

## Test Your Implementation

```bash
python pacman.py -l tinyCorners -p SearchAgent -a fn=bfs,prob=CornersProblem
python pacman.py -l mediumCorners -p SearchAgent -a fn=bfs,prob=CornersProblem
```

Run:

```bash
python autograder.py -q q5
```

---

## Sanity Checks

For `tinyCorners`, the shortest solution is **28 steps**.

A reasonable BFS implementation on `mediumCorners` should expand roughly a few thousand states rather than an enormous number.

If your program is extremely slow, check whether your state contains unnecessary information.

---

## Common Mistakes

- Representing a state using only Pacman's current position.
- Storing visited corners only as one mutable variable inside the overall problem object.
- Mutating a list or set that is shared among multiple successor states.
- Including the entire `GameState` in the search state.
- Forgetting to mark a corner when Pacman enters it.
- Forgetting that the starting position itself may need to be considered.

---

# Question 6 – Corners Heuristic

**Points: 12**

## Your Task

Implement:

```python
cornersHeuristic
```

in:

```text
searchAgents.py
```

You will use this heuristic with A* to solve `CornersProblem` more efficiently.

Complete Question 4 before working on this question.

---

## What the Heuristic Should Estimate

At any state, Pacman may still need to visit several corners.

Your heuristic should estimate a **lower bound** on the remaining cost required to visit all unvisited corners.

A useful heuristic should be:

- inexpensive to compute;
- non-negative;
- `0` at a goal state;
- admissible;
- consistent;
- more informative than always returning `0`.

---

## Admissibility

A heuristic is admissible when it never overestimates the true remaining optimal cost.

In other words:

```text
h(state) <= actual minimum remaining cost
```

You are estimating how much work remains, but your estimate must never claim that the remaining problem is harder than it actually is.

---

## Consistency

For a state `n`, a successor `n'`, and step cost `c`:

```text
h(n) <= c + h(n')
```

Another useful way to think about consistency is that the A* `f` value should not decrease as you move along a path.

Consistency is especially important for A* graph search.

---

## How to Design a Heuristic

A useful approach is to start from a **relaxed version of the problem**.

Ask yourself:

- What obstacles or requirements could I temporarily ignore?
- Can I compute something that is definitely no larger than the real remaining path?
- Can I combine information about multiple remaining corners without overestimating?

Do not attempt to solve the entire remaining search problem inside the heuristic. If the heuristic itself performs an expensive full search, A* loses its efficiency advantage.

---

## Test Your Heuristic

```bash
python pacman.py -l mediumCorners -p AStarCornersAgent -z 0.5
```

Run:

```bash
python autograder.py -q q6
```

---

## Performance Guide

For the standard `mediumCorners` evaluation, the following node counts correspond to the course score for this question:

| Nodes Expanded | Points |
|---:|---:|
| More than 2000 | 0 / 12 |
| At most 2000 | 4 / 12 |
| At most 1600 | 8 / 12 |
| At most 1200 | 12 / 12 |

A heuristic that violates the required correctness properties may receive no credit even if it expands very few nodes.

---

## Debugging Consistency

If you suspect your heuristic is inconsistent:

1. Compare the solution cost returned by UCS and A*.
2. Examine `f = g + h` values along expanded paths.
3. Check whether moving one step causes the heuristic to drop by more than the step cost.
4. Test small states where the true remaining cost is easy to reason about.

---

# Question 7 – Eating All the Food

**Points: 16**

## Your Task

Implement:

```python
foodHeuristic
```

in:

```text
searchAgents.py
```

The `FoodSearchProblem` is already implemented for you.

A state in this problem contains:

- Pacman's current position;
- the food that remains.

The goal is to collect **all food** using as few steps as possible.

Ghosts and power pellets are not part of this search problem.

Complete Question 4 before working on this question.

---

## Why This Problem Is Hard

The number of possible states grows rapidly because two states with Pacman at the same position can still be different if different food dots remain.

A simple uninformed search can therefore expand a very large number of states.

This question is designed to show why a good heuristic matters.

---

## First Check Your A* Implementation

Before designing `foodHeuristic`, verify that your A* implementation works with the provided problem:

```bash
python pacman.py -l testSearch -p AStarFoodSearchAgent
```

For this small test, the optimal solution cost should be **7**.

If this basic case fails, fix A* before spending time on the food heuristic.

---

## Designing the Food Heuristic

Your heuristic must estimate a lower bound on the cost required to collect all remaining food.

Think carefully about what must necessarily happen before all remaining food can be collected.

A good heuristic should use information from the state but remain much cheaper than solving the entire remaining problem exactly.

You may use the `heuristicInfo` dictionary associated with the problem to cache reusable calculations if doing so helps avoid repeated expensive work.

When evaluating an idea, ask:

- Can this estimate ever be larger than the true optimal remaining path?
- Does the estimate become `0` when there is no food left?
- Can moving one step cause the estimate to decrease too much?
- Is computing the heuristic itself fast enough to be useful?

---

## Test Your Heuristic

```bash
python pacman.py -l trickySearch -p AStarFoodSearchAgent
```

Then:

```bash
python autograder.py -q q7
```

---

## Performance Guide

For the standard evaluation:

| Nodes Expanded | Points |
|---:|---:|
| More than 15000 | 4 / 16 |
| At most 15000 | 8 / 16 |
| At most 12000 | 12 / 16 |
| At most 9000 | 16 / 16 |

A very strong heuristic may expand fewer than 7000 nodes, but the official EECE 6399 score for this question remains capped at **16 points**.

The heuristic must still satisfy the required correctness conditions. A fast but invalid heuristic is not a correct solution.

---

## Common Mistakes

- Returning the number of remaining food dots without considering distance.
- Accidentally overestimating the remaining cost.
- Computing a heuristic so expensive that A* becomes slower than a simpler method.
- Forgetting that the food configuration is part of the state.
- Returning a nonzero value when all food has already been collected.

---

# Question 8 – Finding the Closest Food

**Points: 12**

## Your Task

Implement:

```python
findPathToClosestDot
```

in:

```text
searchAgents.py
```

You will use the provided `ClosestDotSearchAgent`.

Instead of planning one globally optimal route through every remaining food dot, this agent repeatedly does the following:

```text
Find the closest remaining food
→ Move to it
→ Recompute from the new position
→ Repeat
```

This strategy is fast, but it is greedy.

---

## The Key Subproblem

The project includes:

```python
AnyFoodSearchProblem
```

This problem should consider **any location containing food** to be a goal.

The main missing piece is its goal test.

Once `AnyFoodSearchProblem` correctly recognizes food locations as goals, `findPathToClosestDot` can solve that search problem using an appropriate search algorithm.

Because ordinary movement steps have equal cost, think about which algorithm guarantees the shortest path to the nearest goal.

---

## Test Your Implementation

```bash
python pacman.py -l bigSearch -p ClosestDotSearchAgent -z .5
```

Then:

```bash
python autograder.py -q q8
```

A correct implementation should solve `bigSearch` quickly. A typical path cost is around **350**, although this is not the globally optimal route for collecting all food.

---

## Why This Method Is Suboptimal

Choosing the closest food right now does not consider how that choice affects future food collection.

A locally best decision can create a poor global route.

You should be able to explain the difference between:

```text
shortest path to the next food
```

and:

```text
shortest path that collects all food
```

These are different optimization problems.

---

# 5. Grading

Project 1 is graded on a **100-point scale**.

| Question | Topic | Points |
|---|---|---:|
| Q1 | Depth-First Search | 12 |
| Q2 | Breadth-First Search | 12 |
| Q3 | Uniform-Cost Search | 12 |
| Q4 | A* Search | 12 |
| Q5 | Corners Problem | 12 |
| Q6 | Corners Heuristic | 12 |
| Q7 | Food Heuristic | 16 |
| Q8 | Closest-Food Search | 12 |
| **Total** |  | **100** |

The local testing framework may display scores using its own internal question scale. The official course grade is reported using the 100-point scale above.

Technical correctness is the primary basis for grading. If necessary, submitted code may also be reviewed manually.

---

# 6. Submission

Your work must be maintained in your assigned GitHub repository.

You are strongly encouraged to commit and push regularly rather than waiting until the deadline.

Check your current changes:

```bash
git status
```

Add the two files you are expected to edit:

```bash
git add search.py searchAgents.py
```

Commit:

```bash
git commit -m "Update Project 1"
```

Push:

```bash
git push
```

The required implementation files are:

- `search.py`
- `searchAgents.py`

The official deadline is listed in **UTRGV Brightspace**.

Follow any additional submission instructions posted there.

---

# 7. Repository and Grading Integrity

The repository contains local testing files so that you can debug your work before submission.

You may use these files for testing, but modifying grading-related files is not a valid way to complete the assignment.

Do not modify files such as:

- `autograder.py`
- `grading.py`
- `testClasses.py`
- `searchTestClasses.py`
- `test_cases/`

The official course grading environment is maintained separately from the files in your repository.

Your score is based on whether your implementation correctly solves the assigned problems.

---

# 8. Academic Integrity

All submitted work must comply with:

- the EECE 6399 syllabus;
- UTRGV academic integrity requirements;
- course policies on collaboration;
- course policies on the use of generative AI tools.

Unless explicitly authorized, the implementation you submit must represent your own work.

Do not:

- copy another student's implementation;
- share completed solutions;
- submit code from previous students;
- submit a publicly available completed solution as your own;
- attempt to manipulate or bypass the grading environment.

When discussing the assignment with others, focus on concepts and debugging strategies rather than exchanging complete solutions.

---

# 9. Recommended Workflow

For each question, use the following process:

```text
Read the question carefully
        ↓
Identify the required function or class
        ↓
Read its existing docstring and surrounding code
        ↓
Determine what the state and search node must contain
        ↓
Implement one small part
        ↓
Test on the provided Pacman layout
        ↓
Run the question-specific autograder
        ↓
Debug
        ↓
Commit
        ↓
Push
```

For example:

```bash
python autograder.py -q q1
git add search.py
git commit -m "Complete DFS"
git push
```

Do not wait until the entire project is finished before committing your work.

---

# 10. Debugging Guide

If a question is failing, check these issues before rewriting everything.

## Search algorithm returns no solution

Check:

- Did you add the start state to the frontier?
- Is your goal test being called correctly?
- Are you generating successors?
- Are you accidentally treating every state as already explored?

## Pacman reaches the goal but the autograder fails

Check:

- Are you returning a list of **actions**?
- Are the actions legal?
- Are you using the provided frontier data structures?
- Are you implementing graph search rather than a maze-specific shortcut?

## Search is extremely slow

Check:

- Are duplicate states being expanded repeatedly?
- Is your state representation much larger than necessary?
- Is your heuristic expensive to compute?
- Are you accidentally storing an entire game state when only a small amount of information is required?

## A* returns the wrong path cost

Check:

- Is priority computed as `g + h`?
- Does `g` contain the full path cost?
- Is the heuristic being called on the successor state?
- Is your heuristic admissible and consistent?

## CornersProblem behaves strangely

Check:

- Does each search state have its own visited-corner information?
- Are you mutating shared state across multiple branches?
- Does entering a corner update the state correctly?
- Does the goal test require all four corners?

## Heuristic gets a low score

First make sure it is **correct**. A fast but inconsistent heuristic can invalidate A*.

After correctness is established, improve how closely the heuristic lower-bounds the true remaining cost while keeping it inexpensive to compute.

---

# 11. Final Checklist

Before submitting, verify that:

- [ ] `depthFirstSearch` is implemented.
- [ ] `breadthFirstSearch` is implemented.
- [ ] `uniformCostSearch` is implemented.
- [ ] `aStarSearch` is implemented.
- [ ] `CornersProblem` is implemented.
- [ ] `cornersHeuristic` is implemented.
- [ ] `foodHeuristic` is implemented.
- [ ] `findPathToClosestDot` is implemented.
- [ ] `AnyFoodSearchProblem` correctly recognizes food locations as goals.
- [ ] `python autograder.py --no-graphics` completes.
- [ ] You reviewed any failed tests rather than ignoring them.
- [ ] Your latest changes are committed.
- [ ] Your latest commit is pushed to your assigned repository.
- [ ] You checked Brightspace for the official deadline and submission instructions.

---

## Project Framework Notice

This repository uses an educational Pacman search framework adapted for **UTRGV EECE 6399 – AI for Autonomous Systems**. Course-facing instructions, grading, and submission procedures in this repository are specific to EECE 6399.

Please retain any copyright or license notices already included in the distributed source files.
