---
title: Keras
date: 2020-08-03 21:20:55
summary: Keras中文文档笔记，从零开始摸爬滚打~
categories: 机器学习
tags: 
	- 机器学习
	- Keras
---

> “我要忘了你的样子，像鱼忘了海的味道”——像鱼

## 1. Keras Sequentical顺序模型

顺序模型是多个网络层的线性堆叠，可以通过将网络层实例的列表传递给`Sequential`的构造器，创建一个`Sequential`模型：

```python
from keras.models import Sequential
from keras.layers import Dense,Activation

model = Sequential([
    Dense(32, input_shape=(784,)),
    Activition('relu'),
    Dense(10),
    Activition('softmax')
])
```

或者使用`.add()`方法将各层添加到模型中

```python
model = Sequential()
model.add(Dense(32, input_dim=784))
model.add(Activition('relu'))
```

### 指定输入数据的尺寸

模型需要知道它所期望的输入尺寸，所以顺序模型的第一层需要接收关于输入尺寸的信息，其余层可以自动推断尺寸。方法如下：

- 传递一个`input_shape`参数，它是一个表示尺寸的元组 (一个由整数或 `None` 组成的元组，其中 `None` 表示可能为任何正整数)。在 `input_shape` 中不包含数据的 batch 大小。
- 某些2D层，如`Dense`，支持通过参数`input_dim`指定输入尺寸；某些3D时序层支持`input_dim`和`input_length`参数。
- 如果你需要为你的输入指定一个固定的 batch 大小（这对 stateful RNNs 很有用），你可以传递一个 `batch_size` 参数给一个层。如果你同时将 `batch_size=32` 和 `input_shape=(6, 8)` 传递给一个层，那么每一批输入的尺寸就为 `(32，6，8)`。

以下代码是等价的：

```python
model = Sequential()
model.add(Dense(32, input_shape=(784,)))
```

```python
model = Sequential()
model.add(Dense(32, input_dim=784))
```

### 模型编译

在训练模型之前，需要配置学习过程，通过`compile`方法完成。它接收三个参数：

- 优化器optimizer。
- 损失函数loss，模型试图最小化的目标函数。
- 评估标准metrics。

一些例子：

```python
# 多分类问题
model.compile(optimizer='rmsprop',loss='categorical_crossentropy',metrics=['accuracy'])

# 二分类问题
model.compile(optimizer='rmsprop',loss='binary_crossentropy',metrics=['accuracy'])

# 均方误差回归问题
model.compile(optimizer='rmsprop',loss='mse')

# 自定义评估标准函数
import keras.backend as K
def mean_pred(y_true, y_pred):
    return K.mean(y_pred)

model.compile(optimizer='rmsprop',
              loss='binary_crossentropy',
              metrics=['accuracy', mean_pred])
```

### 模型训练

Keras模型在输入数据和标签的Numpy矩阵上进行训练，使用`fit`函数：

```python
# 对于二分类
model = Sequential()
model.add(Dense(32, activation='relu',input_dim=100))
mdoel.add(Dense(1,activation='sigmoid'))
model.compile(optimizer='rmsprop',loss='binary_crossentropy',metrics=['accuracy'])

# 生成虚拟数据
import numpy as np
data = np.random.random((1000, 100))
labels = np.random.randint(2, size=(1000, 1))

# 训练模型，以 32 个样本为一个 batch 进行迭代
model.fit(data, labels, epochs=10, batch_size=32)
```

```python
# 对于多分类
model = Sequential()
model.add(Dense(32, activation='relu',input_dim=100))
mdoel.add(Dense(10,activation='softmax'))
model.compile(optimizer='rmsprop',loss='categorical_crossentropy',metrics=['accuracy'])

# 生成虚拟数据
import numpy as np
data = np.random.random((1000, 100))
labels = np.random.randint(10, size=(1000, 1))

# 将标签转换为分类的 one-hot 编码
one_hot_labels = keras.utils.to_categorical(labels, num_classes=10)

# 训练模型，以 32 个样本为一个 batch 进行迭代
model.fit(data, one_hot_labels, epochs=10, batch_size=32)
```

### 样例

#### 基于多层感知器（MLP）的softmax多分类：

```python
import keras
from keras.models import Sequential
from keras.layers import Dense,Activation,Dropout
from keras.optimizers import SGD

# 生成虚拟数据
import numpy as np
x_train = np.random.random((1000, 20))
y_train = keras.utils.to_categorical(np.random.randint(10, size=(1000, 1)), num_classes=10)
x_test = np.random.random((100, 20))
y_test = keras.utils.to_categorical(np.random.randint(10, size=(100, 1)), num_classes=10)

model = Sequential()
# Dense(64) 是一个具有 64 个隐藏神经元的全连接层。
# 在第一层必须指定所期望的输入数据尺寸,在这里，是一个 20 维的向量。
model.add(Dense(64, activation='relu', input_dim=20))
model.add(Dropout(0.5))
model.add(Dense(64, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(10, activation='softmax'))

sgd = SGD(lr=0.01, decay=1e-6, momentum=0.9, nesterov=True)
model.compile(loss='categorical_crossentropy',
              optimizer=sgd,
              metrics=['accuracy'])

model.fit(x_train, y_train,
          epochs=20,
          batch_size=128)
score = model.evaluate(x_test, y_test, batch_size=128)
```

#### 基于多层感知器的二分类：

```python
import numpy as np
from keras.models import Sequential
from keras.layers import Dense, Dropout

# 生成虚拟数据
x_train = np.random.random((1000, 20))
y_train = np.random.randint(2, size=(1000, 1))
x_test = np.random.random((100, 20))
y_test = np.random.randint(2, size=(100, 1))

model = Sequential()
model.add(Dense(64, input_dim=20, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(64, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(1, activation='sigmoid'))

model.compile(loss='binary_crossentropy',
              optimizer='rmsprop',
              metrics=['accuracy'])

model.fit(x_train, y_train,
          epochs=20,
          batch_size=128)
score = model.evaluate(x_test, y_test, batch_size=128)

```

#### 类似 VGG 的卷积神经网络：

```python
import numpy as np
import keras
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten
from keras.layers import Conv2D, MaxPooling2D
from keras.optimizers import SGD

# 生成虚拟数据
x_train = np.random.random((100, 100, 100, 3))
y_train = keras.utils.to_categorical(np.random.randint(10, size=(100, 1)), num_classes=10)
x_test = np.random.random((20, 100, 100, 3))
y_test = keras.utils.to_categorical(np.random.randint(10, size=(20, 1)), num_classes=10)

model = Sequential()
# 输入: 3 通道 100x100 像素图像 -> (100, 100, 3) 张量。
# 使用 32 个大小为 3x3 的卷积滤波器。
model.add(Conv2D(32, (3, 3), activation='relu', input_shape=(100, 100, 3)))
model.add(Conv2D(32, (3, 3), activation='relu'))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Dropout(0.25))

model.add(Conv2D(64, (3, 3), activation='relu'))
model.add(Conv2D(64, (3, 3), activation='relu'))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Dropout(0.25))

model.add(Flatten())
model.add(Dense(256, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(10, activation='softmax'))

sgd = SGD(lr=0.01, decay=1e-6, momentum=0.9, nesterov=True)
model.compile(loss='categorical_crossentropy', optimizer=sgd)

model.fit(x_train, y_train, batch_size=32, epochs=10)
score = model.evaluate(x_test, y_test, batch_size=32)
```









































