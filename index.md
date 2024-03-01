# Architecture for Scheduler and Observing Progress Monitoring Software

```{eval-rst}
.. abstract::

   An architecture for software to generate and present information needed to understand scheduler behavior and monitor survey progress is given. The scope of this architecture document is limited to elements specific to this task, except in so far as the architecture of other infrastructure is required to show how it fits into the environments within which it will operate.

   :tocdepth: 1

```

% Metadata such as the title, authors, and description are set in metadata.yaml

% TODO: Delete the note below before merging new content to the main branch.

:::{note}
**This technote is a work-in-progress.**
:::

## Abstract

Tools for monitoring and exploring scheduler behaviour and survey progress are an important element of Rubin Observatory infrastructure.
A variety of users will require these plots, and require access to them in a variety of contexts and environments.
Examples include telescope operators monitoring the scheduler for problems during observing, managers preparing reports for funding agencies, and scientists from science working groups reviewing the effectiveness of survey strategy as implemented.
This document describes an architecture for development of scheduler and progress monitoring examination and visualization.
The architecutre is intended to avoid duplication of develpoment effort by making it easy to use use common code across different contexts.
Such contexts include both the infrastructure being developed for the observatory and the Rubin Science Platform jupyter notebooks being developed for users and developers.

## Introduction

A variety of users will require tools for monitoring scheduler behaviour and survey progress.
RTN-016 provides a list of examples of use cases and visualizations.
The users, contexts of use, and details of the visulazations listed in RTN-016 are diverse, and not exhaustive: it is expected that many additional uses and visualizations will be discovered and the survey progresses.
Some illustrative use cases include:

- The observing and scheduler scientists will review the current state of the survey and simulations of each upcoming night of observing to identify potential misbehavior by the scheduler before the night begins.
- Observatory staff will review scheduler visualizations to familiarize themselves with the expected behavior of the scheduler over the course of the upcoming night.
- Observatory staff will need to monitor visualizations of scheduler behavior to identify and diagnose problematic behavior during observing.
- Night reports will require elements showing scheduler behavior during the night.
- The project needs to produce periodic status reports that allow the community to assess survey progress, including updating predictions of the results of the ten year survey based on progress to the current time.

Some users will have limited software development skills, or have limited familiarity with the details of the scheduler or observing system, while others will be experts.
For some use cases, the full flexibility of general analyses tools (such as python running in an ad-hoc jupyter notebook) will be required, for example when a developer is debugging a problem.
In other cases, users will need monitoring visualizations that can alert them to problems while requiring minimal attention or interaction.
These visualizations well be needed in the context of observatory operations as part of the "First-look Analysis and Feedback Functionality" (FAFF) infrastructure (described in SITCOMNT-025), as modules to be used in notebooks running on the Rubin Science Platform notebook aspect, as elements of RSP parameterized notebooks (SQR-062), and perhaps in other contexts as well.

## Components

### Figure generators

The `schedview` `python` module provides code to generate table, plots, and other figures intended to provide an undestanding of the scheduler and related data to humans.
Examples of figures produced by such generators include a table of astronomical events, histograms of visit parameters (e.g. R.A., Declination, LST, depth), a table of visits in a given time window, sky maps of scheduler reward basis functions at a given time, and many others; RTN-016 provides a number of examples.
`schedview` provides separate functions to generate plots when combined in a step-by-step "pipe and filter" architucture, with the following filter compononts:

1. **Collection** and **munging**. Code that handles collection of data such as loading files from disk, downloading them from a URL, or querying a database; and filters or reformats the collected data and to make it suitable for direct use by later elements (such as computation of plotting).
   A single visualization may have multiple implementations of its collection element, each supporting a different source for data or operational context.
   In some cases, when all data to be visualized can be generated by the code and configuration itself without additional data (e.g. the phase of the moon), this step may not be necessary.
   Elements in the "collection and munging" submodule of the architecure should contain all functionality specific to a given source of data, and only functionality specific to that data source, such that they provied output independent of the source of the data.
   The `collect` submodule of `schedview` (`schedview.collect`) containts collection and munging code.
2. **Computation**. Some visualizations may require processing and calculation beyond simple reformatting.
   Functionality in computation components includes, for example, calculation of the area of sky covered by a giving visit (its footprint) given the pointing, or computation of basis function rewards at a given time.
   Elements in the "computation" submodule should contain all functionality that is independent of either how the data is collected or how it is displayed, and only functionality that is independent of either how the data is collected or how it is displayed
   The `compute` submodule of `schedview` (`schedview.compute`) contains computing code.
3. **Plotting**. The plotting architectural element creates a visualization object that can be used directly in a dashboard, displayed in a jupyter notebook, or written to disk as a `pdf`, `png`, or `jpeg` file.
   When plotting using `bokeh`, this will be an instance of `bokeh.modules.Figure` or one of its subclasses, e.g. `bokeh.models.Plot`.
   Functionality in the `schedview` repository is intended to be specific, rather than general purpose plotting tools.
   For example, `schedview` includes a function to plot camera pointings with the footprint outline over a healpix map, but the general purpose functionality for making sky maps (including plotting healpix maps and polygons) is provided by a separate module (`uranography`) in a different repository.
   In this way, general purpose plotting code can be reused without requiring `schedview` and its scheduler-specific dependencies.
   The `plot` submodule of `schedview` (`schedview.plot`) contains such `schedview` plotting code.
4. **Workflom** and **presentation**. Workflow architectural elements use colletion, computation, and plotting elements to create finished visualizations.
   Examples include `panel` dashboards and `jupyter` notebooks (paramterized or not).
   Although some driver code resides in the `schedview` module, it is not expect that all such code will be hosted there: some dashboards or notebooks that call collection, computation, and plotting elements and use the results to create finished reports or dashboards are expected to reside in other repositories as well.

Generation of a figure may sometimes requrire multiple collection, munging, or computation components, if multiple sets of data will overplotted on the same figure.

There is no expectation that each visualization be a subclass of the same superclass, or that the components used by any given visualization derive from a common superclass of components of other figure generation functions.
This design permits but does not require such code reuse.

### Simulation generators

Several visualizations will require revised values for survey metrics, calculated using simulations starting from a specific time (e.g. the current time) and running through the end of the survey.
The `opsim` and `maf` submodules of the `rubin_sim` module will be used to run the simulations and calculate corresponding metrics, but infrastructure is required to automatically (and also manually) launch such simulations, collect and archive the results (including scheduler snapshots, `opsim` output, and `maf` output), and provide and interface to provide access to these data.

The survey simulator will need to run automatically under a variety of conditions:

1. During the day before each night of observing, a set of simulations will need to be run for the following night (and maybe two nights) under a variety of weather conditions and starting times. Likely examples include:

   1. Good seeing, first exposure exactly at the nominal starting time.
   2. Good seeing, first exposure a few minutes later than the nominal starting time (simulating a late start due to technical or operational delays).
   3. Poor seeing, first exposure exactly at the nominal starting time.
   4. Good seeing, first exposure two hours later than the nominal starting time (simulating a delayed start due to weather).
   5. Cloud, wind, and seeing conditions predicted by meteorologists, first exposure exactly at the nominal starting time or when the weather is first predicted to be good enough to start observing, whichever is earlier.

2. On a periodic basis (daily? weekly? quarterly?), a set of simulations will need to be run showing how the achieved data collected in that period affects predicted final metrics. Likely examples include:

   1. A simulation beginning at the end of the period, starting with the current actual state of the survey and running through the end of the survey.
   2. A simulation beginning at the start of the period, and running through the end of the survey using baseline conditions. Comparison with the previous simulation will show how differences between the actual and baseline predicted exposures affect the final survey.
   3. A simulation beginning at the end of the period and running through the end of the survey in baseline weather conditions, including no visits during the period in question. This simulation demonstrates how the worst possible observing in that period would have affected the final survey.
   4. A simulation beginning at the start of the period and running through the end of the survey, with no clouds, wind, or bad seeing during the perioud under study, and baseline conditions thereafter. This simulation will show how the best possible observing in the period affects the final survey.

For each of these survey, a suite of MAF metrics will need to be evaluated at the current time and the end of the survey.

The simulation generator will also store resultant visit databases, MAF metrics, and snapshots of the scheduler instances will need to be saved in visit database archives along with corresponding metadata.

In addition to running automatically, the simulation generator will also need to be configured and run manually.
Such manually run simulations will differ from those run using `opsim` directly in that it will handle interactions with an archive a simulations and derived data automatically.

### Dashboards

One way in which users can view visualization generated by ``schedview`` is through dashboards, interactive server-backed web pages that use ``schedview`` (and maybe other tools) to arrange visualizations on a compact user interface.
Dashboards included within ``schedview`` proper should be built using the [``holoviz panel``](https://panel.holoviz.org) module using its [declaritive API](https://panel.holoviz.org/how_to/param/index.html).

Each dashboard included in the ``schedview`` module should be a submodule of the ``schedview.app`` submodule, and consist of a parameterized class (subclass of ``param.Parameterized``) with any data to be visualized as ``param.Parameter`` attributes of this class.
Dependencies between these members, the widgets used to load and manipulate them, and the visualizations made from them are then managed by members of this class, declared using the ``param.depends`` decorator.

The functionality of a dashboard submodule of ``schedview`` should be limited to features specific to that dashboard; functionalty that load data, performs computations, creates visualizations, or performs other operations which may be useful in multiple dashboards should be contained in other modules on which that (and potentially other) dashboards can depend.

:::{note}
It is expected that other dashboard and interfaces may be implemented outside of the ``schedview`` module, but may still use and depend of functionality in ``schedview``, for example to include one or two scheduler related visualizations in an interface mostly unrelated to the scheduler.
The design of such external uses is outside the scope of this document.
This possibility does imply that care should be taken that other parts of ``schedview`` (e.g. implementations of functionality in the ``compute`` or ``plot`` submodules) not depend on the specific implementations of the dashboard described here.
:::

### Simulation and schedular instance archive

Many figure generation functions will require access to previously generated visit databases (actual, simulated, or hybrid), MAF metric values, and instances of the scheduler.
Such databases and scheduler instances will usually be impossible or too computationally expensive to generate as needed, so archives that stores and provide access to visit database and scheduler instances will be required.
Such an archive will need to include metadate necessary to associate visit databases, MAF metrics, and instances of the sceduler with each other.
There will be separate instances of this archive for different contexts: there will be one available at the observatory, and another on the RSP.
These different instances may or may not share the same implementation or API.

### Containers

There will be a container for each dashboard, deployable at the observatory using kubernetes.

## Operational Contexts

% Viewpoint described by IEEE 1016 5.2, Hyde 11.2.2.1
% This wiewpoint sets scope and system boundaries: what is external, what is internal
% provides a "black box" persepctive on the design subject

### The Rubin Observatory Site

:::{note}
TODO: What piece of infrastructure provides the scheduler instances (currently pickles)? If not through the EFD, the source should be mentioned here.
:::

:::{note}
TODO: What are the names of the components that implement the observing queue? The scheduler monitoring software can benefit from direct intergration with it, so entries on the queue can be traced to specific scheduler decisions for visualization.
:::

First-look Analysis and Feedback Functionalty (FAFF) breakout group is addressing the question of how project will provide on-the-fly analysis and display of telemetry and other data at the observatory.
The group has catalogued existing resources, and identified where they need to be augmented, or where new resources need to be constructed, and made recommendations about what resources should be used (SITCOMTN-025).
The group is currently defining how users will interact with existing metrics, analyses, and other artifacts, developing a set of use cases and metrics that need to be scheduled for implementation, creating a corresponding task list and schedule, and preparing user-level training (SITCOMTN-030).
The visualization tools within the scope of this document must be able to operate within the context of the resources and interfaces described by the FAFF breakout group.
SITCOMTN-025 recommends the creation of visualizations using Bokeh applications, to be incorporated into the observatory displays that are provided by the LSST Observatory Visualization Environement (LOVE).

:::{note}
TODO: describe the sources of data the visualization software will need to use, including the EFD and whatever provides the scheduler pickles (or whatever the pickles will be replaced with).
:::

At the observatory, the scheduler and observing progress monitoring software will run on containers deployed using kubernetes.
The containers will include:

- The simulator will retrieve a configured instance of the scheduler, complete simulations both nightly and "on demand," and store the results.
- The pre-night briefing generator will provide a web-base dashboard giving an overview of the active or upcoming night. It will load the configured instance of the scheduler and one or more simulations of the upcoming night, and provide a web-based dashboard giving an overview of the upcoming night.
- `schedview` will provide a web-based interface that allows the user to select an instance of the scheduler from snapshots, visualize the state, and explore its behavior.
- A night report tool will read completed visits and metadata (e.g. from the Engineering and Facility Database), snapshots of the scheduler, and provide a web-based dashboard allowing exploration of the progress and scheduler behavior of the night, calling out indications possible problems.

:::{note}
- TODO: Identify the source of scheduler instances.
- TODO: Identify the archive of results of scheduler simulation.
- TODO: Name the component that runs the simulations.
- TODO: Name the pre-night briefing generator.
- TODO: Name the night report tool.
:::

### Rubin Science Platform's Notebook View

The Rubin Science Platform (RSP) has a "notebook view" that provides a jupyterhub environment.
RSP notebooks provide access to Rubin Observatory software and many data products, but RSP instances not running at the observatory do not provide access to all of the data sources available at the observatory.

Scheduler and observing progress monitoring tools will provide a collection of `python` modules to support flexible exploration of scheduler behavior and progress, both of previously completed observing and simulated future observing.
These modules will include:

- **collection** The `collection` module retrieves data needed for exploration and visualization, and makes it available within the `jupyter` notebook.
- **simulation** Utilities that simplify the execution of `opsim` simulations in the context of understanding past or hypothetical situations, or from a given starting point to the a given time in the future, may be required. Most of the code involved will be contained in `opsim` itself, but some tools to launch simulations with appropriate parameters, archive and organize results, and otherwise intergrate it into the monitoring or progress context may be needed.
- **plotting** Specialized figure for specific kinds of scheduler or progress data will be supported in a `plots` submodule. Examples will include maps in custom sky projections and hourglass plots. In most cases, however, `holoviews` should be usable directly on data returned by the `collection` module, so normal plots (e.g. scatter plots and histograms) will not need specialized code.
- **dashboards** Collections of plots and controls that support specific use cases hand be implemented as dashboards that be displayed within a jupyter notebook.

### Parametrized Notebooks

Instead of being developed ad-hoc, standard jupyter notebooks can be published with customizable parameters to implement live dashboards and reports (SQR-062, SITCOMTN-025).
Progress and scheduler visualizations should support inclusion in these reports.

Parametrized notebooks will require the same set of modules as the RSP context.

### Local

Developers will want to use visualization tools locally, for example on their own laptops.
Code for this context may also be useful even in the RSP notebook. For example, code for a "local" context will need to be able to used data stored in arbitrary local files.
It may be usefull to bypass the production data provided in the RSP and use specially crafted local files for testing.

% Module
% ======

% Composition
% ===========
%
% Viewpoint described in IEEE 1016 5.3, Hyde 11.2.2.2
% "design subject is (recursively) structured into constituent parts and establishes the roles of those parts"
% High level component diagram: shows composition, use, and generalization
% Mostly deprecated in favor of structure and logical viewpoints

% Logical
% =======
%
% Viewpoint described in IEEE 1016 5.4, Hyde 11.2.2.3
% Shows types (classes), interfactes, structural definitions, objects the design uses
% Typically uses UML class diagrams and a data dictionary
% Shows dependency, association, aggregation, composition, inheretance

% Dependency
% ==========
%
% Viewpoint descirbed in IEEE 1016 5.5, Hyde 11.2.2.4
% Mostly deprecated
% UML component diagriams or package diagram with dependencies shown

% Information/Database
% ====================
%
% Viewpoint described in IEEE 1016 5.6, Hyde 11.2.2.5
% Describes *persistent* data usage
% Shows data access schemes, data management strategies, data storage mechanisms

% Patterns
% ========
%
% Viewpoint described in IEEE 1016 5.7, Hyde 11.2.2.7
% Describes design patterns used

% Interfaces
% ==========
%
% Viewpoint described in IEEE 1016 5.8, Hyde 11.2.2.8
% Describes APIs
% UML component diagriams
% Interface specifications for each entity

% Structure
% =========
%
% Viewpoint described in IEEE 1016 5.9, Hyde 11.2.2.8
% UML composite structure diagrams, class diagrams, package diagrams

% Interaction
% ===========
%
% Viewpoint described in IEEE 1016 5.10, Hyde 11.2.2.9
% "main place where you define activities that take place in the software"
% allocates responsibilities in collaborations
% UML interaction diagrams

% State dynamics viewpoint
% ========================
%
% Viewpoint described in IEEE 1016 5.11, Hyde 11.2.2.10
% UML statechart diagram
% describes modes, states, transitions, reactions to events

% Algorithms
% ==========
%
% Viewpoint described in IEEE 1016 5.12, Hyde 11.2.2.11
% describes algorithms

% Resource
% ========
%
% Viewpoint described in IEEE 1016 5.13, Hyde 11.2.2.12
% Deprecated, use context viewpoint instead

% Viewpoint n
% -----------
%
% Design view n
% ^^^^^^^^^^^^^
%
% Design overlays n
% ^^^^^^^^^^^^^^^^^
%
% Design rationales n
% ^^^^^^^^^^^^^^^^^^^

% Make in-text citations with: :cite:`bibkey`.

% Uncomment to use citations

% .. rubric:: References

%

% .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib

% :style: lsst_aa
