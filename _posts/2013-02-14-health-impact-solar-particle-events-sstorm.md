---
layout: post
title: "Quantifying the Health Impact of Solar Particle Events on Astronauts With SStoRM: The Solar Storm Radiation Model"
comments: true
redirect_from: "/2014/02/14/solar-particle-events-sstorm"
permalink: health-impact-solar-particle-events-sstorm
---

![My SEAP internship adviser Ron Turner in front of a poster for SStoRM.](/assets/ron_turner_anser_2007-580x872.jpg)

I participated in the [Science &amp; Engineering Apprentice Program](http://seap.asee.org/program_details) during the summer after my senior year of high school and again the following summer. I worked for [Dr. Ronald Turner](http://www.anser.org/turner-r-publications) at [ANSER](http://www.anser.org/). He studies solar radiation from [solar particle events](http://helios.gsfc.nasa.gov/sep.html) and their health impact on astronauts in outer space. One practical application of his research is the protection of astronauts in outer space from dangerous radiation.

During the internship, I ran a computer simulation of these solar particle events called the BRYNTRN space transport program. This program realistically simulates solar radiation as it penetrates a variable thickness space shielding and it calculates the radiation dose that a man on the other side of the shielding would be exposed to. I used this simulation to calculate the danger that a solar storm would pose to an astronaut in outer space under varying solar storm parameters.

![A screen shot of the time evolution tab of SStoRM.](/assets/sstorm_time_evolution_screenshot.jpeg)

Using this information, I wrote the graphical computer program [SStoRM: The Solar Storm Radiation Model](http://joshualande.github.com/SStoRM/). SStoRM lets users simulate a generic solar storm and modify the storm's parameters such as its energy spectrum and time evolution. Given these, the program presents information on the radiation exposure an astronaut would receive during that particular storm. The program then compares this radiation dose to legislated and biological limits.

It also contains an exercise where a solar storm begins while an astronaut is engaged in extra-vehicular activity and has to flee to shelter. The user can specify certain parameters about how quickly the astronaut can get to their shelter and the program will assess the harm that this particular storm would cause to the astronaut.

[SStoRM](http://joshualande.github.com/SStoRM/) is open source and is hosted on [github](http://joshualande.github.com/SStoRM). It is written in Java so it can be run on any operating system that has [Java Web Start](http://www.oracle.com/technetwork/java/javase/tech/index-jsp-136112.html).You can read my research paper papers I wrote at the end of my internships [here (2004)](/assets/lande_SEAP_paper_2004.pdf) and [here (2005)](/assets/lande_SEAP_paper_2005.pdf).

![A screenshot of the lunar exercise tab of SStoRM.](/assets/sstorm_screenshot.jpeg)
