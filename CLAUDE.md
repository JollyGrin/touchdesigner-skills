# TouchDesigner Scene Builder

This repo is used for creating TouchDesigner scenes from plain English descriptions via the MCP server.

## What This Project Does

Users describe a scene in natural language, and Claude builds it in a running TouchDesigner instance using the TouchDesigner MCP tools.

## Critical Rules

- **NEVER delete `/project1/mcp_webserver_base`** — this is the MCP connection to TouchDesigner.
- **Read LEARNINGS.md before building** — it contains essential patterns and gotchas discovered through trial and error.
- Always verify renders produce non-black pixels using `render.sample(x, y)` before declaring success.

## Scene Building Workflow

### 1. Clear the Existing Scene
Delete all nodes in `/project1` EXCEPT `mcp_webserver_base`. Use `get_td_nodes` to list them first.

### 2. Build the Minimum Viable Scene
Every 3D scene needs these four elements created in order:
1. **Geometry COMP** — with SOPs inside (the shapes)
2. **Camera COMP** — positioned to see the geometry
3. **Light COMP** — at least one light
4. **Render TOP** — with `camera`, `geometry` (use `*`), and `lights` set

### 3. Materials
- **Use Phong materials (`phongMAT`) for reliability.** PBR materials render black without a configured environment map.
- Create the material inside the geometry COMP, then assign via `geo.par.material = '/path/to/mat'`.

### 4. Animation
Set expressions via Python script:
```python
p = op('/project1/geo').par.ry
p.expr = 'absTime.frame * 2'
p.mode = type(p.mode).EXPRESSION
```

### 5. Output Setup
- Create an `out1` TOP connected to the render TOP.
- Create a `window1` COMP with `winop` pointing to the render TOP for Perform mode.

### 6. Verify the Render
Sample pixels across the frame to confirm the scene is visible:
```python
r = op('/project1/render1')
r.sample(x=640, y=360)  # Returns (R, G, B, A)
```
If all black, check: material type (use Phong not PBR), SOP display/render flags, camera position/direction.

## MCP Tool Reference

### Key Tools
| Tool | Purpose |
|------|---------|
| `get_td_info` | Check connection to TouchDesigner |
| `get_td_nodes` | List all nodes under a parent |
| `create_td_node` | Create a new node (specify parentPath, nodeType, nodeName) |
| `delete_td_node` | Delete a node by path |
| `update_td_node_parameters` | Set node parameters (use `properties` key) |
| `get_td_node_parameters` | Read current parameter values |
| `execute_python_script` | Run Python in TD (last expression = return value) |

### Python Script Gotchas
- Use `import td` then `td.absTime.frame` (not bare `absTime`)
- Access `ParMode.EXPRESSION` via `type(par.mode).EXPRESSION`
- No `return` statements — use a bare expression as the last line
- `print()` output is not captured — return strings instead
- Create nodes in script: `op('/project1').create(td.geometryCOMP, 'name')`
- Type refs need `td.` prefix: `td.windowCOMP`, `td.outTOP`, etc.

## File Structure
- `CLAUDE.md` — this file (project instructions for Claude)
- `LEARNINGS.md` — detailed technical learnings and patterns
- `idea_reactor.md` — example scene description (fusion reactor concept)

## Context & memory

- When compacting, always preserve the list of modified files, the task's
  acceptance criteria, the build/test command, the PR URL, and the
  `STATUS:` line contract.
- Auto memory (`~/.claude/projects/<repo>/memory/`) holds Claude-written
  notes — corrections and confirmed approaches, one lesson per file. Don't
  save what the repo, its docs, or git history already record.
