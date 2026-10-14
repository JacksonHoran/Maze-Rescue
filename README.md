# COMP 371 Project 1: Maze Rescue (Imperative Paradigm)

Loyola University Chicago — Department of Computer Science  
**Course:** COMP 371 / COMP 471: Programming Languages (F26)
**Instructor:** Konstantin Läufer  
**Grade:** Points (max 6.0)

---

## Story & Overview

A rescue robot has boarded a damaged space station. The interior structure is unstable, and rooms are unlit or collapsed. The robot must navigate through the corridors, collect crucial energy cells to recharge its failing battery, use an emergency flood-fill sonar scanner to map out connected chambers, and reach the escape pod before its power cell is completely drained!

```
###########
#R..#....E#
#.#.#.##..#
#.#...C#..#
#.#####.#.#
#...C...#.#
###########
```

### Map Legend
* `R`: Robot starting position (mutated to `.` once the game initializes)
* `@`: Robot's current live location on the rendered display
* `E`: Escape pod exit
* `C`: Energy cell (+3 energy bonus when collected)
* `#`: Impassable wall
* `.`: Walkable open corridor / floor

---

## Learning Objectives

In this project, you will:
1. **Transition from Java to Scala 3**: Practice writing idiomatic Scala 3 syntax using significant indentation without unnecessary curly braces `{}`.
2. **Master Imperative & Procedural Programming**: Model dynamic state using built-in Scala mutable data structures (`Array[Array[Char]]`, `Array[Int]`, scalar `var`s, tuples, and `Option`) without domain-model classes, traits, or ADTs.
3. **Analyze Parameter Passing & Aliasing**: Deepen your understanding of call-by-value evaluation, reference copying, aliasing, and why parameter variables cannot be rebound to alter the caller's reference.
4. **Contrast Evaluation Strategies**: Explore Scala's call-by-name evaluation (`=> Boolean`) and see how it differs from call-by-value.
5. **Implement Recursion & Trace Activation Records**: Implement a multi-way recursive flood fill (`revealReachable`), reason about base cases and termination guarantees, and draw call stack frames.
6. **Apply Test-Driven Development (TDD)**: Work with JUnit 5 (Jupiter), interpret compiler contracts under strict typing (`-language:strictEquality`, `-Yexplicit-nulls`, `-Werror`), and write targeted unit tests for boundary conditions.
7. **Reflect on Architectural Paradigms**: Experience firsthand why a purely procedural approach with loosely associated arrays and scalar variables becomes unwieldy as software scales—setting the stage for Object-Oriented design and Algebraic Data Types in Project 2.

---

## Project Structure

```
.
├── build.sbt                               # Build definitions and compiler options
├── src
│   ├── main
│   │   ├── resources
│   │   │   └── mazes                       # External ASCII maze definition files
│   │   │       ├── default.txt             # Standard 11×7 maze
│   │   │       ├── tiny.txt                # Minimal 7×5 maze
│   │   │       ├── corridor.txt            # Long corridor 15×5 maze
│   │   │       ├── chambers.txt            # Multi-chamber 13×7 maze
│   │   │       ├── arena.txt               # Open arena 16×5 maze
│   │   │       └── labyrinth.txt           # Complex 17×8 maze
│   │   └── scala
│   │       └── mazesolver
│   │           └── MazeSolver.scala        # Procedures & interactive CLI entry point
│   └── test
│       └── scala
│           └── mazesolver
│               └── MazeSolverTest.scala    # JUnit 5 Jupiter test suite
└── README.md                               # Assignment specification & written report
```

---

## Functional Requirements

The assignment is staged in four sequential parts:

### Part 1: State & Procedural Abstraction

In `src/main/scala/mazesolver/MazeSolver.scala`, implement the core imperative procedures:

1. **`isWalkable(maze: Array[Array[Char]], row: Int, col: Int): Boolean`**
   * Return `true` if `(row, col)` is within the grid boundaries and the character at that position is not a wall (`'#'`).
2. **`directionDelta(direction: Char): Option[(Int, Int)]`**
   * Map movement commands (`'U'`, `'D'`, `'L'`, `'R'`) to coordinate offset deltas:
     * `'U'` $\rightarrow$ `(-1, 0)`
     * `'D'` $\rightarrow$ `(1, 0)`
     * `'L'` $\rightarrow$ `(0, -1)`
     * `'R'` $\rightarrow$ `(0, 1)`
     * Any other character $\rightarrow$ `None`
3. **`moveRobot(maze: Array[Array[Char]], robot: Array[Int], direction: Char): Boolean`**
   * Check if the destination cell in the specified direction is walkable.
   * If walkable, mutate the `robot` array in place (`robot(0) = newRow`, `robot(1) = newCol`) and return `true`.
   * If blocked or invalid direction, leave `robot` unchanged and return `false`.
4. **`collectCell(maze: Array[Array[Char]], robot: Array[Int]): Int`**
   * If the robot is currently on an energy cell (`'C'`), overwrite the grid cell with floor (`'.'`) and return `1`.
   * Otherwise, return `0`. (An energy cell can only be collected once!)
5. **`isAtExit(maze: Array[Array[Char]], robot: Array[Int]): Boolean`**
   * Return `true` if the robot's current position matches the exit marker (`'E'`).
6. **`render(maze: Array[Array[Char]], robot: Array[Int], energy: Int, cellsCollected: Int): String`**
   * Format the 2-D maze grid into a multi-line string with the robot rendered as `'@'` at its live position, followed by a status line:  
     `s"Energy: $energy  Cells: $cellsCollected"`
7. **`playMoves(maze: Array[Array[Char]], robot: Array[Int], moves: String, startEnergy: Int): Int`**
   * Execute moves sequentially from `moves`.
   * Each successful move costs 1 energy; collecting a cell awards +3 energy. Blocked moves cost 0 energy.
   * Stop immediately if energy reaches 0 (stranded) or if the robot reaches the exit (`'E'`). Return remaining energy.
8. **`playGame(maze: Array[Array[Char]], robot: Array[Int], moves: String, startEnergy: Int): (Int, Int, Boolean)`**
   * Execute moves tracking `(remainingEnergy, cellsCollected, reachedExit)`.

### Part 2: Parameter Passing & Aliasing Lab

Complete and study the parameter-passing procedures:

1. **`moveNorth(position: Array[Int]): Unit`**
   * Decrement `position(0)` by 1. Because Scala uses call-by-value with reference copying for arrays, the caller's array is mutated in place.
2. **`localReset(position: Array[Int]): Unit`**
   * Rebind a local `val replacement = Array(1, 1)`. Explain why assigning to a local variable does not alter which array the caller's variable references.
3. **`repeatUntilStopped(action: => Boolean): Int`**
   * Execute the call-by-name parameter `action` in a while-loop as long as it evaluates to `true`.
   * Return the total count of successful attempts. Explain why call-by-name evaluates `action` anew on every iteration.

### Part 3: Recursive Emergency Scanner

Implement the emergency flood-fill scanner:

1. **`revealReachable(maze: Array[Array[Char]], revealed: Array[Array[Boolean]], row: Int, col: Int): Int`**
   * Base cases:
     * `(row, col)` is out of maze bounds $\rightarrow$ return `0`
     * `maze(row)(col) == '#'` (wall) $\rightarrow$ return `0`
     * `revealed(row)(col) == true` (already visited) $\rightarrow$ return `0`
   * Recursive step:
     * Mark `revealed(row)(col) = true`
     * Recursively explore the 4 cardinal neighbors: up, down, left, right.
     * Return `1 + sum of newly revealed neighbors`.
   * **Termination condition:** Explain why setting `revealed(row)(col) = true` *before* the recursive calls guarantees termination even in mazes with loops.

### Part 4: Written Reasoning & Reflections

Complete all sections in the **Written Report Deliverables** section at the bottom of this `README.md`.

1. **State-Transition Table**: Trace 3 consecutive moves on the small test maze (`R`, `D`, `D`).
2. **Parameter-Passing Memory Diagram**: Visual diagram showing stack frames and heap arrays for `moveNorth` and `localReset`.
3. **Scanner Call-Stack Trace**: Detailed activation record trace for the 4×4 maze starting at `(1, 1)`.
4. **Recursion vs. Iteration Analysis**: Space complexity comparison (call stack vs heap stack).
5. **Procedural vs. OO Reflection**: Analysis of why loosely bundled arrays and procedures become awkward and how ADTs / OO will solve this.

---

## Nonfunctional Requirements

* **Strictly Imperative / Procedural**: Use only built-in types (`Array`, `String`, `Int`, `Boolean`, tuples, and `Option`). **Do NOT create custom classes, case classes, traits, or domain ADTs.**
* **Syntax & Style**: Use Scala 3 significant indentation (2 spaces, no unnecessary `{}`). Code must pass `sbt scalafmtCheckAll`.
* **Compiler Flags**: Code must compile cleanly with `-language:strictEquality`, `-Yexplicit-nulls`, and `-Werror` (warnings treated as errors).
* **Test Verification**: All existing tests in `MazeSolverTest.scala` must pass.
* **Student Unit Tests**: Add at least four original test cases to `StudentMazeSolverTest` covering edge cases.

---

## Extra Credit (+0.5 pt)

Implement **`revealReachableIterative`** in `MazeSolver.scala`:
* Use an explicit mutable stack (`scala.collection.mutable.Stack[(Int, Int)]`) on the heap to perform flood fill instead of recursive call stack frames.
* In your written report, contrast heap allocation with JVM stack frame limits, detailing when and why an iterative stack avoids `StackOverflowError`.

---

## Development & Testing Commands

Open a terminal in the project directory:

```bash
# Compile the project
sbt compile

# Run the test suite
sbt test

# Check code formatting (reformat with 'sbt scalafmtAll')
sbt scalafmtCheckAll

# Run test coverage verification
sbt clean coverage test coverageReport

# Stage the application executable
sbt stage

# Run the interactive game with default maze
sbt run

# Run with a specific sample maze
sbt "run labyrinth.txt"
sbt "run chambers.txt"
```

### Interactive REPL Commands
When running `sbt run`:
* `U`: Move up (costs 1 energy)
* `D`: Move down (costs 1 energy)
* `L`: Move left (costs 1 energy)
* `R`: Move right (costs 1 energy)
* `S`: Emergency scanner flood fill
* `H`: Display help message
* `Q`: Quit

---

## Submission Instructions

1. Ensure all your work is committed and pushed to your assigned GitHub Classroom repository.
2. Verify that the GitHub Actions CI workflow runs and passes cleanly.
3. Submit a brief inline note on Sakai with the URL of your repository before the deadline.

---

## Grading Criteria (Total: 6.0 Points)

| Points | Criterion | Description |
|:------:|:----------|:------------|
| **1.0** | **Navigation & Bounds** | Correct implementation of `isWalkable`, `directionDelta`, and in-place `moveRobot`. |
| **0.5** | **Game Mechanics & State** | Correct implementation of `collectCell`, `isAtExit`, `playMoves`, and `playGame`. |
| **0.5** | **Rendering** | Correct implementation of `render` with robot `@` overlay and formatted status line. |
| **0.5** | **Parameter Passing Lab** | Correct implementation of `moveNorth`, `localReset`, and memory aliasing demonstration. |
| **0.5** | **Call-by-Name Evaluation** | Correct implementation of `repeatUntilStopped` and explanation of re-evaluation. |
| **1.0** | **Recursive Flood Fill** | Correct implementation of `revealReachable` with base cases, 4-way recursion, and termination. |
| **0.5** | **Memory Diagram & Stack Trace** | Completed written deliverables: parameter passing memory diagram and scanner activation record trace. |
| **0.5** | **State Table & OO Reflection** | Completed written deliverables: state-transition table and procedural vs. OO architectural reflection. |
| **0.5** | **Student Unit Tests** | Thorough unit test cases added in `StudentMazeSolverTest` verifying edge cases. |
| **0.5** | **Scala 3 Style & Strict Typing** | Code conforms to `-Yexplicit-nulls`, `-language:strictEquality`, passes `scalafmtCheckAll`, and avoids custom domain classes. |

### Possible Deductions: up to...
* **-1.0 pt**: Modification of existing procedure signatures, test fixtures, or project architecture.
* **-1.0 pt**: Introduction of custom domain classes, case classes, traits, or ADTs (violating the imperative project constraint); we will soon work on a fully-object oriented version of this project!
* **-1.0 pt**: Compiler warnings under `-Werror` or unformatted code failing `scalafmtCheckAll`.
* **-1.0 pt**: Missing AI transcript or affidavit that AI was not used.

---

# Written Report Deliverables (Student Submissions)

> *Fill in your answers to each of the five sections below directly within this file.*

### Deliverable 1: State-Transition Table

Trace the three moves `R`, `D`, `D` on the `smallLines` 5×5 maze starting at `(1, 1)` with `energy = 10` and `cellsCollected = 0`:

```
#####
#R.C#
#.#.#
#..E#
#####
```

| Step | Move Command | Attempted Pos | Walkable? | Updated Robot Pos | Remaining Energy | Cells Collected | Note |
|:----:|:------------:|:-------------:|:---------:|:-----------------:|:----------------:|:---------------:|:-----|
| 0 | *Init* | — | — | `(1, 1)` | 10 | 0 | Initial state |
| 1 | `R` | `(1, 2)` | [TODO: Yes/No] | [TODO] | [TODO] | [TODO] | [TODO] |
| 2 | `D` | `(2, 2)` | [TODO: Yes/No] | [TODO] | [TODO] | [TODO] | [TODO] |
| 3 | `D` | `(2, 2)` | [TODO: Yes/No] | [TODO] | [TODO] | [TODO] | [TODO] |

---

### Deliverable 2: Parameter-Passing & Aliasing Memory Diagram

Illustrate with an ASCII diagram the stack and heap layout during execution of:
1. `moveNorth(pos)`
2. `localReset(pos)`

```
[TODO: Provide your ASCII memory diagram here showing Stack Frames, Callee Parameters, and Heap Arrays]
```

**Written Explanation:**
1. Why does `moveNorth` mutate the caller's array, whereas passing an `Int` variable (e.g. `energy`) cannot be mutated by the callee?
   > [TODO: Your answer here]

2. Why does Scala reject `position = Array(1, 1)` in parameter lists, and what happens when you rebind a local `val replacement = Array(1, 1)`?
   > [TODO: Your answer here]

3. How does call-by-name (`action: => Boolean` in `repeatUntilStopped`) fundamentally differ from passing a reference or passing a primitive value?
   > [TODO: Your answer here]

---

### Deliverable 3: Scanner Call-Stack Trace & Activation Records

Given the 4×4 `tinyLines` maze:
```
####
#..#
#.##
####
```
Trace `revealReachable(maze, revealed, 1, 1)`:

Neighbours are explored in the order up, down, left, right. Indentation shows stack depth.
`push` means a new activation record was created, and `pop` means it returned.

```
push reveal(1,1)                 in bounds, '.', not revealed -> mark revealed(1)(1)
  push reveal(0,1)               wall '#'
  pop  reveal(0,1)  -> 0
  push reveal(2,1)               in bounds, '.', not revealed -> mark revealed(2)(1)
    push reveal(1,1)             already revealed
    pop  reveal(1,1)  -> 0
    push reveal(3,1)             wall '#'
    pop  reveal(3,1)  -> 0
    push reveal(2,0)             wall '#'
    pop  reveal(2,0)  -> 0
    push reveal(2,2)             wall '#'
    pop  reveal(2,2)  -> 0
  pop  reveal(2,1)  -> 1 + 0 + 0 + 0 + 0 = 1
  push reveal(1,0)               wall '#'
  pop  reveal(1,0)  -> 0
  push reveal(1,2)               in bounds, '.', not revealed -> mark revealed(1)(2)
    push reveal(0,2)             wall '#'
    pop  reveal(0,2)  -> 0
    push reveal(2,2)             wall '#'
    pop  reveal(2,2)  -> 0
    push reveal(1,1)             already revealed
    pop  reveal(1,1)  -> 0
    push reveal(1,3)             wall '#'
    pop  reveal(1,3)  -> 0
  pop  reveal(1,2)  -> 1 + 0 + 0 + 0 + 0 = 1
pop  reveal(1,1)  -> 1 + 0 + 1 + 0 + 1 = 3

Total: 13 activation records, maximum depth 3, result 3
Final revealed cells: (1,1), (2,1), (1,2)
```

**Questions:**
1. What values are stored in each activation record (stack frame) on the JVM?
   > Each frame stores the parameters: `row` and `col` as plain `Int` values, and *references* to the shared `maze` and `revealed` arrays (not copies of the arrays). It also stores the running sum `1 + up + down + ...` while it waits for each recursive call to return, plus the return address so the program knows where to continue in the caller when the frame is popped.

2. Which invocations hit the base cases (out of bounds, wall `#`, or already revealed)?
   > **Wall:** `(0,1)`, `(1,0)`, `(3,1)`, `(2,0)`, `(2,2)` (twice), `(0,2)`, and `(1,3)`. **Already revealed:** `(1,1)` twice, once from `(2,1)` and once from `(1,2)`, when each tries to step back to where it came from. **Out of bounds:** none, because the maze has a wall border, so the wall check stops the recursion before it can leave the grid.

3. Why is marking `revealed(row)(col) = true` *prior* to recursive neighbor exploration essential to avoid infinite recursion?
   > Neighbouring cells call each other: `(1,1)` calls `(2,1)`, and `(2,1)` calls back up to `(1,1)`. If `(1,1)` weren't marked yet, that call-back would start over from `(1,1)`, and the two cells would keep calling each other until a `StackOverflowError`. Marking first means the call-back hits the "already revealed" base case and returns 0. Since each cell can only be marked once, there are at most `M*N` cells that recurse further, so the recursion always ends, even in mazes with loops.

4. What is the maximum theoretical stack depth in the worst-case maze of size $M \times N$?
   > About $M \times N$. The worst case is a single winding corridor that passes through every open cell. The recursion follows it one cell at a time without returning, so every cell has a frame on the stack at once, plus one more for the final base-case call. For the 4×4 example the depth is only 3.

---

### Deliverable 4: Recursion vs. Iteration (Call Stack vs. Heap Stack)

Compare `revealReachable` (recursive) with `revealReachableIterative` (explicit heap stack):
* **Memory location of state:**
  > The recursive version keeps its pending work in JVM call-stack frames, one per active call. The call stack is small (usually about 512 KB–1 MB). The iterative version keeps pending work as `(row, col)` tuples in a `mutable.Stack` on the **heap** and only uses one stack frame. In both versions the `maze` and `revealed` arrays are on the heap.
* **Risk of `StackOverflowError` vs. `OutOfMemoryError`:**
  > The recursive version can go as deep as the longest path in the maze, so a very large maze (e.g. 1000×1000 with one long winding corridor) can run out of call-stack space and throw `StackOverflowError`, even with plenty of memory free. The iterative version only uses heap memory, which is much larger (usually gigabytes), so it would need a huge maze to cause an `OutOfMemoryError`.
* **Performance tradeoffs (function invocation overhead vs object allocation):**
  > The recursive version pays for a method call every time (creating a frame and passing arguments) but doesn't create any objects. The iterative version avoids those calls but creates a new tuple object for each push, which adds work for the garbage collector. For the small sample mazes the difference is too small to notice. The real benefit of the iterative version is that it doesn't crash on large mazes, not that it's faster.

---

### Deliverable 5: Architectural Reflection (Procedural vs. Object-Oriented)

In this assignment, all state is represented using raw arrays (`Array[Array[Char]]`, `Array[Int]`) and loose variables passed into top-level procedures.
1. What problems or vulnerabilities arise when multiple procedures directly read and mutate naked arrays?
   > [TODO: Your answer here]

2. How would an Object-Oriented or ADT-based design (e.g., `Maze`, `Robot`, `Cell`, `Position`) improve encapsulation, type safety, and maintainability in Project 2?
   > [TODO: Your answer here]

---

### AI Usage Disclosure
* [ ] No AI tools were used on this assignment.
* [x] AI tools were used (describe tool, prompts, and provide transcript link below or transcript document(s) in doc/ subdirectory):
  > Claude Code (Claude Opus 5.5) was used for Part B code, tests, and Deliverables 3–4. See [doc/ai-usage.md](doc/ai-usage.md) for the tool, prompts, and transcript.
