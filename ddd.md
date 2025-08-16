
# KP彙整表測試資料與報表預覽（最終修正版）

## 測試資料來源 (bom90CompsList)

假設我們有以下 5 個 90料號的資料：

### 90料號: 90NB1202-T00010 (Part90Quantity: 2)

**主板**: 60NB1200-MB1121, M6500XU, Qty: 1, IS_SUB: false **CPU**:

- 01002-01411200, 100-000000963, Qty: 1, IS_SUB: false
- 02004-00751100, C.S GN21-X2-K1-A1 FCBGA1358//NVIDIA GB5C-128 AD107-750-K1-A1, Qty: 1, IS_SUB: false **VRAM**: 03014-00021500, K4ZAF325BC-SC16, Qty: 15, IS_SUB: false **SSD**: SSD料號123, 512GB, Qty: 3, IS_SUB: false **WLAN**: 0C011-00250100, Qty: 3, IS_SUB: false **Battery**: 0B200-03750000, SMP/CA436981G/3S1P/11.55V/50WH, X321 BATT/COS POLY/C31N1905, Qty: 2, IS_SUB: false **VGA**: 60PD1234-VGA001, RTX4060/8GB/GDDR6, Qty: 1, IS_SUB: false

### 90料號: 90NB1202-T00020 (Part90Quantity: 1)

**主板**: 60NB1200-MB1011, M6500XU, Qty: 1, IS_SUB: false **CPU**:

- 01002-01411200, 100-000000963, Qty: 1, IS_SUB: false (相同規格)
- 02046-00020200, C.S AU6465RB63-GCF-GR QFN28//ALCOR USB2.0 FLASH CARD READER, Qty: 1, IS_SUB: false **VRAM**: 03014-00021500, K4ZAF325BC-SC16, Qty: 15, IS_SUB: false (相同規格) **SSD**: SSD料號456, 1TB, Qty: 4, IS_SUB: false (不同容量) **WLAN**: 0C011-00280000, Qty: 2, IS_SUB: false (不同料號) **Battery**: 0B200-04140200, SMP/576981/3S1P/11.61V/70W, K3402ZA BAT/ATL POLY/C31N2105, Qty: 4, IS_SUB: false (不同料號+廠商+規格) **VGA**: 60PD1234-VGA001, RTX4060/8GB/GDDR6, Qty: 1, IS_SUB: false (相同料號+規格)

### 90料號: 90NB1221-T00010 (Part90Quantity: 1)

**主板**: 60NB1220-MB1030, M1605XA, Qty: 1, IS_SUB: false **CPU**: 01002-01410800, 100-000000954, Qty: 1, IS_SUB: false (不同規格) **VRAM**: 03016-00010400, MICRON, MT60B1G16HC-48B:A=>D8BNK, Qty: 4, IS_SUB: false (不同規格) **SSD**: SSD料號789, 512GB, Qty: 1, IS_SUB: false (與第一個相同容量) **WLAN**: 0C011-00281200, Qty: 1, IS_SUB: false (不同料號) **Battery**: 0B200-03750000, SMP/CA436981G/3S1P/11.55V/50WH, X321 BATT/COS POLY/C31N1905, Qty: 1, IS_SUB: false (與第一個相同料號+廠商+規格)

### 90料號: 90NB1241-M00090 (Part90Quantity: 2)

**主板**: 60NB1240-MB1320, M6500QF, Qty: 2, IS_SUB: false
**CPU**:
- 01002-01325000, 100-000000295, Qty: 2, IS_SUB: false (新規格)
- 02004-00653400, C.S GN20-S7-B-MP-KB-A1 FCBGA1358//NVIDIA GB5B-128 GA107-715-KB-A1, Qty: 2, IS_SUB: false **VRAM**: 03014-00021500, K4ZAF325BC-SC16, Qty: 15, IS_SUB: false (相同規格) **SSD**: SSD料號999, 512GB, Qty: 3, IS_SUB: false (相同容量) **WLAN**: 0C011-00250100, Qty: 3, IS_SUB: false (相同料號) **Battery**: 0B200-03750000, DYNA/CA576981F/3S1P/11.61V/70W, X321 BATT/COS POLY/C31N1905, Qty: 2, IS_SUB: false (相同料號+規格，但不同廠商) **VGA**: 60PD5678-VGA002, RTX4070/12GB/GDDR6X, Qty: 1, IS_SUB: false (不同料號+規格)

### 90料號: 90NB1241-T00030 (Part90Quantity: 1)

**主板**: 60NB1240-MB1320, M6500QF, Qty: 1, IS_SUB: false **CPU**: 01002-01325000, 100-000000295, Qty: 1, IS_SUB: true (相同規格但替代料) **VRAM**: 03014-00021500, K4ZAF325BC-SC16, Qty: 15, IS_SUB: true (相同規格但替代料) **SSD**: SSD料號AAA, 2TB, Qty: 1, IS_SUB: false (新容量) **WLAN**: 0C011-00300000, Qty: 1, IS_SUB: false (新料號) **Battery**: 0B200-04140000, DYNA/CA576981F/3S1P/11.61V/70W, K3402 BAT/COS POLY/C31N2105, Qty: 1, IS_SUB: false (新料號+廠商+規格) **VGA**: 60PD5678-VGA002, RTX4070/12GB/GDDR6X, Qty: 1, IS_SUB: true (相同料號+規格但替代料)

---

## 總表 (Part90Summary) 預覽 (部分欄位)

|90料號|Qty|機種名稱|主板|Qty|CPU 料號|CPU 規格|Qty|VRAM 料號|VRAM 廠商|VRAM 規格|Qty|SSD 容量|Qty|WLAN 料號|Qty|Battery 料號|Battery 廠商|Battery 規格|Qty|VGA 料號|VGA 規格|Qty|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|90NB1202-T00010|2|M6500XU|60NB1200-MB1121|2|01002-01411200|100-000000963|2|03014-00021500||K4ZAF325BC-SC16|30|512GB|6|0C011-00250100|6|0B200-03750000|SMP/CA436981G/3S1P/11.55V/50WH|X321 BATT/COS POLY/C31N1905|4|60PD1234-VGA001|RTX4060/8GB/GDDR6|2|
||||||02004-00751100|C.S GN21-X2-K1-A1 FCBGA1358//NVIDIA GB5C-128 AD107-750-K1-A1|2||||||||||||||||
|90NB1202-T00020|1|M6500XU|60NB1200-MB1011|1|01002-01411200|100-000000963|1|03014-00021500||K4ZAF325BC-SC16|15|1TB|4|0C011-00280000|2|0B200-04140200|SMP/576981/3S1P/11.61V/70W|K3402ZA BAT/ATL POLY/C31N2105|4|60PD1234-VGA001|RTX4060/8GB/GDDR6|1|
||||||02046-00020200|C.S AU6465RB63-GCF-GR QFN28//ALCOR USB2.0 FLASH CARD READER|1||||||||||||||||
|90NB1221-T00010|1|M1605XA|60NB1220-MB1030|1|01002-01410800|100-000000954|1|03016-00010400|MICRON|MT60B1G16HC-48B:A=>D8BNK|4|512GB|1|0C011-00281200|1|0B200-03750000|SMP/CA436981G/3S1P/11.55V/50WH|X321 BATT/COS POLY/C31N1905|1||||
|90NB1241-M00090|2|M6500QF|60NB1240-MB1320|4|01002-01325000|100-000000295|4|03014-00021500||K4ZAF325BC-SC16|30|512GB|6|0C011-00250100|6|0B200-03750000|DYNA/CA576981F/3S1P/11.61V/70W|X321 BATT/COS POLY/C31N1905|4|60PD5678-VGA002|RTX4070/12GB/GDDR6X|2|
||||||02004-00653400|C.S GN20-S7-B-MP-KB-A1 FCBGA1358//NVIDIA GB5B-128 GA107-715-KB-A1|4||||||||||||||||
|90NB1241-T00030|1|M6500QF|60NB1240-MB1320|1|01002-01325000 (替代料)|100-000000295|1|03014-00021500 (替代料)||K4ZAF325BC-SC16|15|2TB|1|0C011-00300000|1|0B200-04140000|DYNA/CA576981F/3S1P/11.61V/70W|K3402 BAT/COS POLY/C31N2105|1|60PD5678-VGA002 (替代料)|RTX4070/12GB/GDDR6X|1|

---

## KP彙整表 (KpSummary) 預覽

**重點：每種零件類型都從第1列開始，彼此獨立**

| CPU 規格                                                            | Qty | PCH 規格 | Qty | GPU 規格 | Qty | VRAM 規格                  | Qty | On-Board DIMM 規格 | Qty | SSD 容量 | Qty | DDR 容量 | Qty | HDD 容量 | Qty | WLAN 料號        | Qty | Battery 料號     | Battery 廠商                     | Battery 規格                    | Qty | VGA 料號                | VGA 規格              | Qty |
| ----------------------------------------------------------------- | --- | ------ | --- | ------ | --- | ------------------------ | --- | ---------------- | --- | ------ | --- | ------ | --- | ------ | --- | -------------- | --- | -------------- | ------------------------------ | ----------------------------- | --- | --------------------- | ------------------- | --- |
| 100-000000963                                                     | 3   |        |     |        |     | K4ZAF325BC-SC16          | 75  |                  |     | 512GB  | 13  |        |     |        |     | 0C011-00250100 | 12  | 0B200-03750000 | SMP/CA436981G/3S1P/11.55V/50WH | X321 BATT/COS POLY/C31N1905   | 5   | 60PD1234-VGA001       | RTX4060/8GB/GDDR6   | 3   |
| C.S GN21-X2-K1-A1 FCBGA1358//NVIDIA GB5C-128 AD107-750-K1-A1      | 2   |        |     |        |     | MT60B1G16HC-48B:A=>D8BNK | 4   |                  |     | 1TB    | 4   |        |     |        |     | 0C011-00280000 | 2   | 0B200-04140200 | SMP/576981/3S1P/11.61V/70W     | K3402ZA BAT/ATL POLY/C31N2105 | 4   | 60PD5678-VGA002       | RTX4070/12GB/GDDR6X | 2   |
| C.S AU6465RB63-GCF-GR QFN28//ALCOR USB2.0 FLASH CARD READER       | 1   |        |     |        |     | K4ZAF325BC-SC16 (替代料)    | 15  |                  |     | 2TB    | 1   |        |     |        |     | 0C011-00281200 | 1   | 0B200-03750000 | DYNA/CA576981F/3S1P/11.61V/70W | X321 BATT/COS POLY/C31N1905   | 4   | 60PD5678-VGA002 (替代料) | RTX4070/12GB/GDDR6X | 1   |
| 100-000000954                                                     | 1   |        |     |        |     |                          |     |                  |     |        |     |        |     |        |     | 0C011-00300000 | 1   | 0B200-04140000 | DYNA/CA576981F/3S1P/11.61V/70W | K3402 BAT/COS POLY/C31N2105   | 1   |                       |                     |     |
| 100-000000295                                                     | 4   |        |     |        |     |                          |     |                  |     |        |     |        |     |        |     |                |     |                |                                |                               |     |                       |                     |     |
| C.S GN20-S7-B-MP-KB-A1 FCBGA1358//NVIDIA GB5B-128 GA107-715-KB-A1 | 4   |        |     |        |     |                          |     |                  |     |        |     |        |     |        |     |                |     |                |                                |                               |     |                       |                     |     |
| 100-000000295 (替代料)                                               | 1   |        |     |        |     |                          |     |                  |     |        |     |        |     |        |     |                |     |                |                                |                               |     |                       |                     |     |

---

## 計算邏輯驗證

### CPU 規格統計（規格 + IS_SUB）：

- **100-000000963 (IS_SUB: false)**: 2×1 + 1×1 = **3**
- **C.S GN21-X2-K1-A1... (IS_SUB: false)**: 2×1 = **2**
- **C.S AU6465RB63-GCF-GR... (IS_SUB: false)**: 1×1 = **1**
- **100-000000954 (IS_SUB: false)**: 1×1 = **1**
- **100-000000295 (IS_SUB: false)**: 2×2 = **4**
- **C.S GN20-S7-B-MP-KB-A1... (IS_SUB: false)**: 2×2 = **4**
- **100-000000295 (IS_SUB: true)**: 1×1 = **1**

### VRAM 規格統計（規格 + IS_SUB）：

- **K4ZAF325BC-SC16 (IS_SUB: false)**: 2×15 + 1×15 + 2×15 = **75**
- **MT60B1G16HC-48B:A=>D8BNK (IS_SUB: false)**: 1×4 = **4**
- **K4ZAF325BC-SC16 (IS_SUB: true)**: 1×15 = **15**

### SSD 容量統計（容量 + IS_SUB）：

- **512GB (IS_SUB: false)**: 2×3 + 1×1 + 2×3 = **13**
- **1TB (IS_SUB: false)**: 1×4 = **4**
- **2TB (IS_SUB: false)**: 1×1 = **1**

### Battery 複合統計（料號 + 廠商 + 規格 + IS_SUB）：

- **0B200-03750000 + SMP/CA436981G/3S1P/11.55V/50WH + X321 BATT/COS POLY/C31N1905 (IS_SUB: false)**: 2×2 + 1×1 = **5**
- **0B200-04140200 + SMP/576981/3S1P/11.61V/70W + K3402ZA BAT/ATL POLY/C31N2105 (IS_SUB: false)**: 1×4 = **4**
- **0B200-03750000 + DYNA/CA576981F/3S1P/11.61V/70W + X321 BATT/COS POLY/C31N1905 (IS_SUB: false)**: 2×2 = **4**
- **0B200-04140000 + DYNA/CA576981F/3S1P/11.61V/70W + K3402 BAT/COS POLY/C31N2105 (IS_SUB: false)**: 1×1 = **1**

### VGA 複合統計（料號 + 規格 + IS_SUB）：

- **60PD1234-VGA001 + RTX4060/8GB/GDDR6 (IS_SUB: false)**: 2×1 + 1×1 = **3**
- **60PD5678-VGA002 + RTX4070/12GB/GDDR6X (IS_SUB: false)**: 2×1 = **2**
- **60PD5678-VGA002 + RTX4070/12GB/GDDR6X (IS_SUB: true)**: 1×1 = **1**

---

## 關鍵理解確認

1. **稀疏矩陣邏輯**：每種零件類型從第1列開始填入，彼此完全獨立
2. **替代料分離**：主料和替代料即使規格相同也分開統計
3. **複合分組**：Battery 和 VGA 使用多欄位組合進行 distinct
4. **數量精確計算**：part90Quantity × partQuantity 的正確加總

這次的理解是否完全正確？