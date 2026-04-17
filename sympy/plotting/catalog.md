# Plotting Module Catalog

## Core Plotting (`plotting/`)

### `plot.py`
Main plotting API and data series definitions for matplotlib-based 2D/3D plots.

- `Plot` — container for data series; dispatches rendering to backends (matplotlib, text, default).
- `check_arguments(args, expr_len, nb_of_free_symbols)` — argument parser that groups flat or tuple-wrapped expressions into plot series; handles ambiguity when expr_len == nb_of_free_symbols (e.g., 3D parametric lines where 3 exprs can't be distinguished from a range tuple).
- Public API: `plot()`, `plot_parametric()`, `plot3d()`, `plot3d_parametric_line()`, `plot3d_parametric_surface()`.
- `LineOver1DRangeSeries` — evaluates single expression over 1D range; adaptive subdivision with collinearity check.
- `Parametric2DLineSeries` — 2D parametric curve series; `get_segments()` uses recursive adaptive subdivision with complex-value handling (samples 10 intermediate points when both endpoints are non-real).
- `Parametric3DLineSeries` — 3D parametric curve from three expressions and a range.
- `SurfaceOver2DRangeSeries`, `ParametricSurfaceSeries` — 3D surface data series.
- `Line2DBaseSeries`, `Line3DBaseSeries` — base classes for line series with common range/label logic.
- Backend classes: `MatplotlibBackend`, `TextBackend`, `DefaultBackend`.

### `plot_implicit.py`
Implicit equation/inequality plotter using interval arithmetic rasterization.

- `plot_implicit()` — public API; plots relations (Eq, And, Or, inequalities) over 2D region.
- `ImplicitSeries` — data series for implicit plots; `_get_raster_interval()` recursively subdivides rectangles using interval arithmetic to determine inclusion.

### `textplot.py`
ASCII art plotting for terminal output.

- `textplot(expr, a, b, W=55, H=21)` — evaluates expression and renders as text grid.

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

### `plot.py`
Main `PygletPlot` class for interactive 3D visualization with OpenGL/pyglet.

- `PygletPlot` — top-level plot object; manages plot objects, axes, camera, window, and rendering thread.
- Flexible variable interval syntax: `[var, min, max, steps]` with partial specification — e.g., `[100]` sets only steps, `[-13, 13]` sets only bounds, `[]` uses all defaults from the coordinate mode.
- Coordinate modes: Cartesian, parametric, polar, cylindrical, spherical; auto-detected from argument count.
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
OpenGL window management.

- `PlotWindow` — extends `ManagedWindow`; sets up GL context, coordinates camera and controller.

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

### `managed_window.py`
Pyglet window lifecycle management.

- `ManagedWindow` — wraps pyglet Window with auto event loop in a separate thread; FPS-limited rendering.

### `util.py`
OpenGL and 3D math utilities.

- `get_model_matrix()`, `get_projection_matrix()`, `get_viewport()` — GL state queries.
- `screen_to_model()`, `model_to_screen()` — coordinate transformations.
- `billboard_matrix()` — orients geometry to face camera.
- `parse_option_string()` — parses keyword arguments from string format.
