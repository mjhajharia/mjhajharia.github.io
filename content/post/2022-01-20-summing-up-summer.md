---
title: 'Summing up Summer'
# author: Meenal Jhajharia
date: '2021-08-20'
categories:
  - Blog
tags:
  - GSoC
  - Time Series
  - PyMC
  - Probabilistic Programming
---

[Here's the code used/mentioned in this post](https://gist.github.com/mjhajharia)

Summer's over. if i had to sum it up, it's been, um, a lot. yes, also, *a lot* can be good, sometimes. this is technically a blog post summing up google summer of code, but i'm going to go ahead and make this about my summer, for the sake of crying out loud. Anyhow, i started the summer with an ambitious proposal for all the things that i'm going to easily do, and little to no knowledge of how things have a brilliant knack for going wrong. Don't get me wrong, I'm not disappointed, I'm actually very surprised with the amount of stuff i worked on in about three months. Not satisfied (RIP, more on that later!) but surprised, yes. 

To give you some context, here's how modeling works in PyMC3: You create a generator function to generate random samples of a distribution / model, this is essentially tinkering with scipy / numpy random generators. Earlier this happened in a `random` method in the distribution class. Now, PyMC3 has a lot of refactoring going on for a new version, and that changes stuff, we're now creating random distributions with the `RandomVariable` class in Aesara (revamped good old theano).  The only other important part is the method `logp`, evaluting log likelihood of a distribution for a particular value and then optimizing it during sampling to estimate parameters or whatever it is that we need. The latter is taken care of with a robust sampling system, that's not your worry. Now, things get a little tricky, the entire library is being refactored, there's hardly any available code or documentation that's comprehensively functional, every now and then you stumble on a `NotImplementedError`. 

So, let's get to it, I was building time series models for PyMC3. Here's a roadmap (with a lot of hindsight knowledge) of how it went:

- I started with the statsmodels ARIMA distribution, trying to figure out how to make it bayesian. Statsmodels does a lot of stuff, timeseries analysis is just one of the many, so the code base is quite, *inconvenient* for the lack of a better word. Although, i came across [this](https://www.statsmodels.org/dev/examples/notebooks/generated/statespace_sarimax_pymc3.html) and tried to make it work with aesara and PyMC3, here's a gist (quite literally):

  <script src="https://gist.github.com/mjhajharia/0a75d3cdaf60d6dbcd91bb805af80a21.js"></script>

  Seemingly, all I had to do, for an in-house PyMC3 solution was change the Loglike(Op) into a method for the distribution. Now, here's some blockers for starters: statsmodels has many estimation forms for logp, and also stuff like kalman filters which are very useful in time series analysis.

  So this was just a stint, that did *nothing*, but alright, i learned stuff.

  -  Next I went to [Varstan](https://github.com/asael697/varstan), this had a bayesian model, way more realistic to implement, i guess. SARIMAX models have many constraints and somewhat complicated priors to estimate the Moving-average parts. So i started with implementing ARMA and that worked out fine.

  - Now we have a complete PyMC distribution with the entire code, Aesara Random Variable to generate ARMA (we do not generate non stationary data), and estimate ARIMA(p,d,q) with logp.

    - The tests are a work in progress. Testing Time series distributions is a little more complicated than a standard probability distribution with tidy ways of generating and log likelihood evaluation. Time series models on the other hand, especially ones with Moving average components tend to have different ways to generate and estimate the model, which complicates things a little. Here's the key, they're not really distributions, they're models we're trying to fit into a box of a distribution to wrap it in PyMC3. 

      - A little note on the estimation. Some of you might be familiar with scipy lfilter (it's the one used in signal processing), if you aren't i would ask you to read [this](https://warrenweckesser.github.io/papers/weckesser-scipy-linear-filters.pdf). It's essentially just convolution to create a recurring linear series of products. I'm fairly confident we can tweak Aesara.conv to do this as well.

      - About the estimation part, ideally I would use Aesara.scan or as theano users would remember it as theano.scan, it's the same thing. But, Aesara is difficult to wrap your head around initially, especially since it's in a nascent development stage, it's a powerful tool regardless. So, for now, thanks to Austin's advice i'm sticking with linear algebra, we have lagged matrices for observed values and errors from past esimations, and multiplying these matrices is way more computationally efficient than running long for loops in a function that has a sampler on top of it!

      

      Here's the ARMA code: 

      <script src="https://gist.github.com/mjhajharia/62c90a86c121c3cdc70ad43943dacd2b.js"></script>

      Now, that's done, here's a bunch of other stuff I'll be doing in the next few months:

      - Test ARIMA
      - Figure out how to make time series work with Aesara with Kaustabh and Ricardo( i really appreciate you helping me out so much!!!).  When this is done, we can generate practically any time series distribution with a joint log probability distribution.
      - Make tons of example notebooks and case studies about various time series models in PyMC3, on that note checkout the pymc-examples repository, it's a bunch of really cool stuff!!
      - Make SARIMA work, I've already done more than half of the work anyway, so yes, finish that up! [here's the code for that as well](https://gist.github.com/mjhajharia/fd5e6dfe647474d692ddea86a1bd7dab.js)

    So, it's been an incredible summer. Now, I'm going to take the liberty of making this blog my journal, so here we go, I'm so grateful I spent this time working with the open source community and the folks working to make PyMC3 better everyday!! it's been a genuine delight, making mistakes, people correcting you kindly and saying "it's okay we've all been there!!!", the entire team of developers are amazing here. [Ravin](https://github.com/canyon289) and [Chris](https://github.com/fonnesbeck) have been patiently and consistently showing up for me when I'm stuck, or just telling me it's okay when I mess up, it's great to have that. A few other people I'd really like to thank for making my summer memorable are:

    - [Oriol](https://github.com/OriolAbril) was the first person at PyMC3 that helped me out in random Pull requests, I'd always come with a bundle of doubts and he'd patiently unwrangle them for me, I'm so glad I learned more about xarrays and arviz and open source in general from you!! he's genuinely such an amazing and kind person!!
    - In the last few weeks of the summer, when i was struggling to wrap it all up, [Austin](https://github.com/AustinRochford) and [Michael](https://github.com/michaelosthege) would review my pull requests or just point out stupid things i missed and help me ignore my impostor syndrome a lil. It's great to be around smart people who are also incredibly kind!!
    - hi [Sayam](https://github.com/Sayam753), chess was nice, teach me variational inference someday
    - [Asael](https://github.com/asael697) is doing great work on time-series stuff and i've learned so much from him, hope we work together more!!

    
