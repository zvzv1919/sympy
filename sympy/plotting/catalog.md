# Plotting Module Catalog

## Core Plotting (`plotting/`)

### `plot.py`
Main plotting API and data series definitions for matplotlib-based 2D/3D plots.

- `_arity(f)` — cross-version (Py2/3) introspection helper; returns the number of positional-or-keyword arguments a callable accepts. Used by color/aesthetic logic to decide whether to pass parameters or coordinates to user-supplied color functions.
- `Plot` — central figure container for data series; dispatches rendering to backends (matplotlib, text, default). Manages an internal `_series` list of `BaseSeries` objects.
  - `__init__(*args, **kwargs)` — sets default display configuration (title, xlabel/ylabel, xlim/ylim, axis_center, aspect_ratio, autoscale, margin, axis visibility, xscale/yscale, legend). Applies user keyword overrides via `hasattr`/`setattr` — silently ignores unknown kwargs.
  - `__getitem__`, `__delitem__` — indexed read/delete on `_series`.
  - `__setitem__(index, *args)` — indexed replacement of a series in `_series`; uses `*args` varargs signature, so the value arrives as a tuple even for single assignment (`p[0] = series`).
  - `append()`/`extend()` — add series from another `Plot` or individual `BaseSeries` objects.
- `check_arguments(args, expr_len, nb_of_free_symbols)` — argument grouping helper for matplotlib-based `plot*()` functions only.
  - Groups flat or tuple-wrapped expressions into plot series; three branches: multiple exprs with same range, series with same range, multiple with different ranges.
  - Supplies default range `(-10, 10)` for each free symbol when the user omits explicit range tuples.
  - Distinguishes expressions from ranges by checking if an element is a 3-tuple (symbol, min, max); symbolic expressions are identified by being non-tuple sympified objects.
  - Does NOT determine curve-vs-surface or coordinate mode — each `plot*()` entry point already knows its series type.
  - Explicitly excludes the "series of plots with same range" branch when expr_len == 3, because 3-element tuples are ambiguous between expression groups and range tuples.
- Public API entry points (matplotlib-based):
  - `plot()` — 2D line plots over one variable; accepts multiple expressions sharing the same free symbol. Raises `ValueError` if expressions use different free variables (e.g., `sin(x)` and `cos(y)`).
  - `plot_parametric()` — 2D parametric curves from two expressions over one parameter.
  - `plot3d()` — 3D surface from one expression over two variables.
  - `plot3d_parametric_line()` — 3D parametric curve from three expressions over one parameter; parses args via `check_arguments(args, 3, 1)`, constructs `Parametric3DLineSeries` objects.
  - `plot3d_parametric_surface()` — 3D parametric surface from three coordinate expressions (x, y, z) each over two independent parameters (u, v). Uses `check_arguments(args, 3, 2)`, so inherits the expr_len==3 ambiguity limitation requiring explicit grouping for multiple plots.
- `LineOver1DRangeSeries` — evaluates single expression over 1D range into *numeric* (numpy) arrays; adaptive subdivision with collinearity check.
- `Parametric2DLineSeries` — 2D parametric curve series; `get_segments()` uses recursive adaptive subdivision with complex-value handling (samples 10 intermediate points when both endpoints are non-real).
- `Parametric3DLineSeries` — 3D parametric curve from three expressions and a range.
- `SurfaceBaseSeries` — base class for 3D surfaces; `get_color_array()` dispatches callable coloring by arity and `is_parametric` flag (uses parameter meshes vs coordinate meshes).
- `SurfaceOver2DRangeSeries` — 3D surface from one expression over two variables; `get_meshes()` builds x/y meshgrid and lambdifies the expression.
- `ParametricSurfaceSeries` — 3D parametric surface from three coordinate expressions (x, y, z) over two parameters (u, v); `get_meshes()` lambdifies each expression separately and evaluates on parameter meshgrid to produce numerical coordinate grids.
- `ContourSeries` — contour (iso-level curve) data series for a scalar expression over a 2D x/y domain; inherits `BaseSeries` directly. Aliases `self.get_points = self.get_meshes` so callers can retrieve computed coordinate meshes via the same `get_points()` interface used by other series.
- `Line2DBaseSeries` — base class for 2D line series; defines `get_segments()` which converts raw point arrays into pairwise consecutive coordinate segments (shape [N-1, 2, dim]) for renderer consumption. Supports `steps=True` staircase mode: interleaves/offsets x and y arrays to produce step-function segments before pairing.
- `Line3DBaseSeries` — base class for 3D line series; inherits `get_segments()` from `Line2DBaseSeries` with `_dim=3`.
- `_matplotlib_list(interval_list)` — converts bounding rectangular intervals to x/y coordinate lists for matplotlib `fill()`; returns lists of four `None`s when input is empty (workaround because matplotlib rejects empty lists for `fill`).
- `MatplotlibBackend` — renders all series types via `process_series()`: dispatches 2D/3D lines, surfaces, contours, and implicit plots.
  - `__init__` — enforces dimensional consistency: raises `ValueError` if series mix 2D and 3D data (all series must be uniformly 2D or 3D).
  - Implicit plot rendering: disables axis smart bounds before rendering; interval-arithmetic results rendered with `fill()`; contour-based results use `contour` (equality) vs `contourf` (inequality).
- `TextBackend` — ASCII fallback backend; `show()` raises `ValueError` if more than one series or if series is not `LineOver1DRangeSeries`. Delegates single-expression rendering to `textplot()`.
- `DefaultBackend` — auto-selects `MatplotlibBackend` if matplotlib is available, otherwise `TextBackend`.

### `plot_implicit.py`
Implicit equation/inequality data series; computes raster data via interval arithmetic (rendering handled by backends in `plot.py`).

- `plot_implicit()` — public API; plots relations (Eq, SymPy boolean `And`/`Or`, inequalities) over 2D region. Accepts SymPy boolean combinators in the input expression; does not define ternary truth-value operators.
  - Variable inference: bare expressions auto-wrapped as `Eq(expr, 0)`; single-variable expressions get a synthetic dummy symbol for the missing axis.
- `ImplicitSeries` — data series for implicit plots; `_get_raster_interval()` recursively subdivides rectangles using interval arithmetic to determine inclusion.
  - Jitter mechanism: adds small random perturbations to grid sample boundaries to prevent false positives where aligned intervals (e.g., x∈[1,2], y∈[1,2]) incorrectly satisfy equality expressions (e.g., y==x), which would otherwise render spurious filled rectangles instead of curves.
  - Fallback: if interval evaluation raises `AttributeError`, warns and falls back to uniform mesh grid (`_get_meshes_grid`).

### `textplot.py`
Low-level ASCII art grid renderer (called by `TextBackend`; does not validate input count).

- `textplot(expr, a, b, W=55, H=21)` — evaluates a single expression over [a, b] and prints a text grid.

### `experimental_lambdify.py`
Custom expression-to-function converter for internal plotting use.

- `lambdastr()`, `experimental_lambdify()` — convert sympy expressions to callable functions with namespace translation.
- Caveat: unstable internal API; may be rewritten.

## Interval Arithmetic (`plotting/intervalmath/`)

### `interval_arithmetic.py`
Core `interval` class definition and all its arithmetic/comparison operator overloads, used by implicit plot region testing (not for variable range management or rendering discretization).

- `interval` — represents [start, end] with `is_valid` ternary flag (True/False/None for partial validity).
  - Constructor auto-swaps arguments when given in descending order (upper < lower), so `interval(5, 2)` yields `[2, 5]`.
- Arithmetic operators (`__add__`, `__sub__`, `__mul__`, `__div__`, `__pow__`) and ternary comparison operators, each with scalar and interval-typed dispatch paths.
  - `__mul__`: interval×interval with invalid/uncertain validity expands result to (-inf, inf); interval×scalar simply scales endpoints preserving validity.
- `__rpow__` — handles scalar**interval (reverse power); negative-base logic: invalidates wide exponents, rationalizes point exponents to check denominator parity.
- `_pow_float` — `interval ** float` operator path; rationalizes exponent to rational form and checks numerator/denominator parity for domain validity.
  - Three-way outcome: fully invalid (entire interval negative + even denom), partially valid (spans negative + even denom), or valid.

### `lib_interval.py`
Standalone named math functions (`sin`, `cos`, `exp`, `log`, `sqrt`, etc.) applied to `interval` objects, plus ternary logic operators. Does NOT define the `interval` class or its arithmetic operators (those are in `interval_arithmetic.py`).

- Standalone named math functions on `interval` objects: `Abs`, `exp`, `log`, `sqrt`, `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `sinh`, `cosh`, `tanh`, `acosh`, `asinh`, `atanh`, `ceil`, `floor`.
- Each is a direct function call (e.g., `sqrt(x)`), not the `**` operator path; handles domain validation returning `is_valid=False` outside domain, `is_valid=None` for partial overlap.
- `ceil`, `floor` — set `is_valid=None` when the interval spans a discontinuity (i.e., rounded start ≠ rounded end).
- `And(*args)` — ternary conjunction over 2-tuples of truth values (True/False/None); priority: False > None > True. Used to combine comparison results in range-based curve rendering.
- `Or(*args)` — ternary disjunction over 2-tuples; priority: True > None > False.
- `Min`, `Max` — interval-aware min/max returning new intervals.

## Pyglet 3D Plotting (`plotting/pygletplot/`)

### `__init__.py`
Public entry point for pyglet plotting; defines the `PygletPlot` factory function that wraps the class in `plot.py`.

- `PygletPlot(*args, **kwargs)` — factory function whose docstring documents the full user-facing API:
  - Flexible variable interval syntax: `[var, min, max, steps]` with partial specification — `[]` uses all defaults, `[100]` sets only step count, `[-13, 13]` sets only bounds; omitted args filled from coordinate mode defaults.
  - Automatic coordinate mode detection: 1 expression → Cartesian, 2–3 expressions → parametric; 1 variable → curve, 2 variables → surface. Supports Cartesian, parametric, polar, cylindrical, spherical modes.
  - Calculator-like indexed interface (`p[1] = expr`), per-slot style/color, keyboard controls.
- Caveat: defined inside a try/except block; if pyglet (or any dependency) is not importable, a fallback `PygletPlot` is defined that re-raises the captured exception on every call (lazy-error-raising pattern for optional dependencies).

### `plot.py`
`PygletPlot` class implementation for interactive 3D visualization with OpenGL/pyglet.

- Module-level pyglet dependency check: imports `pyglet.gl` in a try/except at the top of the file; raises `ImportError` immediately with a descriptive message if pyglet is not installed (eager failure, unlike `__init__.py`'s lazy-error pattern).
- `PygletPlot` — top-level user-facing 3D plot class; its docstring is the authoritative definition of auto-detection rules (1 expr → Cartesian, 2–3 → parametric; 1 var → curve, 2 vars → surface) and supported coordinate modes. Manages plot objects, axes, camera, window, and rendering thread.
- `__init__(*fargs, **win_args)` — constructor: extracts `axes` keyword (semicolon-delimited key=value option string like `"style=ordinate; stride=0.25"`), parses it via `parse_option_string()`, and builds `PlotAxes`. Registers initial expressions at index 0 via `self[0] = fargs`. Conditionally shows the window based on `visible` kwarg (default True).
- `__setitem__(i, args)` — indexed assignment (`p[1] = expr`); parses args into a `PlotMode` and stores it.
  - Wraps `GeometryEntity` in a list even though it satisfies `is_sequence()`, preventing geometry objects from being unpacked as multiple arguments.
  - Passes `PlotObject` instances through directly without parsing.

### `plot_mode.py`
Coordinate-system mode registry and argument interpretation only — no coordinate math or coordinate-to-Cartesian conversion (those live in `plot_modes.py`).

- `PlotMode` — internal factory + registry class invoked by `PygletPlot.__setitem__`; resolves a concrete mode subclass from the registry given already-determined variable counts. Does not define auto-detection rules or user-facing API (those belong to `PygletPlot` in `plot.py`).
- `__new__(*args, **kwargs)` — factory method: separates expressions from intervals, counts variables, looks up the matching mode subclass in the registry, and returns an instantiated subclass.
- `_get_mode(mode_arg, i_var_count, d_var_count)` — resolves a mode argument (string alias or class) to a concrete mode class from `_mode_map`/`_mode_default_map`. Delegates to `_get_default_mode` (empty string) or `_get_aliased_mode` (named alias).
- `_get_default_mode(i, d)` / `_get_aliased_mode(alias, i, d)` — look up mode by d_var/i_var count; on `TypeError` (no mode registered for exact i_var count), recursively retry with incremented i_var count up to `_i_var_max` before raising `ValueError`.
- `_interpret_args(args)` — separates raw args into (functions, intervals); enforces ordering: raises `ValueError` if a `PlotInterval` appears before any expression. Handles `GeometryEntity` specially by extracting coordinates via `arbitrary_point()`.
- `_find_i_vars(functions, intervals)` — collects independent variables first from intervals (in order), then from free symbols of functions.
- `_fill_intervals()` — copies default intervals, merges user-provided ranges, then assigns orphan intervals (those without a variable) to remaining unused free parameters.

### `plot_mode_base.py`
Base class providing shared infrastructure for all pyglet plot modes, including rendering display style selection (wireframe/solid/both).

- `PlotModeBase` — common parent for all mode implementations.
- `draw()` — main rendering entry point; checks `style_override` first (class-level, wins if non-empty), else uses instance `_style`.
  - Uses bitmask dispatch (wireframe=1, solid=2, both=3) to selectively draw wireframe and/or solid display lists.
- `_get_evaluator()` — tries fast lambda evaluator first; on exception, falls back to sympy substitution evaluator with a warning.
- `_get_sympy_evaluator()`, `_get_lambda_evaluator()` — abstract methods implemented by concrete modes.
- Thread-safe rendering stack: `push_wireframe()`, `push_solid()` with `_draw_lock`.
- `_render_stack_top()` — consumes top of render stack: compiles callable into GL display list, returns cached list if valid, or regenerates via `_create_display_list()` if `glIsList()` reports the list invalid (e.g., after context loss).
- `_on_calculate()` — triggers vertex/color vertex computation in background threads.
- `_calculate_verts()` / `_calculate_cverts()` — thread-coordination wrappers around concrete `_on_calculate_verts`/`_on_calculate_cverts` callbacks.
  - `_calculate_cverts` skips immediately if geometry verts are still computing (`_calculating_verts` event); busy-waits if a prior color calc is running.
  - `_calculate_verts` guards against concurrent re-entry via `_calculating_verts` event; invokes `bounds_callback` on completion.
- `style` property (`_set_style`) — when style is set to empty string, auto-selects rendering appearance: computes max v_steps across intervals and picks 'both' (wireframe+solid) if ≤ 40, or 'solid' (filled only) if > 40.
- Class-level attributes: `styles` (render style bitmask dict), `style_override` (forces rendering style when non-empty), `i_vars`, `d_vars`, `intervals`, `aliases`, `is_default`.

### `plot_modes.py`
Concrete plot mode classes with coordinate-to-Cartesian conversion logic for all supported coordinate systems.

- `float_vec3(f)` — decorator that coerces all three components of a returned 3-vector to native Python `float`; used only by Cartesian and parametric sympy evaluators.
- `Cartesian2D` — single-variable y=f(x) curve embedded into 3D by returning (x, y, 0.0) in both sympy and lambda evaluators.
- `Cartesian3D` — z=f(x,y) surface mode; sympy evaluator chains two sequential `subs()` calls for x then y.
- `ParametricCurve2D` — 2D parametric curve embedded into 3D by hardcoding z=0.0 in both sympy and lambda evaluators.
- `ParametricCurve3D`, `ParametricSurface` — 3D parametric curve and surface modes using all three coordinate expressions.
- `Polar` — polar-to-Cartesian (r,θ → x,y) curve mode.
- `Cylindrical` — cylindrical-to-Cartesian (r,θ,h → x,y,z) surface mode.
- `Spherical` — spherical-to-Cartesian (r,θ,φ → x,y,z) surface mode; physics convention (azimuth 0–2π, polar 0–π).
- Each implements `_get_sympy_evaluator()` (symbolic substitution + `float()`) and `_get_lambda_evaluator()` (`lambdify` for fast numeric evaluation).

### `plot_interval.py`
Bounded interval representation for pyglet variable ranges (discretized sample points for rendering, not interval arithmetic).

- `PlotInterval` — stores [variable, min, max, steps] with property accessors and validation. Does NOT auto-reorder bounds; assumes caller provides min ≤ max.
  - `__init__(*args)` — flexible constructor: accepts a string (parsed via `eval`), a tuple/list of bounds, copy from another `PlotInterval`, or positional args `(symbol, min, max, steps)` with optional leading symbol.
  - Property setters validate types: `v_min`/`v_max` must be sympifiable to a numeric value (tested via `float(evalf())`), `v_steps` must be a positive integer.
- `fill_from(b)` — merges defaults from another interval for partial specifications.
- `vrange()` — yields v_steps+1 evenly-spaced *symbolic* (sympy) numbers from v_min to v_max; guarded by `@require_all_args` decorator that raises if any interval parameter is `None`.
- `vrange2()` — yields v_steps consecutive adjacent (a, b) pairs sharing endpoints, covering v_min to v_max (used for line segments/mesh cells).
- `frange()` — float version of `vrange()`; evaluates each sympy number to float.

### `plot_curve.py`
Curve rendering for 1D pyglet plots.

- `PlotCurve` — OpenGL vertex computation and caching for pyglet wireframe curve drawing (no argument parsing or series construction).
  - `_on_calculate_verts()` — evaluates parametric positions along the curve parameter range; catches `ZeroDivisionError`/`NameError` during evaluation and stores `None` for undefined points. Thread coordination lives in base class.
  - `_on_calculate_cverts()` — computes per-vertex colors via `ColorScheme.apply_to_curve`.
  - `draw_verts(use_cverts)` — emits OpenGL `GL_LINE_STRIP` segments; breaks the strip at `None` vertices to create visual discontinuities at undefined points.

### `plot_surface.py`
Surface rendering for two-parameter (u, v) pyglet 3D surface plots.

- `PlotSurface` — OpenGL vertex grid for pyglet surface rendering; supports wireframe and solid draw styles (not used by matplotlib-based plotting).
  - `_on_calculate_verts()` — evaluates parametric positions over u×v grid; catches `ZeroDivisionError` and stores `None` for undefined points.
    - Computes per-axis bounding box; zero-span guard sets axis range to 1.0 when surface is flat along that axis to prevent division-by-zero.
  - `draw_verts(use_cverts, use_solid_color)` — emits `GL_QUAD_STRIP` segments; ends and restarts the strip at `None` vertices to create visual gaps at undefined points. When per-vertex coloring is active, falls back to black `(0,0,0)` for any vertex whose computed color is `None`.

### `plot_axes.py`
Coordinate axes OpenGL rendering (not parsing; axes option string parsing occurs in `plot.py`'s `PygletPlot.__init__`).

- `PlotAxes` — draws axes with configurable styles (ordinate, frame, box, none); manages labels, ticks, and per-axis bounding boxes.
  - `adjust_bounds(child_bounds)` — merges child plot bounds into the cumulative bounding box; silently skips any axis whose child bounds contain infinity (`S.Infinity`), leaving that axis unchanged.
  - `reset_bounding_box()` — resets all three axes to `[None, None]` bounds and clears tick arrays.

### `plot_camera.py`
3D camera control for pyglet interactive viewer.

- `PlotCamera` — handles perspective/orthographic projection, position, rotation, zoom, and named viewing-angle presets.
  - `set_rot_preset(preset_name)` — applies a named rotation preset from `rot_presets` dict; catches `AttributeError` to raise `ValueError` with descriptive message for invalid preset names.
  - `euler_rotate()`, `spherical_rotate()` — rotation primitives; `zoom_relative()` — zoom with ortho/perspective branching.
  - `reset()` — restores default distance, position, and rotation matrix.

### `plot_controller.py`
User input handling with 2D/3D mode awareness.

- `PlotController` — maps keyboard/mouse events to camera actions (rotate, zoom, reset).
  - Class-level sensitivity constants: `normal_mouse_sensitivity` / `modified_mouse_sensitivity` and `normal_key_sensitivity` / `modified_key_sensitivity`; shift key toggles from normal to modified (slower) sensitivity for both pointer and keyboard input.
  - `__init__(window, **kwargs)` — initializes boolean `action` state dictionary tracking all user interactions (rotate, zoom, spin, reset, presets, axis toggles, `modify_sensitivity`); accepts `invert_mouse_zoom` option.
  - `update(dt)` — applies accumulated input each frame; branches on `is_2D()`: in 2D mode, arrow keys become translations instead of rotations, and model-Z-axis rotation keys are suppressed entirely.
  - `is_2D()` — returns True when all plotted functions have at most 1 independent variable and at most 2 dependent variables; determines whether dragging translates (2D) or rotates (3D).

### `plot_window.py`
OpenGL window management, rendering loop, and title-bar progress display.

- `PlotWindow` — extends `ManagedWindow`; sets up GL context, coordinates camera and controller.
- `draw()` — called each frame; acquires `_render_lock` to iterate plot functions and collect vertex/color progress in a single pass.
  - Sets initial viewing orientation from the first rendered object's `default_rot_preset` via a `drawing_first_object` flag; only the first surface influences the default camera angle.
  - Progress tracking catches `ValueError` silently: swallows errors from concurrent modification of vertex/color progress counters. This is narrow tolerance for progress display only — it does not govern loop termination or resource cleanup (those are handled by the parent `ManagedWindow` event loop).
- `update_caption()` — formats vertex and color calculation percentages into the window title bar.

### `plot_object.py`
Base renderable object.

- `PlotObject` — base class for all GL-renderable objects; defines `draw()` interface and visibility.

### `plot_rotation.py`
Vector math and rotation utilities.

- `get_sphere_mapping(x, y, width, height)` — maps screen pixel positions to unit sphere for trackball rotation (not coordinate unprojection); clamps to viewport, normalizes, and projects onto sphere surface (or equator if outside radius).
- `cross()`, `dot()`, `mag()`, `norm()` — basic vector operations.
- `get_spherical_rotatation()` — trackball-style rotation matrix from two screen positions via sphere mapping.
  - Returns `None` when both positions map to nearly the same sphere point (dot product ≈ 1.0), guarding against degenerate zero-angle rotations.

### `color_scheme.py`
Color mapping for curves and surfaces.

- `ColorGradient` — interpolates colors across value intervals.
- `ColorScheme` — applies color functions (including lambdified expressions) to vertices.
  - `__call__(x, y, z, u, v)` — evaluates the color function; catches any exception and returns `None` (used as sentinel for failed evaluations).
  - `apply_to_curve(verts, u_set)` — assigns RGB to 1D vertex list; two-pass: compute raw channels + track bounds, then normalize to [0,1] and apply gradient. Skips None vertices (from exceptions or missing verts). Sets `v=None` for single-parameter curves.
  - `apply_to_surface(verts, u_set, v_set)` — two-pass RGB normalization over a u×v vertex mesh: pass 1 computes per-vertex RGB via color function and tracks per-channel min/max bounds; pass 2 rescales each channel to [0,1] via `rinterpolate` then applies gradient. Skips None entries.

### `managed_window.py`
Pyglet window lifecycle and threaded event loop with thread-safe GL lock management.

- `ManagedWindow` — wraps pyglet Window; spawns a separate thread running `__event_loop__` for FPS-limited rendering.
- `__init__(**win_args)` — early-returns immediately (no thread, no window) when `runfromdoctester` kwarg is truthy; otherwise merges default window args and spawns the event-loop thread.
- `__event_loop__()` — the thread function: acquires/releases module-level `gl_lock` via try/finally for both initialization and each per-frame cycle (dispatch, update, draw, flip).
  - On uncaught exception during per-frame rendering: catches exception, sets `has_exit = True` to terminate the loop, and the `finally` block ensures `gl_lock` is released.
  - Initialization errors similarly set `has_exit = True` before the loop starts.
  - After loop termination, calls parent `close()` to release the window and its resources.
- `gl_lock` — module-level `threading.Lock` guarding all OpenGL calls; distinct from the plot-level `_render_lock` in `PlotWindow.draw()` (which serializes plot-function iteration, not GL context access).

### `util.py`
OpenGL state queries, 3D math utilities, numeric range/interpolation helpers, and option parsing.

- `get_model_matrix()`, `get_projection_matrix()` — GL state queries; default to single-precision (`c_float`/`glGetFloatv`).
- `get_viewport()` — returns current GL viewport as integer array.
- `screen_to_model()` — unprojects pixel (screen) coordinates to 3D world coordinates via `gluUnProject`; overrides default single-precision with `c_double`/`glGetDoublev` for matrix retrieval.
- `model_to_screen()` — projects 3D world coordinates to screen coordinates via `gluProject`; also uses double-precision override.
- `billboard_matrix()` — resets the upper-left 3×3 rotation submatrix of the current modelview matrix to identity while preserving translation (row 4) and projection (column 4), so drawn primitives always face the viewer.
- `strided_range(r_min, r_max, stride, max_steps=50)` — generates evenly-spaced tick values between endpoints aligned to stride boundaries; recursively doubles stride when step count exceeds `max_steps` to prevent excessive output.
- `interpolate()`, `rinterpolate()`, `interpolate_color()` — linear interpolation helpers; `rinterpolate` computes inverse ratio.
- `scale_value()`, `scale_value_list()` — normalize values to [0,1] range.
- `create_bounds()`, `update_bounds()` — track per-axis min/max bounding boxes.
- `parse_option_string()` — parses keyword arguments from string format.
