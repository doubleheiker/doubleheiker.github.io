---
title: Deep Learning(1)
date: 2019-02-10 22:02:00
tags: [AI,Deep Learning]
categories: [AI]
description: <center>来自Neural networks and deep learning CHAP1</center>
copyright: true
photo: "http://r.photo.store.qq.com/psb?/V13BnVsC23MbK8/6*gsxnav8Tjbcop0*68BpIadQ.kq0EejKKDiLrzCe7U!/r/dLYAAAAAAAAA"
---
---

# Perceptrons

输入x1, x2...是二进制输入，产生一个二进制输出。
感知器可以用作一种决策器。
感知器可以当作NAND门。
	```
	output= 0 if w⋅x+b≤0
		1 if w⋅x+b>0
	```
权重或者偏差（阈值）轻微的变动会引起结果的很大的变化。
假设我们采用感知器（Perceptrons）网络中的所有权重和偏差，并将它们乘以正常数c，c> 0。神经网络的行为不会更改。

---

# Sigmoid neurons

输入x1, x2...在0~1之间，产生一个在0~1之间的输出。
sigmoid函数：σ(z)≡1/1+e^(−z), z ≡ **wx** + b。
	```
output = 1/(1+exp(−∑jwjxj−b))
Δoutput ≈ (∑j∂output/∂wj)Δwj+(∂output/∂b)Δb
	```
权重或者偏差（阈值）轻微的变动会引起结果较小的变化。
假设我们采用Sigmoid神经元网络中的所有权重和偏差，并将它们乘以正常数c，c> 0。当c→∞时，神经网络的行为就是感知器网络的行为。

---

# The architecture of neural networks

输入层、输入神经元
输出层、输出神经元
隐层、隐层神经元
前馈神经网络：前一层的输出作用于后一层的输入（没有回路）
递归神经网络
输入和输出神经元的设计通常比较简单。比如一个64x64的灰度图像是否是“9”，则输入神经元有4096个，输出神经元只有一个

---

# A simple network to classify handwritten digits

启发式方法识别数字（所以直接识别0-9更容易而不是以4位二进制的方式来识别）

---

# Learning with gradient descent

成本函数：C(w,b) ≡ 1/2n(∑x∥y(x)−a∥^2)，a是输出向量
成本函数值越接近0，说明输出值和拟合的输出值越接近
ΔC ≈ (∂C/∂v1)Δv1 + (∂C/∂v2)Δv2, Δv ≡ (Δv1,…,Δvm)^T, 梯度向量∇C ≡ (∂C/∂v1,…,∂C/∂vm)^T
所以 ΔC ≈ ∇C⋅Δv，令Δv = −η∇C，则 ΔC ≈ −η∇C⋅∇C = −η∥∇C∥^2，所以ΔC≤0，则梯度下降ΔC
更新规则：通过 v→v′=v−η∇C 来更新v（位置）值，η：学习率
可以证明，令 ∥Δv∥=ϵ 当 Δv = −η∇C 时，C下降的最大
利用梯度下降，找出最合适的w和b使成本函数值最小：使用权值和偏差代替上式的v，也就是说v有两个分量w、b，那么更新规则变为：
```
wk→w′k=wk−η∂C/∂wk
bl→b′l=bl−η∂C/∂bl
```
请注意，此成本函数的形式为`C = 1/n(ΣxCx)`，也就是说，它是针对个别培训样例的平均成本为 `Cx ≡ ∥y（x）-a∥^2`。实际上，为了计算梯度∇C，我们需要分别为每个训练输入x计算梯度∇Cx，然后对它们求平均值，∇C=1/nΣx∇Cx。那么当训练的输入非常多的时候，会造成神经网络的学习速度低下

**随机梯度下降**：其思想是通过计算随机选择的训练输入的小样本的∇Cx来估计梯度∇C。通过对这个小样本进行平均，我们可以快速得到真实梯度∇C的良好估计，这有助于加快梯度下降，从而学习。即从所有样本中取一部分样本(mini-batch)进行输入，并近似的认为
`[(∑m,j=1)∇CXj]/m ≈ (∑x∇Cx)/n = ∇C`
我们可以通过计算随机选择的小批量的梯度来估计总梯度。
```
wk→w′k=wk−η/m(∑j∂CXj/∂wk)
bl→b′l=bl−η/m(∑j∂CXj/∂bl)
```
然后选择另一个随机选择的小批量进行训练。直到我们用完样本的输入，就说完成了一轮（epochs）训练。那时我们重新开始一论新的训练

**增量学习**：将小批量的大小设定为1。那么输入一个x，就更新权重和偏差。然后重新选择另一个输入，更新权重和偏差。

# Implementing our network to classify digits

```python
"""
network.py
~~~~~~~~~~

A module to implement the stochastic gradient descent learning
algorithm for a feedforward neural network.  Gradients are calculated
using backpropagation.  Note that I have focused on making the code
simple, easily readable, and easily modifiable.  It is not optimized,
and omits many desirable features.
"""

#### Libraries
# Standard library
import random

# Third-party libraries
import numpy as np

class Network(object):

    def __init__(self, sizes):
        """The list ``sizes`` contains the number of neurons in the
        respective layers of the network.  For example, if the list
        was [2, 3, 1] then it would be a three-layer network, with the
        first layer containing 2 neurons, the second layer 3 neurons,
        and the third layer 1 neuron.  The biases and weights for the
        network are initialized randomly, using a Gaussian
        distribution with mean 0, and variance 1.  Note that the first
        layer is assumed to be an input layer, and by convention we
        won't set any biases for those neurons, since biases are only
        ever used in computing the outputs from later layers."""
        self.num_layers = len(sizes)
        self.sizes = sizes
        self.biases = [np.random.randn(y, 1) for y in sizes[1:]]
        self.weights = [np.random.randn(y, x)
                        for x, y in zip(sizes[:-1], sizes[1:])]

    def feedforward(self, a):
        """Return the output of the network if ``a`` is input."""
        for b, w in zip(self.biases, self.weights):
            a = sigmoid(np.dot(w, a)+b)
        return a

    def SGD(self, training_data, epochs, mini_batch_size, eta,
            test_data=None):
        """Train the neural network using mini-batch stochastic
        gradient descent.  The ``training_data`` is a list of tuples
        ``(x, y)`` representing the training inputs and the desired
        outputs.  The other non-optional parameters are
        self-explanatory.  If ``test_data`` is provided then the
        network will be evaluated against the test data after each
        epoch, and partial progress printed out.  This is useful for
        tracking progress, but slows things down substantially."""
        if test_data: n_test = len(test_data)
        n = len(training_data)
        for j in xrange(epochs):
            random.shuffle(training_data)
            mini_batches = [
                training_data[k:k+mini_batch_size]
                for k in xrange(0, n, mini_batch_size)]
            for mini_batch in mini_batches:
                self.update_mini_batch(mini_batch, eta)
            if test_data:
                print "Epoch {0}: {1} / {2}".format(
                    j, self.evaluate(test_data), n_test)
            else:
                print "Epoch {0} complete".format(j)

    def update_mini_batch(self, mini_batch, eta):
        """Update the network's weights and biases by applying
        gradient descent using backpropagation to a single mini batch.
        The ``mini_batch`` is a list of tuples ``(x, y)``, and ``eta``
        is the learning rate."""
        nabla_b = [np.zeros(b.shape) for b in self.biases]
        nabla_w = [np.zeros(w.shape) for w in self.weights]
        for x, y in mini_batch:
            delta_nabla_b, delta_nabla_w = self.backprop(x, y)
            nabla_b = [nb+dnb for nb, dnb in zip(nabla_b, delta_nabla_b)]
            nabla_w = [nw+dnw for nw, dnw in zip(nabla_w, delta_nabla_w)]
        self.weights = [w-(eta/len(mini_batch))*nw
                        for w, nw in zip(self.weights, nabla_w)]
        self.biases = [b-(eta/len(mini_batch))*nb
                       for b, nb in zip(self.biases, nabla_b)]

    def backprop(self, x, y):
        """Return a tuple ``(nabla_b, nabla_w)`` representing the
        gradient for the cost function C_x.  ``nabla_b`` and
        ``nabla_w`` are layer-by-layer lists of numpy arrays, similar
        to ``self.biases`` and ``self.weights``."""
        nabla_b = [np.zeros(b.shape) for b in self.biases]
        nabla_w = [np.zeros(w.shape) for w in self.weights]
        # feedforward
        activation = x
        activations = [x] # list to store all the activations, layer by layer
        zs = [] # list to store all the z vectors, layer by layer
        for b, w in zip(self.biases, self.weights):
            z = np.dot(w, activation)+b
            zs.append(z)
            activation = sigmoid(z)
            activations.append(activation)
        # backward pass
        delta = self.cost_derivative(activations[-1], y) * \
            sigmoid_prime(zs[-1])
        nabla_b[-1] = delta
        nabla_w[-1] = np.dot(delta, activations[-2].transpose())
        # Note that the variable l in the loop below is used a little
        # differently to the notation in Chapter 2 of the book.  Here,
        # l = 1 means the last layer of neurons, l = 2 is the
        # second-last layer, and so on.  It's a renumbering of the
        # scheme in the book, used here to take advantage of the fact
        # that Python can use negative indices in lists.
        for l in xrange(2, self.num_layers):
            z = zs[-l]
            sp = sigmoid_prime(z)
            delta = np.dot(self.weights[-l+1].transpose(), delta) * sp
            nabla_b[-l] = delta
            nabla_w[-l] = np.dot(delta, activations[-l-1].transpose())
        return (nabla_b, nabla_w)

    def evaluate(self, test_data):
        """Return the number of test inputs for which the neural
        network outputs the correct result. Note that the neural
        network's output is assumed to be the index of whichever
        neuron in the final layer has the highest activation."""
        test_results = [(np.argmax(self.feedforward(x)), y)
                        for (x, y) in test_data]
        return sum(int(x == y) for (x, y) in test_results)

    def cost_derivative(self, output_activations, y):
        """Return the vector of partial derivatives \partial C_x /
        \partial a for the output activations."""
        return (output_activations-y)

#### Miscellaneous functions
def sigmoid(z):
    """The sigmoid function."""
    return 1.0/(1.0+np.exp(-z))

def sigmoid_prime(z):
    """Derivative of the sigmoid function."""
    return sigmoid(z)*(1-sigmoid(z))

```

```python
"""
mnist_loader
~~~~~~~~~~~~

A library to load the MNIST image data.  For details of the data
structures that are returned, see the doc strings for ``load_data``
and ``load_data_wrapper``.  In practice, ``load_data_wrapper`` is the
function usually called by our neural network code.
"""

#### Libraries
# Standard library
import cPickle
import gzip

# Third-party libraries
import numpy as np

def load_data():
    """Return the MNIST data as a tuple containing the training data,
    the validation data, and the test data.

    The ``training_data`` is returned as a tuple with two entries.
    The first entry contains the actual training images.  This is a
    numpy ndarray with 50,000 entries.  Each entry is, in turn, a
    numpy ndarray with 784 values, representing the 28 * 28 = 784
    pixels in a single MNIST image.

    The second entry in the ``training_data`` tuple is a numpy ndarray
    containing 50,000 entries.  Those entries are just the digit
    values (0...9) for the corresponding images contained in the first
    entry of the tuple.

    The ``validation_data`` and ``test_data`` are similar, except
    each contains only 10,000 images.

    This is a nice data format, but for use in neural networks it's
    helpful to modify the format of the ``training_data`` a little.
    That's done in the wrapper function ``load_data_wrapper()``, see
    below.
    """
    f = gzip.open('../data/mnist.pkl.gz', 'rb')
    training_data, validation_data, test_data = cPickle.load(f)
    f.close()
    return (training_data, validation_data, test_data)

def load_data_wrapper():
    """Return a tuple containing ``(training_data, validation_data,
    test_data)``. Based on ``load_data``, but the format is more
    convenient for use in our implementation of neural networks.

    In particular, ``training_data`` is a list containing 50,000
    2-tuples ``(x, y)``.  ``x`` is a 784-dimensional numpy.ndarray
    containing the input image.  ``y`` is a 10-dimensional
    numpy.ndarray representing the unit vector corresponding to the
    correct digit for ``x``.

    ``validation_data`` and ``test_data`` are lists containing 10,000
    2-tuples ``(x, y)``.  In each case, ``x`` is a 784-dimensional
    numpy.ndarry containing the input image, and ``y`` is the
    corresponding classification, i.e., the digit values (integers)
    corresponding to ``x``.

    Obviously, this means we're using slightly different formats for
    the training data and the validation / test data.  These formats
    turn out to be the most convenient for use in our neural network
    code."""
    tr_d, va_d, te_d = load_data()
    training_inputs = [np.reshape(x, (784, 1)) for x in tr_d[0]]
    training_results = [vectorized_result(y) for y in tr_d[1]]
    training_data = zip(training_inputs, training_results)
    validation_inputs = [np.reshape(x, (784, 1)) for x in va_d[0]]
    validation_data = zip(validation_inputs, va_d[1])
    test_inputs = [np.reshape(x, (784, 1)) for x in te_d[0]]
    test_data = zip(test_inputs, te_d[1])
    return (training_data, validation_data, test_data)

def vectorized_result(j):
    """Return a 10-dimensional unit vector with a 1.0 in the jth
    position and zeroes elsewhere.  This is used to convert a digit
    (0...9) into a corresponding desired output from the neural
    network."""
    e = np.zeros((10, 1))
    e[j] = 1.0
    return e
```