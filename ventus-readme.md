## CTS 批量测试脚本使用方法

首先，正常编译 OpenCL-CTS

```bash
mkdir -p build
cmake -S . -B ./build \
        -DCL_INCLUDE_DIR=${VENTUS_INSTALL_PREFIX}/include \
        -DCL_LIB_DIR=${VENTUS_INSTALL_PREFIX}/lib \
        -DOPENCL_LIBRARIES=OpenCL
cmake --build ./build --config Release -j $(nproc)
```

然后，就可以运行 run_test_parallel.py 进行批量测试了。运行前需确保环境变量 `VENTUS_INSTALL_PREFIX` 被正确设置：

```bash
export VENTUS_INSTALL_PREFIX=/path/to/llvm-project/install
```

脚本还可以设置其他各种参数，可以通过运行 `python3 run_test_parallel.py --help` 来查看帮助信息。如无其他需要，运行默认参数即可：

```bash
python3 run_test_parallel.py
```

运行结果会保存在 `logs/` 目录下：

- 所有测例的通过情况保存在 `all_run_tests.log`；
- 默认配置下，通过的子测例会自动删除 log，未通过则保留 log。

此外，为了减小测试过程中生成的 log 文件大小，我们最好关闭 spike 生成的 log（每条指令的执行结果那个）。方法是：将 `ventus-gpgpu-isa-simulator/spike_main/spike_device.cc` 中的​

```cpp
  parser.option('l', 0, 0, [&](const char* s){log = true;});
    parser.option(0, "log-commits", 0,
                [&](const char* s){log_commits = true;});
```

的 2 个 true 修改为 false。

目前，select 和 conversions 测试套运行仍然较慢，有待优化（减少迭代次数）。如果想先跳过这两个测试套，可以在 `test_list_new.json` 中先将这两个测试套全部删掉。

默认配置下，参数 `--filter-state` 会指示程序跳过 `test_list_new.json` 中所有状态为 `fail` 的子测例。这样，默认配置下，脚本执行的所有子测例理论上都应该在 `all_run_tests.log` 中显示为 `[PASS]`。
