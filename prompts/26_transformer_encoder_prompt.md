# 📌 **Transformer Encoder 기반 시계열 예측 프롬프트**  

이 문서는 **GPT를 활용하여 Transformer Encoder 모델을 이용한 시계열 예측 코드를 자동 생성**할 수 있도록 도와줍니다.  
아래 **프롬프트를 GPT에 입력**하면, Transformer Encoder 기반 시계열 예측 코드를 쉽게 생성할 수 있습니다. 🚀  

---

## 🛠️ **1. 라이브러리 불러오기**  
```plaintext
Python에서 TensorFlow/Keras를 사용하여 Transformer Encoder 기반 시계열 예측 모델을 구축하는 코드를 작성해줘.  
필요한 라이브러리는 pandas, numpy, matplotlib, tensorflow.keras 등이 포함되어야 해.
```

📌 **예제 코드**  
```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import tensorflow as tf
from tensorflow.keras.models import Model
from tensorflow.keras.layers import Input, Dense, LayerNormalization, MultiHeadAttention, Dropout
from sklearn.preprocessing import MinMaxScaler
from sklearn.metrics import mean_squared_error
```

---

## 📊 **2. 테슬라(TSLA) 주가 데이터 다운로드 및 시각화**  
```plaintext
yfinance 라이브러리를 사용하여 테슬라(TSLA) 주가 데이터를 가져오고,  
Matplotlib을 이용하여 시계열 그래프를 그리는 코드를 작성해줘.  
최근 2년 치 데이터를 사용하고, "Close" 가격을 기반으로 예측해줘.
```

📌 **예제 코드**  
```python
import yfinance as yf

# 📌 1️⃣ 테슬라(TSLA) 주가 데이터 다운로드 (최근 2년치)
df = yf.download("TSLA", start="2022-01-01", end="2024-01-01")

# 📌 2️⃣ 종가 시계열 시각화
plt.figure(figsize=(12, 5))
plt.plot(df.index, df["Close"], label="Tesla Stock Price (Close)", color="black")
plt.title("Tesla Stock Price Time Series")
plt.xlabel("Date")
plt.ylabel("Close Price (USD)")
plt.legend()
plt.show()
```

---

## 🔍 **3. 데이터 전처리 및 정규화**  
```plaintext
MinMaxScaler를 사용하여 주가 데이터를 0과 1 사이로 정규화하는 코드를 작성해줘.
```

📌 **예제 코드**  
```python
scaler = MinMaxScaler(feature_range=(0, 1))
df["Close_Scaled"] = scaler.fit_transform(df["Close"].values.reshape(-1, 1))
```

---

## 🔄 **4. 데이터셋 생성 (슬라이딩 윈도우 적용)**  
```plaintext
시계열 데이터를 학습에 사용할 수 있도록 슬라이딩 윈도우 방식으로  
X(입력)와 y(출력) 데이터를 생성하는 코드를 작성해줘.  
```

📌 **예제 코드**  
```python
def create_dataset(data, look_back=10):
    X, y = [], []
    for i in range(len(data) - look_back):
        X.append(data[i : i + look_back])
        y.append(data[i + look_back])
    return np.array(X), np.array(y)

look_back = 10  # 입력 데이터 길이
data = df["Close_Scaled"].values
X, y = create_dataset(data, look_back)

# 훈련 및 테스트 데이터 분할
train_size = int(len(X) * 0.8)
X_train, X_test = X[:train_size], X[train_size:]
y_train, y_test = y[:train_size], y[train_size:]

# 차원 변경 (Transformer 입력 형식: (samples, time steps, features))
X_train = X_train.reshape((X_train.shape[0], X_train.shape[1], 1))
X_test = X_test.reshape((X_test.shape[0], X_test.shape[1], 1))
```

---

## 🏗 **5. Transformer Encoder 블록 정의**  
```plaintext
Transformer Encoder 블록을 구성하는 코드를 작성해줘.  
MultiHeadAttention, LayerNormalization, Dropout을 포함해야 해.
```

📌 **예제 코드**  
```python
class TransformerEncoderLayer(tf.keras.layers.Layer):
    def __init__(self, embed_dim, num_heads, ff_dim, rate=0.1):
        super(TransformerEncoderLayer, self).__init__()
        self.att = MultiHeadAttention(num_heads=num_heads, key_dim=embed_dim)
        self.ffn = tf.keras.Sequential([
            Dense(ff_dim, activation="relu"),
            Dense(embed_dim),
        ])
        self.layernorm1 = LayerNormalization(epsilon=1e-6)
        self.layernorm2 = LayerNormalization(epsilon=1e-6)
        self.dropout1 = Dropout(rate)
        self.dropout2 = Dropout(rate)

    def call(self, inputs, training):
        attn_output = self.att(inputs, inputs)
        attn_output = self.dropout1(attn_output, training=training)
        out1 = self.layernorm1(inputs + attn_output)

        ffn_output = self.ffn(out1)
        ffn_output = self.dropout2(ffn_output, training=training)
        return self.layernorm2(out1 + ffn_output)
```

---

## 🏗 **6. Transformer Encoder 모델 구성 및 학습**  
```plaintext
Sequential API를 사용하여 Transformer Encoder 기반 시계열 예측 모델을 구성하고,  
Adam Optimizer와 MSE Loss를 사용하여 학습하는 코드를 작성해줘.
```

📌 **예제 코드**  
```python
embed_dim = 16
num_heads = 2
ff_dim = 32
num_layers = 2

inputs = Input(shape=(look_back, 1))
x = inputs

for _ in range(num_layers):
    x = TransformerEncoderLayer(embed_dim, num_heads, ff_dim)(x)

x = Dense(50, activation="relu")(x)
x = Dense(1)(x)
outputs = x[:, -1, :]

model = Model(inputs=inputs, outputs=outputs)

# 모델 컴파일 및 학습
model.compile(optimizer=Adam(learning_rate=0.001), loss="mse")
history = model.fit(X_train, y_train, epochs=50, batch_size=16, validation_data=(X_test, y_test), verbose=1)

# 학습 곡선 시각화
plt.figure(figsize=(12, 5))
plt.plot(history.history["loss"], label="Train Loss")
plt.plot(history.history["val_loss"], label="Validation Loss")
plt.title("Model Loss")
plt.xlabel("Epochs")
plt.ylabel("Loss")
plt.legend()
plt.show()
```

---

## 🔮 **7. 예측 수행 및 성능 평가**  
```plaintext
학습된 Transformer Encoder 모델을 사용하여 테스트 데이터의 종가를 예측하고,  
Mean Squared Error(MSE)를 평가하는 코드를 작성해줘.
```

📌 **예제 코드**  
```python
# 예측 수행
y_pred = model.predict(X_test)

# MSE 평가
test_mse = mean_squared_error(y_test, y_pred)
print(f"📌 Test MSE: {test_mse:.4f}")

# 예측 결과 시각화
plt.figure(figsize=(12, 5))
plt.plot(y_test, label="Actual", color="black")
plt.plot(y_pred, label="Predicted (Transformer Encoder)", linestyle="--", color="red")
plt.title("Tesla Stock Price Prediction using Transformer Encoder")
plt.xlabel("Time Step")
plt.ylabel("Stock Price (Scaled)")
plt.legend()
plt.show()
```

---

📌 **이 프롬프트를 사용하여 GPT에서 직접 코드 생성을 시도해보세요! 🚀**  

Colab에서 직접 실행하려면 아래 링크를 클릭하세요.  
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nhjung-phd/TimeSeriesAnalysis/blob/main/notebooks/26_transformer_encoder.ipynb)

