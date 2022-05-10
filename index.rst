:tocdepth: 1

.. sectnum::

.. Metadata such as the title, authors, and description are set in metadata.yaml

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is a work-in-progress.**

Abstract
========

Tools for monitoring and exploring scheduler behaviour and survey progress are an important element of Rubin Observatory infrastructure. A variety of users will require these plots, and require access to them in a variety of contexts and environments. Examples include telescope operators monitoring the scheduler for problems during observing, managers preparing reports for funding agencies, and scientists from science working groups reviewing the effectiveness or suvey strategy as implemented. Tools for generating these visualizations require much of the same functionality as other elements of Rubin Observatory infrastructure. This document describes and architecture for development of scheduler and progress monitoring visualizations intended to avoid duplication of develpoment effort by making it easy to use use common code across different contexts (e.g. both the "First-look Analysis and Feedback Functionality" (FAFF) infrastructure being developed for the observatory and the Rubin Science Platform jupyter notebooks being developed for users and developers), and to itself reuse code developed for scheduler construction and strategy evaluationn (opsim and MAF). 

Introduction
============

Purpose
^^^^^^^

A variety of users will require tools for monitoring scheduler behaviour and survey progress. RTN-016 provides a list of examples of use cases and visualizations. The users, contexts of use, and details of the visulazations listed in RTN-016 are diverse, and not exhaustive: it is expected that many additional uses and visualizations will be discovered and the survey progresses. Some users will have limited software development skills, or have limited familiarity with the details of the scheduler or observing system, while others will be experts. For some use cases, the full flexibility of general analyses tools (such as python running in an ad-hoc jupyter notebook) will be required, for example when a developer is debugging a problem. In other cases, users will need monitoring visualizations that can alert them to problems while requiring minimal attention or interaction. These visualizations well be needed in the context of observatory operations as part of the "First-look Analysis and Feedback Functionality" (FAFF) infrastructure (described in SITCOMNT-025), as modules to be used in notebooks running on the Rubin Science Platform notebook aspect, as elements of RSP parameterized notebooks (SQR-062), and perhaps in other contexts as well.

Scope
^^^^^

This document describes an architecture for software specific to construction of schedulery behaviour and survey progress monitoring, particularly elements that can either be shared among multiple visualizations. It does not describe the design of specific visualizations except as illustrative examples. In so far as the architecture or design of other elements (e.g. FAFF or the RSP) are described, it is only to provide context.

Intended Audience
^^^^^^^^^^^^^^^^^

This document is primarily intended to be a guide for software developers either writing or mainitaining scheduler or survey progress monitoring software, or integrating such software into other systems.

Illustrative use
^^^^^^^^^^^^^^^^

Consider a night on which the scheduler does something unexpected, and a team of developers uses jupyter notebooks to explore the behavior on that night. They will ultimately develop notebooks that create visualizations that are diagnostics useful for identifying problems (whether or not there was actually a problem on that night). If the developers follow a standard architecture, the code they develop will be easily adopted to be included in other environments, such as the FAFF system at the observatory. If there is existing code developed for other visualizations, it may be reused, reducing the development effort needed for the current use. If the developers extend existing tools, or package their new tools in a consistent way, it will be of use in current efforts.


Operational Contexts
====================

FAFF
----

First-look Analysis and Feedback Functionalty (FAFF) breakout group is addressing the question of how project will provide on-the-fly analysis and display of telemetry and other data at the observatory. The group has catalogued existing resources, and identified where they need to be augmented, or where new resources need to be constructed, and made recommendations about what resources should be used (SITCOMTN-025). The group is currently defining how users will interact with existing metrics, analyses, and other artifacts, developing a set of use cases and metrics that need to be scheduled for implementation, creating a corresponding task list and schedule, and preparing user-level training (SITCOMTN-030). The visualization tools within the scope of this document must be able to operate within the context of the resources and interfaces described by the FAFF breakout group. SITCOMTN-025 recommends the creation of visualizations using Bokeh applications, to be incorporated into the observatory displays that are provided by the LSST Observatory Visualization Environement (LOVE).

Rubin Science Platform's Notebook View
--------------------------------------

The Rubin Science Platform (RSP) has a "notebook view" that provides a jupyterhub environment. RSP notebooks provide access to Rubin Observatory software and many data products, but RSP instances not running at the observatory do not provide access to all of the data sources available at the observatory.

Parametrized Notebooks
----------------------

Instead of being developed ad-hoc, standard jupyter notebooks can be published with customizable parameters to implement live dashboards and reports (SQR-062, SITCOMTN-025). Progress and scheduler visualizations should support inclusion in these reports. 

Local
-----

Developers will want to use visualization tools locally, for example on their own laptops. Code for this context may also be useful even in the RSP notebook. For example, code for a "local" context will need to be able to used data stored in arbitrary local files. It may be usefull to bypass the production data provided in the RSP and use specially crafted local files for testing.

Resources
=========

Bokeh
-----

*Very* short description of the bokeh architecture (few sentences).


MAF
---

Many of the metrics to be visualized are already implemented in MAF using matplotlib. When they become important to implement in this framework, they should use the MAF code in ``rubin_sim`` rather than implement new code. For example, there should be a standard way for maps from metrics calculated on a healpix spatial slicer to be converted to bokeh DataSource objects, so the same software for visualizing maps can be used for arbitrary maps from MAF.

Note MAF metrics are often take to long to be usefully included in dynamic callbacks, but this will not be necessary for many uses.


Components
==========

Application
-----------

A bokeh "application" or server provides a web server that serves a bokeh application that can support python callbacks and incorporated into LOVE displays. They can also be used in jupyter notebooks, but are not the only way to include bokeh plots in jupyter notebooks, and can be more tricky than other methods.

The bokeh appliction for a visualization can be very light-weight, perhaps only import statements and three or four lines of code: it is primarily a harness for combining the data access, processing, and visualization provided by other modules.

Data access
-----------

A data access component retrieves data and returns it in a format that can be conveniently converted into a ``bokeh.models.DataSource``, typically a dictionary of lists or numpy arrays, or a ``pandas.DataFrame``. See the `API <https://docs.bokeh.org/en/latest/docs/user_guide/data.html>`__ for ``bokeh.models.ColumnDataSource``.

Different contexts may need different implementations of data access components, but the different implementations should have the same API and return the same python objects.

Processing
----------

Often, a visualization not need to present data as provided by the data source, but rather the results of a computation created using input from any number of data sources. (This number may be zero.) This code should be isolated from both data access and data visualization.

Visualization
-------------

Generation
^^^^^^^^^^

Data visualization code transforms data provided by "Data Access" and "Processing" components into Bokeh "models." 

Callbacks
^^^^^^^^^

Interactive elements of plots are implemented by registering callbacks with user interface events. In Bokeh, these callbacks can be implemented in either javascript or python.

Callbacks implemented in python can use the full array of tools available in a python environment, including functionality provided by the data access and processing components. However, they can require significant communication between the server and browser, introducing significant overhead and sluggish responses.

Callbacks can be implemented in javascript instead, in which case the calculations are executed by the user's browser. No communication overhead is required, but the implementation of callbacks is limited to tools available in javascript.

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
