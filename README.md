## 常见问题：RDkit error encountered 报错处理

### 问题现象
生成分子过程中出现如下警告/报错：[2026-02-04 14:31:20,663::sample::INFO] Success: O=C1CCc2c(cccc2C2CNNC2Nc2ccccn2)N1[2026-02-04 14:31:20,669::sample::WARNING] RDkit error encountered.

### 报错原因分析
理论上以下三种场景均可能触发该RDKit异常，但经实际测试验证，**仅计算Vina得分时会触发**，其他场景无影响：
1. **Vina对接工具限制**：Vina对分子3D结构要求极高，哪怕缺失单个氢原子，都会导致RDKit解析异常；
2. **轨迹保存问题**：生成的轨迹分子属于"中间态"，结构不完整，RDKit写入SDF文件时必然报错；
3. **属性计算异常**：部分分子计算SA/QED得分时，RDKit会因"原子类型识别偏差"抛出警告级异常。

### 根本原因
Vina分数计算异常的核心原因是：**meeko 与 openbabel 版本不兼容**。

### 解决方案
为避免升降版本引发其他依赖问题，直接修改 `docking_vina.py` 文件，关键修改点如下：
- 移除 `from openbabel import pybel` 导入语句；
- 将 `PrepLig` 类的实现改为纯RDKit处理分子；
- 读取SDF文件时，使用 `Chem.SDMolSupplier(input_mol, removeHs=False)` 保留氢原子（避免结构缺失）。

### 使用方法
将本项目提供的 `docking_vina.py` 文件替换掉原文件即可解决该问题。
