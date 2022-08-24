.. include:: <s5defs.txt>

========================================
``schedview`` Archictecture and Overview
========================================

:URL:    https://github.com/lsst/rtn-037/blob/main/outline.rst
:Author: Eric Neilsen
:Date:   $Date: 2022-08-22 $

.. container:: handout

   This is an outline and presentation of RTN-037.
   Both the text and the architecure described are in the "prototype" phase.

.. contents::
   :class: handout

Purpose
=======

* ``schedview`` is a "home" for code filling requirements described in RTN-016.
* Provides visualizations for:
   * Predicting Rubin Observatory/LSST scheduler behavior.
   * Identifying problems with the scheduler (before and after use).
   * Current survey progress.
   * Extrapolated survey progress.

Operational Contexts
====================

* At the observatory:
   - Before observing (pre-night plan or briefing).
   - During observing (part of FAFF?).
   - After observing (night summary or night log).
* On the Rubin Science Platform.
* In parametrized notebooks.
* On a developer or scientist's laptop.

Design considerations
=====================

* Maximize code re-use across contexts.
   * Code that is the same across contexts should live in different submodules
     from code that might vary across contexts.
* Add levels of indirection or abstraction only when necessary.
* Maximize flexibility.
   * Do not wrap elements of the ``bokeh`` API in other functions that limit
     access to ``bokeh`` functionality and parameters, or require maintenance
     to add support for use of new ``bokeh`` parameters or functionality.
* Conceptual cohesion.

``schedview``'s role
====================

* ``schedview`` is *not* high-level code into which other functionality fits.
* ``schedivew`` *is*:
   * A ``git`` repository to hold code for scheduler-related visualizations.
      * More operational than visualizations in MAF,
          * but there will probably be overlap. 
   * A recommended architectural pattern & set of interfaces.
      * very much a prototype!
   * A set of utilities that might be useful for multiple visualizations.

``bokeh`` "models"
==================

* Abstractions represented in ``python`` objects, ``json`` serializations, and ``javascript`` objects.
* Parametrize all plot elements.
   * Axes, data sources, ticks, tick labels, tick label formatters, glyphs, axes, buttons, sliders, callbacks, full plots, html text, etc.
   * Models reference other models.
* A ``bokeh`` "document" encapsulates all models for a given visualization.

``bokeh`` code
==============

* The ``bokeh`` ``python`` module.
   * Tools for creating ``python`` objects representing ``bokeh`` models, and serializing them to ``json``.
   * Tools for creating ``html`` representations (with embedded ``javascript`` and ``json``) and providing serving them as ``jupyter`` cells or stand-alone web applications.
* The ``bokeh`` ``javascript`` library.
   * Creates visualizations themselves.
   * Handles ``javascript`` callbacks.
   * Passes ``python`` callbacks to the server (if possible).

``schedviews`` visualization elements
=====================================

:Collection: Gets data from external sources.
:Munging: Reformats or pre-processes data.
:Computation: Calculation beyond filtering and reformatting (e.g. simulation).
:Plotting: Generates a ``bokeh.models.Figure`` instance.
:Application:
   * Adds ``python`` callbacks to plot elements.
   * Implements a web server to serve the visualization and handle ``python`` callbacks.

Customizing for different contexts
==================================

* A given visualization may have multiple implementations of its collection
  submodule, if different contexts have access to underlying data following
  different APIs.
* If the access in different contexts result in significantly different python
  data structures, different contexts might require different munge steps, but
  I expect this to be less common.
* Computation and plotting modules should be the same across all contexts.
* The base ``bokeh`` module can display the ``bokeh.models.Figure`` produced by
  the plotting step in ``jupyter`` directly, or export it to an html file:
  separate code for displaying the visualization is only needed for the ``bokeh``
  stand-alone app.

``schedview`` submodules
========================

:Collection: ``schedview.collect``
:Munging: ``schedview.munge``
:Computation: (will be) ``schedview.compute``
:Plotting: ``schedview.plot``
:Application: ``schedview.app``

Code that might be reused by several visualizations currently "lives" in the
submodule it will be used by.


.. container:: handout

   Maybe the munge module should be dropped, and it's functionality
   absorbed into either collection or computation.

Example: ``sched_maps``
=======================

* Shows what the scheduler will do at a given time.
   * Currently: if no observing happens between "now" and that time.
   * Planned: if simulated visits happen as simulated.
* Maps and tables of basis functions and rewards used by the scheduler,
   * for an adjustable time,
   * in a variety of projections, with adjustable parameters.

``sched_maps`` submodules
=========================

:``schedview.collect.scheduler_pickle``:
    Loads an instance of the ``rubin_sim`` scheduler from a pickle file.
:``schedview.plot.SphereMap``:
    Class to generate ``bokeh`` models for display of healpix sky maps.
:``schedview.plot.scheduler``:
    Defines ``SchedulerDisplay``, a class that creates ``bokeh`` models for the scheduler visualization,
    including callbacks defined in javascript and an API for modifying the models.
:``schedview.app.sched_maps``:
    Defines ``SchedulerDisplayApp``, which adds app specific UI elements and a
    top-level driver to start the server.

"Missing" ``sched_maps`` submodules
===================================

:``schedview.munge.??``:
    The ``scheduler`` object has methods that can provide data in a format
    ``bokeh`` data source constructors can accept as arguments, so there is
    nothing for a munge step to do in this use case.
:``schedview.compute.simulate``:
    (planned) will run simulations to a given time.

Near-term plan
==============

* Test ``sched_maps`` app during observing.
* Incorporate executions of scheduler simulations in ``sched_maps``.
* Write some initial API guidelines for interfaces between elements.
* Add more visualizations.

