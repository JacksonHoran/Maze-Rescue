# Maze Rescue — Proposed Work Split

A starting point for discussion. The goal is an even split of **actual work**: about half
the code and about half the written report each, not an even split of rubric points. Line
counts below are rough estimates of finished code.

---

## Partner A — Robot Movement, Game Loop & Aliasing

**Code** (`src/main/scala/mazesolver/MazeSolver.scala`): ~50 lines

| Procedure | Est. lines | Notes |
|:--|:--:|:--|
| `moveRobot` | ~8 | Uses B's `isWalkable` + `directionDelta` |
| `collectCell` | ~5 | |
| `isAtExit` | ~2 | |
| `playMoves` | ~15 | Most debugging: stop at 0 energy / at exit, blocked moves are free |
| `playGame` | ~18 | Same loop as `playMoves`, plus tracking cells and exit |
| `moveNorth`, `localReset` | ~3 | Part 2 aliasing lab |

**Written report** (`README.md`): ~half

- **Deliverable 1:** State-transition table (`R`, `D`, `D`), which traces your own game logic.
- **Deliverable 2:** Stack/heap memory diagram for `moveNorth` / `localReset` + 3 questions.
- **Deliverable 5:** Procedural vs. OO reflection (2 short essay questions).

**Student tests** (`StudentMazeSolverTest`)
- Completely trapped robot (existing stub)
- Energy exhaustion mid-journey (existing stub)

---

## Partner B — Grid Helpers, Rendering, Call-by-Name & Flood Fill

**Code** (`src/main/scala/mazesolver/MazeSolver.scala`): ~50 lines

| Procedure | Est. lines | Notes |
|:--|:--:|:--|
| `isWalkable` | ~3 | **Do first.** A's `moveRobot` needs it |
| `directionDelta` | ~6 | **Do first.** A's `moveRobot` needs it |
| `render` | ~12 | String building with `@` overlay + status line |
| `repeatUntilStopped` | ~5 | Call-by-name loop |
| `revealReachable` | ~10 | Recursive flood fill: the conceptually hardest piece |
| `revealReachableIterative` | ~15 | Extra credit (+0.5), explicit `mutable.Stack` |

**Written report** (`README.md`): ~half

- **Deliverable 3:** Call-stack trace of `revealReachable` on the 4×4 maze + 4 questions (the single biggest written item).
- **Deliverable 4:** Recursion vs. iteration comparison (3 parts, pairs with the extra credit).

**Student tests** (`StudentMazeSolverTest`)
- Scanner on cyclic corridors terminates (existing stub)
- One more original edge case (e.g. recursive and iterative fill agree on a non-square maze)

---

## Why this is even

| | Partner A | Partner B |
|:--|:--|:--|
| Code | ~50 lines, mostly game-loop logic and edge cases | ~50 lines, fewer but harder (recursion, extra credit) |
| Writing | 1 small table + 1 diagram + 5 short answers | 1 long trace + 4 short answers + 3-part comparison |
| Student tests | 2 | 2 |

If the extra credit is skipped, B is lighter on code. In that case B should also write the
AI usage disclosure and do the final coverage/CI pass.

---

## Shared / Together

- **Style:** Scala 3 significant indentation; code must compile under `-Yexplicit-nulls`, `-language:strictEquality` and `-Werror`, and pass `sbt scalafmtCheckAll`.
- **Reviews:** review each other's PRs.
- **Proofreading:** read each other's written answers. You'll each want to understand the other half for exams.
- **AI usage disclosure:** fill in the section at the bottom of `README.md`. A missing disclosure costs **−1.0**.
- **Final check:** run `sbt clean coverage test coverageReport`, confirm CI is green, then submit the repo URL on Sakai.

---

## Workflow

1. **Never commit to `main`.** Each person works on their own branch and opens a PR:
   - `partA-movement-game`
   - `partB-helpers-floodfill`
2. **B pushes `isWalkable` + `directionDelta` early** (a tiny first PR) so A can test `moveRobot`. A can stub them locally until then.
3. Both edit `MazeSolver.scala` and `README.md`, but in different functions/sections, so conflicts should be small. Pull `main` often.
4. Run before every push:
   ```bash
   sbt scalafmtAll
   ```
   ```bash
   sbt test
   ```
5. Don't change existing procedure signatures, test fixtures, or project layout (**−1.0**). No custom classes / case classes / traits (**−1.0**).

## Suggested timeline

| When | Milestone |
|:--|:--|
| Day 1 | B merges `isWalkable` + `directionDelta`; both start main code |
| Day 2–3 | Finish code on branches, open PRs, review and merge |
| Day 4 | Written deliverables + student tests |
| Day 5 | Proofread each other's writing, play-through with `sbt run`, coverage + CI, submit |

## Open questions to decide together

- Are we doing the extra credit?
- Who writes the AI usage disclosure, and do transcripts go in `doc/`?
- Deadline, and how often do we sync?
