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
Some illustrative use cases include:

- The observing and scheduler scientists will review the current state of the survey and simulations of each upcoming night of observing to identify potential misbehavior by the scheduler before the night begins.
- Observatory staff will review scheduler visualizations to familiarize themselves with the expected behavior of the scheduler over the course of the upcoming night.
- Observatory staff will need to monitor visualizations of scheduler behavior to identify and diagnose problematic behavior during observing.
- TODO NIGHT REPORTS
- The project (who?) needs to produce periodic status reports that allow the community to assess survey progress, including updating predictions of the results of the ten year survey based on progress to the current time.

Some users will have limited software development skills, or have limited familiarity with the details of the scheduler or observing system, while others will be experts.
For some use cases, the full flexibility of general analyses tools (such as python running in an ad-hoc jupyter notebook) will be required, for example when a developer is debugging a problem.
In other cases, users will need monitoring visualizations that can alert them to problems while requiring minimal attention or interaction.
These visualizations well be needed in the context of observatory operations as part of the "First-look Analysis and Feedback Functionality" (FAFF) infrastructure (described in SITCOMNT-025), as modules to be used in notebooks running on the Rubin Science Platform notebook aspect, as elements of RSP parameterized notebooks (SQR-062), and perhaps in other contexts as well.

Operational Contexts
====================

..
   Viewpoint described by IEEE 1016 5.2, Hyde 11.2.2.1
   This wiewpoint sets scope and system boundaries: what is external, what is internal
   provides a "black box" persepctive on the design subject

The Rubin Observatory Site
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note::
   TODO: What piece of infrastructure provides the scheduler instances (currently pickles)? If not through the EFD, the source should be mentioned here.

.. note::
   TODO: What are the names of the components that implement the observing queue? The scheduler monitoring software can benefit from direct intergration with it, so entries on the queue can be traced to specific scheduler decisions for visualization. 

First-look Analysis and Feedback Functionalty (FAFF) breakout group is addressing the question of how project will provide on-the-fly analysis and display of telemetry and other data at the observatory.
The group has catalogued existing resources, and identified where they need to be augmented, or where new resources need to be constructed, and made recommendations about what resources should be used (SITCOMTN-025).
The group is currently defining how users will interact with existing metrics, analyses, and other artifacts, developing a set of use cases and metrics that need to be scheduled for implementation, creating a corresponding task list and schedule, and preparing user-level training (SITCOMTN-030).
The visualization tools within the scope of this document must be able to operate within the context of the resources and interfaces described by the FAFF breakout group.
SITCOMTN-025 recommends the creation of visualizations using Bokeh applications, to be incorporated into the observatory displays that are provided by the LSST Observatory Visualization Environement (LOVE).

.. note::
   TODO: describe the sources of data the visualization software will need to use, including the EFD and whatever provides the scheduler pickles (or whatever the pickles will be replaced with).

At the observatory, the scheduler and observing progress monitoring software will run on containers deployed using kubernetes.
The containers will include:

- **FIXME: name for service that runs opsim** The simulator will run retrieve a configured instance of the scheduler from **FIXME**, complete simulations both nightly and "on demand," and store the results in **FIXME**.
- **FIXME: name for briefing dashboard** The pre-night briefing generator will provide a web-base dashboard giving an overview of the active or upcoming night. It will read the configured instance of the scheduler from **FIXME** and one or more simulations of the upcoming night from *FIXME*, and provide a web-based dashboard giving an overview of the upcoming night.
- ``schedview``. ``schedview`` will provide a web-based interface that allows the user to select an instance of the scheduler from snapshots stored in **FIXME**, visualize the state, and explore its behavior.
- **FIXME: name for night report service**. A night report tool will read completed visits and metadata from the Engineering and Facility Database, snapshots of the scheduler from **FIXME**, and provide a web-based dashboard allowing exploration of the progress and scheduler behavior of the night, calling out indications possible problems. **FIXME: Maybe a static report is better here? Maybe ``schedview`` can be extended to serve both purposes? Do we need different night report tools for different audiences? Does this tool provide a separate report from other night report stuff, or is it all integrated?**

Rubin Science Platform's Notebook View
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Rubin Science Platform (RSP) has a "notebook view" that provides a jupyterhub environment.
RSP notebooks provide access to Rubin Observatory software and many data products, but RSP instances not running at the observatory do not provide access to all of the data sources available at the observatory.

Scheduler and observing progress monitoring tools will provide a collection of ``python`` modules to support flexible exploration of scheduler behavior and progress, both of previously completed observing and simulated future observing.
These modules will include:

- **collection** The ``collection`` module retrieves data needed for exploration and visualization, and makes it available within the ``jupyter`` notebook. **FIXME: Where will this data come from?**
- **simulation** Utilities that simplify the execution of ``opsim`` simulations in the context of understanding past or hypothetical situations, or from a given starting point to the a given time in the future, may be required. Most of the code involved will be contained in ``opsim`` itself, but some tools to launch simulations with appropriate parameters, archive and organize results, and otherwise intergrate it into the monitoring or progress context may be needed. **FIXME: Think this through more, or just put it all in opsim?**
- **plotting** Specialized figure for specific kinds of scheduler or progress data will be supported in a ``plots`` submodule. Examples will include maps in custom sky projections and hourglass plots. In most cases, however, ``holoviews`` should be usable directly on data returned by the ``collection`` module, so normal plots (e.g. scatter plots and histograms) will not need specialized code.
- **dashboards** Collections of plots and controls that support specific use cases hand be implemented as dashboards that be displayed within a jupyter notebook.

Parametrized Notebooks
^^^^^^^^^^^^^^^^^^^^^^

Instead of being developed ad-hoc, standard jupyter notebooks can be published with customizable parameters to implement live dashboards and reports (SQR-062, SITCOMTN-025).
Progress and scheduler visualizations should support inclusion in these reports.

Parametrized notebooks will require the same set of modules as the RSP context.

Local
^^^^^

Developers will want to use visualization tools locally, for example on their own laptops.
Code for this context may also be useful even in the RSP notebook. For example, code for a "local" context will need to be able to used data stored in arbitrary local files.
It may be usefull to bypass the production data provided in the RSP and use specially crafted local files for testing.

Resources
=========

MAF
^^^

Many of the metrics to be visualized are already implemented in MAF using matplotlib.
When they become important to implement in this framework, they should use the MAF code in ``rubin_sim`` rather than implement new code.
For example, there should be a standard way for maps from metrics calculated on a healpix spatial slicer to be converted to bokeh DataSource objects, so the same software for visualizing maps can be used for arbitrary maps from MAF.

Note MAF metrics often take to long to be usefully included in dynamic callbacks, but this will not be necessary for many uses.

Bokeh
^^^^^

Bokeh is a plotting tool which consists of two major components: 

The ``BokehJS`` ``javascript`` library
  for displaying a (possibly interactive) visualization specified by a collection of objects serialized in a ``json`` file.
The ``bokeh`` ``python`` module
  provides tools for creating and manipulating plot elements and collecting them into a ``bokeh.document`` serializing them in ``json`` that can be used by ``BokehJS``.
  The module also includes tools for and generating html, jupyter cells, or full applications that use ``BokehJS`` to display the modeled visualizations.

Bokeh can be used in serveral ways, including:

- Within a jupyter notebook.
- As a stand-alone web service.
- Embedded within a bespoke web page.
- Within a dashboard created using holoviz panel or lumen.
- As a "back-end" for higher level plotting and dashboard constuction tools in holoviz.

Components
==========

Observatory Site Service containers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There will be a collection of containers deployed at the observatory, each of which hosts a web service that shows a dashboard that presents a collection of tables and visualizations useful for observing.
These will include:

- **Pre-night briefing dashboard**, showing visualizations and data useful for verifying the scheduler's readiness for an approaching night of observing, letting the observatory staff know what to expect, and preparing them to identify anamolous behavior that might require intervention.
- **Scheduler viewer**, showing visualizations and data that help observatory staff and others understand the scheduler state and behavior when it is active.
- **Night summary dashboard**, providing visualizations and data tha summarizes the previous night. This might be implemented as an element of a different system or display whose scope extends beyond the scheduler itself. **FIXME: look into how the general purpose night summary will be implemented**

.. note::
   TODO: opsim simulation will need to be run in support of the pre-night briefing and night summary. At least one (and possibly several) simulations will need to be run automatically before each night of observing, and the automatic simulations may need to be supplemented or replaced through human intervention. Finally, the results of these simulations need to be supplied to the dashboards. Should this be done with an additional container, within the scope of this document? Is there an existing element of the site infrastructure that should supply this survice?

The ``schedview`` python module
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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
3. Portability across different environments.
   Similar visualization will be needed in a variety of contexts, including within displays shown by LOVE, at the observatory, Rubin Science Platform jupyter notebooks, jupyter notebooks run locally on laptops, and parametrized notebooks.
   Some parts of the code will need to be shared across all of these environments, while others will need to have different implementations on different environments. Code that can be shared across different environments should be isolated in separate components from that which may vary.

The first two of these are standard software design principles.
The third enables use in the environments listed in `Operational Contexts`_: code that needs to be different in different contexsts should be isolated in separate modules, such that context-independent code can be shared amoung context while context-dependent code is not.

Division roughly corresponding to stages an a data flow from the source to the ultimate presention satisfies these goals:

1. **Collection**. Code that handles collection of data such as loading files from disk, downloading them from a URL, or querying a database. 
   A single visualization may have multiple implementations of its collection element, each supporting a different source for data or operational context.
   This ``collect`` submodule of ``schedview`` containts collection code.
2. **Munging**. Code that filters or reformats the collected data and places it in a format that can be used directly to instantiate ``bokeh`` data sources.
   See the `Providing Data <https://docs.bokeh.org/en/latest/docs/user_guide/data.html>`_ page of the ``bokeh`` documentation.
   The ``bokeh`` API is flexible, and in many cases well be able to accept data as read.
   In these cases, a visualization may not include this element at all.
   The ``munge`` submodule of ``schedview`` contains code for munging.
3. **Computation**. Some visualizations may require processing and calculation beyoned that is reasonably considered munging, for example running an ``opsim`` simulation.
   If this code is included within the ``schedview`` module at all, it should be placed in the ``compute`` submodule.
4. **Plotting**. The plotting architectural element constructs a high-level ``bokeh`` object (an instance of ``bokeh.models.Plot``, ``bokeh.modules.Figure``) from data provided by earlier steps. 
   Not all operational context support ``python`` callbacks, so only ``javascript`` callbacks should be included in this element.
   This element may also include an API for modifying the models within the plot, thereby supporting manipulation of the plot using ``python`` code, for   example in cells of a ``jupyter`` notebook.
   The ``plot`` submodule of ``schedview`` contains this code.
5. **Application**. Full ``bokeh`` "applications" support ``python`` callbacks, not just ``javascript`` ones. 
   The applications architectural element supplements the plotting code from the plotting element to include ``python`` callbacks where they are useful.
   Whenever possible, the callbacks should be implemented my simple callback registrations of ``bokeh`` model events to calls of the API for modifying the plot's ``bokeh`` models already implemented in the plotting element.
   The applications submodule also includes the code required to start the ``bokeh`` application itself.
   The ``app`` submodle of ``schedviews`` contains this code.

Not all visualizations will require all achitectural elements.
In particular, many will not require munging or computation elemenets.
Visualizations to be run in multiple contexts may need multiple implementations of the collection element, while the ``app`` element may not be implemented for visualizations than never need to be run as an independent ``bokeh`` application.

Although different visualizations will share this architecture, they may not share any actual code: there is **no** requirement that all implementations of a given element be classes that inheret from a common parent class, for example. 

Survey simulator
^^^^^^^^^^^^^^^^

Severaly visualizations will require revised values for survey metric, calculated using simulations starting from a specific time (e.g. the current time) and running through the end of the survey.
The ``opsim`` and ``maf`` submodules of the ``rubin_sim`` module will be used to run the simulations and calculate corresponding metrics, but infrastructure is required to automatically (and manually) launch such simulations, collect and archive the results (including scheduler snapshots, ``opsim`` output, and ``maf`` output), and provide and interface to provide access to these data.

The survey simulator will need to run automatically under a variety of conditions:

1. In the day following each night of observing, a simulation will need to be run from the end of that night through the end of the survey, using baseline simulation weather.
2. In the day following each night of observing, a simulation will need to be run from the end of the next night through the end of the survey, using baseline simulation weather, but excluding data from the most recent night. That is, simulate the result of the survey if the entire next night is lost, and the rest of the simualtion after that follows baseline conditions.
3. In the day following each night of observing, a simulation will need to be run from the end of that night through the end of the next night, with no clouds and good seeing, follow by a simulation of the rest of the survey under baseline conditions. That is, simulate the result of the survey if the entire next night is clear with good seeing, and the rest of the simualtion after that follows baseline conditions.
4. In the day following each night of observing, a simulation will need to be run from the end of that night through the end of the next night, with no clouds and poor seeing, follow by a simulation of the rest of the survey under baseline conditions. That is, simulate the result of the survey if the entire next night is clear with poor seeing, and the rest of the simualtion after that follows baseline conditions.

For each of these survey, a suite of MAF metrics will need to be evaluated at the current time, the end of the following night, and the end of the survey.

Survey visit database and scheduler instance archive
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 

..
   Viewpoints are a concept used in IEEE standards for architecture and design documents.
   I copy my notes on them here to serve as inspiration for organizing this document, but do not intend to follow the standards.
   In the notes below, I mix the architecture and design viewtypes.

..
   Module
   ======


..
   Composition
   ===========

   Viewpoint described in IEEE 1016 5.3, Hyde 11.2.2.2
   "design subject is (recursively) structured into constituent parts and establishes the roles of those parts"
   High level component diagram: shows composition, use, and generalization
   Mostly deprecated in favor of structure and logical viewpoints

..
   Logical
   =======

   Viewpoint described in IEEE 1016 5.4, Hyde 11.2.2.3
   Shows types (classes), interfactes, structural definitions, objects the design uses
   Typically uses UML class diagrams and a data dictionary
   Shows dependency, association, aggregation, composition, inheretance

..
   Dependency
   ==========

   Viewpoint descirbed in IEEE 1016 5.5, Hyde 11.2.2.4
   Mostly deprecated
   UML component diagriams or package diagram with dependencies shown

..
   Information/Database
   ====================

   Viewpoint described in IEEE 1016 5.6, Hyde 11.2.2.5
   Describes *persistent* data usage
   Shows data access schemes, data management strategies, data storage mechanisms

..
   Patterns
   ========

   Viewpoint described in IEEE 1016 5.7, Hyde 11.2.2.7
   Describes design patterns used

..
   Interfaces
   ==========

   Viewpoint described in IEEE 1016 5.8, Hyde 11.2.2.8
   Describes APIs
   UML component diagriams
   Interface specifications for each entity

..
   Structure
   =========

   Viewpoint described in IEEE 1016 5.9, Hyde 11.2.2.8
   UML composite structure diagrams, class diagrams, package diagrams

..
    Interaction
    ===========

    Viewpoint described in IEEE 1016 5.10, Hyde 11.2.2.9
    "main place where you define activities that take place in the software"
    allocates responsibilities in collaborations
    UML interaction diagrams

..
   State dynamics viewpoint
   ========================

   Viewpoint described in IEEE 1016 5.11, Hyde 11.2.2.10
   UML statechart diagram
   describes modes, states, transitions, reactions to events


..
   Algorithms
   ==========

   Viewpoint described in IEEE 1016 5.12, Hyde 11.2.2.11
   describes algorithms

..
   Resource
   ========

   Viewpoint described in IEEE 1016 5.13, Hyde 11.2.2.12
   Deprecated, use context viewpoint instead

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
