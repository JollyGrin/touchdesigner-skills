# TouchDesigner MCP Learnings

Hard-won lessons from building scenes via the TouchDesigner MCP server.

## MCP Tool Usage

### Parameter Updates
- The `update_td_node_parameters` tool uses `properties` (not `parameters`) as the key.
- Pass values as a JSON object: `{"tx": 0, "ty": 1, "tz": 2}`

### Python Script Execution
- `execute_python_script` runs in a limited context — not all TD globals are available directly.
- `absTime` is NOT available in the script context. Use `import td` then `td.absTime.frame`.
- `ParMode` enum is NOT available as a global or via `td.ParMode`. Access it from an existing parameter: `type(par.mode).EXPRESSION`.
- `print()` output is NOT captured in the return value. To get data back, make the last line an expression that evaluates to a string.
- `return` cannot be used (scripts are not functions). Use a bare expression as the last line.
- TD type references (e.g., `windowCOMP`, `outTOP`) require `import td` then `td.windowCOMP`, `td.outTOP`, etc.
- Creating nodes via script: `op('/project1').create(td.geometryCOMP, 'name')`

### Setting Expressions on Parameters
```python
p = op('/project1/coin').par.ry
p.expr = 'absTime.frame * 2'
p.mode = type(p.mode).EXPRESSION  # Access ParMode enum from the par itself
```

### Connecting Nodes via Script
```python
op('/project1/out1').inputConnectors[0].connect(op('/project1/render1'))
```

## Rendering Pipeline

### Minimum Viable 3D Scene
A renderable scene requires at minimum:
1. **Geometry COMP** — contains SOPs (the 3D shapes)
2. **Camera COMP** — viewpoint
3. **Light COMP** — at least one light source
4. **Render TOP** — with `camera`, `geometry`, and `lights` parameters set

### Render TOP Geometry Parameter
- Can be a specific path like `/project1/coin` or wildcard `*` for all geometry in the network.
- Both forms work, but `*` is simpler for single-scene setups.

### Materials: Use Phong, Not PBR (for basic scenes)
- **PBR materials render BLACK** without a properly configured Environment Light with an environment map.
- **Phong materials work reliably** with standard point/directional lights.
- Phong parameter names: `diffr/diffg/diffb`, `specr/specg/specb`, `shininess`, `ambr/ambg/ambb`.
- PBR parameter names: `basecolorr/g/b`, `metallic`, `roughness`, `specularlevel`.
- Assign material to geometry COMP via: `coin.par.material = '/project1/coin/gold_phong'`

### SOP Flags Inside Geometry COMPs
- The SOP with `display=True` and `render=True` flags is what gets rendered.
- Always verify both flags after creating SOPs: `tube.display = True; tube.render = True`

### Camera Setup
- Default camera looks down -Z axis.
- Camera at `tz=1.5` sees geometry at origin.
- Use `rx` (negative values) to tilt camera downward to center objects.
- Check near/far planes (defaults 0.1–1000.0 are fine for most scenes).

### Debugging Black Renders
1. Sample pixels: `render.sample(x=640, y=360)` — returns RGBA tuple.
2. Scan the whole frame to find where geometry actually appears (it may be off-center).
3. Test with default geometry (no material) to isolate material vs. pipeline issues.
4. Remove material to check if geometry renders with defaults — if yes, material is the problem.
5. Force cook: `op('/project1/render1').cook(force=True)`

## Perform / Output

### Viewing the Render
- Clicking on `render1` in the network editor shows the output in the node viewer.
- For Perform mode, a **Window COMP** must exist and be configured:
  ```python
  w = op('/project1').create(td.windowCOMP, 'window1')
  w.par.winop = '/project1/render1'
  ```
- Set the Window Operator in **Dialogs > Perform Settings** to the window COMP path.
- An `out1` TOP connected to `render1` helps with output routing.

### MCP Webserver Base
- The `mcp_webserver_base` COMP is the MCP connection — **never delete it**.
- Path: `/project1/mcp_webserver_base`

## Common Node Types (for `create_td_node`)

### SOPs (Geometry)
`tubeSOP`, `torusSOP`, `sphereSOP`, `boxSOP`, `gridSOP`, `circleSOP`, `lineSOP`, `noiseSOP`

### TOPs (Textures/Images)
`renderTOP`, `outTOP`, `noiseTOP`, `feedbackTOP`, `bloomTOP`, `levelTOP`, `compositeTOP`

### CHOPs (Channels/Animation)
`noiseCHOP`, `lfoCHOP`, `mathCHOP`, `filterCHOP`, `nullCHOP`, `audiofileinCHOP`

### MATs (Materials)
`phongMAT`, `pbrMAT`, `constantMAT`

### COMPs (Components)
`geometryCOMP`, `cameraCOMP`, `lightCOMP`, `environmentlightCOMP`, `windowCOMP`, `baseCOMP`

## Animation Patterns

### Continuous Rotation
```python
p = op('/path').par.ry
p.expr = 'absTime.frame * 2'  # degrees per frame
p.mode = type(p.mode).EXPRESSION
```

### Expression Context
- Inside expressions, `absTime.frame`, `absTime.seconds`, `me.time.frame` are available.
- In `execute_python_script`, use `td.absTime.frame` instead.

## Tube SOP as Coin/Disc
- `height=0.05` for thin disc
- `cap=True` to close the ends
- `rad1=0.5, rad2=0.5` for uniform radius
- `orient=y` for vertical disc
- `rows=2, cols=40` for smooth circular edge
