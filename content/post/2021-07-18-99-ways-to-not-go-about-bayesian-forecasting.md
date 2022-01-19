---
title: '99 ways to not go about Bayesian Forecasting'
# author: Meenal Jhajharia
date: '2021-07-18'
categories:
  - Blog
tags:
  - GSoC
  - Time Series
  - PyMC
  - Probabilistic Programming
---

Long time no see huh? it’s been a long, quiet and eventful summer so far. In the first week of July, I started working on expanding time series modelling capabilities in PyMC3. 

I find myself at a loss of coherent statements to stitch together the last five weeks or so. Anyhow, let’s start, I spent the first two weeks reading about how other people implemented ARMA / ARIMA / SARIMA / SARIMAX. It was overwhelming and mostly resulted in me exacerbating my own impostor syndrome :P

Before, we go ahead, here are a few things:

- [mjhajharia/sarimax-pymc: SARIMAX Model classes for Pymc3 (GSoC'21) (github.com)](https://github.com/mjhajharia/sarimax-pymc) - I am storing all my code (useful and experimental both), along with good resources on learning more about Time series.
- Here are my notes on [ARMA](https://meenal.xyz/arma.html), hope they’re useful!!!

I will provide some context about these models, after all, apparently, these blogs should be useful, and not just me venting into the abyss. So what was I *actually* doing? Say, you have certain data of monthly rainfall over a few years, now you would want to *predict* or estimate or I should say forecast future rainfall trends? that’s Time Series Analysis for you, It’s simply you regressing on data to find a pattern and make predictions, the difference is the data has timestamps (usually linear).

Essentially, it’s a Bayesian version of a simple model to make predictions. Bayesian Inference *tries* to estimate the parameters of the model by treating them as Random Variables instead of fixed values. So, we set a prior(our prior beliefs or presumptions about the parameters), which is followed by evaluating the model likelihood et cetera. All of this is beyond the scope of this blog post, but look out for [Thomas Wiecki’s upcoming course](https://twiecki.io/pages/an-intuitive-guide-to-bayesian-statistics.html). So, back to our model - the end goal of my summer at PyMC3 is building a class of models called **S**easonal **A**uto**R**egressive **I**ntegrated **M**oving **A**verage with e**X**ogenous factors, or SARIMAX. Time series models can be generally broken down into three components for analysis and computation:



Now, Seasonal is, um, just the Seasonality component; then there’s residuals / white noise, recently I’ve come across other fancy (and [useful specific ways of talking about the interchangeably used  terms](https://stats.stackexchange.com/questions/130900/error-terms-vs-innovations)). Let’s cut to the meat now, shall we? so SARIMAX is basically a certain combination of an AR and MA process. Autoregression models rely on past values of the variable, that is to say, it is a regression of the variable against itself. On the other hand, Moving Average models use the errors in past forecasts in a linear regression form. ARMA combines these, so far so great? *but* ARMA does not work on non-stationary data. oops(?) It’s alright, we can difference the data and make it Stationary, that’s ARIMA for you. What if my data has seasonal components too? SARIMA. Sounds robust enough, but oh wait, what if I want to account for external weather forecast variables in my model, something that isn’t affected by the variables present in the system (these are exogenous variables), well my man then we have SARIMAX. This was a bottom-up approach.

Alright, we have enough context, let’s go back to my story now, so yes, it’s been two weeks, I have not written a single line of code, I keep asking my mentors about which fancy idea should I implement, they’re amazingly supportive, but I need to actually *start* somewhere. I mostly spent this time contributing to issues that were *not* a part of my project, I think I was just procrastinating because I was unsure if I could do it. [Junpeng](https://github.com/junpenglao) recommended looking at [Varstan/sarima.stan](https://github.com/asael697/varstan/blob/master/inst/stan/Sarima.stan)) for starters, thanks for the amazing work [Asael](https://github.com/asael697)!! Great, I have an implementation which is Bayesian, now I just need to incorporate it into PyMC3, fair enough. There’s a fun part, it’s in Stan (shout-out to Stan-devs, amazing library for statistical stuff), I’ll admit: I can barely write code in C based stuff. I figured, I will simply convert all of the stan code to python with PyMC3 and numpy, and later deal with it. That wasn’t too hard, now I have 200 lines of code, that doesn’t work, that I don’t *quite* understand. I think I can still wing it :sobs.

I couldn’t xD, so [PyMC3](https://github.com/pymc-devs/pymc3) uses Aesara for computational operations on multi-dimensional arrays. It’s like Numpy, but better in the context of Statistical Modelling. Previously, Theano was used, so [Aesara]([aesara-devs/aesara: Aesara is a Python library that allows one to define, optimize, and efficiently evaluate mathematical expressions involving multi-dimensional arrays. (github.com)](https://github.com/aesara-devs/aesara)) is new and not robustly documented as of now, anyhow, after asking 2372832 questions a day in PyMC slack channels i converted all the code to Aesara + PyMC v4, it was a messy Pull request that I have deleted, half of the model was in *init*. Clearly, that didn’t work, I also made a Gist in this week from statsmodels ARIMA implementation, using that model instance, just running Bayesian parameter estimation using an Op class for Aesara, that was working, but not an in-house PyMC3 solution. I know, I’ve talked about a lot of things that didn’t work, so the last thing I tried was, writing code and not testing it, and simultaneously looking at 7 different implementations of ARIMA models. At the very best, the whole thing was an incredibly profound hotchpotch. I would’ve mentioned all the ARIMA implementations I came across in case it helps anyone, for references, [Bayesforecast](https://github.com/asael697/bayesforecast) is a good place, selfishly, I’d say wait for a month, we’ll have a PyMC SARIMA class up and running for usage and references :) but well, there were a few issues:

- [auto.Arima()](https://otexts.com/fpp2/arima-r.html) etc are in R and hence out of the question
- Many of the models weren’t Bayesian
- Most were not being actively developed.

I was whining about my lack of progress, and [Marco](https://github.com/MarcoGorelli) suggested taking a simpler approach, I could simply implement ARMA, then iteratively test it and make it an ARIMA model, and so on. Now, ARMA is a hundred times simpler than SARIMAX, it has a reliable implementation in the Stan Documentation. Also, AR processes already exist in PyMC3 so simply adding Moving Average component wouldn’t be that difficult. I’ve been doing that now, it feels less overwhelming, I might actually finish my work *somehow*. So, here are some things I’ve learnt / things not to do / things to do that you might find useful!!!!

Things I got right:

- Everything was well planned (Github projects!!)
- Regular communication about where I’m finding potholes in my own work
- I read a *lot* about what I was about to implement
- Looking through a lot of existing implementations

Things I could’ve done better:

- I didn’t spend as much time as I should have on understanding the existing PyMC- Aesara engine. That did catch up to me when I was implementing my own model. LEARN THE LIBRARY YOU WILL BE WORKING ON!!! it sounds incredibly obvious but yes, also, for developers, your documentation, examples, start-up guide are so so so important!!!!!!!! accessibility for the win.
- I should’ve tested every small piece of code I wrote as [Ravin](http://canyon289.github.io/) suggested, whatever you write is too abstract to be useful unless tested. Also, having 5 lines of perfectly tested, working code is better than 50 lines of buggy code.
- I should’ve started from the least ambitious approach, bottom-up is always better, get the simplest prototype working and then iterate, I promise you can add all those *important* things later. 

Things I plan to do now:

- Restructure and Test the ARMA code I have written.
- Create an ARIMA model
- Add Kalman Filter
- Play around with state-space forms of Dynamic Linear models in general 
- Extend ARIMA to SARIMA / SARIMAX and create a separate class of models
- Jupyter Notebooks to explain using these SARIMA models and if possible, a function for model fitting. Right now, except `auto.arima()` in R, I think most models need $$p, d, q$$ in $$ARIMA(p,d,q)$$ to estimate the model parameters, being able to evaluate those in a more or less static approach would be wonderful. I love making notebooks explaining models, it is literally fun, I find them better than the actual model building for some reason.

That’s all folks, don’t be me.