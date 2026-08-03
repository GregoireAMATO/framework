# 01 - Preflight

Guard action that verifies AIDD is installed and captures the output mode for the session.

**No delegation, no scaffolding.** Halts the entire flow if the prerequisite is not met.

## Input

The user's raw product idea, captured from the invocation message.

## Output

`go_no_go` (`go` or `abort`) and, when `go`, `output_mode` (`file | ticket | both`, captured once,
default `file`, propagated to actions 05 and 06). If `go_no_go = abort`, no further action runs
and no files or directories are created.

## Process

1. **Check.** Verify that `aidd_docs/` exists at the repo root.
   - If absent, return the message below and halt immediately. Produce nothing, create nothing,
     do not scaffold the framework or the memory bank, do not proceed to step 2.

     > **AIDD is not installed (`aidd_docs/` not found). Install AIDD first; this orchestrator
     > does not scaffold the framework or the memory bank.**
   - If present, continue to step 2.
2. **Capture.** Ask the user once where epics and user stories should be saved:

   > **Where should epics and user stories be saved?**
   >
   > - `file` — written to `aidd_docs/` (default, no external tools required)
   > - `ticket` — created in the configured ticketing tool
   > - `both` — written to file AND created as tickets
   >
   > Press Enter to accept the default (`file`).

   Wait for the response. Store it as `output_mode`, defaulting to `file` on an empty reply.
3. **Confirm.** Emit `go_no_go = go` and report readiness:

   > **Preflight passed.** AIDD is installed. Output mode: `<output_mode>`.
   > Starting the greenfield chain: brainstorm → prd → bootstrap → epic-breakdown → user-stories.

   Do not re-ask `output_mode` in any subsequent action.

## Test

- **Missing `aidd_docs/`**: invoke with `aidd_docs/` absent. Expect the exact abort message
  `"AIDD is not installed (aidd_docs/ not found). Install AIDD first; this orchestrator does not
  scaffold the framework or the memory bank."` Confirm zero files created, zero directories
  created, no further step executed.
- **Present `aidd_docs/`**: invoke with `aidd_docs/` present. Expect step 2 to prompt the
  output-mode question.
- **Default output mode**: user presses Enter without typing. Expect `output_mode = file`.
- **Explicit mode — ticket**: user types `ticket`. Expect `output_mode = ticket`.
- **Explicit mode — both**: user types `both`. Expect `output_mode = both`.
- **Propagation contract**: confirm `output_mode` is available to actions 05 and 06 without
  being re-asked.
