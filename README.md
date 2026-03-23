# 8-to-3 Encoder 專案說明

本文件重點說明 Verilog 中 `always @(*) begin ... end` 的語法與使用時機，並提供簡單範例。

## 1. `always @(*) begin ... end` 是什麼

`always` 區塊用來描述「當某些訊號改變時，要執行的一段程式碼」。

- `@(...)`：事件觸發條件（敏感度列表）
- `*`：代表「區塊內所有被讀取的訊號」都會自動加入敏感度列表
- `begin ... end`：包住多行敘述

因此：

- `always @(*)` 通常用來描述組合邏輯
- 只要輸入改變，輸出就立即重新計算
- 不需要手動列出每個敏感訊號，較不容易漏寫

## 2. 為什麼組合邏輯常用 `always @(*)`

早期寫法常見：

    always @(a or b or sel) begin
        ...
    end

若漏掉某個訊號，模擬行為可能與實際硬體不一致。

`always @(*)` 可以讓工具自動推導敏感度列表，降低錯誤風險。

## 3. `begin ... end` 的角色

在 `always` 裡面若只有一行敘述，可不寫 `begin ... end`；
若有多行敘述，建議一定使用 `begin ... end` 提升可讀性並避免錯誤。

## 4. 簡單範例：2 對 1 多工器（MUX）

以下是最常見的組合邏輯範例：

    module mux2to1 (
        input  wire a,
        input  wire b,
        input  wire sel,
        output reg  y
    );

    always @(*) begin
        if (sel)
            y = b;
        else
            y = a;
    end

    endmodule

說明：

- 當 `sel=0`，輸出 `y=a`
- 當 `sel=1`，輸出 `y=b`
- 因為是組合邏輯，所以用 `always @(*)`

## 5. 本專案中的實際範例（8 對 3 編碼器）

本專案的 [encoder_8_to_3.v](encoder_8_to_3.v) 使用：

    always @(*) begin
        case (in)
            8'b0000_0001: out = 3'b000;
            8'b0000_0010: out = 3'b001;
            8'b0000_0100: out = 3'b010;
            8'b0000_1000: out = 3'b011;
            8'b0001_0000: out = 3'b100;
            8'b0010_0000: out = 3'b101;
            8'b0100_0000: out = 3'b110;
            8'b1000_0000: out = 3'b111;
            default:      out = 3'b000;
        endcase
    end

這表示：

- 只要 `in` 改變，就重新計算 `out`
- `case` 負責 one-hot 到 3-bit 的編碼
- `default` 可避免未涵蓋情況造成不預期結果

## 6. 使用 `always @(*)` 的常見注意事項

1. 請對所有輸出在所有分支都賦值
   - 若某些分支沒有賦值，工具可能推導出 latch（鎖存器）

2. 組合邏輯常用阻塞賦值 `=`
   - 例如 `out = ...`
   - 時序邏輯（如 `always @(posedge clk)`）才常用非阻塞賦值 `<=`

3. 建議提供 `default` 或完整 `if/else`
   - 可讓行為更明確，也較容易除錯

## 7. 總結

`always @(*) begin ... end` 是 Verilog 描述組合邏輯最常用、最安全的寫法之一：輸入一變，輸出就依照區塊內邏輯立即更新。
