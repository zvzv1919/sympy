# sympy/plotting — Catalog

> Part of [SymPy](../catalog.md). 2-D and 3-D plotting with backends for matplotlib, pyglet, and text-based output.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports the public plotting API: `plot`, `plot_parametric`, `plot3d`, `plot3d_parametric_line`, `plot3d_parametric_surface`, `plot_implicit`, `textplot`, and `PygletPlot`. |
| `plot.py` | Core plotting module defining the `Plot` class, data series classes (`LineOver1DRangeSeries`, `Parametric2DLineSeries`, `SurfaceOver2DRangeSeries`, etc.), backend wrappers for matplotlib and text output, the public convenience functions `plot`, `plot_parametric`, `plot3d`, `plot3d_parametric_line`, and `plot3d_parametric_surface`, and the `check_arguments` helper that validates and normalizes arguments into (expressions, ranges) tuples. `check_arguments` assigns a default fallback interval of (-10, 10) when no range is provided, and has an explicit limitation where three-element expression groups cannot be distinguished from range tuples. |
| `plot_implicit.py` | Implicit plotting using interval arithmetic (with a fallback adaptive sampling algorithm for rendering). Defines `ImplicitSeries` and the `plot_implicit` function for rendering equations, inequalities, and boolean combinations of expressions. Does not contain general argument validation or default-range assignment logic — see `plot.py`'s `check_arguments` for that. |
| `textplot.py` | Provides `textplot`, a function that renders a crude ASCII-art plot of a single-variable SymPy expression over a given interval to the terminal. |
| `experimental_lambdify.py` | Internal lambdify variant used by the plotting module to convert SymPy expressions into callable numerical functions. Translates expression strings to use math/numpy/mpmath and handles edge cases that the standard `lambdify` does not. |
| `intervalmath/__init__.py` | Package init for the interval math subpackage; re-exports the `interval` class and all interval-aware math functions (sin, cos, exp, log, sqrt, etc.). |
| `intervalmath/interval_arithmetic.py` | Defines the `interval` class representing a floating-point interval with a validity flag. Implements arithmetic operators and comparisons used by `plot_implicit` for adaptive region subdivision. |
| `intervalmath/lib_interval.py` | Implements interval-aware versions of standard math functions (exp, log, sin, cos, tan, sqrt, Abs, floor, ceil, etc.) and boolean operations (And, Or) over `interval` objects, using numpy for speed. |
| `pygletplot/__init__.py` | Package init for the pyglet-based plotting backend; defines and exports the `PygletPlot` factory function. Documents the auto-detection logic that infers curve vs. surface rendering from the number of free variables (one = curve, two = surface) and determines the coordinate mode from the number of supplied expressions (one = rectilinear/Cartesian, two or three = parameter-driven). Also documents flexible interval syntax, curvilinear coordinate support, the indexed calculator-like interface, and keyboard controls. |
| `pygletplot/plot.py` | Implements `PygletPlot`, the main class for the pyglet backend. Manages a collection of plot functions, coordinates rendering via `PlotWindow`, and supports interactive features like color schemes, coordinate modes, and image saving. |
| `pygletplot/color_scheme.py` | Defines `ColorGradient` and `ColorScheme` classes for mapping surface/curve vertex data to RGB colors in the pyglet backend. Supports user-defined color functions, named presets, and gradient interpolation. |
| `pygletplot/managed_window.py` | Provides `ManagedWindow`, a pyglet `Window` subclass that runs its event loop in a separate thread, serving as the base class for `PlotWindow`. |
| `pygletplot/plot_axes.py` | Implements `PlotAxes` and helper classes (`PlotAxesOrdinate`, `PlotAxesFrame`) for rendering labeled coordinate axes with tick marks in the pyglet 3-D plot window. |
| `pygletplot/plot_camera.py` | Defines `PlotCamera`, which manages the OpenGL camera (projection, rotation, zoom, panning) for the pyglet plotting window, supporting both perspective and orthographic modes. |
| `pygletplot/plot_controller.py` | Implements `PlotController`, which translates keyboard and mouse input into camera rotation, zoom, and other interactive actions in the pyglet plot window. |
| `pygletplot/plot_curve.py` | Defines `PlotCurve`, a `PlotModeBase` subclass that calculates and renders 1-D parametric curves (wireframe line strips) in the pyglet backend. |
| `pygletplot/plot_interval.py` | Defines `PlotInterval`, a helper class representing a named variable range with step count, used to generate evaluation grids for pyglet plot modes. |
| `pygletplot/plot_mode.py` | Defines `PlotMode`, the grandparent class for all pyglet plot modes. Handles mode registration, lookup by alias or variable count, argument parsing, and coordinate-mode selection. |
| `pygletplot/plot_mode_base.py` | Defines `PlotModeBase`, the intended parent class for concrete plot modes. Provides threaded vertex/color calculation, display-list management, evaluator setup, and the base `draw` implementation. |
| `pygletplot/plot_modes.py` | Defines concrete plot modes for the pyglet backend: `Cartesian2D`, `Cartesian3D`, `ParametricCurve2D`, `ParametricCurve3D`, `ParametricSurface`, `Polar`, `Cylindrical`, and `Spherical`. |
| `pygletplot/plot_object.py` | Defines `PlotObject`, a minimal base class for anything that can be drawn in the pyglet plot window, providing a `visible` flag and a `draw` method stub. |
| `pygletplot/plot_rotation.py` | Utility functions for computing 3-D rotation matrices from sphere-mapped screen coordinates, used for interactive trackball-style rotation in the pyglet backend. |
| `pygletplot/plot_surface.py` | Defines `PlotSurface`, a `PlotModeBase` subclass that calculates and renders 2-D parametric surfaces (wireframe and solid quad strips) in the pyglet backend. |
| `pygletplot/plot_window.py` | Implements `PlotWindow`, the `ManagedWindow` subclass that sets up OpenGL state, coordinates the camera and controller, and drives the per-frame drawing loop for pyglet plots. |
| `pygletplot/util.py` | OpenGL and math utility functions for the pyglet backend: model/projection matrix retrieval, screen-to-model coordinate conversion, interpolation, billboard matrices, vector operations, and option-string parsing. |
