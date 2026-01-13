
---

# 1) CELL 2 — Load dữ liệu cổ phiếu BID (vnquant)

```python
import vnquant.data as dt

loader = dt.DataLoader(symbols="BID",
                       start="2024-01-01",
                       end="2024-11-30",
                       minimal=True,
                       data_source="cafe")
data = loader.download()
data = data.stack()
data = data.reset_index()
data = data.set_index('date')
data.head()
```

### ✅ Block này làm gì?

Bạn đang **tải dữ liệu lịch sử cổ phiếu BID** từ ngày `2024-01-01` đến `2024-11-30`, sau đó  **chuyển dữ liệu về dạng DataFrame dễ xử lý** .

### Giải thích từng dòng:

* `import vnquant.data as dt`
  → gọi module vnquant để lấy dữ liệu chứng khoán VN.
* `dt.DataLoader(...)`
  → tạo một loader để lấy dữ liệu.
  * `symbols="BID"`: mã cổ phiếu
  * `start/end`: khoảng thời gian
  * `minimal=True`: lấy dữ liệu “tinh gọn” (thường đủ xài)
  * `data_source="cafe"`: nguồn CafeF
* `data = loader.download()`
  → tải dữ liệu về (thường ra dạng bảng nhiều cấp index).
* `data = data.stack()`
  → **chuyển cột dạng nhiều tầng thành dạng “dài” (long format)**
  Nói đơn giản: giúp dữ liệu từ dạng “bảng phức tạp” thành dạng dễ xử lý hơn.
* `data = data.reset_index()`
  → đưa index thành cột thường (để tiện chỉnh sửa).
* `data = data.set_index('date')`
  → đưa cột `date` về làm index (mốc thời gian).
* `data.head()`
  → xem 5 dòng đầu để check dữ liệu OK chưa.

### 📌 Output bạn nhận được

Một DataFrame theo ngày, thường có cột:
`open, high, low, close, adjust, volume_match, value_match ...`

---

# 2) CELL 3 — Vẽ biểu đồ giá + khối lượng BID (2 trục Y)

```python
import matplotlib.pyplot as plt

fig, ax1 = plt.subplots(figsize=(14, 8))

ax1.plot(data.index, data['open'], label='Open Price', color='blue')
ax1.plot(data.index, data['high'], label='High Price', color='green')
ax1.plot(data.index, data['low'], label='Low Price', color='red')
ax1.plot(data.index, data['close'], label='Close Price', color='purple')
ax1.plot(data.index, data['adjust'], label='Adjusted Price', color='orange')

ax1.set_xlabel('Thời gian')
ax1.set_ylabel('Giá (VNĐ)', color='black')
ax1.tick_params(axis='y', labelcolor='black')
ax1.legend(loc='upper left')

ax2 = ax1.twinx()
ax2.plot(data.index, data['volume_match'],
         label='Volume Match', color='brown', linestyle='dashed')
ax2.plot(data.index, data['value_match'],
         label='Value Match', color='cyan', linestyle='dashed')

ax2.set_ylabel('Khối lượng/Giá trị giao dịch', color='black')
ax2.tick_params(axis='y', labelcolor='black')
ax2.legend(loc='upper right')

plt.title('Sự thay đổi giá và giao dịch của cổ phiếu BID theo thời gian')
ax1.grid()
plt.show()
```

### ✅ Block này làm gì?

Bạn vẽ 1 chart gồm:

* **Trục Y bên trái** : giá cổ phiếu (`open/high/low/close/adjust`)
* **Trục Y bên phải** : thanh khoản (`volume_match/value_match`)

### Ý nghĩa từng phần:

* `plt.subplots(figsize=(14,8))` → tạo khung hình to dễ nhìn
* `ax1.plot(...)` → vẽ các đường giá
* `ax1.twinx()` → tạo **trục Y thứ 2** để vẽ volume/value (vì scale khác giá)
* `linestyle='dashed'` → đường gạch để phân biệt với đường giá
* `legend` chia 2 bên để không đè nhau

### ✅ Bạn đọc biểu đồ thế nào?

* Close tăng đều + volume tăng → xu hướng tăng “có lực”
* Giá tăng nhưng volume yếu → tăng “mỏng”, dễ đảo chiều
* Volume tăng mạnh nhưng giá giảm → có thể “xả hàng”

---

# 3) CELL 4 & 5 — Load + Plot cổ phiếu ACB

Hai cell này giống BID, chỉ khác:

```python
symbols="ACB"
```

### ✅ Mục tiêu

* làm tương tự để có chart ACB
* chuẩn bị cho bước so sánh sau

📌 Đây là “pattern” bạn dùng:
**Load một mã → vẽ một mã**

---

# 4) CELL 6 & 7 — Load + Plot cổ phiếu MBB

Cũng y chang, đổi thành:

```python
symbols="MBB"
```

### ✅ Mục tiêu

Bạn phân tích nhóm ngân hàng: BID – ACB – MBB.

---

# 5) CELL 9 — So sánh lợi suất trung bình ngày của BID, ACB, MBB

```python
import pandas as pd

loader = dt.DataLoader(
    symbols=["BID", "ACB", "MBB"],
    start="2024-01-01",
    end="2024-11-30",
    minimal=True,
    data_source="cafe"
)

data = loader.download()
data = data.stack()
data.reset_index(inplace=True)
data.set_index('date', inplace=True)

data['daily_return'] = data.groupby('Symbols')['close'].pct_change()

average_daily_returns = data.groupby('Symbols')['daily_return'].mean()

print("Lợi suất hàng ngày trung bình của từng cổ phiếu:")
print(average_daily_returns * 100)
```

### ✅ Block này làm gì?

Bạn lấy dữ liệu 3 mã cùng lúc rồi tính:

1. **Daily return** = lợi suất mỗi ngày
   [
   daily_return = \frac{close_t - close_{t-1}}{close_{t-1}}
   ]
2. **Average daily return** = lợi suất trung bình ngày của từng cổ phiếu

### Giải thích các điểm quan trọng:

* `symbols=["BID","ACB","MBB"]`
  → tải một lần, khỏi lặp code nhiều
* `groupby('Symbols')['close'].pct_change()`
  → tính % thay đổi theo ngày **cho từng mã riêng**
  * nếu không groupby thì dữ liệu 3 mã trộn sẽ sai bét
* `mean()`
  → lợi suất trung bình ngày
* `* 100`
  → in ra dạng %

### 📌 Ý nghĩa thực tế

Cổ phiếu nào có `average_daily_return` cao hơn thì **tăng trưởng trung bình tốt hơn** trong giai đoạn đó.
Nhưng nhớ: return trung bình **không nói hết rủi ro** (cần volatility nữa).

---

# 6) CELL 11 — Tính EMA & SMA (Moving Averages) cho 3 cổ phiếu

```python
data = data.sort_values(by=['Symbols', 'date'])

data['EMA_30'] = data.groupby('Symbols')['close'].transform(
    lambda x: x.ewm(span=30, adjust=False).mean())
data['EMA_60'] = data.groupby('Symbols')['close'].transform(
    lambda x: x.ewm(span=60, adjust=False).mean())
data['EMA_200'] = data.groupby('Symbols')['close'].transform(
    lambda x: x.ewm(span=200, adjust=False).mean())

data['SMA_30'] = data.groupby('Symbols')['close'].transform(
    lambda x: x.rolling(window=30).mean())
data['SMA_60'] = data.groupby('Symbols')['close'].transform(
    lambda x: x.rolling(window=60).mean())
data['SMA_200'] = data.groupby('Symbols')['close'].transform(
    lambda x: x.rolling(window=200).mean())

print(data[['Symbols', 'SMA_30', 'SMA_60', 'SMA_200',
      'EMA_30', 'EMA_60', 'EMA_200']].tail())
```

### ✅ Block này làm gì?

Bạn tính **đường trung bình động** để đánh giá xu hướng:

* **SMA** (Simple Moving Average): trung bình cộng thuần
* **EMA** (Exponential Moving Average): ưu tiên dữ liệu gần nhất hơn

### Vì sao phải `sort_values(by=['Symbols','date'])`?

Đây là dòng rất quan trọng.

✅ Vì moving average là tính theo thời gian.
Nếu dữ liệu không sort đúng thứ tự ngày → MA sai.

### SMA & EMA khác nhau thế nào?

* SMA 30:
  * mỗi ngày trong 30 ngày có trọng số như nhau
* EMA 30:
  * ngày gần nhất nặng hơn → phản ứng nhanh hơn

### Ý nghĩa của 30 / 60 / 200

* 30 ngày: xu hướng ngắn hạn
* 60 ngày: trung hạn
* 200 ngày: dài hạn (rất hay dùng trong phân tích kỹ thuật)

### Đọc tín hiệu cơ bản:

* **Giá > MA200** → xu hướng dài hạn tốt
* **EMA30 cắt lên EMA60** → dấu hiệu tăng
* **EMA30 cắt xuống EMA60** → dấu hiệu giảm

---

# ✅ CELL 13 — Tính lợi suất hàng ngày (Daily Return)

```python
# Tính toán lợi suất hàng ngày
data['daily_return'] = data.groupby('Symbols')['close'].pct_change()
```

### Block này làm gì?

✅ Tạo cột **daily_return** cho từng cổ phiếu.

### Giải thích kỹ:

* `groupby('Symbols')`: chia dữ liệu theo từng mã (BID riêng, ACB riêng…)
* `['close']`: lấy giá đóng cửa
* `.pct_change()`: tính % thay đổi so với ngày trước

Công thức chính là:
[
daily_return_t = \frac{close_t - close_{t-1}}{close_{t-1}}
]

### Vì sao quan trọng?

Giá cổ phiếu tuyệt đối khó so sánh (mã 20k vs 80k), nên **return** là chuẩn để:

* so sánh tăng trưởng
* đo rủi ro (volatility)
* tính tương quan

⚠️ Lưu ý:

* Ngày đầu tiên của mỗi mã sẽ bị **NaN** (không có ngày trước).

---

# ✅ CELL 14 — Pivot daily_return thành bảng dạng “mỗi cột là 1 mã”

```python
# Pivot dữ liệu để mỗi cột đại diện cho daily_return của một cổ phiếu
daily_returns_pivot = data.pivot_table(
    values='daily_return', index='date', columns='Symbols')

# Kiểm tra dữ liệu đã pivot
print(daily_returns_pivot.head())
```

### Block này làm gì?

✅ Chuyển dữ liệu từ dạng “dài” (long format) sang dạng “rộng” (wide format)

**Trước pivot (long):**

| date       | Symbols | daily_return |
| ---------- | ------- | ------------ |
| 2024-01-02 | BID     | 0.01         |
| 2024-01-02 | ACB     | -0.02        |

**Sau pivot (wide):**

| date       | BID  | ACB   | MBB   |
| ---------- | ---- | ----- | ----- |
| 2024-01-02 | 0.01 | -0.02 | 0.005 |

### Tại sao phải pivot?

Vì các bước tiếp theo như:

* **tính correlation**
* **vẽ heatmap**
* **tính risk portfolio**
  … đều cần dạng bảng này.

---

# ✅ CELL 15 — (…)

Cell này notebook của bạn đang để `...`
=> nghĩa là **bị placeholder** hoặc bạn chưa code phần đó.

📌 Nếu bạn muốn mình gợi ý cell này nên làm gì, thì thường sau pivot sẽ là:

* làm sạch NaN
* hoặc normalize data
  Ví dụ hợp lý:

```python
daily_returns_pivot = daily_returns_pivot.dropna()
```

---

# ✅ CELL 16 — Vẽ Heatmap correlation (Seaborn)

Notebook bạn có import seaborn:

```python
import seaborn as sns
```

Và (thường sẽ có) dạng:

```python
corr_matrix = daily_returns_pivot.corr()

plt.figure(figsize=(8,6))
sns.heatmap(corr_matrix, annot=True, cmap="coolwarm")
plt.title("Ma trận tương quan lợi suất")
plt.show()
```

### Block này làm gì?

✅ Tính **ma trận tương quan** giữa các cổ phiếu dựa trên daily_return.

### Ý nghĩa correlation:

* Corr ≈ 1: đi cùng nhau rất mạnh (cùng tăng/cùng giảm)
* Corr ≈ 0: gần như độc lập
* Corr ≈ -1: đi ngược nhau

📌 Trong tài chính:

* Corr cao → diversification kém (rủi ro hệ thống)
* Corr thấp/âm → diversification tốt

⚠️ Note:
Bạn đang dùng seaborn để vẽ heatmap. Nó đẹp, nhưng nhớ là  **seaborn chỉ là visualization** , cốt lõi vẫn là `.corr()`.

---

# ✅ CELL 18 — Nhắc lại lợi suất hàng ngày (Daily return)

Bạn có cell comment:

```python
# Lợi suất hàng ngày
```

Thường cell này sẽ tiếp tục dùng `daily_returns_pivot` hoặc chuẩn bị cho risk calculation.

Nếu notebook bạn có dòng kiểu:

```python
daily_returns_pivot.describe()
```

Thì mục tiêu là:
✅ xem thống kê: mean, std, min, max cho từng mã.

---

# ✅ CELL 19 — Import numpy

```python
import numpy as np
```

### Block này làm gì?

✅ Gọi numpy để tính toán số học, thống kê.

Từ đây trở đi notebook sẽ bắt đầu tính:

* Percentile
* VaR
* mô phỏng, hoặc prediction

---

# ✅ CELL 20 — Thiết lập Confidence level cho VaR

```python
confidence_level_99 = 1  # VaR ở mức 99% --> 1% phía dưới
```

### Block này làm gì?

✅ Bạn đang chuẩn bị tính  **Value at Risk (VaR)** .

Nhưng dòng này có vấn đề logic nhỏ:

* VaR 99% nghĩa là **xét 1% trường hợp xấu nhất**
* Thông thường:
  * `alpha = 0.01`
  * hoặc `confidence_level = 0.99`

✅ Viết đúng hơn sẽ là:

```python
alpha = 0.01   # 1% tail loss
```

Hoặc:

```python
confidence_level = 0.99
alpha = 1 - confidence_level
```

### VaR hiểu đơn giản:

> “Trong ngày xấu, bạn có thể lỗ tối đa khoảng bao nhiêu với xác suất 99%?”

Ví dụ VaR = -2%
→ 99% ngày bạn không lỗ quá 2%, nhưng 1% ngày có thể lỗ hơn.

---

# ✅ CELL 21 — Plot / Visual risk

Bạn có:

```python
import matplotlib.pyplot as plt
```

Cell này thường là bạn sẽ vẽ:

* phân phối daily_return (histogram)
* đường VaR (vạch đỏ)

Ví dụ chuẩn:

```python
returns = daily_returns_pivot['BID'].dropna()
var_99 = np.percentile(returns, 1)

plt.hist(returns, bins=50)
plt.axvline(var_99, linestyle="--")
plt.title("VaR 99% - BID")
plt.show()
```

### Ý nghĩa:

✅ Nhìn được “đuôi trái” (tail risk), chỗ có rủi ro lớn.

---

# ✅ CELL 24–28 — Dự đoán giá cổ phiếu (Machine Learning Pipeline)

Đây là đoạn notebook chuyển qua  **forecasting/prediction** .

## ✅ CELL 24 — Chuẩn bị dữ liệu

```python
# Bước 1: Chuẩn bị dữ liệu
```

Thông thường bước này gồm:

* chọn 1 cổ phiếu mục tiêu (ví dụ BID)
* tạo feature từ giá quá khứ: lag 1, lag 2, MA…

Ví dụ chuẩn:

```python
df = data[data['Symbols']=="BID"].copy()
```

---

## ✅ CELL 25 — Chọn target

```python
# Bước 2: Chọn mục tiêu dự đoán (target)
```

Target thường là:

* `close` ngày hôm sau
* hoặc `close` chính ngày đó

Ví dụ:

```python
y = df['close']
```

---

## ✅ CELL 26 — Chia dữ liệu train/test

```python
# Bước 3: Chia dữ liệu
```

Ví dụ:

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, shuffle=False)
```

⚠️ Lưu ý rất quan trọng trong time-series:
✅ **shuffle=False** vì dữ liệu có thứ tự thời gian, trộn lên là sai.

---

## ✅ CELL 27 — Train model dự đoán

```python
# Bước 4: Áp dụng mô hình dự đoán
```

Thường bạn dùng:

* Linear Regression
* RandomForest
* SVR

Ví dụ:

```python
from sklearn.linear_model import LinearRegression
model = LinearRegression()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
```

---

## ✅ CELL 28 — Vẽ so sánh giá thật vs giá dự đoán

```python
# Bước 5: Trực quan hóa kết quả
import matplotlib.pyplot as plt

plt.figure(figsize=(10, 6))
plt.plot(y_test.values, label="Giá thực tế", color="blue")
plt.plot(y_pred, label="Giá dự đoán", color="red")
plt.title("So sánh giá thực tế và giá dự đoán")
plt.xlabel("Mẫu")
plt.ylabel("Giá cổ phiếu")
plt.legend()
plt.show()
```

### Block này làm gì?

✅ Vẽ 2 đường:

* xanh: giá thật (y_test)
* đỏ: giá mô hình dự đoán (y_pred)

### Cách đọc biểu đồ:

* Đường đỏ bám sát xanh → dự đoán ok
* Đỏ lệch xa hoặc “lag” → mô hình yếu / feature chưa đủ

⚠️ Note:
Trục X đang là “mẫu” (sample index), nếu muốn đẹp hơn có thể dùng date thật.

---

# ✅ Tổng kết phần bạn vừa làm (Cell 13 → 28)

Bạn đã xây 3 tầng phân tích:

### (1) Lợi suất (Return)

* pct_change
* so sánh tăng trưởng

### (2) Tương quan (Correlation)

* pivot bảng daily_return
* heatmap

### (3) Rủi ro + Dự đoán

* VaR (tail risk)
* ML model dự đoán close

---

# ✅ Gợi ý nâng cấp để bài “ăn điểm” hơn (rất đáng)

Nếu đây là bài nộp assignment, bạn nên thêm 2 thứ:

### ✅ 1) RMSE / MAE cho prediction

Đừng chỉ plot, hãy đo lỗi:

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error
rmse = mean_squared_error(y_test, y_pred, squared=False)
mae = mean_absolute_error(y_test, y_pred)
print("RMSE:", rmse, "MAE:", mae)
```

### ✅ 2) Annualized Volatility + Sharpe ratio

Tài chính rất thích:

* Volatility năm:
  [
  \sigma_{year} = \sigma_{daily}\sqrt{252}
  ]
* Sharpe:
  [
  Sharpe = \frac{mean(daily_return)}{std(daily_return)}\sqrt{252}
  ]
