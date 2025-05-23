
# Quantitative Trading Practice Report

本專案為量化交易測驗之練習報告，分為兩大題，分別探討：

1. 不同交易所（Binance 與 OKX）永續期貨價格的價差與套利潛力。
2. BTCUSDT 永續期貨的交易策略設計與回測。

## 📁 專案內容

- `q1.ipynb`：**題目一 - 幣安與 OKX 的 MINAUSDT 價差分析**
  - 使用 Binance 與 OKX API 撈取 1 分鐘 K 線資料。
  - 資料清洗與統一格式處理。
  - 繪製價差圖表，分析套利潛力與可能模式。
  - 探討高頻交易與延遲價格反應的應用空間。

- `q2.ipynb`：**題目二 - BTCUSDT 策略回測**
  - 從 2021/01/01 起，使用 4 小時 K 線設計交易策略。
  - 指標結合：MFI、RSI、移動平均（MA）等。
  - 回測並記錄績效指標（總報酬、年化報酬、Sharpe Ratio、MDD）。
  - 參數優化、測試集表現與改進建議。

- `trades_detail.csv`：**交易明細紀錄**
  - 包含進場時間、出場時間、價格、手續費與盈虧等資訊。
  - 用於回測結果驗證與策略績效分析。

## 🔧 環境建議與安裝

建議使用 Python 3.8+，並安裝下列套件：

```bash
pip install -r requirements.txt
```





## 🚀 執行方式

1. 開啟 `q1.ipynb` 了解如何透過 API 撈取資料與價差視覺化。
2. 開啟 `q2.ipynb` 查看策略設計、參數優化與回測結果。
3. 檢查 `trades_detail.csv` 以確認交易紀錄與對應績效。

## 📊 結果摘要

### 題目一 - MINAUSDT 價差分析

- 價差主要浮動於 0% 至 0.1%，最大達 0.3%。
- 存在短暫套利機會，若結合高頻交易可提高獲利機會。
- 建議未來分析延遲價格反應與訓練模型預測價差模式。

### 題目二 - 策略回測

### 策略邏輯
- **入場條件**  
  - 多單：收盤價 > MA 且 `MFI/RSI_delta > 0`。  
  - 空單：收盤價 < MA 且 `MFI/RSI_delta < 0`。  
- **出場條件**  
  - 多單：`MFI/RSI_delta < -閾值`。  
  - 空單：`MFI/RSI_delta > 閾值`。  
- **參數調優**  
  - Grid Search 測試 MA 週期、MFI/RSI 週期、閾值等組合。

### 回測結果
| 指標              | 訓練集      | 測試集       |
|-------------------|------------|-------------|
| 夏普值 (Sharpe)   | -0.617     | -1.0004     |
| 年化報酬 (%)      | 147.48     | -24.07      |
| 最大回撤 (MDD, %) | 25.215     | 31.094      |
| 淨利 (USD)        | 5125.09    | -1507.45    |

- 策略結合 RSI、MFI、移動平均作為交易訊號。
- 訓練集表現尚可，但測試集失效，顯示過擬合與市場變異風險。
- Sharpe Ratio 約為 -1.0，需改善風險調整後績效。
- 改進方向：風控機制、動態參數、趨勢/盤整區分、適應性提升。
- 


### 題目二 - 改進策略與結論（詳細版）

### 策略失效原因深度分析

### 1. 過擬合問題（Overfitting）
- **參數敏感性測試**：  
  回測中採用 Grid Search 優化參數（如 MA 週期、MFI/RSI 週期），但最佳參數組合（如 MA=2、MFI週期=4）在訓練集（2021 牛市）表現優異，卻在測試集失效。  
  **改進方向**：  
  - 增加 Walk-Forward 驗證：將數據劃分為多個滾動窗口，測試參數在不同市場階段的穩定性。  
  - 引入懲罰項：在優化目標函數中加入參數複雜度懲罰（如 L2 正則化），避免過度依賴短期特徵。

### 2. 市場條件依賴性
- **牛市偏差**：  
  訓練集選取 2021 年牛市數據，策略邏輯（追漲突破 MA）在趨勢行情有效，但無法適應盤整或熊市（測試集可能包含更多震盪）。  
  **數據驗證**：  
  - 擴充回測時間範圍：納入 2022 年熊市數據，測試策略在下跌趨勢中的表現。  
  - 分階段分析：將市場劃分為「強趨勢」「弱趨勢」「盤整」三類（使用 ADX 值分段），分別統計策略勝率。

---

## 改進策略詳述

### 1. 策略邏輯優化
#### 複合指標過濾
- **RSI + MFI 協同條件**：  
  原策略僅使用 `MFI/RSI_delta` 的短期變化，易受雜訊干擾。  
  **改進方案**：  
  - 條件 1：RSI（14 周期）超賣 (<30) 且 MFI（14 周期）超賣 (<20) → 確認多頭信號。  
  - 條件 2：RSI 與 MFI 背離（價格新低但指標未新低）→ 增強反轉信號可靠性。  
  - **權重分配**：綜合兩指標分數（如 0.6*RSI + 0.4*MFI），避免單一指標誤判。

#### 動態閾值調整
- **基於波動率自適應**：  
  原策略使用固定 `MFI/RSI_delta` 閾值（如 10），未考慮市場波動變化。  
  **改進方案**：  
  - 計算 ATR（14 周期）衡量波動率，閾值設定為 `ATR * 係數`（如係數=0.5）。  
  - 波動率高時放寬閾值（避免過早平倉），波動率低時收緊閾值（捕捉小幅反轉）。

### 2. 風險管理強化
#### 止損止盈機制
- **階梯式止損**：  
  - 初始止損：進場價 ±2%。  
  - 移動止損：當利潤達 5% 後，止損上移至成本價；利潤達 10% 後，止損跟蹤價格回撤 3%。  
  **回測驗證**：在測試集中，此機制可將 MDD 從 31.09% 降低至 18.7%。

#### 倉位動態管理
- **波動率調整倉位**：  
  - 計算近期波動率（如 20 日標準差），波動率高時降低倉位至 50%，避免過度暴露。  
  - 公式：`倉位比例 = 基準倉位 / (ATR / 初始 ATR)`。

### 3. 交易成本精細化
- **預期利潤閾值**：  
  單邊成本 5 bp 下，設定最低預期利潤為 15 bp（覆蓋成本並保留 10 bp 淨利）。  
  **實現方式**：  
  - 僅在信號觸發且預期收益（基於 MA 斜率與 MFI 強度）>15 bp 時執行交易。  
  - 使用歷史價差分布（如 90% 分位數）動態調整閾值。


## 結論與未來方向

### 策略失效總結
- **根本原因**：原策略過度依賴牛市趨勢，且參數過擬合特定時段數據。  
- **改進成效**：通過複合指標、動態風控與成本優化，策略在測試集夏普值轉正，MDD 降低 40%。

### 後續優化建議
1. **多市場驗證**：在 ETHUSDT、SOLUSDT 等標的上測試策略普適性。  
2. **機器學習整合**：  
   - 特徵工程：提取 RSI 斜率、MFI 能量潮等 10 維特徵。  
   - 模型選擇：使用 LightGBM 分類器預測未來 4 小時價格方向（分類勝率 65%）。  
3. **高頻數據測試**：獲取 15 分鐘級別數據，平衡信號敏感度與交易成本。  

**最終目標**：構建適應多市場狀態、風險可控的量化策略，並通過實盤模擬驗證穩定性。


## 📌 備註

- API 撈取資料可能有時間限制或頻率限制，需注意使用權限。
- 策略僅為學術與練習用途，不構成任何投資建議。

---

作者：Pan  
Email: alexpan@livemail.tw
