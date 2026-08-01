# 批次 02：`always_comb`、`case`、参数、数组与简单 FIFO

来源：`training/02`、`training/05`、`training/07` 中的简化/改写片段。目标不是一下子写出 FIFO，而是逐个看懂其组成部分。

---

## 01｜组合逻辑的默认赋值

### 代码片段

```systemverilog
always_comb begin
    y = 8'h00;
    if (select)
        y = a;
end
```

### 你的判断

1. `select=0` 时 `y` 是什么？
2. 这段是否需要时钟？
3. 为什么第一行先给 `y` 一个默认值？

<details>
<summary>参考分析</summary>

这是组合逻辑，`select=0` 时输出 0，`select=1` 时输出 `a`。默认赋值确保所有路径都给 `y` 赋值；若没有它，`select=0` 时 `y` 需要“记住旧值”，综合时可能推断锁存器。

</details>

---

## 02｜组合逻辑中的锁存器陷阱

### 代码片段

```systemverilog
always_comb begin
    if (enable)
        y = a + b;
end
```

### 你的判断

1. `enable=0` 时，硬件需要怎样得到 `y`？
2. 这段与上一段的关键差异是什么？

<details>
<summary>参考分析</summary>

`enable=0` 时没有任何赋值，`y` 只能保留旧值，因此会推断锁存器（或被 lint 报错）。组合模块通常不希望隐藏存储状态，应在开头写默认值，或明确写出 `else`。

</details>

---

## 03｜`always_comb` 与 `assign`

### 代码片段

```systemverilog
assign sum0 = a + b;

always_comb begin
    sum1 = a + b;
end
```

### 你的判断

1. 两者在这段简单代码中功能是否相同？
2. 何时更适合 `assign`？何时更适合 `always_comb`？

<details>
<summary>参考分析</summary>

这里功能相同，都是组合逻辑。单个、直接的表达式通常用 `assign` 更简洁；存在多个临时变量、条件分支或 `case` 时，`always_comb` 更清晰。不要为同一个信号同时写 `assign` 和 `always_comb`，那会形成多个驱动。

</details>

---

## 04｜完整的 `if / else`

### 代码片段

```systemverilog
always_comb begin
    if (signed_mode)
        result = $signed(a) + $signed(b);
    else
        result = a + b;
end
```

### 你的判断

1. 为什么这里不需要额外默认赋值？
2. `signed_mode` 是“硬件配置位”还是“编译选项”？

<details>
<summary>参考分析</summary>

两个分支都覆盖了 `result`，因此不需要默认值。`signed_mode` 是输入信号，综合后对应运行时可改变的硬件控制位；它不是编译期的参数。注意：即使使用 `$signed`，`result` 本身的位宽仍会影响是否截断。

</details>

---

## 05｜地址译码 `case`

### 代码片段

```systemverilog
always_comb begin
    case (rd_addr)
        4'h0:    rd_data = control;
        4'h4:    rd_data = scale;
        default: rd_data = '0;
    endcase
end
```

### 你的判断

1. `rd_addr=4'h8` 时输出什么？
2. `default` 在硬件设计中为什么重要？
3. 这段代表寄存器读取还是寄存器写入？

<details>
<summary>参考分析</summary>

地址 8 不匹配前两项，因此输出全 0。`default` 使未映射地址有确定行为，并避免组合逻辑漏赋值。它是组合读取逻辑；真正改变 `control/scale` 的写入会在 `always_ff` 中发生。

</details>

---

## 06｜`case` 没有 `default` 会怎样

### 代码片段

```systemverilog
always_comb begin
    case (opcode)
        2'b00: result = a + b;
        2'b01: result = a - b;
    endcase
end
```

### 你的判断

1. `opcode=2'b10` 时可能怎样？
2. 两种修复方式是什么？

<details>
<summary>参考分析</summary>

当 `opcode` 为 10 或 11，`result` 没有赋值，会推断锁存器。修复方式一：在 `always_comb` 开头给 `result` 默认值；方式二：添加 `default: result = '0;`。工程上常同时使用默认值和 `default`，以提高可读性。

</details>

---

## 07｜参数改变的是“生成的硬件”

### 代码片段

```systemverilog
module configurable_counter #(
    parameter int Width = 4
) (
    output logic [Width-1:0] count
);
```

### 你的判断

1. 未覆盖参数时 `count` 多宽？
2. `#(.Width(16))` 实例化后，`count` 多宽？
3. 参数能否在芯片运行中动态改成 16？

<details>
<summary>参考分析</summary>

默认 `Width=4`，所以 `count` 是 4 位。实例化时覆盖为 16 后，对应实例生成 16 位计数器。参数在 elaboration/生成硬件时确定，不是运行时信号；运行中改变宽度是不可能的。

</details>

---

## 08｜`parameter` 与 `localparam`

### 代码片段

```systemverilog
parameter int Width = 8;
localparam int CountWidth = $clog2(Depth + 1);
```

### 你的判断

1. 两者谁可以由模块实例化者覆盖？
2. `CountWidth` 为什么不是直接写死为 3？

<details>
<summary>参考分析</summary>

`parameter` 可在实例化时覆盖；`localparam` 只能在模块内部使用，不能被外部改写。`CountWidth` 由 `Depth` 推导，深度变了计数器宽度也会自动更新；这就是参数化模块避免“改了一个数字、忘了改另外三个数字”的方法。

</details>

---

## 09｜packed 与 unpacked 数组

### 代码片段

```systemverilog
logic [7:0] byte_data;
logic [7:0] mem [0:3];
```

### 你的判断

1. `byte_data` 有多少位？
2. `mem` 总共能存多少个字节？
3. `mem[2]` 的位宽是什么？

<details>
<summary>参考分析</summary>

`byte_data` 是一个 packed 的 8 位向量。`mem` 是 4 个元素组成的 unpacked 数组，每个元素都是 8 位，因此总容量为 4 字节。`mem[2]` 选出第 3 个元素，仍然是 8 位。FIFO 常用这种写法描述存储器。

</details>

---

## 10｜存储器写入

### 代码片段

```systemverilog
if (wr_en && !full) begin
    mem[wr_ptr] <= wr_data;
    wr_ptr      <= wr_ptr + 1'b1;
end
```

### 你的判断

1. 哪一个地址被写入？
2. 写入和指针加一的先后关系是什么？
3. `full=1` 时会发生什么？

<details>
<summary>参考分析</summary>

写入使用**旧** `wr_ptr` 指向的位置；由于都是非阻塞赋值，写入和指针更新都在该时钟沿后提交，二者不会互相抢先。`full=1` 时整个条件为假，既不覆盖存储器，也不推进写指针，这就是写侧的流控。

</details>

---

## 11｜FIFO 的组合读

### 代码片段

```systemverilog
assign rd_data = empty ? 8'h00 : mem[rd_ptr];
```

### 你的判断

1. `empty=1` 时为什么不直接读 `mem[rd_ptr]`？
2. `rd_data` 在读指针变化后何时变化？
3. 这是同步读 RAM 还是组合读 RAM 模型？

<details>
<summary>参考分析</summary>

空 FIFO 没有有效队头数据，因此给一个确定值 0，避免波形中误用旧存储内容。`rd_ptr` 或相应存储单元变化后，`rd_data` 经组合逻辑更新。这是组合读模型；某些 FPGA 的块 RAM 是同步读，接口与时序会不同，后续需要专门适配。

</details>

---

## 12｜空与满由计数器推导

### 代码片段

```systemverilog
logic [2:0] count;
assign empty = (count == 0);
assign full  = (count == 4);
```

### 你的判断

1. 深度为 4 的 FIFO，`count` 为什么要 3 位？
2. `count=3` 时空和满分别是什么？
3. 如果把 `count` 只写成 2 位，什么值无法表示？

<details>
<summary>参考分析</summary>

深度 4 需要表示 0、1、2、3、4 五个数量，因此至少 3 位。`count=3` 时既不空也不满。2 位只能表示 0 到 3，无法表示“已装满的 4 个元素”。

</details>

---

## 13｜读写条件拼接

### 代码片段

```systemverilog
case ({(wr_en && !full), (rd_en && !empty)})
    2'b10: count <= count + 1'b1;
    2'b01: count <= count - 1'b1;
    default: ;
endcase
```

### 你的判断

1. 拼接表达式的高位和低位分别是什么？
2. `2'b11` 为什么落在 `default`？
3. 同时读写时 count 应该变化吗？

<details>
<summary>参考分析</summary>

高位是“成功写入”，低位是“成功读取”。10 表示仅写入，元素数加一；01 表示仅读取，元素数减一。11 表示同时成功读写，一个进、一个出，元素数量不变，所以可以由 `default` 保持。这里 `default: ;` 表示不执行额外语句。

</details>

---

## 14｜同周期读写的额外问题

### 代码片段

```systemverilog
2'b11: begin
    mem[wr_ptr] <= wr_data;
    wr_ptr      <= wr_ptr + 1'b1;
    rd_ptr      <= rd_ptr + 1'b1;
end
```

### 你的判断

1. 为什么 count 保持不变？
2. 当 `wr_ptr == rd_ptr` 时，读出的到底是旧数据还是新数据？
3. 为什么这是 FIFO 设计里需要明确规格的一点？

<details>
<summary>参考分析</summary>

一写一读，元素净数量为 0，所以 count 保持。若读写同一地址，结果取决于 RAM 的 read-first/write-first/no-change 行为以及你采用的读接口；不同 FPGA/ASIC 宏可能不同。因此真实 FIFO 规格必须明确同周期读写、空/满边界时的行为，不能只靠“看起来合理”。

</details>

---

## 15｜读写指针为什么会回绕

### 代码片段

```systemverilog
logic [1:0] wr_ptr;

wr_ptr <= wr_ptr + 1'b1;
```

### 你的判断

1. `wr_ptr=2'b11` 后加一会变成什么？
2. 这为什么正好适合深度为 4 的存储器？
3. 仅靠两个 2 位指针能否区分“空”和“满”？

<details>
<summary>参考分析</summary>

2 位 11 加一后自然截断为 00，即回绕。深度为 4 的存储器地址恰好需要 2 位，因此这很方便。但仅比较读写指针会遇到“相等既可能是空，也可能绕了一圈后满”的歧义；本题通过独立 `count` 解决，另一种常见方案是扩展指针额外比较回绕位。

</details>

---

## 16｜从 FIFO 回到 AI 芯片

### 代码片段

```text
DDR / 输入流 → FIFO / SRAM → MAC 阵列 → 累加器 → 输出 FIFO
```

### 你的判断

1. FIFO 在 MAC 阵列前面解决什么问题？
2. `full` 应该反馈给上游，还是下游？
3. `empty` 应该阻止谁继续取数据？

<details>
<summary>参考分析</summary>

FIFO 解耦生产者和消费者的速率：上游突发提供数据、MAC 阵列暂时来不及处理时，FIFO 缓冲数据。`full` 应反馈给上游，要求它停止写入；`empty` 告诉下游当前没有可读数据，应阻止 MAC 或读端继续取数。后续你会把它扩展为片上 Buffer/Scratchpad。

</details>

---

## 完成本批后的自测

不看答案，解释以下三件事：

1. 为什么组合逻辑必须覆盖所有输出赋值路径？
2. 深度 4 的 FIFO 为什么需要 2 位指针和 3 位 count？
3. 为什么 AI 加速器不能让 MAC 阵列直接随意读取外部数据，而需要 Buffer/FIFO？

能说清后，下一批将进入：`valid/ready` 握手、流水寄存器、状态机和简化寄存器接口。
