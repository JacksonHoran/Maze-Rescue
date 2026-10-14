# AI Usage Disclosure

## Tool

- **Tool:** Claude Code (Claude desktop app, Code tab)
- **Model:** Claude Opus 5.5 (Anthropic)
- **Used by:** Jackson Horan (Part B)

## What AI was used for

| Area | AI contribution |
|:--|:--|
| Planning | Proposed a division of work between partners (`WORK_SPLIT.md`), then rebalanced it so code and writing are roughly 50/50 |
| Code (`MazeSolver.scala`) | Implemented `isWalkable`, `directionDelta`, `render`, `repeatUntilStopped`, `revealReachable`, and `revealReachableIterative` (extra credit) |
| Tests (`MazeSolverTest.scala`) | Wrote two `StudentMazeSolverTest` cases: flood fill on a cyclic corridor, and recursive vs. iterative flood fill agreement on all sample mazes |
| Written report (`README.md`) | Wrote Deliverable 3 (scanner call-stack trace + 4 questions) and Deliverable 4 (recursion vs. iteration), then simplified the wording on request |

All AI-generated code was compiled and tested with `sbt test` and checked with `sbt scalafmtCheckAll`.
Partner A's work (Part A) is not covered here.

## Prompts

The prompts given to the AI, in order:

1. "i have to work on this with a partner, can you make a sensible divide of the work that needs to get done and add it to a mardown file so i can discuss with my partner"
2. "is this work split about 50% of the actual workload not points wise?"
3. "yea can you make them about 50% of the literal work each, try to make them even on code and written writing"
4. "ok can either partner do all their work at once then hand off to the next?"
5. "ok ill take on part b so i can get started now, create the new branch and implement my wokr making sure not to water mark any commits and dont push anything to remote till i give the go ahead"
6. "can you reset those commits and just leave all the changes in staged so i can look at them, i want the commits erased from the history"
7. "is everything for part b done and in the diff?"
8. "can you make the written response answers that you wrote but simpler, still technical but a littel simpler"
9. "mark the AI disclosure as yes, then create a MD file in a doc/ directory and link to it in the readme"
10. "yes export the transcript and link it"

## Transcript

The full session transcript is in [ai-transcript.md](ai-transcript.md). It was converted from the Claude Code session log: every prompt and reply is included, tool calls are summarized, and long tool output is truncated.
