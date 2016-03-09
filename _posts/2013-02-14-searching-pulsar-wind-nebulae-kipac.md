---
layout: post
title: Searching for Pulsar Wind Nebulae at the Kavli Institute for Particle Astrophysics and Cosmology
comments: true
redirect_from: "/2013/02/14/stanford-university-research/"
permalink: searching-pulsar-wind-nebulae-kipac
---

![The Crab Nebula observed by Hubble.](/assets/crab_nebula_hubble-940x940.jpg)

## Pulsar Wind Nebulae

![An artists rendering of the Fermi Gamma-ray Space Telescope. The image is from http://www.nasa.gov/mission_pages/GLAST/multimedia/glast_rendering3.html](/assets/fermi_gamma_ray_space_telescope-580x773.jpg)

I got a PhD from [Stanford University](http://www.stanford.edu) in
Physics. In graduate school I studied the Gamma-ray emission
from [pulsar wind
nebulae](http://en.wikipedia.org/wiki/Pulsar_wind_nebula). 
A [pulsar](http://en.wikipedia.org/wiki/Pulsar) is a rapidly rotating
neutron star. Pulsars were first discovered in 1967 by [Jocelyn
Bell Burnell](http://en.wikipedia.org/wiki/Jocelyn_Bell_Burnell).
Pulsars are typically powered by the energy released when a neutron
star slows down. Much of this energy is released as a wind of
electrons, which interacts with the interstellar medium. This forms
a diffuse cloud called a pulsar wind nebulae that can be observed
in gamma-rays.

## The Fermi Gamma-ray Space Telescope

During my thesis, I observed pulsar wind nebulae with the [Large Area Telescope (LAT)](http://fermi.gsfc.nasa.gov/science/instruments/lat.html), the main scientific instrument on board the [Fermi Gamma-ray Space Telescope (Fermi)](http://fermi.gsfc.nasa.gov). Fermi, pictured on the right, is a pair-conversion gamma-ray telescope. Cosmic gamma-rays are interesting to study because they originate in the most extreme and energetic astrophysical environments. [Here](http://www-glast.stanford.edu/instrument.html) is a good description of the design of the instrument. Fermi was launched in June of 2008 with a designed mission length of 5 years and a goal of operating for 10 years. [Here](http://fermi.sonoma.edu/multimedia/GLASTPromoVideo.mp4) is an artist rendering of the mission launch.

## ["Fermi-LAT Search for Pulsar Wind Nebulae around gamma-ray Pulsars"](http://iopscience.iop.org/0004-637X/726/1/35)

![The distribution in energy of the gamma-ray emission from the source Westerlund 2 detected by Fermi.](/assets/pulsar_wind_nebulae_search_-HESS-J1023-575-580x580.jpg)

The first detection of gamma-ray emission from a pulsar wind nebula was the crab nebulae, detected by a previous gamma-ray detector called the Energetic Gamma Ray Experiment Telescope ([EGRET](http://heasarc.gsfc.nasa.gov/docs/cgro/egret)) in [1991](http://adsabs.harvard.edu/abs/1993ApJ...409..697N). Since then, the [Vela X](http://iopscience.iop.org/0004-637X/713/1/146) and [MSH 15-52](http://iopscience.iop.org/0004-637X/714/1/927/fulltext) pulsar wind nebulae were detected by Fermi in 2010.

The first paper I was involved with was a search for new pulsar wind nebulae that emit gamma-ray radiation. In this paper, we selected 54 pulsars which had been [previously detected by Fermi](http://arxiv.org/abs/0910.1608). Because the emission from pulsars is pulsating in time, we could select and remove the pulsed emission. In this so-called off-pulse window, we performed a search for gamma-ray emission that could come from pulsar wind nebulae.

![The age of the pulsar (on the x axis) compared to the energy output of the pulsar (on the y axis). The red stars are pulsar wind nebulae detected by Fermi.](/assets/edot_dsquared_vs_age-580x551.jpg)

In this analysis, we discovered a new pulsar wind nebula associated with the [Westerlund 2 region](http://tevcat.uchicago.edu/?mode=1;id=132) (see above). In addition, we performed a population study of four gamma-ray emitting pulsar wind nebula. We observed that young and highly energetic pulsars power gamma-ray emitting pulsar wind nebulae.

The full text of the paper can be read for free at the [arXiv](http://arxiv.org/abs/1011.2076).

## ["Search for Spatially Extended Fermi-LAT Sources Using Two Years of Data"](http://iopscience.iop.org/0004-637X/756/1/5/)

![The supernova remnant RX J1713.7-3946 observed by Fermi. The blue contours are the SNR as observed by H.E.S.S.](/assets/extended_source_search_RX_J1713.7_3946-580x581.jpg)

The next paper I worked on focused on developed new methods to study the spatial structure of sources detected by Fermi. Before Fermi, previous gamma-ray telescopes had relatively low statistics and a notoriously-poor angular resolution. Given its improved angular resolution and better statistics, Fermi was the first gamma-ray detector capable of spatially resolving a large number of gamma-ray sources.

Studying the spatial structure of a Fermi source is important because it is often difficult to find a counterpart for a Fermi source observed at other wavelengths (for example, [radio](http://en.wikipedia.org/wiki/Radio_astronomy), [optical](http://en.wikipedia.org/wiki/Visible-light_astronomy), or [x-ray](http://en.wikipedia.org/wiki/X-ray_astronomy)). The second Fermi source catalog detected 1,873 sources, but 575 of them [could not be associated to a known source counterpart](http://www.nasa.gov/mission_pages/GLAST/news/gamma-ray-census.html). In addition, often there are several potential counterparts to a Fermi source. In many situations, the spatial shape of the observed source can be compared to potential counterparts to uniquely classify the source.

![This monte-carlo simulation was used to show that we could use pointlike to study spatially-extended sources. Each curve represents ~90,000 independent simulations.](/assets/ts_ext_simulation-580x562.jpg)

In the paper, we developed a new method to study spatially-extended Fermi sources. We used the Fermi data fitting-fitting package called pointlike (described [here]({% post_url 2013-02-01-pointlike-fitting-package %})) and added functionality to the program to fit the shape of an assumed spatially-extended Fermi source. We then defined a test for the statistical significance of the detection of extension and validated this test with an involved monte-carlo simulation. Next, we computed the sensitivity of Fermi to detected spatially-extended sources. We then validated this test by applying it to a [active galactic nucleus](http://en.wikipedia.org/wiki/Active_galactic_nucleus), Fermi sources which are known to be pointlike.

![The source HESS J1616-508 (left) and HESS J1615-518 (right). The source on the left is likely a pulsar wind nebula.](/assets/hess_j1614_and_j1616-580x581.jpg)

Finally, we took this new method and applied it to two years of Fermi data to search for new spatially-extended Fermi sources. In our search, we discovered seven new spatially-extended sources, bringing the total number of extended sources to 21. In particular, we detected the source RX J1713.7-3946 to be spatially-extended (see the picture above) and the extended sources HESS J1616-508 and HESS J1615-518 (pictures on the left). The location of the extended Fermi sources is overlaid on a map of the gamma-ray sky below. For these sources, this spatial analysis was important for identifying the nature of the gamma-ray emission. The full text of the paper can be read for free at the [arXiv](http://arxiv.org/abs/1207.0027).

![A map of the gamma-ray sky. Overlaid are the 21 spatially-extended sources observed by Fermi. The orange triangles represent new extended sources discovered by the analysis.](/assets/allsky_extended_sources_color-580x356.jpg)

## The pointlike maximum-likelihood package

During my PhD, I have been involved with the development of a maximum-likelihood package to fit LAT data. In the process, I have learned about software management, code refactoring, interface design, version control, and issue tracking. I have written about the program [here]({% post_url 2013-02-01-pointlike-fitting-package %}).

## PhD Thesis

You can find my PhD thesis through the [Stanford Library](http://purl.stanford.edu/zj578kk6428) or on the [arXiv](http://arxiv.org/abs/1401.6718).

The slides from my PhD talk are on [slideshare](http://www.slideshare.net/joshualande/neutron-star-powered-nebulae).
