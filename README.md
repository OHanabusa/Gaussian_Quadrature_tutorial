# ガウス求積入門 — 実装の基礎と任意区間への拡張

---

## 導入 — なぜ数値積分を学ぶのか？

- **解析解が得られない積分が多い**\
  実データや複雑関数の積分は閉形式で求められないことが多い。
- **数値計算の現場で実装必須**\
  FEM、モンテカルロ法、ベイズ推定などで積分器を自作する機会が増加。
- **ガウス求積は高精度・低コスト**\
  同じ精度を得るのに台形則や Simpson 則よりも少ない関数評価で済む。

> **目的**：ガウス求積公式と係数表を示し、任意区間に一般化した上で実装と演習を行う。

---

## 1.1 基本公式

$$
\int_{-1}^{1} f(x)\,dx \;\approx\; \sum_{i=1}^{n} w_i\,f(x_i) \qquad (1)
$$

**解説**：ガウス点 \(x_i\) は Legendre 多項式 \(P_n\) の零点、重み \(w_i\) は誤差最小になる係数。\(2n-1\) 次までの多項式で誤差ゼロ。

| \(n\) | ガウス点 \(x_i\)                                        | 重み \(w_i\)                                                             |
| ----- | --------------------------------------------------- | ---------------------------------------------------------------------- |
| 2     | \(\pm0.577350269189626\)                            | 1.000000000000000                                                      |
| 3     | \(\pm0.774596669241483,\;0\)                        | 0.555555555555556 (両端), 0.888888888888889 (中央)                         |
| 4     | \(\pm0.861136311594053,\;\pm0.339981043584856\)     | 0.347854845137454 (外側), 0.652145154862546 (内側)                         |
| 5     | \(\pm0.906179845938664,\;\pm0.538469310105683,\;0\) | 0.236926885056189 (最外), 0.478628670499366 (中間), 0.568888888888889 (中央) |

---

## 2.0 章の狙い

標準区間以外では式 (1) を直接使えない。一次変換で \([x_1,x_2]\) を \([-1,1]\) に写し、ヤコビアンでスケールを補正して利用可能にする。

## 2.1 変数変換

$$
\xi\in[-1,1],\quad x(\xi)=\frac{x_2-x_1}{2}\,\xi+\frac{x_1+x_2}{2}. \qquad (2)
$$

## 2.2 ヤコビアン

$$
\frac{dx}{d\xi}=\frac{x_2-x_1}{2}=J. \qquad (3)
$$

## 2.3 一般区間用ガウス求積

$$
\boxed{\displaystyle
\int_{x_1}^{x_2} f(x)\,dx \;\approx\; J\,\sum_{i=1}^{n} w_i\,f\left(\tfrac{x_2-x_1}{2}\,\xi_i+\tfrac{x_1+x_2}{2}\right)
}\qquad (4)
$$

**解説**：式 (4) は標準区間のガウス点・重みをそのまま再利用する万能公式。\(J\) が区間長スケールを補正する。

---

# 3章 │ Python 実装テンプレート（多点数対応・任意区間）

```python
# === Gauss-Legendre データ: n 点数 → (ガウス点リスト, 重みリスト) ===
gauss_data = {
    2: ([-0.577350269189626, 0.577350269189626], [1.0, 1.0]),
    3: ([-0.774596669241483, 0.0, 0.774596669241483],
        [0.555555555555556, 0.888888888888889, 0.555555555555556]),
    4: ([-0.861136311594053, -0.339981043584856,
          0.339981043584856,  0.861136311594053],
        [0.347854845137454, 0.652145154862546,
         0.652145154862546, 0.347854845137454]),
    5: ([-0.906179845938664, -0.538469310105683, 0.0,
          0.538469310105683,  0.906179845938664],
        [0.236926885056189, 0.478628670499366, 0.568888888888889,
         0.478628670499366, 0.236926885056189]),
}

def gauss_interval(f, a, b, n=2):
    """[a,b] 区間の n 点ガウス求積 (n=2〜5) — ★★ 空欄を埋めて完成させよう ★★"""
    # 1) ガウス点と重みを取得
    xg, wg = gauss_data[n]

    # 2) ヤコビアン J = (b-a)/2 を計算  ↓↓↓
    J = ...  # TODO: ここを計算式に置き換え

    # 3) 区間中心 c = (a+b)/2 を計算  ↓↓↓
    c = ...  # TODO: ここを計算式に置き換え

    # 4) 積分値を格納する変数を初期化
    s = 0.0

    # 5) ガウス点ごとにループして重み付き評価値を加算
    for xi, wi in zip(xg, wg):
        x = ...  # TODO: 変換後の評価点 x = J*xi + c
        s += ...  # TODO: 重みをかけて加算 wi * f(x)

    # 6) 最後にヤコビアン J を掛けて返す
    return ...  # TODO: J * s を返す
```

> **埋め方ガイド**
>
> 1. `J` は `0.5 * (b - a)`
> 2. `c` は `0.5 * (a + b)`
> 3. 変換後の評価点は `x = J * xi + c`
> 4. ループ内で `s += wi * f(x)`
> 5. 関数の戻り値は `J * s`\
>    これら 5 カ所を **TODO** 行に記述すれば完成します。

> **使い方**：`n` に 2〜5 を指定して任意点数を選択可。

---

### 3.1 サンプル計算例

被積分関数:

$$
f(x) = 2x^2 + 3x + 1
$$

解析積分:

$$
\int_{0}^{2} (2x^2 + 3x + 1)\,dx = \left[\frac{2}{3}x^3 + \frac{3}{2}x^2 + x\right]_{0}^{2} = \frac{40}{3} \approx 13.333333333
$$

```python
f = lambda x: 2*x**2 + 3*x + 1
approx = gauss_interval(f, 0, 2, n=2)
print(approx)  # -> 13.333333333333334
```

解析値 \(=40/3\) と一致し、誤差ゼロを確認。

---

## 3.2 演習問題（解答付き）

| # | 被積分関数             | 区間        | 指示                                                                      | 正しい積分値                              |
| - | ----------------- | --------- | ----------------------------------------------------------------------- | ----------------------------------- |
| 1 | \(2x^2 + 3x + 1\) | \([1,3]\) | **2点** ガウス求積で実装                                                         | \(\frac{94}{3}\approx31.333333333\) |
| 2 | \(\sin(3x)\)      | \([0,1]\) | **テーラー展開 3 次まで**: \(\sin(3x)\approx3x-\frac{(3x)^3}{6}\) を用い **2点** で計算 | 0.375                               |
| 3 | \(x^4 - x^2 + 1\) | \([0,2]\) | **3点** ガウス求積を用いる                                                        | 5.733333333                         |

> **ヒント**：問3は \(n=3\) を `gauss_interval` に渡して解くこと。

---

# 4章 │ 2変数ガウス求積法の導出

数値積分を 2 変数関数に拡張するため、まず物理空間と参照空間の座標系を定義し、一次写像とヤコビアン行列を導入します。最後に、一般的な 2 重積分からガウス求積の形に至るまでをステップ・バイ・ステップで示します。

---

## 4.1 座標の定義とヤコビアン行列

- **物理空間の座標**  
  \[
    \mathbf{x} = (x,\,y), \quad (x,y)\in [x_1,x_2]\times[y_1,y_2].
  \]

- **参照空間（パラメータ空間）の座標**  
  \[
    \boldsymbol{\xi} = (\xi,\,\eta), \quad (\xi,\eta)\in[-1,1]^2.
  \]

参照空間から物理空間へのアフィン一次写像を
\[
  \mathbf{x} = \mathbf{F}(\boldsymbol{\xi})
  = J\,\boldsymbol{\xi} + \mathbf{c},
\]
として定義します。ここで

\[
  J
  =
  \begin{pmatrix}
    \frac{x_2 - x_1}{2} & 0 \\
    0 & \frac{y_2 - y_1}{2}
  \end{pmatrix},
  \qquad
  \mathbf{c}
  =
  \begin{pmatrix}
    \frac{x_1 + x_2}{2} \\
    \frac{y_1 + y_2}{2}
  \end{pmatrix}.
\]

**解説**：  
- 行列 \(J\) の対角要素が \(x\)、\(y\) それぞれのスケール変換を担い、  
- ベクトル \(\mathbf{c}\) が区間の中心位置を表します。

---

## 4.2 2重積分の一般形

もとの物理空間上の積分は

\[
  I
  =
  \int_{x_1}^{x_2} \int_{y_1}^{y_2} f(x,y)\,\mathrm{d}y\,\mathrm{d}x.
\]

**解説**：  
\(\mathrm{d}y\,\mathrm{d}x\) は物理空間の面積要素です。

---

## 4.3 変数変換とヤコビアンの導入

参照空間の微小面積要素を \(\mathrm{d}\xi\,\mathrm{d}\eta\) とすると、一次写像により

\[
  \mathrm{d}x\,\mathrm{d}y
  =
  \bigl|\det J\bigr|\;\mathrm{d}\xi\,\mathrm{d}\eta,
\]
\[
  \det J
  =
  \frac{x_2 - x_1}{2}\times\frac{y_2 - y_1}{2}.
\]

**解説**：  
ヤコビアン \(\bigl|\det J\bigr|\) が面積スケーリングを補正し、参照空間上の面積要素を物理空間上の面積要素に変換します。

---

## 4.4 参照空間上の積分式

変数変換を適用すると、

\[
  I
  =
  \int_{-1}^{1} \int_{-1}^{1}
    f\bigl(\mathbf{F}(\xi,\eta)\bigr)\;
    \bigl|\det J\bigr|\;\mathrm{d}\eta\,\mathrm{d}\xi.
\]

**解説**：  
- 関数 \(f\) の引数が \(\mathbf{F}(\xi,\eta)\) によって物理空間の点にマッピングされ、  
- ヤコビアン \(\bigl|\det J\bigr|\) が面積要素の変形を補正します。

---

## 4.5 2変数ガウス求積公式

参照空間上で 1 次元ガウス求積をそれぞれの変数に適用し、組み合わせることで

\[
  I
  \approx
  \sum_{i=1}^{n}\sum_{j=1}^{n}
    w_i\,w_j\,
    \bigl|\det J\bigr|\,
    f\!\bigl(\mathbf{F}(\xi_i,\eta_j)\bigr).
\]

- \(\{\xi_i,w_i\}\) は \(\xi\) 方向のガウス点と重み、  
- \(\{\eta_j,w_j\}\) は \(\eta\) 方向のガウス点と重みです。

**解説**：  
独立した 2 回の 1 次元ガウス求積を二重和で評価し、ヤコビアンをかけるだけで高精度な 2 次元積分が得られます。


