\# user-flow-dev



&#x20; A Claude Code skill that maintains a living registry of \*\*user flows\*\* — behavioral descriptions of how a product should work — under

&#x20; `.claude/user-flows/` in your project. Designed for long Claude Code sessions where context drifts and old flows quietly break.



&#x20; Deliberately \*\*not\*\* called "user stories" — these are behavioral specs with branches and acceptance criteria, not backlog items.



&#x20; ## What it does



&#x20; - \*\*Infers domains\*\* from your codebase (auth, payments, plus app-specific ones like `hunt-builder`) — you never name them.

&#x20; - \*\*Generates a dense, scannable index\*\* so an agent can find relevant flows without reading every domain file.

&#x20; - \*\*Forces a clarification round\*\* before adding a flow, so branches and edges actually get captured.

&#x20; - \*\*Verifies acceptance criteria against real code\*\* with a strict no-vibes-based-verification rule.

&#x20; - \*\*Tracks tasks per flow\*\* and archives them on completion.



&#x20; ## Subcommands



&#x20; | Command | What it does |

&#x20; |---|---|

&#x20; | `/user-flow-dev init` | Scan codebase, infer domains, propose flows, write files. |

&#x20; | `/user-flow-dev add <description>` | Conversational add of a single flow with required clarification round. |

&#x20; | `/user-flow-dev pending` | List flows with no associated task. Read-only. |

&#x20; | `/user-flow-dev task <FLOW-ID>` | Generate a task file for one flow. |

&#x20; | `/user-flow-dev check \[domain]` | Verify acceptance criteria still hold against current code. |

&#x20; | `/user-flow-dev done <TASK-ID>` | Archive a completed task. |



&#x20; ## Install (personal, all projects)



&#x20; Clone into your personal skills directory:



&#x20; ```bash

&#x20; git clone https://github.com/<your-username>/user-flow-dev.git \~/.claude/skills/user-flow-dev



&#x20; On Windows (PowerShell):



&#x20; git clone https://github.com/<your-username>/user-flow-dev.git $HOME\\.claude\\skills\\user-flow-dev



&#x20; The /user-flow-dev slash command appears the next time you start Claude Code.



&#x20; Install (project-local, just this repo)



&#x20; git clone https://github.com/<your-username>/user-flow-dev.git .claude/skills/user-flow-dev



&#x20; Commit .claude/skills/user-flow-dev/ if you want the team to share it; otherwise add it to .gitignore.



&#x20; Update



&#x20; cd \~/.claude/skills/user-flow-dev \&\& git pull



&#x20; File structure produced



&#x20; .claude/user-flows/

&#x20;   overview.md          ← one-line-per-flow index

&#x20;   domains/

&#x20;     <domain>.md        ← per-domain summary, flow index, full flow detail

&#x20;   tasks/

&#x20;     todo.md            ← one-line-per-task index

&#x20;     archive/           ← completed task files (kept, never deleted)

&#x20;     <domain>/          ← active task files



&#x20; After init, the skill offers to add a small instruction block to your project's CLAUDE.md so future Claude sessions read overview.md on session start   

&#x20; and call /user-flow-dev done <TASK-ID> when finishing implementation work.



&#x20; Design constraints



&#x20; - Token efficiency. Indices are dense pipe-delimited lines (UF-AUTH-001 | Sign in with SSO | auth | tags: sso, session). An agent reads overview.md     

&#x20; first to decide which domain file to load.

&#x20; - Scope safety. All file writes are confined to .claude/user-flows/. The CLAUDE.md update during init is preview-then-confirm, never silent.

&#x20; - No content redundancy. The index line is duplicated (overview + domain index) for scannability; the content (branches, AC) lives in exactly one place.

&#x20; - Behavioral, not implementation. Flow detail describes what the user experiences — not tables, RLS, endpoints, or library calls.



&#x20; License



&#x20; MIT

