:tocdepth: 1

.. sectnum::

.. Metadata such as the title, authors, and description are set in metadata.yaml

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is a work-in-progress.**

Abstract
========

Tools for monitoring and exploring scheduler behaviour and survey progress are an important element of Rubin Observatory infrastructure.
A variety of users will require these plots, and require access to them in a variety of contexts and environments.
Examples include telescope operators monitoring the scheduler for problems during observing, managers preparing reports for funding agencies, and scientists from science working groups reviewing the effectiveness of survey strategy as implemented.
This document describes an architecture for development of scheduler and progress monitoring examination and visualization.
The architecutre is intended to avoid duplication of develpoment effort by making it easy to use use common code across different contexts.
Such contexts include both the "First-look Analysis and Feedback Functionality" (FAFF) infrastructure being developed for the observatory and the Rubin Science Platform jupyter notebooks being developed for users and developers.

Introduction
============

A variety of users will require tools for monitoring scheduler behaviour and survey progress.
RTN-016 provides a list of examples of use cases and visualizations.
The users, contexts of use, and details of the visulazations listed in RTN-016 are diverse, and not exhaustive: it is expected that many additional uses and visualizations will be discovered and the survey progresses.
Some users will have limited software development skills, or have limited familiarity with the details of the scheduler or observing system, while others will be experts. For some use cases, the full flexibility of general analyses tools (such as python running in an ad-hoc jupyter notebook) will be required, for example when a developer is debugging a problem.
In other cases, users will need monitoring visualizations that can alert them to problems while requiring minimal attention or interaction.
These visualizations well be needed in the context of observatory operations as part of the "First-look Analysis and Feedback Functionality" (FAFF) infrastructure (described in SITCOMNT-025), as modules to be used in notebooks running on the Rubin Science Platform notebook aspect, as elements of RSP parameterized notebooks (SQR-062), and perhaps in other contexts as well.

Operational Contexts
====================

FAFF
^^^^

First-look Analysis and Feedback Functionalty (FAFF) breakout group is addressing the question of how project will provide on-the-fly analysis and display of telemetry and other data at the observatory.
The group has catalogued existing resources, and identified where they need to be augmented, or where new resources need to be constructed, and made recommendations about what resources should be used (SITCOMTN-025).
The group is currently defining how users will interact with existing metrics, analyses, and other artifacts, developing a set of use cases and metrics that need to be scheduled for implementation, creating a corresponding task list and schedule, and preparing user-level training (SITCOMTN-030).
The visualization tools within the scope of this document must be able to operate within the context of the resources and interfaces described by the FAFF breakout group.
SITCOMTN-025 recommends the creation of visualizations using Bokeh applications, to be incorporated into the observatory displays that are provided by the LSST Observatory Visualization Environement (LOVE).

Rubin Science Platform's Notebook View
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Rubin Science Platform (RSP) has a "notebook view" that provides a jupyterhub environment.
RSP notebooks provide access to Rubin Observatory software and many data products, but RSP instances not running at the observatory do not provide access to all of the data sources available at the observatory.

Parametrized Notebooks
^^^^^^^^^^^^^^^^^^^^^^

Instead of being developed ad-hoc, standard jupyter notebooks can be published with customizable parameters to implement live dashboards and reports (SQR-062, SITCOMTN-025).
Progress and scheduler visualizations should support inclusion in these reports. 

Local
^^^^^

Developers will want to use visualization tools locally, for example on their own laptops.
Code for this context may also be useful even in the RSP notebook. For example, code for a "local" context will need to be able to used data stored in arbitrary local files.
It may be usefull to bypass the production data provided in the RSP and use specially crafted local files for testing.

Resources
=========

Bokeh
^^^^^

Bokeh consists of two major components: 

The ``BokehJS`` ``javascript`` library
  for displaying a (possibly interactive) visualization specified by a collection of objects serialized in a ``json`` file.
The ``bokeh`` ``python`` module
  provides tools for creating and manipulating plot elements and collecting them into a ``bokeh.document`` serializing them in ``json`` that can be used by ``BokehJS``.
  The module also includes tools for and generating html, jupyter cells, or full applications that use ``BokehJS`` to display the modeled visualizations.

The key concept in understanding ``bokeh`` is that of a model.
Each ``bokeh`` model has three representations of interest: an instance of a ``python`` class, ``json`` that can be included in a ``bokeh`` document, and an instance of of ``javascript`` object. A few representative examples of bokeh models are:

- **Sources of data**, for example, ``bokeh.models.ColumnDataSource`` is a table of data which can be created from python dictionaries or ``pandas.DataFrames`` using the ``bokeh`` python module.
- **Axes**
- **Glyphs**, for example markers of various shapes, lines, bars, wedges, and other markers that might appear on a plot.
- **Annotations**
- **Scales**
- **Selections**, specifying elements users can use to interactively select data on a plot.
- **Layouts**, specifying how different plot elements should be arranged.
- **Callbacks**, which map events to calls of either ``javascript`` code (embedded within the callback objects themselves) or ``python`` code (if the ``BokehJS`` library can contact the appropriate server run built using the ``bokeh`` python module.)
- **Controls** such as sliders, buttons, and text input boxes.
- **Plots** combine other models into a unified whole in top-level layout.

The ``bokeh`` python module includes both a low-level API, which supports direct maniuplation of these models, and a high-level API, which resembles the plotting interfaces of other python plotting packages such as ``matplotlib``, and requires less understanding of the underlying architecture.

In the simplest use, a developer uses the ``bokeh`` python module to build the models needed by desired plot, and either exports the result to an ``html`` file or displays it in a ``jupyter`` cell.
The ``html`` (either exported to its own file or displayed in the ``jupyter`` cell) contains everything required for the plot, including the data itself and the javascript code for the javascript callbacks: interactive elements implemented using javascript callbacks are fully functional when loaded from the static html files.

If bokeh is being used within a jupyter notebook, the user can continue to modify of the ``python`` objects representing the ``bokeh`` models, and the changes can be "emmitted" to the ``javascript`` used by the plot, modifying the ``javascript`` objects and corresponding visualization accordingly.

If the web server built into the ``bokeh`` python module is being used to serve the plot's html, the python object representing the bokeh models can (optionally) be updated automatically when the javascript objects in the user's browser are update (e.g. by the user hitting a button, entering text in a text box, or triggering a javascript callback). Furthermore, callbacks written in python can be triggered, updating the python reperentation of the models and pushing the updates to the javascript represenations in the user's browser.

MAF
^^^

Many of the metrics to be visualized are already implemented in MAF using matplotlib.
When they become important to implement in this framework, they should use the MAF code in ``rubin_sim`` rather than implement new code.
For example, there should be a standard way for maps from metrics calculated on a healpix spatial slicer to be converted to bokeh DataSource objects, so the same software for visualizing maps can be used for arbitrary maps from MAF.

Note MAF metrics often take to long to be usefully included in dynamic callbacks, but this will not be necessary for many uses.


Components
==========

Rationale
^^^^^^^^^

Many of the different visualizations to be supported by ``schedview`` may have little actual code in common, beyond what is already contained in various supporting modules such as ``bokeh`` and ``rubin_sim``.
There are, however, a few reasons why grouping the different visualizations in the same framework makes sense.

1. Some types of plots are specific to astronomy, and the ``bokeh`` high-level API does not support all of the redundant code between different visualizations.
   Examples of this include display of ``healpix`` data in different projections, with annotations for different astronomical features (e.g. the galactic plane.)
2. Although the existing high-level tools in ``bokeh`` and ``rubin_sim`` may eliminate most of the code reduncancy between different visualizations, the code will be easier to maintain and reuse in different contexts if different visualizations follow a common architecture.
   So, the ``schedview`` framework can be used to guide the construction of new visualizations even when the new visualization may not depend on any existing elements of the ``schedview`` code.

These motivations result in an implementation of ``schedview`` with two important features: a handful of modules that support creation of plots specific to astronomy, access to data in Rubin Observatory specific environments, or visualization of data from ``rubin_sim``, and a set of guiding principles that drive the architecture of visualizations.

The architectural elements described here, therefore, do not describe the architecture of the ``schedview`` package as a whole, but rather present an architecture that should guide the construction of each visualization individually: different visualizations will typically have all of these same components, but may not share any implementation of any of them.

Even within the ``schedview`` module, exceptions should be expected: this list of compononts should be considered a guide, not a set of rigid rules.

The division of visualizations into these architectural components is driven by several considerations:

1. Conceptual coherence. The components should be intuitive to a human.
2. Minimal coupling.
3. Portability across different environments. Similar visualization will be needed in a variety of contexts, including within displays shown by LOVE, at the observatory, Rubin Science Platform jupyter notebooks, jupyter notebooks run locally on laptops, and parametrized notebooks.
Some parts of the code will need to be shared across all of these environments, while others will need to have different implementations on different environments.
Code that can be shared across different environments should be isolated in separate components from that which may vary.

The first two of these are standard software design principles.
The third enables use in the environments listed in `Operational Contexts`_: code that needs to be different in different contexsts should be isolated in separate modules, such that context-independent code can be shared amoung context while context-dependent code is not.

Division roughly corresponding to stages an a data flow from the source to the ultimate presention satisfies these goals:

1. **Collection**. Code that handles collection of data such as loading files from disk, downloading them from a URL, or querying a database. 
   A single visualization may have multiple implementations of its collection element, each supporting a different source for data or operational context.
   This code is organized into the ``collect`` submodule of ``schedview``.
2. **Munging**. Code that filters or reformats the collected data and places it in a format that can be used directly to instantiate ``bokeh`` data sources.
   See the `Providing Data <https://docs.bokeh.org/en/latest/docs/user_guide/data.html>`_ page of the ``bokeh`` documentation.
   The ``bokeh`` API is flexible, and in many cases well be able to accept data as read.
   In these cases, a visualization may not include this element at all.
   This code is organized into the ``munge`` submodule of ``schedview``.
3. **Computation**. Some visualizations may require processing and calculation beyoned that is reasonably considered munging, for example running an ``opsim`` simulation.
   If this code is included within the ``schedview`` module at all, it should be placed in the ``compute`` submodule.
4. **Plotting**. The plotting architectural element constructs a high-level ``bokeh`` object (an instance of ``bokeh.models.Plot``, ``bokeh.modules.Figure``) from data provided by earlier steps. 
   Not all operational context support ``python`` callbacks, so only ``javascript`` callbacks should be included in this element.
   This element may also include an API for modifying the models within the plot, thereby supporting manipulation of the plot using ``python`` code, for   example in cells of a ``jupyter`` notebook.
   This code is organized into the ``plot`` submodule of ``schedview``. 
5. **Application**. Full ``bokeh`` "applications" support ``python`` callbacks, not just ``javascript`` ones. 
   The applications architectural element supplements the plotting code from the plotting element to include ``python`` callbacks where they are useful.
   Whenever possible, the callbacks should be implemented my simple callback registrations of ``bokeh`` model events to calls of the API for modifying the plot's ``bokeh`` models already implemented in the plotting element.
   The applications submodule also includes the code required to start the ``bokeh`` application itself.
   This code is organized in the ``app`` submodle of ``schedviews``.

Not all visualizations will require all achitectural elements.
In particular, many will not require munging or computation elemenets.
Visualizations to be run in multiple contexts may need multiple implementations of the collection element, while the ``app`` element may not be implemented for visualizations than never need to be run as an independent ``bokeh`` application.

Although different visualizations will share this architecture, they may not share any actual code: there is **no** requirement that all implementations of a given element be classes that inheret from a common parent class, for example. 

Interfaces
==========

Data provided to Bokeh
^^^^^^^^^^^^^^^^^^^^^^

There should be some standard conventions for column names in the instances of ``ColumnDataSource``, so that (for example) a healpix maps generated from different data can be mixed and matched with different visualizations of healpix maps.





..
   Viewpoint n
   -----------

   Design view n
   ^^^^^^^^^^^^^

   Design overlays n
   ^^^^^^^^^^^^^^^^^

   Design rationales n
   ^^^^^^^^^^^^^^^^^^^

.. Make in-text citations with: :cite:`bibkey`.
.. Uncomment to use citations
.. .. rubric:: References
.. 
.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
