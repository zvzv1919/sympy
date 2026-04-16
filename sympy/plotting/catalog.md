# sympy/plotting — Plotting

Symbolic expression plotting with multiple backends (matplotlib, text, pyglet). Supports 2D/3D line plots, parametric plots, surface plots, implicit equation plots, and interactive 3D visualization via pyglet.

## Glossary

- **Data Series**: Objects (`BaseSeries` subclasses) that hold the symbolic expression(s) and range(s) for a single plotted entity and know how to produce numerical point/mesh data.
- **Backend**: A renderer (`BaseBackend` subclass) that takes data series and draws them (matplotlib, text, or pyglet).
- **Adaptive sampling**: Recursive subdivision that adds points where a curve is not approximately collinear, producing smoother plots with fewer total points.
- **Interval arithmetic**: Approximate floating-point interval evaluation used for implicit plot region detection. *Not* a rigorous IEEE-754 implementation — intended only for plotting.

---

## Core Plotting

### `__init__.py`
Public re-exports: `plot`, `plot_parametric`, `plot3d`, `plot3d_parametric_line`, `plot3d_parametric_surface`, `plot_implicit`, `textplot`, `PygletPlot`, `plot_backends`.

### `plot.py`
Central plotting module. Defines the `Plot` container, all data series classes, backends, and the top-level convenience functions.

**Classes — Data Series hierarchy**

- `BaseSeries` — abstract base; flags `is_2Dline`, `is_3Dline`, `is_3Dsurface`, `is_contour`, `is_implicit`, `is_parametric`.
- `Line2DBaseSeries(BaseSeries)` — base for 2D lines; provides `get_segments`, `get_color_array`.
  - `List2DSeries` — line from explicit coordinate lists.
  - `LineOver1DRangeSeries` — `f(x)` over a range; supports adaptive sampling (`get_segments` recurses to depth 12).
  - `Parametric2DLineSeries` — `(x(t), y(t))` parametric curve; adaptive sampling.
- `Line3DBaseSeries(Line2DBaseSeries)` — base for 3D lines.
  - `Parametric3DLineSeries` — `(x(t), y(t), z(t))` parametric 3D curve.
- `SurfaceBaseSeries(BaseSeries)` — base for 3D surfaces; provides `get_color_array`.
  - `SurfaceOver2DRangeSeries` — `f(x,y)` surface over a 2D range.
  - `ParametricSurfaceSeries` — `(x(u,v), y(u,v), z(u,v))` parametric surface.
- `ContourSeries(BaseSeries)` — contour plot data (currently unused by the public API).

**Classes — Backends**

- `BaseBackend` — minimal interface (`show`, `save`, `close`).
- `MatplotlibBackend(BaseBackend)` — full matplotlib renderer.
  - `process_series()` iterates data series and dispatches to matplotlib line collections, contours, or 3D surfaces.
  - Handles global options (title, axis labels, scale, limits, legend, margin).
- `TextBackend(BaseBackend)` — delegates to `textplot`; supports only a single `LineOver1DRangeSeries`.
- `DefaultBackend(BaseBackend)` — factory: returns `MatplotlibBackend` if matplotlib is available, otherwise `TextBackend`.

**Top-level functions**

- `plot(*args, **kwargs)` — plot one or more univariate expressions; returns `Plot`.
- `plot_parametric(*args, **kwargs)` — 2D parametric plot.
- `plot3d(*args, **kwargs)` — 3D surface plot.
- `plot3d_parametric_line(*args, **kwargs)` — 3D parametric line.
- `plot3d_parametric_surface(*args, **kwargs)` — 3D parametric surface.
- `check_arguments(args, expr_len, nb_of_free_symbols)` — normalizes mixed argument forms `(expr, range)` into uniform tuples.

**Utilities**

- `centers_of_segments(array)`, `centers_of_faces(array)` — midpoint helpers for color arrays.
- `flat(x, y, z, eps=1e-3)` — collinearity test for adaptive sampling.
- `_matplotlib_list(interval_list)` — converts bounding rectangles to matplotlib `fill` lists.
- `unset_show()` — global flag to suppress display during tests.

**Caveats**: Performance is deliberately sacrificed for code simplicity. A new backend instance is created on every `show()` call.

### `plot_implicit.py`
Implicit equation / inequality plotting via interval arithmetic with contour-mesh fallback.

- `ImplicitSeries(BaseSeries)` — data series for implicit plots.
  - `get_raster()` — tries interval-arithmetic adaptive meshing; falls back to uniform mesh grid.
  - `_get_raster_interval(func)` — recursive interval subdivision (depth controlled by `depth` parameter, max 4 extra levels). Returns a list of bounding rectangles to fill.
  - `_get_meshes_grid()` — uniform numpy mesh; uses `contour` for equalities, `contourf` for inequalities.
- `plot_implicit(expr, x_var, y_var, **kwargs)` — public entry point.
  - Supports `Eq`, relational operators, and boolean combinations (`And`, `Or`).
  - Auto-detects free symbols and default ranges `(-5, 5)`.

### `textplot.py`
ASCII art plotting of a single-variable expression to the terminal.

- `textplot(expr, a, b, W=55, H=18)` — evaluates `expr` over `[a, b]` at `W` points, normalizes to `H` rows, and prints characters `.`, `/`, `\\` with y-axis labels.

### `experimental_lambdify.py`
Internal lambdification engine for the plotting module. Converts sympy expressions to callable Python functions with translation to numpy, cmath, math, or intervalmath namespaces.

- `vectorized_lambdify(args, expr)` — callable class; returns numpy-masked real arrays. Falls back through: numpy → python cmath (vectorized) → evalf-wrapped.
- `lambdify(args, expr)` — callable class; returns a single real float or `None`. Used by adaptive sampling. Falls back through: cmath → python math → evalf-wrapped.
- `experimental_lambdify(*args, **kwargs)` — factory returning a `Lambdifier` instance.
- `Lambdifier` — the core engine:
  - `str2tree(exprstr)` / `tree2str(tree)` — parse expression string into a `(func, args)` tree.
  - `tree2str_translate(tree)` — apply function/string translation dictionaries.
  - `translate_func`, `translate_str` — look up replacements (e.g. `sin` → `np.sin`).
  - `get_dict_str()`, `get_dict_fun()` — build translation dicts from class-level tables for numpy, math, cmath, intervalmath.
  - `sympy_expression_namespace(expr)` — crawl expression tree to build a namespace dict.
  - `sympy_atoms_namespace(expr)` — collect `Symbol`/`NumberSymbol` atoms for the namespace.

**Caveats**: Explicitly unstable / internal. Does not use `lambdarepr` and may be rewritten. Breaks if symbols collide with function names (e.g. `Symbol('sin')`).

---

## Interval Arithmetic (`intervalmath/`)

Sub-package providing approximate interval arithmetic for implicit plotting. Uses numpy floats (no IEEE-754 rounding guarantees). Not suitable for general-purpose interval math — use mpmath instead.

### `intervalmath/__init__.py`
Re-exports `interval` and all functions from `lib_interval`.

### `intervalmath/interval_arithmetic.py`
Defines the `interval` class.

- `interval(start, end, is_valid=True)` — represents a closed floating-point interval.
  - `is_valid` tracks domain validity: `True` (valid), `False` (outside domain), `None` (partially valid / discontinuous).
  - Comparison operators (`<`, `>`, `==`, `!=`, `<=`, `>=`) return `(result, validity)` 2-tuples with three-valued logic (`True`/`False`/`None`).
  - Arithmetic: `+`, `-`, `*`, `/`, `**` with proper endpoint combinations.
  - `__pow__` delegates to `_pow_int` (integer) or `_pow_float` (fractional) or `exp(other * log(self))` for interval exponents.
  - Properties: `mid`, `width`, `__contains__`.
- `_pow_float(inter, power)` — handles fractional powers with domain checks (even numerator, even denominator).
- `_pow_int(inter, power)` — handles integer powers (odd vs even exponent sign considerations).

### `intervalmath/lib_interval.py`
Library of interval-aware mathematical functions, all accepting `int`/`float`/`interval` and returning `interval`.

**Monotonic functions**: `exp`, `log`, `log10`, `atan`, `sqrt`, `sinh`, `tanh`, `asinh`.

**Periodic / non-monotonic functions** (with quadrant-aware min/max):
- `sin(x)`, `cos(x)` — divide range into quarter-period buckets to find extrema.
- `tan(x)` — defined as `sin(x) / cos(x)`.
- `cosh(x)` — handles sign-straddling intervals.

**Inverse trig / hyperbolic** (with domain checks): `asin`, `acos`, `acosh`, `atanh`.

**Rounding**: `ceil`, `floor` — return `is_valid=None` when discontinuous over the interval.

**Extrema**: `imin(*args)`, `imax(*args)`.

**Three-valued logic**: `And(*args)`, `Or(*args)` — reduce 2-tuples of `(value, validity)` with three-valued `And`/`Or` semantics.

`Abs(x)` — interval absolute value.

---

## Pyglet Plotting (`pygletplot/`)

Legacy interactive 3D plotting backend using the pyglet OpenGL library. Runs the render loop in a separate thread. Supports Cartesian, parametric, polar, cylindrical, and spherical coordinate modes with customizable color schemes.

### `pygletplot/__init__.py`
Exports `PygletPlot` factory function (wraps `plot.PygletPlot` from within the sub-package). Catches import errors if pyglet is unavailable.

### `pygletplot/plot.py`
Top-level pyglet plot container.

- `PygletPlot` — main class; manages a function dictionary, render lock, axes, and a `PlotWindow`.
  - `__setitem__(i, args)` — parses arguments via `PlotMode` and stores the resulting mode.
  - `show()`, `close()`, `saveimage()`, `clear()`, `append()`.
  - `wait_for_calculations()` — blocks until all vertex/color calculations finish.
  - `adjust_all_bounds()` — recomputes axis bounding box from all functions.
- `ScreenShot` — captures the GL framebuffer to a PNG via PIL.

### `pygletplot/plot_mode.py`
Mode registry and argument interpretation.

- `PlotMode(PlotObject)` — grandparent class for all plot modes. Acts as a factory via `__new__`: parses arguments, infers independent/dependent variable counts, and looks up the correct registered mode subclass.
  - `_register()` — class method; registers a mode under its aliases in the `_mode_map`.
  - `_get_mode(mode_arg, i, d)` — resolves a mode string/class to a concrete subclass.
  - `_interpret_args(args)` — splits args into function expressions and `PlotInterval`s.
  - `_find_i_vars`, `_fill_i_vars`, `_fill_intervals` — variable and interval resolution.

### `pygletplot/plot_mode_base.py`
Base implementation for concrete plot modes.

- `PlotModeBase(PlotMode)` — provides threading, display-list management, style/color properties, and the `draw()` method.
  - Manages wireframe and solid render stacks (GL display lists).
  - `_on_calculate()` spawns a thread that calls `_calculate_verts()` then `_calculate_cverts()`.
  - `_get_evaluator()` — tries lambda evaluator first, falls back to sympy `.subs()`.
  - Style property: `'wireframe'`, `'solid'`, `'both'`.
  - Color property: wraps value in `ColorScheme`.

### `pygletplot/plot_modes.py`
Concrete coordinate-mode implementations. Each defines `i_vars`, `d_vars`, default `intervals`, `aliases`, and evaluator methods.

- `Cartesian2D(PlotCurve)` — `y = f(x)`, alias `'cartesian'`, default.
- `Cartesian3D(PlotSurface)` — `z = f(x,y)`, alias `'cartesian'`, default.
- `ParametricCurve2D(PlotCurve)` — `(x(t), y(t))`, alias `'parametric'`, default.
- `ParametricCurve3D(PlotCurve)` — `(x(t), y(t), z(t))`, alias `'parametric'`, default.
- `ParametricSurface(PlotSurface)` — `(x(u,v), y(u,v), z(u,v))`, alias `'parametric'`, default.
- `Polar(PlotCurve)` — `r = f(t)`, alias `'polar'`.
- `Cylindrical(PlotSurface)` — `r = f(θ, h)`, alias `'cylindrical'`/`'polar'`.
- `Spherical(PlotSurface)` — `r = f(θ, φ)`, alias `'spherical'`.

All modes are registered at module load time via `_register()`.

### `pygletplot/plot_curve.py`
- `PlotCurve(PlotModeBase)` — rendering for 1D curves (wireframe only). Evaluates vertices over `t_interval`, updates bounding box, and builds GL `LINE_STRIP` display lists.

### `pygletplot/plot_surface.py`
- `PlotSurface(PlotModeBase)` — rendering for 2D surfaces. Evaluates a `u × v` vertex grid and builds GL `QUAD_STRIP` display lists for both wireframe and solid styles.

### `pygletplot/color_scheme.py`
Color mapping for pyglet plots.

- `ColorGradient` — multi-stop linear gradient; maps `(r, g, b)` floats through interpolated color stops.
- `ColorScheme` — accepts a callable, a string name, or symbolic expressions + gradient tuples.
  - `apply_to_curve(verts, u_set, ...)` — computes per-vertex colors for curves.
  - `apply_to_surface(verts, u_set, v_set, ...)` — computes per-vertex colors for surfaces.
  - Built-in schemes: `'rainbow'`, `'zfade'`, `'zfade3'`, `'zfade4'`.

### `pygletplot/plot_interval.py`
- `PlotInterval` — represents `[var, min, max, steps]` for a single independent variable. Properties validate types. Provides `vrange()` (sympy numbers), `vrange2()` (consecutive pairs), and `frange()` (floats).

### `pygletplot/plot_object.py`
- `PlotObject` — trivial base class with a `visible` flag and a `draw()` stub.

### `pygletplot/managed_window.py`
- `ManagedWindow(pyglet.Window)` — pyglet window that runs its event loop in a separate thread. Provides overridable `setup()`, `update(dt)`, `draw()` hooks. FPS capped at 30.

### `pygletplot/plot_window.py`
- `PlotWindow(ManagedWindow)` — the actual plot window. Sets up OpenGL state (depth test, line smoothing, blending, antialiasing), creates `PlotCamera` and `PlotController`, and renders all functions + plot objects each frame. Updates window caption with calculation progress.

### `pygletplot/plot_camera.py`
- `PlotCamera` — manages projection (perspective or pseudo-ortho) and modelview transforms. Supports rotation presets (`xy`, `xz`, `yz`, `perspective`), spherical rotation, euler rotation, zoom, and mouse-translate.

### `pygletplot/plot_controller.py`
- `PlotController` — handles keyboard and mouse input. Maps keys to rotation, zoom, camera reset, preset, axes toggle, and screenshot actions. Supports sensitivity modifier (shift key) and 2D/3D aware behavior.

### `pygletplot/plot_rotation.py`
Spherical rotation math for trackball-style mouse interaction.

- `get_sphere_mapping(x, y, width, height)` — maps screen coordinates to a unit sphere.
- `get_spherical_rotatation(p1, p2, width, height, theta_multiplier)` — computes a GL rotation matrix from two screen points.
- Helpers: `cross`, `dot`, `mag`, `norm`.

### `pygletplot/util.py`
OpenGL and math utilities for pyglet plotting.

- GL matrix queries: `get_model_matrix`, `get_projection_matrix`, `get_viewport`.
- Coordinate transforms: `screen_to_model`, `model_to_screen`.
- Direction vectors: `get_direction_vectors`, `get_view_direction_vectors`, `get_basis_vectors`.
- `billboard_matrix()` — removes rotation so primitives always face the viewer.
- Bounds: `create_bounds`, `update_bounds`.
- Interpolation: `interpolate`, `rinterpolate`, `interpolate_color`.
- `scale_value`, `scale_value_list` — normalize to `[0, 1]`.
- `strided_range(r_min, r_max, stride)` — generate tick positions.
- `parse_option_string(s)` — parse `"key=value; ..."` option strings.
- `dot_product`, `vec_sub`, `vec_mag`.

### `pygletplot/plot_axes.py`
Axis rendering for pyglet plots.

- `PlotAxes(PlotObject)` — top-level axes manager; supports styles `'none'`, `'frame'`, `'ordinate'`. Manages bounding box, tick calculation, stride, and label font.
- `PlotAxesBase(PlotObject)` — base renderer; provides `draw_text`, `draw_line`.
- `PlotAxesOrdinate(PlotAxesBase)` — draws ordinate-style axes with tick marks and labels.
- `PlotAxesFrame(PlotAxesBase)` — frame/box style (stub, `draw_axis` not implemented).
