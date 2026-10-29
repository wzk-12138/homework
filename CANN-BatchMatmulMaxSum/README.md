# BatchMatmulMaxSum · Ascend C

CANNJudge 昇腾算子挑战赛「上合赛区」**BatchMatmulMaxSum** 赛题的 Ascend C 算子实现，已在 Ascend 910B（CANN 9.0.0）上编译运行并通过对拍验证（最大误差 ~2.4e-7）。

## 算子语义

对批量矩阵乘结果做两级归约：

```
Y[b] = Σ_{i=0..M-1}  Max_{j=0..N-1}  ( X1[b] · X2[b] )[i, j]
```

- 输入：`X1` 形状 `(B, M, K)`，`X2` 形状 `(B, K, N)`，支持 `transposeX1 / transposeX2`（仅改变物理存储布局，不改变逻辑 shape）。
- 中间：BatchMatMul → `(B, M, N)`；沿 N 维取最大值（初值 `-FLT_MAX`，全负行返回最大负数）；沿 M 维求和。
- 输出：`Y` 形状 `(B,)`，fp32。

## 本仓库内容

仅含算子源码 `kernel.asc`（提交到 CANNJudge 答题框的内容）。

## 提交到 CANNJudge

将 `kernel.asc` 全文粘贴到赛题答题框即可。提交形态 `npu_kernel_dev`，对应 CANN 版本 9.0.0。

## NPU 上板测试（完整工程）

host 驱动、CMake、数据生成与对拍脚本在 GitCode 仓库：

https://gitcode.com/Lumos_1/batchmatmulmaxsum-cannlab

在 CANNLab（Ascend 910B, CANN 9.0.0）中：

```bash
git clone https://gitcode.com/Lumos_1/batchmatmulmaxsum-cannlab.git
cd batchmatmulmaxsum-cannlab
bash run.sh   # 自动：设环境 → 编译 → 生成用例 → 跑 kernel → 与 golden 对拍
```

结尾出现 `=== PASSED ===` 即验证通过。

## 本地数值验证（无需 NPU）

numpy 参考实现 `bmm_maxsum_ref.py` 运行 `python3 bmm_maxsum_ref.py`，覆盖四种 transpose 组合、非对齐尾块、fp16/bf16、全负相似度行，相对误差 < 1e-3。
