---
title: Machine Learnng endeavours
date: 2021-01-05 21:00:00 +13:00
categories: [Machine Learning]
tags: [Machine-Learning, ML, F#, Python. Mcts, Deep-Learning, Keras]
description: Overview of project on solving the card game spider solitare
published: true
image: /assets/img/2021-01-03-playing-cards.jpg
---

# Overview of the project 

[Spider Solitare](https://en.wikipedia.org/wiki/Spider_(solitaire)) is a card game. It is similar to solitaire, but I believe it bit harder (particularly the four suit variation). 

Several versions can be playing online for free. 

This project has a few simple goals, some of which have been achieved:

- Learn F# to code up a game
    - This was targetting a 'clean code' approach
    - Understand testing (property-based testing) in F#
- Understand how deep learning works    
    - GPU requirements
    - Different architectures (CNN, LSTM, etc)
- Can I code something super-human?
    - Can the software play a game better than I?
    - The media leads us to believe this
- What skill-set is required to be good at Machine-Learning
    - How to measure the progress of the software?
    - What are the data requirements? 
- Will deep learning take my Job?
- A fun side-project

I could have picked up any project to use for the deep-learning aspect. I choose to use my own approach because I wanted a strong understanding of how the technology worked. 
I have found that following online examples and tutorials do not always provide a detailed understanding as all the hard parts of the problem have already been solved. Once the tutorial is applied to a different challenge, it requires customizing at best or a completely new approach in the worst case. 

# Current State of the project

I feel that I have met the first few goals (though my code may not reflect this). The last one is a blog post itself; however, I also feel I have a sufficient understanding of how deep-learning will affect our jobs. 

The reaming goals on this project for me are

- Can I code something super-human?
- What skill-set is required to be good at Machine-Learning
- A fun side-project

# Current solution overview

## Strategy

The approach is a basic version of how AlphaGo works:
- build a simulator of the game
- Use MCTS to play game
- Evaluate the MCST with two neural networks 
    - policy network (which is the next best move)
    - value network (how good is the state we are in)
    - train each network iteratively using supervised learning. 

## Technology    

The game and MCTS algorithm were coded in F# (as this was a learning goal). F# was not yet up to the task of deep learning, so Python with Keras was used to define the neural networks. 

To train, the MCTS algorithm is run and the games are written out to a CSV file. That file is then fed to the Python neural networks to train on. Once trained, the neural networks a run via an HTTP layer. The MCTS algorithm queries the neural network via HTTP calls. 

## Performance

For a single suit spider solitaire game, up to 4 'runs' (a run is a term I use for ordering the cards Ace through to King) can be completed. To finish an entire game, 8 runs must be completed. Running on my Macbook Pro 16" takes less than 5 mins to complete. Greater performance can be achieved with longer times, and injection of domain knowledge (I am aiming for a general solution). 

# What I am working on now

Given the performance state above is very low, I am working on greatly improving the performance of the policy network. My current understanding is this the last major section that provides a significant jump in performance. 

To work on this, it has required more attention to the random data that I have collected. The data either had bugs in it, a bad model was used, or there was bias in the data set. Neural networks, it turns out, are extremely sensitive to this. They won't hit 100% if the data is not linearly separable. 

# Background on the project

This project arose organically - mostly out of my lack of coding ability and understanding of the challenge. At university, I wrote a python implementation of Sudoku, along with an algorithm to solve it. 10 games could be solved in about 3 seconds. If you like solving them; my advice is to NOT do this. I haven't played Sudoku much since (In my mind it's a solved problem). 

I took this line of thinking to Spider Solitare - how hard can it be? At the time I did not put any thought into the difficulty of the game. Agile, T-Shirt sizing, and the sorts are used in business for a reason! In addition to under-estimating the challenge of solving the game, I also under-estimated the challenge of writing the game. I only lasted a few months when I initially began. 

I picked the challenge back up when I wanted to learn F# and property-based testing. I was told that these are good tools to reduce the number of bugs, so it seemed like a good fit. 

After mostly, solving the game implementation I took up the challenge of the machine-learning aspect. AlphaGo was an inspiration for this, as DeepMind is stating that this can play better for the developers. 

Will there be a day, when we as developers can write software to pass a test that we can't pass as humans? 

Since then, I have to fail at many different approaches to solving spider solitaire. Given that I am learning machine-learning techniques I have stuck with the project. I'm also making progress to solving it, given that it can now get through a few runs for many games it has never been trained on. 


