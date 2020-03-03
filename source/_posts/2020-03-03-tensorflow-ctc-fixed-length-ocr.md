---
title: 使用 TensorFlow 做定长文本识别
date: 2020-03-03 05:15:31
tags:
  - tensorflow
  - ctc
  - cnn
  - ocr
---

实现定长验证码的识别. 使用 `tensorflow` `2.1` 版的 `keras` 高级接口来训练模型.
模型使用了 `cnn`, `rnn`, `ctc` 来预测 `4` 位的验证码.  
项目地址: [ocr_shixin](https://github.com/dust8/ocr_shixin)

```python
import matplotlib.pyplot as plt
import tensorflow as tf
import numpy as np

%matplotlib inline
%config InlineBackend.figure_format = 'retina'
```

## 探索数据

预处理数据可以看 `preprocess.py`

```python
from config import labels_to_text
from data_sequence import ShiXinSequence

sx_sequence = ShiXinSequence('./dataset/binary')

(x_train,y_train,_,_),_ = sx_sequence[0]
plt.title(labels_to_text(y_train[1]))
plt.imshow(np.squeeze(x_train[1],-1),cmap='gray')
```

    <matplotlib.image.AxesImage at 0x1f4aa567bc8>

![png](/assert/2020-03-03-1.png)

## ShiXin 模型

```python
from config import OUTPUT_PATH,TARGET_PATH
from network.model import ShiXinModel


# 创建和编译模型
sx_train_model = ShiXinModel()
sx_train_model.compile()

callbacks = sx_train_model.get_callbacks(logdir=OUTPUT_PATH,checkpoint=TARGET_PATH, monitor="loss",verbose=1)
```

## 训练模型

```python
h = sx_train_model.fit_generator(generator=sx_sequence,epochs=15,callbacks=callbacks,verbose=1)
```

    Epoch 1/15
    345/345 [==============================] - 46s 133ms/step - loss: 15.6317
    Epoch 2/15
    345/345 [==============================] - 44s 128ms/step - loss: 13.4181
    Epoch 3/15
    345/345 [==============================] - 44s 128ms/step - loss: 4.6008
    Epoch 4/15
    345/345 [==============================] - 45s 131ms/step - loss: 0.6347
    Epoch 5/15
    345/345 [==============================] - 45s 130ms/step - loss: 0.2896
    Epoch 6/15
    345/345 [==============================] - 46s 134ms/step - loss: 0.1451
    Epoch 7/15
    345/345 [==============================] - 46s 134ms/step - loss: 0.0937
    Epoch 8/15
    345/345 [==============================] - 47s 137ms/step - loss: 0.0306
    Epoch 9/15
    345/345 [==============================] - 47s 137ms/step - loss: 0.0362
    Epoch 10/15
    345/345 [==============================] - 48s 140ms/step - loss: 0.0111
    Epoch 11/15
    345/345 [==============================] - 49s 142ms/step - loss: 0.0521
    Epoch 12/15
    345/345 [==============================] - 49s 143ms/step - loss: 0.0319
    Epoch 13/15
    345/345 [==============================] - 50s 145ms/step - loss: 0.3152
    Epoch 14/15
    345/345 [==============================] - 51s 146ms/step - loss: 0.1556
    Epoch 15/15
    345/345 [==============================] - 51s 148ms/step - loss: 0.0471

## 预测

因为 ctc 的训练和预测输入和输出不一样,所以需要新建一个模型来载入已经训练好的权重

```python
sx_predict_model = ShiXinModel()
sx_predict_model.compile()
sx_predict_model.load_checkpoint(TARGET_PATH)
```

```python
out = sx_predict_model.predict(x=np.array([x_train[1]]))
out
```

    WARNING:tensorflow:From c:\users\tom\appdata\local\programs\python\python37\lib\site-packages\tensorflow_core\python\keras\backend.py:5783: sparse_to_dense (from tensorflow.python.ops.sparse_ops) is deprecated and will be removed in a future version.
    Instructions for updating:
    Create a `tf.sparse.SparseTensor` and use `tf.sparse.to_dense` instead.





    array([[ 3,  4,  7, 17]], dtype=int64)

```python
from pathlib import Path

from PIL import Image


imgs = list(Path('./dataset/val_binary').glob('*'))

x_v = np.array( [np.expand_dims(np.array(Image.open(filename)), -1) / 255.0 for filename in imgs])
out = sx_predict_model.predict(x=x_v)
for idx ,val in enumerate(out):
    print(f'y_pred:{labels_to_text(val)} y_true:{imgs[idx].name[:4]}')
```

    y_pred:aafe y_true:AAFE
    y_pred:bxmp y_true:bxmp
    y_pred:f4rh y_true:f4rh
    y_pred:fmew y_true:fmew
    y_pred:gemt y_true:gemt
    y_pred:hl8n y_true:hl8n
    y_pred:mrhu y_true:mrhu
    y_pred:rvay y_true:rvay
    y_pred:sdtn y_true:sdtn
    y_pred:txy7 y_true:txy7
    y_pred:y7m8 y_true:y7m8

> 这里可以看到 `aafe` 也正确识别出来了, 如果解码算法有问题的话, 可能只能识别出一个 `a` 字符

## 模型结构

```python
from tensorflow.keras.utils import plot_model
from IPython.display import Image

plot_model(sx_predict_model.model, to_file='ctc.png', show_shapes=True)
Image('ctc.png')
```

![ctc.png](/assert/2020-03-03-2.png)

```python
sx_predict_model.model.summary()
```

    Model: "model_2"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #
    =================================================================
    the_input (InputLayer)       [(None, 70, 160, 1)]      0
    _________________________________________________________________
    conv1 (Conv2D)               (None, 70, 160, 16)       160
    _________________________________________________________________
    batch_normalization_3 (Batch (None, 70, 160, 16)       64
    _________________________________________________________________
    activation_3 (Activation)    (None, 70, 160, 16)       0
    _________________________________________________________________
    max1 (MaxPooling2D)          (None, 35, 80, 16)        0
    _________________________________________________________________
    conv2 (Conv2D)               (None, 35, 80, 16)        2320
    _________________________________________________________________
    batch_normalization_4 (Batch (None, 35, 80, 16)        64
    _________________________________________________________________
    activation_4 (Activation)    (None, 35, 80, 16)        0
    _________________________________________________________________
    max2 (MaxPooling2D)          (None, 17, 40, 16)        0
    _________________________________________________________________
    conv3 (Conv2D)               (None, 17, 40, 16)        2320
    _________________________________________________________________
    batch_normalization_5 (Batch (None, 17, 40, 16)        64
    _________________________________________________________________
    activation_5 (Activation)    (None, 17, 40, 16)        0
    _________________________________________________________________
    max3 (MaxPooling2D)          (None, 8, 40, 16)         0
    _________________________________________________________________
    permute_1 (Permute)          (None, 40, 8, 16)         0
    _________________________________________________________________
    time_distributed_1 (TimeDist (None, 40, 128)           0
    _________________________________________________________________
    gru_1 (Bidirectional)        (None, 40, 1024)          1972224
    _________________________________________________________________
    gru_2 (Bidirectional)        (None, 40, 1024)          4724736
    _________________________________________________________________
    out_dense (Dense)            (None, 40, 37)            37925
    =================================================================
    Total params: 6,739,877
    Trainable params: 6,739,781
    Non-trainable params: 96
    _________________________________________________________________

## 导出模型

```python
!python export.py
```

    Export path: Shixin\1
    Already saved a model, cleaning up
    Saved mode: Shixin\1


    WARNING:tensorflow:From C:\Users\tom\AppData\Local\Programs\Python\Python37\lib\site-packages\tensorflow_core\python\ops\resource_variable_ops.py:1786: calling BaseResourceVariable.__init__ (from tensorflow.python.ops.resource_variable_ops) with constraint is deprecated and will be removed in a future version.
    Instructions for updating:
    If using Keras pass *_constraint arguments to layers.

## 使用 TensorFlow Serving 和 Docker 部署模型

```python
!docker run -p 8501:8501 --mount type=bind,source=d:/workspace/ocr_shixin/Shixin/,target=/models/Shixin -e MODEL_NAME=Shixin -t -d tensorflow/serving
```

    df5d7fe164023375968ae4a7892e7845a5ecde08a95614efc25246262c479fca

## Python 客户端示例

```python
import requests
from tensorflow.keras import backend as K


url = 'http://localhost:8501/v1/models/Shixin:predict'
data = {
    "instances":x_v.tolist()
}

response = requests.post(url,json=data)
out = np.array(response.json()['predictions'])
out = K.get_value(
            K.ctc_decode(
                out,
                np.ones(out.shape[0]) * out.shape[1]
            )[0][0]
        )

for i in out:
    print(labels_to_text(i))
```

    aafe
    bxmp
    f4rh
    fmew
    gemt
    hl8n
    mrhu
    rvay
    sdtn
    txy7
    y7m8

## 参考链接

- [captcha_break](https://github.com/ypwhs/captcha_break)
- [andwritten-text-recognition](https://github.com/arthurflor23/handwritten-text-recognition)
