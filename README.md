# CO-LAB-RISCV Template

## 目录说明

- `core` 存放你的核的代码，你可以不使用框架，仅需要确保最外层模块名为`top_axi_wrapper`，且信号一致即可。
- `test`存放以下测试：

    - `mul`：乘法器单元测试，需要乘法器按照以下interface编写，并保存到`core/mul.sv`：

        ```sv
        typedef enum {
            MUL, MULH, MULHU, MULHSU
        } mul_op;

        module mul(
            input               clock,
            input               reset,
            input        [63:0] in1,
            input        [63:0] in2,
            input               mul_word,   // 0: 64-bit, 1: 32-bit
            input               en,         // enable
            input        mul_op op,
            output       [63:0] out,
            output              out_valid,
            input               out_ready
        );
        ```

        握手遵循以下要求：
        1. 当乘法器空闲时，en为0，out_valid为0，out可为任意值。
        2. 当某一时刻有乘法请求时，en为1，直到out_valid为1时，乘法运算结束。
        3. out_ready指示乘法器是否需要保持当前输出状态，用于流水线暂停时防止乘法器结果流失，如果为0，则下一拍上升沿时乘法器保持out输出，out_valid为1。
        4. out_ready与out_valid之间不可存在组合逻辑。

    - `div`：除法器测试，需要乘法器按照以下interface编写，并保存到`core/mul.sv`：

        ```sv
        typedef enum {
            DIV, DIVU, REM, REMU
        } div_op;

        module div(
            input               clock,
            input               reset,
            input        [63:0] in1,
            input        [63:0] in2,
            input               div_word, // 0: 64-bit, 1: 32-bit
            input               en,
            input        div_op op,
            output       [63:0] out,
            output              out_valid,
            input               out_ready
        );
        ```

        信号要求与乘法类似。

    - test_workbench 运行整个核的测试框架
        - `soft` 在你的CPU核上运行的软件，默认为一个串口输出的演示程序，具体可以查看Makefile。
        - `sim`：整合[soc-simulator](https://github.com/cyyself/soc-simulator)与[cemu](https://github.com/cyyself/cemu)的文件，提供Verilator运行环境。
        
            - 编译CPU核到Verilator

                直接`make`，编译后结果为`./obj_dir/Vtop_axi_wrapper`，会在与CEMU差分测试的情况下运行soft。
            - 波形图调试

                执行`./obj_dir/Vtop_axi_wrapper -trace [TRACE_TIME]`，其中`[TRACE_TIME]`替换为波形图输出时间，可以为`1000000000`。
            - RISC-V Tests

                - 首先确保已经编译riscv工具链，且已加入环境变量PATH，前缀为`riscv64-unknown-linux-gnu-`
                - 确保CPU核已经编译完成
                - 使用`make riscv-test-build`进行编译，如果报错需要排查，建议先`rm -rf riscv-test`以及`rm -rf riscv-test`，打开Makefile一条条手动运行。
                - 运行`./run_riscv_test.py`执行测试
                - 如果测试不通过，观察错误的测试点，例如测试点为`rv64mi-p-ma_fetch`，可以使用以下命令单独运行并输出波形图：`./obj_dir/Vtop_axi_wrapper ./tests-bin/rv64mi-p-ma_fetch.bin -rvtest -trace 10000`