---
layout: post
title: Calibrating X-ray Powder Diffraction Data at the Stanford Synchrotron Radiation Laboratory
comments: true
redirect_from: "/2013/02/14/slac-suli-internship/"
permalink: calibrating-powder-diffraction-data-ssrl
---

![Here I am at the end of the 2007 SULI internship with my adviser
Sam Webb.](/assets/IMG_1524b.jpg)

![The participants of the 2007 SLAC SULI
internship.](/assets/IMG_1738b-580x386.jpg)

During the summer of 2007, I participated in a [Science Undergraduate
Laboratory Internship](http://www-group.slac.stanford.edu/aao/suli.asp) at
the [SLAC National Accelerator Laboratory](http://slac.stanford.edu/) .
I worked at the [Stanford Synchrotron Radiation
Laboratory](http://www-ssrl.slac.stanford.edu/) and my advisers
were [Sam Webb](http://www-ssrl.slac.stanford.edu/~swebb/) and
Apurva Mehta. During the internship, I worked on a program to
calibrate and reduce [x-ray powder
diffraction](http://en.wikipedia.org/wiki/Powder_diffraction) data.

![A typical diffraction pattern. This image is being viewed with
the Area Diffraction
Machine.](/assets/diffraction-data-window.jpeg)

X-ray powder diffraction is a method of imagining crystals with
x-rays. The diffraction pattern that comes from the crystal are
concentric rings and the ring spacing can be used to infer the
crystalline structure of the material.

In order to analyze the diffraction data, you have to convert from
the CCD image of the diffraction pattern to physical scattering
angles. This mapping requires knowledge of the detector geometry.
In particular, the imaging detector can often be somewhat rotated,
changing the observed circles into conic sections.

The detector geometry is typically computed using a calibration
source with a known diffraction pattern. I wrote a least-squares
fitting algorithm that can calibrate the diffraction detector by
fitting a known calibration source. I then wrote code to uses the
calibrations to perform a radial integral of the sample data which
can be used to measure the diffraction peaks.

![A screenshot of the calibration tab of the Area Diffraction
Machine.](/assets/area_diffraction_machine_screenshot.jpeg)

I continued working on this project during my senior year of college
as a senior project. I added many features to the code such as an
graphical pixel-masking feature, a macro language for automating
the program, and a detailed manual which describes how to use it.

The program is called the [Area Diffraction
Machine](http://code.google.com/p/areadiffractionmachine), and it
has a robust graphical user interface which allows the program to
be adopted by a general audience of x-ray diffraction scientists.
The code is released under the [GNU
GPL](http://www.gnu.org/licenses/old-licenses/gpl-2.0.html), and
is hosted on [Google
Code](http://code.google.com/p/areadiffractionmachine). At the end
of my SULI internship, I wrote a research paper describing my work
that you can
read [here](/assets/lande_SULI_paper_2007.pdf).

For my work, I was awarded at the end of the summer the [Ernest
Coleman Award for Scholarship and
Citizenship](http://today.slac.stanford.edu/a/2007/08-20.htm):
> The award, named after one of the first directors of the SULI program's predecessor, is nominated by the summer interns themselves. The recipient is chosen based not only on scientific achievement, but also on the support and help given to other interns. And Lande is no exception.
> 
> "He's been fantastic," beamed SLAC beamline scientist Sam Webb, who mentored Lande's summer project. "He's accomplished twice as much as we expected [[from here](http://today.slac.stanford.edu/a/2007/08-20.htm)]

![A picture of me receiving the Ernest Coleman Award for Scholarship
and Citizenship.](/assets/IMG_1680b.jpg)
