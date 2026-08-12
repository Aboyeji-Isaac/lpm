# Living Project Memory (LPM)

Living Project Memory (LPM) is an open-source, AI-agnostic protocol project for preserving engineering execution continuity across interruptions.

AI-assisted development can be interrupted by usage limits, lost connectivity, application or terminal crashes, machine shutdowns, context-window exhaustion, or a change of agent or machine. Afterward, the central problem is not merely missing conversation context: neither the developer nor the next agent may be able to determine confidently what happened, what remains incomplete, or where execution should resume.

LPM proposes an append-only execution event history recorded alongside meaningful engineering work. The history should distinguish declared intent from observed facts and make execution state reconstructable after an interruption. Memory is the mechanism; continuity is the purpose.

## Project status

LPM is currently in the specification and design phase. This repository is a controlled reconstruction from documented design history after the original unpushed local repository became unavailable. The reconstructed files are new artifacts, not recovered originals.

The current milestone is to review and, when ready, accept the draft problem statement in [`rfcs/0001-problem-statement.md`](rfcs/0001-problem-statement.md). Event schemas, storage formats, command-line tools, recovery behavior, and integrations are not yet specified or implemented.

