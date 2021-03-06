# [Scalable Machine Learning](https://hackmd.io/@distributed-systems-engineering/scalable-ml)

###### tags: `data-engineering`

course website: https://event.cwi.nl/lsde/2019/ml.shtml

## Overview

==Neural networks== consists of layers of nodes, where the neighboring layers have connections between these nodes. The layers can be ==Fully Connected (FC)== or partially connected, e.g. to preserve the spatial structure of an image (==convolution layers==). An input signal spreads forward over these connections, which have a weight, as the product of weight and input signal. The "score" of a network node is the sum of all incoming weighted signals. It is fed into an ==activation function==; popular such functions are ==tanh, sigmoid or ReLU== (the most popular). The final activation of a node is produced by a ==loss function== that takes into account a ==regularization== term. This regularization is intended to penalize complex models over simple ones. The models are trained by taking the derivative over the activation (the "gradient") and ==backpropagating the error== (the difference between output and correct supervised training answer) from the output side, layer by layer. The weights are then adjusted, weighted by a ==learning rate==.

It is perfectly possible to implement neural networks in statistics packages such as NumPy or R. But, this is not the easiest and also not the fastest way to work with neural networks. ==Deep learning frameworks such as Tensorflow and PyTorch== allow one to specify a network topology, score, activation and loss functions; and then generate most machinery for you. Specifically, they generate the code to compute gradients, and handle all backpropagation.

TensorFlow is the deep learning framework developed by Google Brain. Its language allows to create a (static) neural network specification that is then compiled by TensorFlow into efficient training (e.g. doing operator fusion) and deployment programs (e.g. doing quantization). These programs run on CPUs, but also GPUs and TPUs (as discussed later). PyTorch is a similar framework, developed by Facebook. Rather than a fully new language, it embeds neural network specifications in plain Python. This creates more flexibility to experiment and to make neural processing flexible, but also means less optimization is possible and the deployed model will always depend on python. ==Keras== is an example of an even higher level interface for deep learning. It comes with predefined optimizer strategy, and loss, regularization and activation functions, and works on top of TensorFlow.

==Neural networks consist of weights and weights are numbers==. When looking deep down at how this is in the end handled by computer hardware, typically, these weights are represented in one of these ways:

- ==32-bits or 16-bits floating point numbers (FP32, FP16)==. A floating point number consists of a mantissa (integer number), that is to be multiplied by some power of ten. When people use floating point numbers, typically some rounding error occurs because the numbers have a big range, but only 65536 (2 to the power 16) resp. 4 billion (2 to the power 32) points in that range are exactly representable as FP16 resp FP32.
- ==8-bits integer (aka byte), or 16-bits integer==. While these are only whole numbers between 0 and 255 (resp 65535), or if signed between -128 and 127 (resp -32768 to 32767 for 16-bits), they can be turned into fractional numbers (decimals) by putting an imaginary dot in the binary number. This is called a ==fixed-precision number representation==.

==Calculations on bytes are 10x cheaper than in FP32 in terms of area on the chip as well as energy consumption==. ==Another energy factor is memory access==: it is very expensive to access memory beyond the chip.
Regarding scalability in deep learning, the following hardware/scalability challenges appear:

- training ever more complex models on ever more data takes enormous computational power. This means it takes a lot of time, or a lot of hardware, and it always takes a lot of energy.
- models are becoming ever larger. This makes it harder to deploy them, e.g. to mobile devices.
- the energy consumption of even model deployment (let alone training) can be high, causing high cost when deploying models on a server in the cloud, or draining your mobile battery quickly.

To counter this we have discussed a number of optimizations that make models smaller or reduce the computational needs of model training and deployment.

- ==Node pruning==: throw away weights close to 0; i.e. weights that hardly affect the outcomes. Retrain the remaining network. ==This technique can reduce the size of the network by 10x without precision loss==.
- ==Trained quantization==: do a k-means cluster on the weight values; create a dictionary of k center-points. Then use log2(k) bits to store the weights. One can apparently go down to 2-bits for FC layers and 4-bits for convolutions.
- ==Standard quantization==: it generally acknowledged that training should happen using FP32, but after training the network and creating a deployable version, ==one can switch to fixed-point numbers==.
- ==Mixed-precision training==: during training one can compute gradients (deltas) in FP16; but keep the weights in FP32. Most calculations then are in FP16. Doing most in FP16 saves a lot of energy. Or, because a FP16 instruction take less chip area, the hardware can contain even more execution units and hence is capable of doing more work in parallel. ==The Volta GPU introduces mixed FP32/FP16 instructions to take advantage of this==. CPUs have evolved along Moore's law but their speed has not improved much in the last 10 years, and the amount of cores per chip is also no longer increasing. More power now comes from so-called ==SIMD instructions (a Single Instruction performs an operation on Multiple Data items)==. SIMD has been in CPUs for 20 years (e.g. MMX, SSE, AVX instruction sets). This feature evolved from 64-bits SIMD to 128-bits in 15 years, but recently has grown through 256-bits to 512-bits SIMD quickly in new CPU designs. SIMD with 512-bits means: one "+" (plus), * (multiply), etc. instruction works on 64x8-bits numbers (or 32x16-bits, or 16x32-bits). Intel has a launched a chip for deep learning called Knights Landing (KNL) that provides 7 TFLOP (FP32) thanks to SIMD. With that, this special KNL CPU is the fastest CPU for machine learning by some margin.

Despite that, the deep learning community has already switched to training with GPU (graphics processors). GPUs arguably enabled the breakthrough of deep learning. A CPU chip has few powerful cores that can do very different things at the same time. To make that possible, the chip needs to contain a lot of control logic and cache memories. In a GPU chip there is almost no memory or control logic, it consist of very many simple cores. These GPU cores must all do the same thing at the same time on different data -- it is like a 1000x SIMD processor. ==GPUs are great for doing heavy computation in parallel==. GPUs traditionally come on graphics cards that are placed in an extension slot of the computer. Graphics cards have their own memory, which is faster than normal memory, but also smaller. The higher bandwidth this provides is needed to feed the GPU with input data quickly enough. One has to be careful that the GPU does not become bandwidth-bound, the deep learning model must fit in this memory.

Programming GPUs is hard, one has to use low-level programming interfaces like ==CUDA (NVIDIA only)==, or ==OpenCL (slower)==. All major deep learning libraries (TensorFlow, PyTorch, MXNET, etc) support training and model evaluation on GPUs. ==Machine Learning training is at least 10x faster on GPUs than on CPUs==.

NVidia is market leader in GPUs. Its current architecture is codenamed Pascal (1.5x faster than KNL for ML), and it has announced its new GPU architecture codenamed Volta. Volta is 1.5x faster than Pascal (10 TFLOP) as a GPU, and even 12x faster thanks to new so-called TensorCores (that provide 120 TFLOP for ML training). ML applications are thus influencing mainstream GPU designs.

To make deep learning scalable, one can try to parallelize it across multiple devices (e.g. GPUs), that may also be placed in different computers, connected by some network. Given so many layers, nodes and multiplications and additions that are independent, this should be scalable. However, parallel speedup is as always limited by communication overheads. Here are some approaches to do parallel training:

- ==hyper-parameter search==: build totally independent models in parallel, using different neural network configurations, score/activation/loss functions. Here, there are zero dependencies and this will scale. The TensorFlow-spark integration allows (only) such paralellism.
- ==Data parallel training==: training happens in mini-batches. The idea is to give different GPUs different batches. There is still a need to communicate the deltas between all GPUs, though, and this communication can slow down the training. Also, this effectively multiplies the mini-batch size by the amount of GPUs and too big batches will deteriorate the convergence speed (such that the speed gain thanks to parallel processing is diminished by training taking longer).
- ==Model parallel==: split the model over different GPUs. This works very well for convolution layers, since these are only locally connected. Partitioning the input image spatially over GPUs therefore requires little communication, only in the fringes of the partitions. For fully connected layers the idea can also be applied: the weights are partitioned. Still, during training the activations need to be communicated at every iteration.

Can parallelism make deep learning training scalable? Most deep learning frameworks offer multi-GPU training, as long as these multiple GPUs are located inside the same computer. This tends to scale nicely, but the practical maximum number of GPUs in a machine is 8 or so. Please note that NVIDIA GPUs inside the same machine can be directly connected to each other with an ==NVlink cable, which has much higher bandwidth (up to 300GB/s in Volta)==, and is 100x faster than a fast computer network (10Gbits=1GByte/sec ethernet). Therefore, ==parallelism works best between multiple GPUs on the same machine==.

Not many frameworks support distributing machine learning over multiple computers. It is best supported in Distributed TensorFlow, which offers various distribution methods and parameter synchronization algorithms, but scalability is not trivial to achieve with it.

## References

- Course Materials
    - [Scalable Machine Learning](https://github.com/cyyeh/large-scale-data-engineering/blob/master/machine-learning/05-Scalable%20Machine%20Learning.pdf)
- Supplementary Materials
    - [In-Datacenter Performance Analysis of a Tensor Processing Unit](https://github.com/cyyeh/large-scale-data-engineering/blob/master/machine-learning/tpu.pdf)
    - [Deep Compression: Compressing Deep Neural Networks with Pruning, Trained Quantization and Huffman Coding](https://github.com/cyyeh/large-scale-data-engineering/blob/master/machine-learning/deepcompression.pdf)
    - [Distributed TensorFlow (TensorFlow Dev Summit 2017)](https://www.youtube.com/watch?v=la_M6bCV91M)
    - [CS231n: Convolutional Neural Networks for Visual Recognition](http://cs231n.stanford.edu/)
    - [Transfer learning with a pretrained ConvNet](https://www.tensorflow.org/tutorials/images/transfer_learning)
    - [Deep Learning with PyTorch: A 60 Minute Blitz](https://pytorch.org/tutorials/beginner/deep_learning_60min_blitz.html)
