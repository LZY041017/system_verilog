# 批次 01：从 MAC 读懂 SystemVerilog 基础

来源：当前项目的 `rtl/mac_pe.sv` 与 `tb/mac_pe_tb.sv`，部分片段为保持聚焦而做了删减或对照改写。

建议顺序：每次读 4 个片段，控制在 10 分钟；先把你的答案写在纸上，再点开参考分析。

---

## 01｜端口中的 `logic`

### 代码片段

```systemverilog
input  logic               clk,
input  logic               rst_n,
input  logic               valid,
output logic signed [31:0] acc_out
```

### 你的判断

1. `acc_out` 的位宽、符号属性分别是什么？
2. 为什么 `clk` 不需要写 `[0:0]`？
3. `logic` 是否等于“它一定是寄存器”？

<details>
<summary>参考分析</summary>

`acc_out` 是 32 位有符号四态信号；`signed` 决定它参与算术表达式时按有符号数解释。未写位宽的 `clk`、`rst_n`、`valid` 默认都是 1 位。`logic` 是 SystemVerilog 推荐的信号类型，不等同于“寄存器”；它既可被组合逻辑驱动，也可被时序过程驱动，但同一信号不应由多个过程同时驱动。

</details>

### 波形观察点

在 GTKWave 中把 `acc_out` 的显示格式改为 **Signed Decimal**，比较它与默认二进制显示的差异。

---

## 02｜`signed` 放在哪里

### 代码片段

```systemverilog
input logic signed [7:0] a,
input logic signed [7:0] b,
logic signed [15:0] product;
```

### 你的判断

1. `a = 8'hFE` 时，它代表 254 还是 -2？
2. 两个 signed int8 相乘，为什么 `product` 需要 16 位？
3. 若把 `a` 的 `signed` 删除，负数乘法可能出现什么问题？

<details>
<summary>参考分析</summary>

`8'hFE` 在 signed 8 位解释下是 -2。两个 N 位有符号整数的完整乘积通常需要 2N 位；因此 int8 x int8 放到 16 位。删掉 `signed` 后，同样的位模式会按无符号数参与运算，例如 `8'hFE` 会变成 254，这会让负数乘法结果错误。

</details>

---

## 03｜连续赋值

### 代码片段

```systemverilog
assign product = a * b;
```

### 你的判断

1. 这行代码是否等待时钟？
2. `a` 在 20 ns 改变后，`product` 何时更新？
3. 它描述的是组合逻辑还是存储状态？

<details>
<summary>参考分析</summary>

`assign` 描述连续的组合逻辑，不等待任何时钟沿。当 `a` 或 `b` 变化时，`product` 会在仿真 delta cycle 后重新计算。它不保存历史状态；真实硬件中对应乘法组合路径（综合工具可能映射为 DSP 或逻辑）。

</details>

---

## 04｜时序过程的触发条件

### 代码片段

```systemverilog
always_ff @(posedge clk or negedge rst_n) begin
    // ...
end
```

### 你的判断

1. 这个过程会被哪两种事件唤醒？
2. `rst_n` 从 1 变 0 时，是否需要等待时钟？
3. 这是什么类型的复位？

<details>
<summary>参考分析</summary>

它在 `clk` 上升沿，或 `rst_n` 下降沿触发。因为复位边沿直接出现在敏感列表中，`rst_n` 拉低会立刻执行复位分支，不等待时钟，因此这是**异步低有效复位**。若只有 `@(posedge clk)`，并在内部判断 `if (!rst_n)`，则会是同步复位。

</details>

---

## 05｜复位优先级

### 代码片段

```systemverilog
if (!rst_n)
    acc_out <= '0;
else if (clear)
    acc_out <= '0;
else if (valid)
    acc_out <= acc_out + product;
```

### 你的判断

1. `clear=1` 且 `valid=1` 时发生什么？
2. 为什么 `clear` 必须放在 `valid` 前面？
3. 这段优先级如何用一句话复述？

<details>
<summary>参考分析</summary>

`clear=1` 时清零，`valid` 被忽略。`if / else if` 从上到下只会执行第一个成立分支，因此顺序就是硬件行为的一部分。完整优先级：**异步复位 > 清零 > 有效累加 > 保持**。

</details>

---

## 06｜没有 `else` 的保持

### 代码片段

```systemverilog
else if (valid)
    acc_out <= acc_out + product;
```

### 你的判断

1. 如果 `valid=0`，这一拍 `acc_out` 变成什么？
2. 为什么这里没有 `else` 也不会推断锁存器？

<details>
<summary>参考分析</summary>

`valid=0` 时这个时序过程没有对 `acc_out` 赋新值，触发器自然保持上一个值。锁存器风险主要出现在 `always_comb` 中：组合过程没有覆盖全部赋值路径时，综合工具才需要推断存储。这里本来就在 `always_ff` 中，保持就是触发器的正常行为。

</details>

---

## 07｜非阻塞赋值 `<=`

### 代码片段

```systemverilog
acc_out <= acc_out + product;
```

### 你的判断

1. 右侧的 `acc_out` 是旧值还是新值？
2. 如果把它写成 `acc_out = acc_out + product;`，在复杂时序块中会有什么风险？

<details>
<summary>参考分析</summary>

在时钟沿到来时，右侧先读取旧 `acc_out` 和旧/当前组合 `product`，再把更新安排到当前时间步末尾统一提交。`<=` 模拟真实触发器“同时采样、同时更新”的行为。时序逻辑中混用阻塞赋值 `=` 容易产生仿真顺序依赖，多个寄存器看起来不像同时更新。

</details>

---

## 08｜符号扩展拼接

### 代码片段

```systemverilog
{{16{product[15]}}, product}
```

### 你的判断

1. 这段表达式最终宽度是多少？
2. `product[15]` 为 1 时，高 16 位是什么？
3. 若直接把 `product` 加到 32 位 `acc_out`，为什么仍建议显式写出它？

<details>
<summary>参考分析</summary>

内部 `{16{product[15]}}` 把符号位重复 16 次，再与 16 位 `product` 拼接，得到 32 位数。若 `product[15]=1`，高 16 位全为 1，负数才能保持负值。语言在一些表达式中会进行自动扩展，但 signed/unsigned 混合时很容易产生歧义；显式符号扩展让位宽和意图都清楚。

</details>

---

## 09｜`'0` 的含义

### 代码片段

```systemverilog
acc_out <= '0;
a       = '0;
```

### 你的判断

1. `'0` 是 1 位、8 位还是 32 位？
2. 为什么它比 `32'd0` 更适合通用初始化？

<details>
<summary>参考分析</summary>

`'0` 是“按左侧目标的宽度填满 0”的无尺寸字面量。赋给 `acc_out` 时是 32 位 0，赋给 `a` 时是 8 位 0。它对参数化模块尤其方便，因为目标宽度变化时不必手改常量位数。

</details>

---

## 10｜测试平台中的时钟

### 代码片段

```systemverilog
always #5 clk = ~clk;
```

### 你的判断

1. 时钟周期是多少？
2. 第一个上升沿发生在多少时间？
3. 为什么它不应该出现在可综合 RTL 中？

<details>
<summary>参考分析</summary>

每 5 个时间单位翻转一次，因此完整周期为 10 个时间单位。若初值为 0，第一个上升沿在 5。`#5` 是仿真延迟，不能综合成真实电路中的时钟发生器；它只属于 testbench。

</details>

---

## 11｜为什么在下降沿改变输入

### 代码片段

```systemverilog
@(negedge clk);
valid = valid_i;
a = a_i;
b = b_i;
@(posedge clk);
```

### 你的判断

1. 输入为什么不在上升沿再改？
2. 上升沿到来时，DUT 看到的是新输入还是旧输入？
3. 这里用阻塞赋值 `=` 是否合理？

<details>
<summary>参考分析</summary>

在下降沿驱动输入，可以让输入在下一个上升沿前稳定半个周期，避免 testbench 与 DUT 在同一上升沿竞争。到上升沿时 DUT 看到新输入。这里是 testbench 的顺序激励代码，使用阻塞赋值是合理的：我们希望语句按顺序立刻更新信号。

</details>

---

## 12｜为什么要等 `#1`

### 代码片段

```systemverilog
@(posedge clk);
#1;
if (acc_out !== expected) begin
    $error("FAIL");
end
```

### 你的判断

1. `#1` 主要在等待什么？
2. 若没有 `#1`，可能检查到什么值？
3. 这个 `#1` 是否表示真实芯片传播延迟？

<details>
<summary>参考分析</summary>

它等待 DUT 在上升沿触发的非阻塞赋值提交完成。若立即检查，testbench 有可能仍读到更新前的 `acc_out`。这里的 `#1` 是仿真调度上的方便做法，不应理解为真实硬件真的延迟了 1 ns。

</details>

---

## 13｜`!==` 与 `!=`

### 代码片段

```systemverilog
if (acc_out !== expected)
```

### 你的判断

1. `!==` 比 `!=` 多检查了什么？
2. 测试平台为什么更偏向 `!==`？

<details>
<summary>参考分析</summary>

`!==` 是 case inequality：它把 `X` 和 `Z` 也当成可比较的位。普通 `!=` 遇到未知值可能产生 `X`，导致 `if` 不按预期进入失败分支。自检 testbench 应把未知值视为错误，因此常用 `===` / `!==`。

</details>

---

## 14｜自动 task

### 代码片段

```systemverilog
task automatic drive_and_check(
    input logic signed [7:0] a_i,
    input logic signed [31:0] expected,
    input string test_name
);
```

### 你的判断

1. task 在 testbench 中的作用是什么？
2. `automatic` 解决什么问题？
3. `expected` 为什么是 32 位 signed？

<details>
<summary>参考分析</summary>

task 把“驱动输入、等一个时钟、检查结果”的重复流程封装起来。`automatic` 表示每次调用拥有独立的局部变量与参数存储，适合未来并发调用。`acc_out` 是 32 位有符号数，所以 expected 使用完全相同的类型，避免比较时出现位宽或符号歧义。

</details>

---

## 15｜波形导出

### 代码片段

```systemverilog
$dumpfile("build/mac_pe_tb.vcd");
$dumpvars(0, mac_pe_tb);
```

### 你的判断

1. 两行分别做什么？
2. `0` 的层级深度意味着什么？
3. 为什么 VCD 能帮助定位“测试失败但看不出原因”的情况？

<details>
<summary>参考分析</summary>

`$dumpfile` 指定 VCD 输出文件；`$dumpvars` 指定记录哪些层级的信号。深度为 0 表示从 `mac_pe_tb` 开始递归记录全部子层级，所以 DUT 内部的 `product` 也会被记录。终端只告诉你“期望值不等于实际值”，而波形能显示是哪一个周期、哪组输入、哪条控制信号先出错。

</details>

---

## 16｜综合与仿真的边界

### 代码片段

```systemverilog
$display("PASS: %s -> %0d", test_name, acc_out);
$finish;
```

### 你的判断

1. 它们是否会变成芯片电路？
2. 这两行分别对仿真器做了什么？

<details>
<summary>参考分析</summary>

它们都不可综合，不会变成芯片电路。`$display` 向仿真终端打印信息；`$finish` 正常结束仿真。把它们放在 `tb/` 而不是 `rtl/`，正是硬件设计代码和验证代码的基本边界。

</details>

---

## 完成本批后的自测

不看答案，用三句话解释当前 MAC：

1. 数据如何从 `a/b` 到 `acc_out`？
2. 哪些信号决定本周期是否累加？
3. testbench 如何避免和 DUT 在同一上升沿争夺输入，并如何确认结果？

如果三句都能说清，再进入下一批：`always_comb`、`case`、参数、数组与简单 FIFO。
