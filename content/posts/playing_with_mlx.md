+++
title = 'Playing with Apple’s MLX'
date = 2025-12-25T03:12:31-05:00
draft = false
+++

Earlier this year at WWDC 2025 (Apple's annual developer conference) I got to meet the people who work on Apple's frameworks and APIs. Most notably, I spoke with the people working on the MLX framework. While I'd heard of MLX in passing, I had no experience with it. I was surprised to see how close it is to PyTorch and how easy it was to jump into. I've played with it since then and wanted to make an intro post around it.


## Precursor - what frameworks already exist?

| Framework    | Language             | Some (not all) use cases                                                              |
|--------------|----------------------|---------------------------------------------------------------------------------------|
| Scikit-learn | Python               | Basic model/data analysis                                                             |
| Keras*       | Python               | Pre-built layers, Sequential API                                                      |
| Tensorflow   | Python, C/C++, JS    | General Deep Learning model training                                                  |
| JAX          | Python               | Distributed CPU/GPU/TPU model training, numerical computing, Just In time compilation |
| PyTorch      | Python C/C++         | General Deep Learning model training                                                  |
| MLX          | Python, Swift, C/C++ | Unified Memory, Lazy and dynamic computation                                          |

 *Note: Keras is wrapped around JAX, Tensorflow or PyTorch


 There's a lot of overlap between MLX and other ML libraries, but in general MLX is trying to address two areas:
1) Optimization for devices that share memory between the CPU and GPU (Apple Silicon)
2) Lazy evaluation AND dynamically constructed graphs

The MLX framework specifically addresses the second point with function transformations that optimize your computation graph or the differentiation processes themselves. 


## What is MLX?

For training, MLX takes advantage of the Unified Memory model that Apple's Silicon devices (iPhone, Macbook Pro, etc.) have. Non-unified Memory devices have the CPU and GPU be fully separate components, but on Apple Silicon, they're literally on the same chip. This has the advantage of sharing data between the CPU and GPU way faster and splitting operations between them with less complexity.

MLX let's you tinker with what unit you want to compute on, while also stepping in to prevent race conditions or breaks in your computation graph. MLX also has some starter layers and modules (`mlx.nn`), while also providing some faster operations (`mlx.fast`) if you want to accelerate some operations.

The lazy evaluation means while you may specify operations (e.g. addition, multiplication), when combined with the dynamically constructed graphs, changing the size of your inputs won't massively slow down your compilation.

**Note:** MLX is different from Apple's [CoreML framework](https://developer.apple.com/documentation/coreml) which you'd use for loading a model you've already built elsewhere and customizing/personalizing it on-device.


## Get started


Install the mlx package
	Via uv: `uv add mlx`
    Or via pip: `pip install mlx`
You can also install it yourself via git or build a CUDA version with the instructions [on the offical site](https://ml-explore.github.io/mlx/build/html/install.html)

Many of the array operations come straight from numpy, with many of operations not actually being executed until their evaluated. For example,

```
import mlx.core as mx

a = mx.array([1,2,3,4])
b = mx.ones(4)
c = a+b
print(c)
```

The line `c = a+b` only builds the computation graph, while the calculation isn't actually done until I run `print(c)` and try to see `c`'s value. 

---
To take advantage of the unified memory of Apple Silicon devices, during evaluation, MLX uses its lazy evaluation procedure to determine how to break up the work of your computational graph across your GPU/CPU:

```
import mlx.core as mx

x_input = mx.array([0.0, (mx.pi/2), (mx.pi/2)*3, mx.pi])

sin_input = mx.sin(x_input)
cos_input = mx.cos(x_input)

addition = mx.add(sin_input, cos_input)
difference = mx.subtract(sin_input, cos_input)
```

 However, you can also specify where the computation is run:

```
import mlx.core as mx

x_input = mx.array([0.0, (mx.pi/2), (mx.pi/2)*3, mx.pi])

sin_input = mx.sin(x_input)
cos_input = mx.cos(x_input)

addition = mx.add(sin_input, cos_input, stream=mx.cpu)
difference = mx.subtract(sin_input, cos_input, stream=mx.gpu)

```

 
---
For building a basic neural network, the `mlx.nn` module supports basic layers, similar to PyTorch. You can create a subclass of `mlx.nn.Module` and fill in the calls to `__init__()`, which is called on the object's initialization and `__call__()`, which like PyTorch's `forward()` is called when a forward pass is done:


```
import mlx.core as mx
import mlx.nn as nn

class basicNetwork(nn.Module):
    def __init__(self, in_dims: int, out_dims: int):
        super().__init__()

        self.layers = []

    def __call__(self, x):
        return x

```


---

There's also more tricks to play with (distributing computation across devices, using a CUDA backend, Interfacing with the Metal framework for specialized GPU operations, function transformations, ...) but I'll have to save that for another blog post.

## In closing
Long term, I believe the pendulum will swing from cloud-based inference, back to local-first designs and at-home models/LLMs. One of the books I've been reading through, [Bayesian Methods for Interaction and Design](https://www.cambridge.org/core/books/bayesian-methods-for-interaction-and-design/721123C200F67FD94DA8DDFD561162A8), has me convinced that there's a real opportunity for the next wave of HCI tools to be not only personalized (e.g. Generative User Interfaces, individual cognitive modeling) and context aware (e.g. Scene Analysis), but also fully local and on-device. MLX seems to be working towards that goal via consumer hardware, but I still have questions including how to take advantage of the Neural Engine, or how to export MLX-trained models directly to CoreML (I'm not sure there's an API for that). I'm excited about this area so I'm looking forward to playing around with MLX more as it develops!

---
No LLMs were used in writing or preparation of this post. 