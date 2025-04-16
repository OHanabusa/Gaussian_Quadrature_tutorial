# ガウス求積による数値積分チュートリアル

このチュートリアルでは、**ガウス求積 (Gaussian Quadrature)** を用いて定積分を数値的に計算する方法を、理論の解説、Python実装、および簡単な演習を通して学びます。

---

## 📘 目次

1. [背景と動機](#背景と動機)
2. [ガウス求積の理論](#ガウス求積の理論)
3. [Pythonによる実装](#pythonによる実装)
4. [具体例](#具体例)
5. [演習問題](#演習問題)
6. [参考文献](#参考文献)

---

## 背景と動機

数値積分は、解析的に積分できない関数や、離散的なデータから積分値を求めたい場合に有効な手法です。中でも**ガウス求積法**は高精度な結果が得られる手法として知られ、有限要素法（FEM）や物理シミュレーションなどで多用されています。

---

## ガウス求積の理論

### 基本アイデア

ガウス求積法では、区間 \([-1, 1]\) 上の積分を以下の形で近似します：

$$
\int_{-1}^{1} f(x)\, dx \approx \sum_{i=1}^{n} w_i f(x_i)
$$

ここで、\(x_i\) は重み付き積分の計算に最適な点（ガウス点）、\(w_i\) は対応する重みです。

### 一般の区間への変換

区間 \([a, b]\) の積分に変換するには、以下を用います：

$$
\int_{a}^{b} f(x) dx = \frac{b-a}{2} \int_{-1}^{1} f\left(\frac{b-a}{2}x + \frac{a+b}{2}\right) dx
$$

---

## Pythonによる実装

```python
import numpy as np

def gauss_quadrature(f, a, b, n):
    # ガウス点と重みの取得（Legendre多項式）
    [x, w] = np.polynomial.legendre.leggauss(n)
    
    # 変換された積分の計算
    t = 0.5 * (x + 1) * (b - a) + a
    return 0.5 * (b - a) * np.dot(w, f(t))
```

---

## 具体例

次の積分を計算します：

$$
\int_0^1 e^{-x^2} dx
$$

```python
import numpy as np
result = gauss_quadrature(lambda x: np.exp(-x**2), 0, 1, n=4)
print(f"Approximate integral: {result:.8f}")
```

比較用にSciPyの結果：

```python
from scipy.integrate import quad
actual, _ = quad(lambda x: np.exp(-x**2), 0, 1)
print(f"Actual integral: {actual:.8f}")
```

---

## 演習問題

1. $$\int_0^\pi \sin(x) dx$$ をガウス求積で計算せよ。
2. $$\int_0^2 x^3 - x + 1 dx$$ を解析解と比較せよ。
3. 精度と点数 $$n$$ の関係を調べよ（n=2〜10）
4. 2次元ガウス求積を使って $$\int_0^1 \int_0^1 e^{-(x^2 + y^2)} dxdy$$ を求めよ。

---

## 参考文献

- Quarteroni, A., et al. "Numerical Mathematics." Springer.
- Burden, R. L., and Faires, J. D. "Numerical Analysis."
- SciPy documentation: [https://docs.scipy.org/doc/scipy/reference/generated/numpy.polynomial.legendre.leggauss.html](https://docs.scipy.org/doc/scipy/reference/generated/numpy.polynomial.legendre.leggauss.html)
