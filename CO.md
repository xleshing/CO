## 1. MIPS 指令集架構 (ISA)
MIPS 指令集架構是一種精簡指令集（RISC），設計上強調簡單且規律性，以提升效能。

- **指令格式**：MIPS 主要有三種指令格式：
  - **R 格式**：適用於寄存器間操作，包含操作碼 (opcode)、來源寄存器 (rs, rt)、目標寄存器 (rd)、函數碼 (funct) 等欄位。
  - **I 格式**：用於立即數、分支和載入/存儲操作，包含操作碼、來源寄存器、目標寄存器以及偏移量/立即數。
  - **J 格式**：適用於跳躍指令，包含操作碼和目標地址欄位。

以下是 MIPS 指令集架構 (ISA) 中的 R、I、J 格式，以及性能指標、虛擬指令與組合指令、CPU 時間與性能分析的更詳細解說：

---
 
#### R 格式 (Register Format)
R 格式主要用於寄存器間操作，如算術運算和邏輯運算。

| 欄位名稱 | 位數 | 說明 |
| -------- | ---- | ---- |
| opcode   | 6    | 操作碼，指令類型的標識 |
| rs       | 5    | 第一個來源寄存器 |
| rt       | 5    | 第二個來源寄存器或目標寄存器 |
| rd       | 5    | 目標寄存器 |
| shamt    | 5    | 移位數量，用於移位指令，其餘指令中設定為 0 |
| funct    | 6    | 函數碼，指定指令的具體操作，如加法、減法等 |

- **常見指令**：
  - `add rd, rs, rt`：將寄存器 `rs` 和 `rt` 的值相加，結果儲存在 `rd`。
  - `sub rd, rs, rt`：將寄存器 `rs` 減去 `rt`，結果儲存在 `rd`。
  - `sll rd, rt, shamt`：將 `rt` 的值左移 `shamt` 位，結果儲存在 `rd`。

#### I 格式 (Immediate Format)
I 格式適用於涉及立即數的操作、分支指令和記憶體訪問（載入/存儲）指令。

| 欄位名稱 | 位數 | 說明 |
| -------- | ---- | ---- |
| opcode   | 6    | 操作碼 |
| rs       | 5    | 來源寄存器 |
| rt       | 5    | 目標寄存器或基址寄存器 |
| immediate| 16   | 立即數或偏移量 |

- **常見指令**：
  - `addi rt, rs, immediate`：將 `rs` 的值加上立即數 `immediate`，結果存儲在 `rt`。
  - `lw rt, offset(rs)`：從記憶體地址 `rs + offset` 中載入值至 `rt`。
  - `beq rs, rt, offset`：當 `rs` 和 `rt` 相等時，跳轉至 `PC + 4 + offset * 4` 的位置。

#### J 格式 (Jump Format)
J 格式主要用於跳躍指令，適合遠距離的無條件跳轉。

| 欄位名稱 | 位數 | 說明 |
| -------- | ---- | ---- |
| opcode   | 6    | 操作碼 |
| address  | 26   | 目標地址，透過 `PC` 的高 4 位和低位 26 位組合成 32 位目標地址 |

- **常見指令**：
  - `j target`：跳轉至指定的目標地址。
  - `jal target`：跳轉至目標地址並將返回地址存入 `$ra` 寄存器。

---

- **常見指令**：
  - **算術指令**：`add`, `sub`, `addi` 等。
  - **邏輯指令**：`and`, `or`, `xor`, `nor` 等。
  - **移位指令**：`sll`（左移位），`srl`（右移位）。
  - **分支指令**：`beq`（相等時分支），`bne`（不相等時分支），`j`（無條件跳躍），`jal`（跳躍並鏈接）。
  - **資料傳輸指令**：`lw`（載入字），`sw`（存儲字）。

- **指令組合與虛擬指令**：MIPS 缺少某些直接指令（例如 `blt`, `bge`），可以透過 `slt`, `bne`, `beq` 組合來模擬。例如，若要實現「小於則跳躍」，可以透過 `slt` 和 `bne` 的組合來達成。

## 2. 定址模式 (Addressing Modes)
定址模式指令操作數的來源。MIPS 定址模式的分類和使用情境如下：

  - **Register Addressing**：直接從寄存器中讀取操作數。
  - **Immediate Addressing**：立即數作為操作數（如 `addi`）。
  - **Base Addressing**：基址加偏移量，用於存取陣列或結構（如 `lw`, `sw`）。
  - **PC-Relative Addressing**：相對於 PC 的偏移量，通常在分支指令中使用（如 `beq`, `bne`）。
  - **Pseudo-direct Addressing**：直接跳轉的地址，適用於跳躍指令（如 `j`, `jal`）。

- **PC-Relative Addressing 與 Base Addressing 的區別**：
  - **PC-Relative** 基於當前指令位置的偏移，用於分支控制。
  - **Base Addressing** 基於基址寄存器和偏移量，用於資料存取。

## 3. 資料儲存的 Endian 模式
  - **Big Endian**：數據的高位元組存儲在低地址。
  - **Little Endian**：數據的低位元組存儲在低地址。
  - 示例：以 `0x12345678` 為例，在 Big Endian 格式下，按順序 `12 34 56 78` 存儲；在 Little Endian 格式下，則是 `78 56 34 12`。

## 4. 跳躍指令 (jr 與 jal)
  - **jr (jump register)**：從寄存器中取出地址並跳轉，用於子程序返回。
  - **jal (jump and link)**：跳轉至指定地址，並將返回地址保存在 `$ra` 中。用於調用子程序，方便返回時使用 `jr $ra`。

## 5. 暫存器搬移
MIPS 沒有 `move` 指令，可以使用 `add` 指令與 `$zero` 實現寄存器搬移。例如，將 `$s1` 搬移至 `$t2` 可表達為 `add $t2, $s1, $zero`。

## 6. Pipeline 與執行階段
MIPS 處理器透過流水線 (Pipeline) 技術來提高指令吞吐量，將指令分為以下五個執行階段：

  - **IF (Instruction Fetch)**：從記憶體中取出指令。
  - **ID (Instruction Decode)**：解碼指令並讀取寄存器數據。
  - **EX (Execute)**：ALU 執行算術或邏輯運算。
  - **MEM (Memory Access)**：進行記憶體訪問，適用於載入/存儲指令。
  - **WB (Write Back)**：將結果寫回寄存器。

## 7. Pipeline Hazards (流水線危障)
流水線中的指令間有依賴關係，會導致三種危障：

  - **結構性危障 (Structural Hazard)**：多條指令需要相同的硬體資源，導致衝突。
  - **資料危障 (Data Hazard)**：指令間存在數據依賴，當指令需要依賴前一指令的結果時會發生。
    - **解決方法**：透過前饋 (Forwarding) 或插入 NOP 指令來解決。
  - **控制危障 (Control Hazard)**：分支指令改變控制流，導致不確定應執行哪條指令。
    - **解決方法**：透過分支預測技術（如靜態預測和動態預測）來減少控制危障的影響。

## 8. 前饋與分支預測
  - **前饋 (Forwarding)**：允許 ALU 的運算結果在寫回寄存器前即被使用，以減少資料危障帶來的延遲。
  - **分支預測 (Branch Prediction)**：
    - **靜態預測**：預測分支不成立（如跳過分支指令）。
    - **動態預測**：基於過去的分支結果，儲存分支歷史並根據歷史預測未來。

## 9. 性能指標與計算
  - **CPI (Cycles Per Instruction)**：平均每條指令所需的時鐘週期數，用於評估指令執行效率。
  - **MIPS (Million Instructions Per Second)**：指令執行速度，MIPS 值越高代表效能越好。
  - **CPU 執行時間公式**：
    \[
    \text{CPU 時間} = \frac{\text{指令數量} \times \text{CPI}}{\text{時鐘頻率}}
    \]
  - **Rpeak 計算浮點運算單元數量 (FPU)**：使用超級計算機中的核心數量、核心頻率和理論峰值運算速度 (Rpeak) 來估算浮點運算單元。

## 10. 虛擬指令與組合指令
  - MIPS 中，某些虛擬指令 (Pseudo Instructions) 並不存在，但可透過其他指令組合來實現。例如 `bge` 可以用 `slt` 和 `bne` 指令來實現。
  - 常見組合：
    - `move $t2, $s1` 可以替換為 `add $t2, $s1, $zero`。
    - `bge $s1, $s2, L` 可以替換為 `slt $t0, $s1, $s2` 和 `beq $t0, $zero, L`。

---

MIPS 的一些指令需要透過其他指令組合來實現，這些指令被稱為「虛擬指令 (Pseudo Instructions)」。虛擬指令並不是真正的硬體指令，而是由編譯器將其轉換為硬體可執行的基本指令組合。

#### 常見虛擬指令和組合
- **移動指令 (move)**：MIPS 沒有 `move` 指令，通常使用 `add` 指令和 `$zero` 寄存器來實現。例如，將 `$s1` 移至 `$t2` 可寫成：
  ```assembly
  add $t2, $s1, $zero
  ```
- **大於等於條件跳躍 (bge)**：MIPS 沒有 `bge` 指令，可以用 `slt` 和 `beq` 組合來模擬。例如，要達成 `bge $s1, $s2, L`，可以寫成：
  ```assembly
  slt $t0, $s2, $s1
  beq $t0, $zero, L
  ```
- **小於條件跳躍 (blt)**：`blt` 指令可用 `slt` 和 `bne` 實現。例如：
  ```assembly
  slt $t0, $s1, $s2
  bne $t0, $zero, L
  ```


## 11. CPU 時間與性能分析
  - **時鐘週期 (Clock Cycle)**：每秒的時鐘信號次數，決定 CPU 指令執行速度。
  - **CPI 與時鐘週期時間的關係**：時鐘週期越短、CPI 越低則性能越高。使用多階段 Pipeline 時，CPI 會低於單階段 Pipeline。
  - **執行時間與 MIPS 評估**：CPU 時間的增減與 MIPS 數值的變化可以有效指標化 CPU 的性能差異。

---

#### 1. CPU 時間計算
CPU 的執行時間可以透過指令數量、CPI 和時鐘頻率來計算：

$$ CPU 時間 = {{指令數量} \times {CPI}}\over {時鐘頻率} $$

其中：
- **指令數量**：程式中需要執行的指令數量。
- **CPI**：每條指令平均需要的時鐘週期數。
- **時鐘頻率**：CPU 每秒運行的時鐘週期數，通常以 GHz 為單位。

#### 2. CPU 性能比較
若要比較兩個 CPU 的效能，可以使用執行時間公式來進行。假設有兩台 CPU，分別為 CPU A 和 CPU B：

- **CPU A**：時鐘週期時間為 500 ns，CPI 為 1.5。
- **CPU B**：時鐘週期時間為 400 ns，CPI 為 1.8。

比較方式：
1. 計算每台 CPU 的執行時間：
   - **CPU A** 的執行時間：\( 500 \times 1.5 = 750 \) ns。
   - **CPU B** 的執行時間：\( 400 \times 1.8 = 720 \) ns。
2. 比較兩者：
   - **CPU B 較快**，其性能是 **CPU A 的 \( \frac{750}{720} \approx 1.042 \) 倍**。

#### 3. IPC (Instructions Per Cycle)
IPC 是指每個時鐘週期內完成的指令數量，可透過以下公式計算：

$$ IPC = \frac{1}{CPI} $$

IPC 值越高，CPU 的指令吞吐量越

高。現代 CPU 通常會採用多重流水線技術來提升 IPC 值，以增加單位時間內的指令完成數。

#### 4. 系統效能瓶頸與改善
影響 CPU 效能的因素還包括記憶體存取延遲、分支預測準確性和 Pipeline 停滯現象等。常見的改善方法包括：

- 增加前饋 (Forwarding) 機制，解決資料危障。
- 使用分支預測來降低控制危障的影響。
- 引入多重 Pipeline 或超純量結構來提高 IPC。

---

這些筆記涵蓋了 MIPS 指令集的各個面向，從基本的指令格式、定址模式到 Pipeline 運作及危障處理，並詳細說明了常見的組合指令

及性能評估方法。這樣的筆記不僅可以作為考試的準備，也能幫助更深入地理解 MIPS 架構。



### 9. 性能指標與計算

在評估 CPU 性能時，MIPS 使用以下幾個關鍵指標：

#### 1. CPI (Cycles Per Instruction)
CPI 是指平均每條指令需要的時鐘週期數，越低的 CPI 表示每條指令所需的時間越少，性能越高。

- **計算公式**：
  \[
  \text{CPI} = \frac{\text{總時鐘週期}}{\text{指令數量}}
  \]
- CPI 的影響因素包括指令組成、指令執行結構（如 Pipeline）、記憶體速度等。

#### 2. MIPS (Million Instructions Per Second)
MIPS 是每秒執行的百萬條指令數，表示 CPU 執行速度。MIPS 的值越高，代表 CPU 性能越好。

- **計算公式**：
  \[
  \text{MIPS} = \frac{\text{時鐘頻率（MHz）}}{\text{CPI}}
  \]
- **例子**：若 CPU 時鐘頻率為 2 GHz（2000 MHz），CPI 為 2，則 MIPS 值為 \( \frac{2000}{2} = 1000 \) MIPS。

#### 3. CPU 執行時間
CPU 的執行時間是衡量一個程式執行時間的指標，公式為：
\[
\text{CPU 時間} = \text{指令數} \times \text{CPI} \times \text{時鐘週期時間}
\]
其中，時鐘週期時間等於 \( \frac{1}{\text{時鐘頻率}} \)。