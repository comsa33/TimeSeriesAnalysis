# 📌 **다항 회귀 기반 시계열 예측 프롬프트**  

이 문서는 **GPT를 활용하여 다항 회귀를 이용한 시계열 예측 코드를 자동 생성**할 수 있도록 도와줍니다.  
아래 **프롬프트를 GPT에 입력**하면, 원하는 다항 회귀 모델을 활용한 예측 코드를 쉽게 생성할 수 있습니다. 🚀  

---

## 🛠️ **1. 라이브러리 불러오기**  
```plaintext
Python에서 다항 회귀(Polynomial Regression) 모델을 활용하여 시계열 데이터를 예측하는 코드를 작성해줘.  
필요한 라이브러리는 pandas, numpy, matplotlib, scikit-learn가 포함되어야 해.
```

📌 **예제 코드**  
```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
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

## 🔍 **3. 데이터 전처리 및 다항 회귀를 위한 변수 변환**  
```plaintext
Pandas를 사용하여 날짜 데이터를 숫자로 변환하고,  
이를 입력 변수(X)로 설정하여 다항 회귀 모델에 적합한 형식으로 데이터를 변환하는 코드를 작성해줘.
```

📌 **예제 코드**  
```python
# 날짜 데이터를 숫자로 변환
df["Date"] = df.index
df["Date"] = df["Date"].map(pd.Timestamp.toordinal)  # 날짜를 숫자로 변환

# 독립 변수(X)와 종속 변수(y) 설정
X = df[["Date"]].values  # 입력 변수 (날짜)
y = df["Close"].values  # 종속 변수 (종가)
```

---

## 📈 **4. 학습 및 테스트 데이터 분할**  
```plaintext
데이터를 학습용(80%)과 테스트용(20%) 데이터로 분할하는 코드를 작성해줘.
```

📌 **예제 코드**  
```python
train_size = int(len(X) * 0.8)
X_train, X_test = X[:train_size], X[train_size:]
y_train, y_test = y[:train_size], y[train_size:]

print(f"Train Size: {len(X_train)}, Test Size: {len(X_test)}")
```

---

## 🔍 **5. 다항 회귀 모델 학습**  
```plaintext
Scikit-learn의 PolynomialFeatures를 사용하여 다항 변환을 수행한 후,  
LinearRegression 모델을 사용하여 학습하는 코드를 작성해줘.
```

📌 **예제 코드**  
```python
# 다항 차수 설정 (예: 3차 다항식)
degree = 3
poly = PolynomialFeatures(degree=degree)
X_train_poly = poly.fit_transform(X_train)
X_test_poly = poly.transform(X_test)

# 다항 회귀 모델 학습
model = LinearRegression()
model.fit(X_train_poly, y_train)

# 회귀 계수 출력
print(f"회귀 계수: {model.coef_}")
print(f"절편: {model.intercept_:.4f}")
```

---

## 🚀 **6. 예측 수행**  
```plaintext
훈련된 다항 회귀 모델을 사용하여 테스트 데이터의 종가를 예측하는 코드를 작성해줘.
```

📌 **예제 코드**  
```python
# 예측 수행
y_pred_train = model.predict(X_train_poly)
y_pred_test = model.predict(X_test_poly)
```

---

## 🔍 **7. 성능 평가 (MSE)**  
```plaintext
예측한 주가와 실제 주가 간의 차이를 비교하여,  
Mean Squared Error(MSE)를 계산하는 코드를 작성해줘.
```

📌 **예제 코드**  
```python
train_mse = mean_squared_error(y_train, y_pred_train)
test_mse = mean_squared_error(y_test, y_pred_test)

print(f"📌 Training MSE: {train_mse:.4f}")
print(f"📌 Test MSE: {test_mse:.4f}")
```

---

## 📉 **8. 결과 시각화**  
```plaintext
Matplotlib을 사용하여 실제 주가와 예측 주가를 비교하는 그래프를 그리는 코드를 작성해줘.
```

📌 **예제 코드**  
```python
plt.figure(figsize=(12, 5))
plt.scatter(X, y, label="Actual", color="black", s=10)  # 실제 데이터
plt.plot(X, model.predict(poly.transform(X)), label="Predicted (Polynomial Regression)", linestyle="--", color="red")
plt.title("Tesla Stock Price Prediction using Polynomial Regression")
plt.xlabel("Date")
plt.ylabel("Stock Price (USD)")
plt.legend()
plt.show()
```

---

📌 **이 프롬프트를 사용하여 GPT에서 직접 코드 생성을 시도해보세요! 🚀**  
프롬프트를 입력하면, 자동으로 코드를 생성해줍니다.  

Colab에서 직접 실행하려면 아래 링크를 클릭하세요.  
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/nhjung-phd/TimeSeriesAnalysis/blob/main/notebooks/13_Polynomial_Regression.ipynb)

