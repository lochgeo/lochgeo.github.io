---
layout: post
title:  "Transformers - Age of ChatGPT"
date:   2023-06-06 01:22:00
comments: True
categories: [Software, AI]
excerpt_separator: "<!--more-->"
---

A transformer model is a neural network architecture that can automatically transform one type of input into another type of output. It was first introduced in the paper "Attention Is All You Need" by Vaswani et al. (2017). Transformer models are distinguished by their use of self-attention, a mechanism that allows them to learn long-range dependencies between input and output sequences. This makes them well-suited for tasks such as machine translation, text summarization, and question answering.

Transformer models are typically composed of two parts: an encoder and a decoder. The encoder takes the input sequence and produces a sequence of hidden representations. The decoder then takes these hidden representations and produces the output sequence. The encoder and decoder are both made up of a stack of self-attention layers. Each self-attention layer takes the hidden representations from the previous layer and produces new hidden representations that are weighted by their attention to each other. This allows the model to learn long-range dependencies between the input and output sequences.

<!--more-->

I am a total noob in this area and the video linked below gave me a good introduction to the Transformer architecture. Andrej explains the basic concepts of Transformer models in a clear and concise way. He also provides complete code for implementing a Transformer model in PyTorch. Its a good introduction for anyone wondering all the fuss is about generative AI.

<iframe width="480" height="320" src="https://www.youtube.com/embed/kCc8FmEb1nY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>