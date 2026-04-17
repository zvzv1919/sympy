# Plotting Module Catalog

## Core Plotting (`plotting/`)

### `plot.py`
Main plotting API and data series definitions for matplotlib-based 2D/3D plots.

- `Plot` — container for data series; dispatches rendering to backends (matplotlib, text, default). Supports indexed access (`__getitem__`, `__setitem__`, `__delitem__`) and `append()`/`extend()` for series manipulation.
- `check_arguments(args, expr_len, nb_of_free_symbols)` — argument parser that groups flat or tuple-wrapped expressions into plot series; handles ambiguity when expr_len == nb_of_free_symbols (e.g., 3D parametric lines where 3 exprs can't be distinguished from a range tuple).
- Public API entry points (matplotlib-based):
  - `plot()` — single-expression 2D plots over one variable.
  - `plot_parametric()` — 2D parametric curves from two expressions over one parameter.
  - `plot3d()` — 3D surface from one expression over two variables.
  - `plot3d_parametric_line()` — 3D parametric curve from three expressions over one parameter.
  - `plot3d_parametric_surface()` — 3D parametric surface from three coordinate expressions (x, y, z) each over two independent parameters (u, v).
- `LineOver1DRangeSeries` — evaluates single expression over 1D range; adaptive subdivision with collinearity check.
- `Parametric2DLineSeries` — 2D parametric curve series; `get_segments()` uses recursive adaptive subdivision with complex-value handling (samples 10 intermediate points when both endpoints are non-real).
- `Parametric3DLineSeries` — 3D parametric curve from three expressions and a range.
- `SurfaceBaseSeries` — base class for 3D surfaces; `get_color_array()` dispatches callable coloring by arity and `is_parametric` flag (uses parameter meshes vs coordinate meshes).
- `SurfaceOver2DRangeSeries`, `ParametricSurfaceSeries` — 3D surface data series.
- `Line2DBaseSeries`, `Line3DBaseSeries` — base classes for line series with common range/label logic.
- `_matplotlib_list(interval_list)` — converts bounding rectangular intervals to x/y coordinate lists for matplotlib `fill()`; returns lists of four `None`s when input is empty (workaround because matplotlib rejects empty lists for `fill`).
- `MatplotlibBackend` — renders all series types via `process_series()`: dispatches 2D/3D lines, surfaces, contours, and implicit plots.
  - Implicit plot rendering: interval-arithmetic results rendered with `fill()`; contour-based results use `contour` (equality) vs `contourf` (inequality).
- `TextBackend` — ASCII fallback backend; `show()` raises `ValueError` if more than one series or if series is not `LineOver1DRangeSeries`. Delegates single-expression rendering to `textplot()`.
- `DefaultBackend` — auto-selects `MatplotlibBackend` if matplotlib is available, otherwise `TextBackend`.

### `plot_implicit.py`
Implicit equation/inequality data series; computes raster data via interval arithmetic (rendering handled by backends in `plot.py`).

- `plot_implicit()` — public API; plots relations (Eq, And, Or, inequalities) over 2D region.
  - Variable inference: bare expressions auto-wrapped as `Eq(expr, 0)`; single-variable expressions get a synthetic dummy symbol for the missing axis.
- `ImplicitSeries` — data series for implicit plots; `_get_raster_interval()` recursively subdivides rectangles using interval arithmetic to determine inclusion.

### `textplot.py`
Low-level ASCII art grid renderer (called by `TextBackend`; does not validate input count).

- `textplot(expr, a, b, W=55, H=21)` — evaluates a single expression over [a, b] and prints a text grid.

### `experimental_lambdify.py`
Custom expression-to-function converter for internal plotting use.

- `lambdastr()`, `experimental_lambdify()` — convert sympy expressions to callable functions with namespace translation.
- Caveat: unstable internal API; may be rewritten.

## Interval Arithmetic (`plotting/intervalmath/`)

### `interval_arithmetic.py`
Core interval class for bounded floating-point interval computations.

- `interval` — represents [start, end] with `is_valid` ternary flag (True/False/None for partial validity).
- Supports arithmetic operators (+, -, *, /, **) and ternary comparison operators.

### `lib_interval.py`
Interval-aware math function library and ternary logic operators for implicit plotting.

- Math functions operating on `interval` objects: `Abs`, `exp`, `log`, `sqrt`, `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `sinh`, `cosh`, `tanh`, `acosh`, `asinh`, `atanh`.
- Each function handles domain validation, returning `is_valid=False` outside domain, `is_valid=None` for partial overlap.
- `And(*args)` — ternary conjunction over 2-tuples of truth values (True/False/None); priority: False > None > True. Used to combine comparison results in range-based curve rendering.
- `Or(*args)` — ternary disjunction over 2-tuples; priority: True > None > False.
- `Min`, `Max` — interval-aware min/max returning new intervals.

## Pyglet 3D Plotting (`plotting/pygletplot/`)

### `__init__.py`
Public entry point for pyglet plotting; defines the `PygletPlot` factory function that wraps the class in `plot.py`.

- `PygletPlot(*args, **kwargs)` — factory function whose docstring documents the full user-facing API:
  - Flexible variable interval syntax: `[var, min, max, steps]` with partial specification — `[]` uses all defaults, `[100]` sets only step count, `[-13, 13]` sets only bounds; omitted args filled from coordinate mode defaults.
  - Coordinate mode selection (Cartesian, parametric, polar, cylindrical, spherical); auto-detected from expression/variable count.
  - Calculator-like indexed interface (`p[1] = expr`), per-slot style/color, keyboard controls.

### `plot.py`
`PygletPlot` class implementation for interactive 3D visualization with OpenGL/pyglet.

- `PygletPlot` — top-level plot object; manages plot objects, axes, camera, window, and rendering thread.
- Supports indexed assignment (`p[1] = expr`) for adding/replacing plot functions.

### `plot_mode.py`
Plot mode registry and argument interpretation.

- `PlotMode` — registry class mapping (d_var count, i_var count) to concrete mode classes.
- `_interpret_args()` — classifies raw arguments into expressions, intervals, and options.
- `_find_i_vars()`, `_find_d_vars()` — infer independent/dependent variables from expressions.

### `plot_mode_base.py`
Base class providing shared infrastructure for all pyglet plot modes.

- `PlotModeBase` — common parent for all mode implementations.
- `_get_evaluator()` — tries fast lambda evaluator first; on exception, falls back to sympy substitution evaluator with a warning.
- `_get_sympy_evaluator()`, `_get_lambda_evaluator()` — abstract methods implemented by concrete modes.
- Thread-safe rendering stack: `push_wireframe()`, `push_solid()` with `_draw_lock`.
- `_on_calculate()` — triggers vertex/color vertex computation in background threads.
- Class-level attributes: `i_vars`, `d_vars`, `intervals`, `aliases`, `is_default`.

### `plot_modes.py`
Concrete plot mode implementations for various coordinate systems.

- `Cartesian2D`, `Cartesian3D` — Cartesian curve/surface modes.
- `ParametricCurve2D`, `ParametricCurve3D`, `ParametricSurface3D` — parametric modes.
- `Polar`, `Cylindrical`, `Spherical` — curvilinear coordinate modes.
- Each implements `_get_sympy_evaluator()` and `_get_lambda_evaluator()` for its coordinate transform.

### `plot_interval.py`
Bounded interval representation for pyglet variable ranges.

- `PlotInterval` — stores [variable, min, max, steps] with property accessors and validation.
- `fill_from(b)` — merges defaults from another interval for partial specifications.

### `plot_curve.py`
Curve rendering for 1D pyglet plots.

- `PlotCurve` — calculates and caches vertices for wireframe curve drawing.
  - `_on_calculate_verts()` — evaluates parametric positions; catches `NameError`/`ZeroDivisionError` and stores `None` for failed points.
  - `draw_verts(use_cverts)` — emits OpenGL `GL_LINE_STRIP` segments; breaks the strip at `None` vertices to create visual discontinuities at undefined points.

### `plot_surface.py`
Surface rendering for 2D pyglet plots.

- `PlotSurface` — calculates 2D vertex grid; supports wireframe and solid draw styles.

### `plot_axes.py`
Coordinate axes rendering.

- `PlotAxes` — draws axes with configurable styles (ordinate, frame, box, none); manages labels and ticks.

### `plot_camera.py`
3D camera control.

- `PlotCamera` — handles perspective/orthographic projection, position, rotation, zoom, and preset angles.

### `plot_controller.py`
User input handling.

- `PlotController` — maps keyboard/mouse events to camera actions (rotate, zoom, reset).

### `plot_window.py`
OpenGL window management, rendering loop, and title-bar progress display.

- `PlotWindow` — extends `ManagedWindow`; sets up GL context, coordinates camera and controller.
- `draw()` — main render loop; acquires render lock, iterates plot functions to draw them, and collects vertex/color computation progress in the same pass to avoid locking twice per frame.
- `update_caption()` — formats vertex and color calculation percentages into the window title bar.

### `plot_object.py`
Base renderable object.

- `PlotObject` — base class for all GL-renderable objects; defines `draw()` interface and visibility.

### `plot_rotation.py`
Vector math and rotation utilities.

- `cross()`, `dot()`, `mag()`, `norm()` — basic vector operations.
- `get_spherical_rotatation()` — trackball-style quaternion rotation from mouse input.

### `color_scheme.py`
Color mapping for curves and surfaces.

- `ColorGradient` — interpolates colors across value intervals.
- `ColorScheme` — applies color functions (including lambdified expressions) to vertices.
  - `__call__(x, y, z, u, v)` — evaluates the color function; catches any exception and returns `None` (used as sentinel for failed evaluations).
  - `apply_to_curve(verts, u_set)` — assigns RGB to 1D vertex list; two-pass: compute raw channels + track bounds, then normalize to [0,1] and apply gradient. Skips None vertices (from exceptions or missing verts). Sets `v=None` for single-parameter curves.
  - `apply_to_surface(verts, u_set, v_set)` — same two-pass normalization for 2D vertex grid (u×v mesh); skips None entries.

### `managed_window.py`
Pyglet window lifecycle management.

- `ManagedWindow` — wraps pyglet Window with auto event loop in a separate thread; FPS-limited rendering.

### `util.py`
OpenGL and 3D math utilities.

- `get_model_matrix()`, `get_projection_matrix()`, `get_viewport()` — GL state queries.
- `screen_to_model()`, `model_to_screen()` — coordinate transformations.
- `billboard_matrix()` — orients geometry to face camera.
- `parse_option_string()` — parses keyword arguments from string format.
