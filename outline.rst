.. include:: <s5defs.txt>

========================================
``schedview`` Archictecture and overview
========================================

:URL:    https://github.com/lsst/rtn-037/outline.rst
:Author: Eric Neilsen
:Date:   $Date: 2022-08-22 $

.. container:: handout

   This is an outline and presentation of RTN-037.
   Both the text and the architecure described are in the "prototype" phase.

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

* At the observatory.
   - Before observing (pre-night plan or briefing).
   - During observing (part of FAFF?).
   - After observing (night summary or night log).
* On the Rubin Science Platform.
* In parametrized notebooks.
* On a developer or scientists laptop.

``schedview``'s role
====================

* ``schedview`` is *not* high level code into which other functionality fits.
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
* Parameterize all plot elements.
   * Axes, data sources, ticks, tick labels, tick label formatters, glyphs, axes, buttons, sliders, callbacks, full plots, html text, etc.
   * Models reference other models.
* A ``bokeh`` "document" encapsulates all models for a given visualization.

``bokeh`` code
==============

* The ``bokeh`` ``python`` module.
   * Tools for creating ``python`` objects representing ``bokeh`` models, and serealizing them to ``json``.
   * Tools for creating ``html`` representations (with embedded ``javascript`` and ``json``) and providing serving them as ``jupyter`` cells or stand-alone web applications.
* The ``bokeh`` ``javascript`` library.
   * Creates visualizations themselves.
   * Handles ``javascript`` callbacks.
   * Passes ``python`` callbacks to the server (if possible).

Visualization elements
======================

:Collection: Gets data from external sources.
:Munging: Reformats or pre-processes data.
:Computation: Calculation beyond filtering and reformatting (e.g. simulation).
:Plotting: Generates a ``bokeh.models.Figure`` instance.
:Application:
   * Adds ``python`` callbacks to plot elements.
   * Implements a web server to serve the visualization and handle ``python`` callbacks.

