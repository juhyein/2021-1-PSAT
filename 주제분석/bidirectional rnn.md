```python
import pandas as pd
import numpy as np
import ast
```


```python
from tensorflow.keras.callbacks import EarlyStopping, ModelCheckpoint
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.preprocessing.sequence import pad_sequences
from tensorflow.keras.utils import to_categorical
from tensorflow.keras.models import Sequential
import tensorflow as tf
import tensorflow_addons as tfa
from sklearn.preprocessing import LabelEncoder
```


```python
info_token =  pd.read_csv('RE_REAL_SOGAE_TOKEN.csv')
sogae = info_token['real_sogae_tokens']
sogae_list = sogae.apply(lambda x: ast.literal_eval(x)).tolist()
```


```python
data = pd.read_csv("FINAL_RE_PLC_review_tokenized_okt_30377.csv")
data['review_tokens'] = data['review_tokens'].apply(lambda x: ast.literal_eval(x))
```


```python
tokenizer = Tokenizer()
tokenizer.fit_on_texts(data['review_tokens'])
X = tokenizer.texts_to_sequences(data['review_tokens']) # Sequence 변환
max_len=82
X = pad_sequences(X, max_len)
le = LabelEncoder()
y = le.fit_transform(data.plc)
y = to_categorical(y)
vocab_size = len(tokenizer.word_index) + 2
```


```python
embedding_matrix=np.load('embedding_mat.npy')
```

## Bidirectional rnn 구현


```python
import tensorflow as tf
import matplotlib.pyplot as plt
from tensorflow import keras
from tensorflow.keras import layers
from tensorflow.keras import Sequential, Model
from tensorflow.keras.preprocessing.sequence import pad_sequences
from pprint import pprint
%matplotlib inline
from tensorflow.keras.layers import Dense, Embedding, Bidirectional, SimpleRNN, Concatenate, Dropout, SpatialDropout1D, LayerNormalization
from tensorflow.keras import Input, Model
from tensorflow.keras import optimizers
import os
```

Train / Test Split


```python
'''from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X,y,train_size=0.8, random_state=4)'''
```




    'from sklearn.model_selection import train_test_split\nX_train, X_test, y_train, y_test = train_test_split(X,y,train_size=0.8, random_state=4)'



Hyperparameter Tuning via KerasTuner

## baseline model


```python
model = Sequential()
model.add(Embedding(vocab_size, 300, weights=[embedding_matrix], input_length=max_len, trainable=False))
model.add(SpatialDropout1D(rate =0.3))
model.add(Bidirectional(SimpleRNN(128)))
model.add(LayerNormalization())
model.add(Dense(units = 300, activation='relu'))
model.add(Dropout(rate = 0.3))
model.add(Dense(349, activation='softmax'))
model.compile(optimizer='adam', loss='categorical_crossentropy', metrics=[tfa.metrics.FBetaScore(beta=0.5,average='micro',num_classes=349)])
```


```python
model.summary()
```

    Model: "sequential"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    embedding (Embedding)        (None, 82, 300)           4641300   
    _________________________________________________________________
    spatial_dropout1d (SpatialDr (None, 82, 300)           0         
    _________________________________________________________________
    bidirectional (Bidirectional (None, 256)               109824    
    _________________________________________________________________
    layer_normalization (LayerNo (None, 256)               512       
    _________________________________________________________________
    dense (Dense)                (None, 300)               77100     
    _________________________________________________________________
    dropout (Dropout)            (None, 300)               0         
    _________________________________________________________________
    dense_1 (Dense)              (None, 349)               105049    
    =================================================================
    Total params: 4,933,785
    Trainable params: 292,485
    Non-trainable params: 4,641,300
    _________________________________________________________________
    


```python
early_stopping = EarlyStopping(monitor='loss', patience=5 , mode='min')
model_checkpoint = ModelCheckpoint('fbeta_rnn', monitor = 'fbeta_score', mode = 'max', verbose = 1, save_best_only = True)
```


```python
history=model.fit(X, y, epochs=100,batch_size=256, callbacks=[model_checkpoint,early_stopping])
```

    Epoch 1/100
    119/119 [==============================] - 55s 391ms/step - loss: 5.0400 - fbeta_score: 0.0469
    
    Epoch 00001: fbeta_score improved from -inf to 0.05787, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 2/100
    119/119 [==============================] - 46s 389ms/step - loss: 4.6010 - fbeta_score: 0.0838
    
    Epoch 00002: fbeta_score improved from 0.05787 to 0.08638, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 3/100
    119/119 [==============================] - 49s 412ms/step - loss: 4.4151 - fbeta_score: 0.1063
    
    Epoch 00003: fbeta_score improved from 0.08638 to 0.10327, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 4/100
    119/119 [==============================] - 52s 437ms/step - loss: 4.3397 - fbeta_score: 0.1160
    
    Epoch 00004: fbeta_score improved from 0.10327 to 0.11917, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 5/100
    119/119 [==============================] - 47s 392ms/step - loss: 4.1947 - fbeta_score: 0.1331
    
    Epoch 00005: fbeta_score improved from 0.11917 to 0.13392, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 6/100
    119/119 [==============================] - 48s 399ms/step - loss: 4.1068 - fbeta_score: 0.1463
    
    Epoch 00006: fbeta_score improved from 0.13392 to 0.14531, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 7/100
    119/119 [==============================] - 55s 466ms/step - loss: 4.0174 - fbeta_score: 0.1575
    
    Epoch 00007: fbeta_score improved from 0.14531 to 0.15581, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 8/100
    119/119 [==============================] - 46s 390ms/step - loss: 3.9706 - fbeta_score: 0.1635
    
    Epoch 00008: fbeta_score improved from 0.15581 to 0.16414, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 9/100
    119/119 [==============================] - 47s 398ms/step - loss: 3.9465 - fbeta_score: 0.1614
    
    Epoch 00009: fbeta_score did not improve from 0.16414
    Epoch 10/100
    119/119 [==============================] - 47s 391ms/step - loss: 3.9529 - fbeta_score: 0.1609
    
    Epoch 00010: fbeta_score did not improve from 0.16414
    Epoch 11/100
    119/119 [==============================] - 47s 392ms/step - loss: 3.8453 - fbeta_score: 0.1772
    
    Epoch 00011: fbeta_score improved from 0.16414 to 0.17385, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 12/100
    119/119 [==============================] - 51s 429ms/step - loss: 3.8230 - fbeta_score: 0.1806
    
    Epoch 00012: fbeta_score improved from 0.17385 to 0.18116, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 13/100
    119/119 [==============================] - 50s 420ms/step - loss: 3.7730 - fbeta_score: 0.1857
    
    Epoch 00013: fbeta_score improved from 0.18116 to 0.18817, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 14/100
    119/119 [==============================] - 57s 475ms/step - loss: 3.7096 - fbeta_score: 0.1928
    
    Epoch 00014: fbeta_score improved from 0.18817 to 0.19518, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 15/100
    119/119 [==============================] - 50s 422ms/step - loss: 3.6593 - fbeta_score: 0.2002
    
    Epoch 00015: fbeta_score improved from 0.19518 to 0.19943, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 16/100
    119/119 [==============================] - 48s 398ms/step - loss: 3.9783 - fbeta_score: 0.1564
    
    Epoch 00016: fbeta_score did not improve from 0.19943
    Epoch 17/100
    119/119 [==============================] - 49s 413ms/step - loss: 3.7304 - fbeta_score: 0.1895
    
    Epoch 00017: fbeta_score did not improve from 0.19943
    Epoch 18/100
    119/119 [==============================] - 48s 399ms/step - loss: 3.6284 - fbeta_score: 0.2006
    
    Epoch 00018: fbeta_score improved from 0.19943 to 0.20088, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 19/100
    119/119 [==============================] - 50s 421ms/step - loss: 3.5490 - fbeta_score: 0.2112
    
    Epoch 00019: fbeta_score improved from 0.20088 to 0.21085, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 20/100
    119/119 [==============================] - 49s 412ms/step - loss: 3.5015 - fbeta_score: 0.2201
    
    Epoch 00020: fbeta_score improved from 0.21085 to 0.21599, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 21/100
    119/119 [==============================] - 66s 558ms/step - loss: 3.4831 - fbeta_score: 0.2221
    
    Epoch 00021: fbeta_score improved from 0.21599 to 0.22013, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 22/100
    119/119 [==============================] - 53s 446ms/step - loss: 3.4345 - fbeta_score: 0.2249
    
    Epoch 00022: fbeta_score improved from 0.22013 to 0.22520, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 23/100
    119/119 [==============================] - 60s 502ms/step - loss: 3.4040 - fbeta_score: 0.2310
    
    Epoch 00023: fbeta_score improved from 0.22520 to 0.22678, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 24/100
    119/119 [==============================] - 66s 553ms/step - loss: 3.3786 - fbeta_score: 0.2322
    
    Epoch 00024: fbeta_score improved from 0.22678 to 0.23445, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 25/100
    119/119 [==============================] - 58s 484ms/step - loss: 3.3399 - fbeta_score: 0.2397
    
    Epoch 00025: fbeta_score improved from 0.23445 to 0.23505, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 26/100
    119/119 [==============================] - 53s 447ms/step - loss: 3.3185 - fbeta_score: 0.2429
    
    Epoch 00026: fbeta_score improved from 0.23505 to 0.24074, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 27/100
    119/119 [==============================] - 48s 400ms/step - loss: 3.2961 - fbeta_score: 0.2431
    
    Epoch 00027: fbeta_score improved from 0.24074 to 0.24301, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 28/100
    119/119 [==============================] - 52s 434ms/step - loss: 3.2547 - fbeta_score: 0.2498
    
    Epoch 00028: fbeta_score improved from 0.24301 to 0.24785, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 29/100
    119/119 [==============================] - 51s 428ms/step - loss: 3.2239 - fbeta_score: 0.2478
    
    Epoch 00029: fbeta_score improved from 0.24785 to 0.25019, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 30/100
    119/119 [==============================] - 50s 419ms/step - loss: 3.1966 - fbeta_score: 0.2542
    
    Epoch 00030: fbeta_score improved from 0.25019 to 0.25312, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 31/100
    119/119 [==============================] - 53s 447ms/step - loss: 3.1719 - fbeta_score: 0.2593
    
    Epoch 00031: fbeta_score improved from 0.25312 to 0.25565, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 32/100
    119/119 [==============================] - 51s 427ms/step - loss: 3.1622 - fbeta_score: 0.2589
    
    Epoch 00032: fbeta_score improved from 0.25565 to 0.26122, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 33/100
    119/119 [==============================] - 49s 415ms/step - loss: 3.1275 - fbeta_score: 0.2675
    
    Epoch 00033: fbeta_score improved from 0.26122 to 0.26563, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 34/100
    119/119 [==============================] - 52s 435ms/step - loss: 3.1003 - fbeta_score: 0.2670
    
    Epoch 00034: fbeta_score did not improve from 0.26563
    Epoch 35/100
    119/119 [==============================] - 52s 434ms/step - loss: 3.0950 - fbeta_score: 0.2678
    
    Epoch 00035: fbeta_score improved from 0.26563 to 0.27020, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 36/100
    119/119 [==============================] - 48s 406ms/step - loss: 3.1535 - fbeta_score: 0.2642
    
    Epoch 00036: fbeta_score did not improve from 0.27020
    Epoch 37/100
    119/119 [==============================] - 49s 409ms/step - loss: 3.0956 - fbeta_score: 0.2685
    
    Epoch 00037: fbeta_score did not improve from 0.27020
    Epoch 38/100
    119/119 [==============================] - 48s 403ms/step - loss: 3.0358 - fbeta_score: 0.2754
    
    Epoch 00038: fbeta_score improved from 0.27020 to 0.27471, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 39/100
    119/119 [==============================] - 40s 339ms/step - loss: 3.0240 - fbeta_score: 0.2789
    
    Epoch 00039: fbeta_score improved from 0.27471 to 0.27636, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 40/100
    119/119 [==============================] - 29s 240ms/step - loss: 2.9913 - fbeta_score: 0.2801
    
    Epoch 00040: fbeta_score improved from 0.27636 to 0.27992, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 41/100
    119/119 [==============================] - 38s 317ms/step - loss: 2.9825 - fbeta_score: 0.2865
    
    Epoch 00041: fbeta_score improved from 0.27992 to 0.28255, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 42/100
    119/119 [==============================] - 33s 281ms/step - loss: 2.9514 - fbeta_score: 0.2924
    
    Epoch 00042: fbeta_score improved from 0.28255 to 0.28617, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 43/100
    119/119 [==============================] - 29s 246ms/step - loss: 2.9618 - fbeta_score: 0.2849
    
    Epoch 00043: fbeta_score did not improve from 0.28617
    Epoch 44/100
    119/119 [==============================] - 36s 300ms/step - loss: 2.9185 - fbeta_score: 0.2955
    
    Epoch 00044: fbeta_score improved from 0.28617 to 0.29058, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 45/100
    119/119 [==============================] - 44s 369ms/step - loss: 2.9228 - fbeta_score: 0.2952
    
    Epoch 00045: fbeta_score improved from 0.29058 to 0.29266, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 46/100
    119/119 [==============================] - 42s 354ms/step - loss: 2.9176 - fbeta_score: 0.2932
    
    Epoch 00046: fbeta_score did not improve from 0.29266
    Epoch 47/100
    119/119 [==============================] - 52s 436ms/step - loss: 2.8781 - fbeta_score: 0.3009
    
    Epoch 00047: fbeta_score improved from 0.29266 to 0.29582, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 48/100
    119/119 [==============================] - 33s 280ms/step - loss: 2.8597 - fbeta_score: 0.3030
    
    Epoch 00048: fbeta_score improved from 0.29582 to 0.29710, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 49/100
    119/119 [==============================] - 47s 395ms/step - loss: 2.8441 - fbeta_score: 0.3035
    
    Epoch 00049: fbeta_score improved from 0.29710 to 0.29819, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 50/100
    119/119 [==============================] - 34s 284ms/step - loss: 2.8468 - fbeta_score: 0.3044
    
    Epoch 00050: fbeta_score improved from 0.29819 to 0.30401, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 51/100
    119/119 [==============================] - 48s 404ms/step - loss: 2.8247 - fbeta_score: 0.3053
    
    Epoch 00051: fbeta_score did not improve from 0.30401
    Epoch 52/100
    119/119 [==============================] - 40s 333ms/step - loss: 2.8312 - fbeta_score: 0.3056
    
    Epoch 00052: fbeta_score improved from 0.30401 to 0.30474, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 53/100
    119/119 [==============================] - 38s 322ms/step - loss: 2.8124 - fbeta_score: 0.3104
    
    Epoch 00053: fbeta_score improved from 0.30474 to 0.30819, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 54/100
    119/119 [==============================] - 40s 340ms/step - loss: 2.8226 - fbeta_score: 0.3123
    
    Epoch 00054: fbeta_score improved from 0.30819 to 0.30987, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 55/100
    119/119 [==============================] - 35s 298ms/step - loss: 2.8000 - fbeta_score: 0.3133
    
    Epoch 00055: fbeta_score did not improve from 0.30987
    Epoch 56/100
    119/119 [==============================] - 47s 398ms/step - loss: 2.7806 - fbeta_score: 0.3154
    
    Epoch 00056: fbeta_score improved from 0.30987 to 0.31445, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 57/100
    119/119 [==============================] - 31s 264ms/step - loss: 2.7579 - fbeta_score: 0.3154
    
    Epoch 00057: fbeta_score did not improve from 0.31445
    Epoch 58/100
    119/119 [==============================] - 34s 288ms/step - loss: 2.7625 - fbeta_score: 0.3115
    
    Epoch 00058: fbeta_score did not improve from 0.31445
    Epoch 59/100
    119/119 [==============================] - 52s 439ms/step - loss: 2.7410 - fbeta_score: 0.3200
    
    Epoch 00059: fbeta_score improved from 0.31445 to 0.31728, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 60/100
    119/119 [==============================] - 32s 268ms/step - loss: 2.7425 - fbeta_score: 0.3154
    
    Epoch 00060: fbeta_score did not improve from 0.31728
    Epoch 61/100
    119/119 [==============================] - 31s 261ms/step - loss: 2.7057 - fbeta_score: 0.3303
    
    Epoch 00061: fbeta_score improved from 0.31728 to 0.32004, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 62/100
    119/119 [==============================] - 42s 350ms/step - loss: 2.7204 - fbeta_score: 0.3219
    
    Epoch 00062: fbeta_score did not improve from 0.32004
    Epoch 63/100
    119/119 [==============================] - 38s 320ms/step - loss: 2.7044 - fbeta_score: 0.3255
    
    Epoch 00063: fbeta_score improved from 0.32004 to 0.32209, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 64/100
    119/119 [==============================] - 31s 260ms/step - loss: 3.0004 - fbeta_score: 0.2787
    
    Epoch 00064: fbeta_score did not improve from 0.32209
    Epoch 65/100
    119/119 [==============================] - 38s 317ms/step - loss: 2.8076 - fbeta_score: 0.3088
    
    Epoch 00065: fbeta_score did not improve from 0.32209
    Epoch 66/100
    119/119 [==============================] - 43s 365ms/step - loss: 2.7360 - fbeta_score: 0.3177
    
    Epoch 00066: fbeta_score did not improve from 0.32209
    Epoch 67/100
    119/119 [==============================] - 33s 278ms/step - loss: 2.7375 - fbeta_score: 0.3186
    
    Epoch 00067: fbeta_score did not improve from 0.32209
    Epoch 68/100
    119/119 [==============================] - 40s 338ms/step - loss: 2.6990 - fbeta_score: 0.3253
    
    Epoch 00068: fbeta_score did not improve from 0.32209
    Epoch 69/100
    119/119 [==============================] - 46s 385ms/step - loss: 2.6842 - fbeta_score: 0.3252
    
    Epoch 00069: fbeta_score improved from 0.32209 to 0.32281, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 70/100
    119/119 [==============================] - 33s 280ms/step - loss: 2.6598 - fbeta_score: 0.3303
    
    Epoch 00070: fbeta_score improved from 0.32281 to 0.32834, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 71/100
    119/119 [==============================] - 47s 395ms/step - loss: 2.6654 - fbeta_score: 0.3331
    
    Epoch 00071: fbeta_score did not improve from 0.32834
    Epoch 72/100
    119/119 [==============================] - 36s 299ms/step - loss: 2.6645 - fbeta_score: 0.3325
    
    Epoch 00072: fbeta_score improved from 0.32834 to 0.32883, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 73/100
    119/119 [==============================] - 48s 407ms/step - loss: 2.6031 - fbeta_score: 0.3413
    
    Epoch 00073: fbeta_score improved from 0.32883 to 0.33325, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 74/100
    119/119 [==============================] - 36s 302ms/step - loss: 2.6275 - fbeta_score: 0.3362
    
    Epoch 00074: fbeta_score did not improve from 0.33325
    Epoch 75/100
    119/119 [==============================] - 46s 389ms/step - loss: 2.6285 - fbeta_score: 0.3320
    
    Epoch 00075: fbeta_score did not improve from 0.33325
    Epoch 76/100
    119/119 [==============================] - 40s 332ms/step - loss: 2.6149 - fbeta_score: 0.3369
    
    Epoch 00076: fbeta_score did not improve from 0.33325
    Epoch 77/100
    119/119 [==============================] - 43s 360ms/step - loss: 2.5905 - fbeta_score: 0.3381
    
    Epoch 00077: fbeta_score improved from 0.33325 to 0.33789, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 78/100
    119/119 [==============================] - 39s 332ms/step - loss: 2.6036 - fbeta_score: 0.3364
    
    Epoch 00078: fbeta_score did not improve from 0.33789
    Epoch 79/100
    119/119 [==============================] - 45s 377ms/step - loss: 2.5890 - fbeta_score: 0.3410
    
    Epoch 00079: fbeta_score improved from 0.33789 to 0.33907, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 80/100
    119/119 [==============================] - 40s 334ms/step - loss: 2.5729 - fbeta_score: 0.3481
    
    Epoch 00080: fbeta_score improved from 0.33907 to 0.34003, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 81/100
    119/119 [==============================] - 47s 396ms/step - loss: 2.5720 - fbeta_score: 0.3482
    
    Epoch 00081: fbeta_score improved from 0.34003 to 0.34292, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 82/100
    119/119 [==============================] - 47s 392ms/step - loss: 2.5530 - fbeta_score: 0.3479
    
    Epoch 00082: fbeta_score improved from 0.34292 to 0.34450, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 83/100
    119/119 [==============================] - 38s 323ms/step - loss: 2.5485 - fbeta_score: 0.3518
    
    Epoch 00083: fbeta_score did not improve from 0.34450
    Epoch 84/100
    119/119 [==============================] - 47s 399ms/step - loss: 2.5657 - fbeta_score: 0.3457
    
    Epoch 00084: fbeta_score did not improve from 0.34450
    Epoch 85/100
    119/119 [==============================] - 44s 371ms/step - loss: 2.5463 - fbeta_score: 0.3479
    
    Epoch 00085: fbeta_score did not improve from 0.34450
    Epoch 86/100
    119/119 [==============================] - 48s 403ms/step - loss: 2.5540 - fbeta_score: 0.3478
    
    Epoch 00086: fbeta_score did not improve from 0.34450
    Epoch 87/100
    119/119 [==============================] - 45s 377ms/step - loss: 2.5491 - fbeta_score: 0.3492
    
    Epoch 00087: fbeta_score did not improve from 0.34450
    Epoch 88/100
    119/119 [==============================] - 44s 366ms/step - loss: 2.5449 - fbeta_score: 0.3494
    
    Epoch 00088: fbeta_score improved from 0.34450 to 0.34924, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 89/100
    119/119 [==============================] - 47s 397ms/step - loss: 2.5144 - fbeta_score: 0.3547
    
    Epoch 00089: fbeta_score improved from 0.34924 to 0.34944, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 90/100
    119/119 [==============================] - 48s 401ms/step - loss: 2.5079 - fbeta_score: 0.3550
    
    Epoch 00090: fbeta_score did not improve from 0.34944
    Epoch 91/100
    119/119 [==============================] - 43s 363ms/step - loss: 2.5118 - fbeta_score: 0.3549
    
    Epoch 00091: fbeta_score did not improve from 0.34944
    Epoch 92/100
    119/119 [==============================] - 48s 406ms/step - loss: 2.5244 - fbeta_score: 0.3515
    
    Epoch 00092: fbeta_score did not improve from 0.34944
    Epoch 93/100
    119/119 [==============================] - 43s 363ms/step - loss: 2.4718 - fbeta_score: 0.3630
    
    Epoch 00093: fbeta_score improved from 0.34944 to 0.35306, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 94/100
    119/119 [==============================] - 47s 393ms/step - loss: 2.4887 - fbeta_score: 0.3616
    
    Epoch 00094: fbeta_score did not improve from 0.35306
    Epoch 95/100
    119/119 [==============================] - 46s 387ms/step - loss: 2.4717 - fbeta_score: 0.3628
    
    Epoch 00095: fbeta_score improved from 0.35306 to 0.35339, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 96/100
    119/119 [==============================] - 43s 365ms/step - loss: 2.5003 - fbeta_score: 0.3575
    
    Epoch 00096: fbeta_score improved from 0.35339 to 0.35481, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 97/100
    119/119 [==============================] - 46s 391ms/step - loss: 2.4972 - fbeta_score: 0.3592
    
    Epoch 00097: fbeta_score improved from 0.35481 to 0.35675, saving model to fbeta_rnn
    INFO:tensorflow:Assets written to: fbeta_rnn\assets
    Epoch 98/100
    119/119 [==============================] - 49s 410ms/step - loss: 2.6217 - fbeta_score: 0.3354
    
    Epoch 00098: fbeta_score did not improve from 0.35675
    Epoch 99/100
    119/119 [==============================] - 46s 386ms/step - loss: 2.5510 - fbeta_score: 0.3499
    
    Epoch 00099: fbeta_score did not improve from 0.35675
    Epoch 100/100
    119/119 [==============================] - 49s 415ms/step - loss: 2.5130 - fbeta_score: 0.3557
    
    Epoch 00100: fbeta_score did not improve from 0.35675
    


```python
#model.save('rnn_fbeta.h5')
```


```python
tmp = keras.models.load_model('rnn_fbeta.h5')
```


```python
tmp.summary()
```

    Model: "sequential"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    embedding (Embedding)        (None, 82, 300)           4641300   
    _________________________________________________________________
    spatial_dropout1d (SpatialDr (None, 82, 300)           0         
    _________________________________________________________________
    bidirectional (Bidirectional (None, 256)               109824    
    _________________________________________________________________
    layer_normalization (LayerNo (None, 256)               512       
    _________________________________________________________________
    dense (Dense)                (None, 300)               77100     
    _________________________________________________________________
    dropout (Dropout)            (None, 300)               0         
    _________________________________________________________________
    dense_1 (Dense)              (None, 349)               105049    
    =================================================================
    Total params: 4,933,785
    Trainable params: 292,485
    Non-trainable params: 4,641,300
    _________________________________________________________________
    


```python
loaded_model= keras.models.load_model('rnn_fbeta.h5')
```


```python
#!pip install tqdm
```


```python
from tqdm.notebook import tqdm
```


```python
mmm = tmp.layers[:-2]
model_partial = Sequential(mmm)
res = []
correct_index=[]
for i, vec in enumerate(tqdm(X)):
    score = loaded_model.predict(np.array([vec]))
    score = np.argsort(score)
    if data.plc[i] in le.inverse_transform(score[0][-10:]).tolist():
        result = model_partial.predict(vec)
        res.append(result)
        correct_index.append(i)
```


    HBox(children=(HTML(value=''), FloatProgress(value=0.0, max=30377.0), HTML(value='')))


    WARNING:tensorflow:Model was constructed with shape (None, 82) for input KerasTensor(type_spec=TensorSpec(shape=(None, 82), dtype=tf.float32, name='embedding_input'), name='embedding_input', description="created by layer 'embedding_input'"), but it was called on an input with incompatible shape (None, 1).
    


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```


```python

```
