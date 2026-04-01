In this task, you'll validate an MCP server with MCP Inspector, find bugs by testing tool behavior, and fix the broken code.

## Workflow at a glance

👩‍💻
1. Set up your environment
2. Run baseline tests
3. Smoke-test the server with MCP Inspector
4. Test the tools and identify bugs
5. Fix the bugs in `get_build_history`
6. Re-run tests and verify the fixes
7. Check your work
8. Submit


## 1. Set up your environment

Once you log in to your GitHub account, the repository for this task will be added automatically.

1. Confirm that `mcp-validating-debugging-practice` appears in your account. 
2. Clone the repo locally and open it in your editor (Cursor, VS Code, etc.)
3. Install dependencies:
    
    ```
    uv sync
    ```
    

**Prerequisites**:

- Python ≥ 3.12
- [uv](https://docs.astral.sh/uv/) package manager
- Node.js (required for MCP Inspector)

### Key files

- `server.py` — server entry point (stdio + SSE transports)
- `app.py` — FastMCP server initialization
- `tools.py` — tool handlers (`analyze_hotspots`, `analyze_file_activity`, `get_build_history`, etc.)
- `analysis.py` — core analysis functions
- `git_utils.py` — `GitRepository` adapter
- `mcp_tests/` — existing test suite

## 2. Run the baseline tests

Your goal in this task is to investigate the broken behavior, fix the code, and get the test suite back to green.

First, run the test suite — some tests should fail:

```
uv run python -m pytest mcp_tests/-v
```

## 3. Smoke test the server with MCP Inspector

Now test the server from an MCP client's point of view.

Start MCP Inspector:

```
npx-y @modelcontextprotocol/inspector
```

In the Inspector UI, connect to your server over stdio:

```
uv run python server.py
```

Confirm that:

- The connection status is green
- The tools list is populated
- The server is callable from Inspector

## 4. Test the tools

### 4.1. Test `analyze_hotspots`

Call `analyze_hotspots` in MCP Inspector with example parameters (for example, `days: 30`).

Check that the results look sensible:

- File paths look real
- Risk scores and commit counts are plausible
- Results are ranked by risk score

### 4.2. Test `get_build_history` and identify the bugs

Now test `get_build_history`.

Try it with:

- No arguments
- A `status` filter
- A `branch` filter

You should observe the following bugs:

- Filtering by `status: "failed"` returns successful builds instead of failed ones
- `total_builds` reflects the filtered count instead of the full dataset size
- The module-level `BUILDS` list can be mutated between calls

## 5. Fix the bugs in `get_build_history`

Fix all three issues in `tools.py`.

Your fixes should:

- Correct the status filter so matching statuses are kept
- Make `total_builds` reflect the full dataset size
- Copy the shared list before working with it: `builds = list(BUILDS)`

## 6. Verify your fixes

Run the tests again until everything passes:

```
uv run python -m pytest tests/-v
```

Reconnect to MCP Inspector and verify the behavior manually:

- `get_build_history` with `status: "failed"` returns only failed builds
- `get_build_history` with no filters reports the full total count

### **Optional: Test with a coding agent**

1. Add the MCP server to your Claude Code/Cursor config (use the same stdio command as in Inspector).
2. Ask: "What are the hotspot files in this repo?"
3. Confirm the agent calls `analyze_hotspots` and produces a useful answer based on the tool output.

## **🚀 Speed up with a prompt (optional)**

You can use AI to generate a first debugging pass. Do this only **after** running the tests and reproducing the issue in MCP Inspector.

You'll still need to review the code, re-run the tests locally, and make sure the final implementation satisfies the submission checklist.

### Prompt

```
You are working in an MCP server repository with tools defined in `tools.py`.

Your task is to debug and fix `get_build_history`.

Goals:
- keep the existing MCP server structure intact
- fix the broken behavior in `get_build_history`
- make all tests pass

What to verify and fix:
1. The status filter is currently inverted
   - when filtering with `status="failed"`, the tool should keep failed builds, not exclude them
2. `total_builds` is incorrect
   - it must reflect the full dataset size, not the filtered result count
3. The shared module-level `BUILDS` list may be mutated between calls
   - create a copy first with `builds = list(BUILDS)`

Validation steps:
- run `uv run python -m pytest tests/ -v`
- make sure the full test suite passes
- preserve the tool’s existing public interface
- do not rewrite unrelated parts of the project

Also sanity-check that:
- MCP Inspector can still connect
- `analyze_hotspots` still returns sensible ranked hotspot results
- `get_build_history(status="failed")` returns only failed builds
- `get_build_history()` reports the correct total count
```

## 7. Check your work

Before submitting, test your functionality locally and review the submission checklist. 

Run the tests one final time and make sure your fixes work in:

- `pytest`
- MCP Inspector

### ✅ Submission checklist

**Validation**

- [ ]  MCP Inspector connects successfully, and the tools list is visible
- [ ]  `analyze_hotspots` returns sensible ranked results
- [ ]  All required tests pass locally

**`get_build_history` fixes**

- [ ]  The status filter works correctly
- [ ]  `total_builds` reflects the full dataset size, not the filtered result count
- [ ]  `get_build_history` does not mutate shared build data between calls

## **8. Submit your task**

1. Commit your changes.
2. Push to GitHub.
3. Return to the lesson and click "Submit."
