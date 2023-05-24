:tocdepth: 1

.. sectnum::

.. Metadata such as the title, authors, and description are set in metadata.yaml

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is a work-in-progress.**

Abstract
========

Calibration production and verification has relied on notebooks for the majority of the visualization, a choice that scales poorly as we move from the analysis of LATISS images to the full camera.  The metric and plotting interfaces currently defined in [``analysis_tools``](https://github.com/lsst/analysis_tools)  provide a much better way to organize and present the data from these calibration steps.  However, there is still some missing functionality that needs to be added for this to fully solve this problem.  This tech note will present an initial set of issues and goals for this integration.

Introduction
============

The May 2023 Commissioning Science Validation Bootcamp presented a wonderful introduction to the existing metrics and visualization tools already available in the LSST Science Pipelines.  The work on calibration production and verification has not previously used these tools, but given their advanced state, switching now before the full camera comes online is a priority.  This should hopefully resolve some of the difficulty of managing and visualizing the results that are already sufficiently complicated for just the LATISS data.  The goal of this technote is define the additional tasks that are necessary to add to the [``cp_pipe``](https://github.com/lsst/cp_pipe) and [``cp_verify``](https://github.com/lsst/cp_verify) packages to take advantage of these tools, to identify any functionality missing from ``analysis_tools`` and ``Sasquatch``, and to present a roadmap for this process.

``cp_verify`` currently generates a series of plots and image views in a series of notebooks.  This is reasonably convenient for an individual (or small group) checking the quality of calibrations for a single detector, but as either the number of interested parties or detectors grow, this becomes more difficult.  We have also wanted a way to present results from the calibration construction and verification similar to the "run reports" the Camera Team produces from their [``eo_pipe``](https://github.com/lsst-camera-dh/eo_pipe) based code.  The plotting ability of ``analysis_tools`` (and the associated Plot Navigator) present an excellent way to present the things currently found in the notebooks, and by turning the numerical results into ``Metrics``, we can also track these values over time via ``Sasquatch``.


Calibration Products Improvements
=================================

Metrics
-------

The ``cp_verify`` package should organize the quality test results (as defined in DMTN-101) that are being calculated into formal ``Metrics``.  As it is unlikely that there is an amplifier level dimension, these probably need to report the minimum, maximum, and median values found per detector, as well as the number of amplifiers that failed the test requirements.  As the variety of most of these tests is fairly limited, defining these metrics does not seem like a problem.  Generating these metrics can be added to the existing ``MergeTasks``, at the expense of ensuring that ``analysis_tools`` is a prerequisite for ``cp_verify``.  This seems like the obvious first step for this integration.

Plotting
--------

Plots are currently generated in notebooks, and those that are generally useful should be migrated to use the ``analysis_tools`` plot types.  As these are written to the Butler, they can still be accessed and displayed in those notebooks.  This also seems like an easy switch, and by generating these plots as part of the appropriate pipeline tasks, they can be parallelized, reducing the complications as we switch to the full camera.

Image Views
-----------

One of the main other aspects of the notebooks is the ability to flip through the set of residual images, such as a bias frame that has had only overscan and bias subtracted, to look for any remaining structure in the image that may indicate quality issues with the calibration.  As these can be displayed in matplotlib, it seems reasonable to pre-generate these as "plots" in the verification pipelines.  This is not an issue for LATISS, but constructing full focal plane images will be needed for the full camera.  The ``visualizeVisit`` pipeline can bin and mosaic the individual detectors together, but it's unclear now if that final image will be too highly binned to be useful for identifying problems.


Missing Functionality
=====================

A few changes are needed to support the inclusion of calibration products in these tools.  This is intended to be a preliminary list that we can distribute to identify if the changes need any special knowledge of the system, and who can make those changes.

Plotting
--------

One style of plot that is essential for calibration products is a focal plane plot that displays quantities binned over individual detectors or amplifiers.  An implementation of this is available on the [DM-32450](https://jira.lsstcorp.org/browse/DM-32450) branch of ``afw``, but does not present the conveinent interface that the ``analysis_tools`` code has.  There is an existing ``FocalPlanePlot``, which does plot quantities on the focal plane using a regular grid of points.

My initial thought is to subclass this existing plot type, transfer any useful portion of the DM-32450 code into that subclass, with the goal being to construct a drop in replacement for the existing class that will use the detector and amplifier level as the bins for the input data.  I'm unsure if any of the code in the current ``makePlot`` method can be pulled into additional functions to prevent a large block of duplicated lines in the subclass ``makePlot``, and would appreciate any guidence on this.

Plot Navigator
--------------

The [Plot Navigator](https://usdf-rsp.slac.stanford.edu/plot-navigator/dashboard_gen3) can identify the existing collections from any of the Butler repositories USDF, but currently only shows plots that are generated for ``tract`` and ``visit`` products.  This is incompatible with the current calibration processing, which operates on ``exposure`` level products.  If this is not already trivially supported (i.e., new plot dimensions automatically propagate from generated plots), then some work will be needed to add this support.  Adding support for calibrations themselves is likely even more challenging, as these have even fewer useful dimensions, in addition to the validity date ranges that separate different calibrations in the same collections.  Again, if we want to support plots for calibrations, this support will need to be added.

Roadmap
=======

TBD.


.. Make in-text citations with: :cite:`bibkey`.
.. Uncomment to use citations
.. .. rubric:: References
.. 
.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
