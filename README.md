问题描述：生成分子时出现诸如此类的报错
[2026-02-04 14:31:20,663::sample::INFO] Success: O=C1CCc2c(cccc2C2CNNC2Nc2ccccn2)N1
[2026-02-04 14:31:20,669::sample::WARNING] RDkit error encountered.
这是由于以下三种情况只要有一个发生都会出发RDkit error encountered.
Vina：对接工具对分子 3D 结构的要求极高，哪怕分子少一个氢原子，都会触发 RDKit 解析异常；
轨迹保存：生成的轨迹分子是 “中间态”，结构不完整，RDKit 写入 SDF 时必然报错；
属性计算：部分分子的 SA/QED 得分计算时，RDKit 会因 “原子类型识别偏差” 抛出警告级异常；
经过测试，只有在计算Vina得分时才会触发该异常，其他情况下均无影响。
Vina分数异常根本原因是 meeko 和 openbabel 版本不兼容。为了避免继续升降版本导致的其它问题，这里我选择直接修改docking_vina.py
- 移除了 from openbabel import pybel 导入
- 将 PrepLig 类改为使用 RDKit 处理分子
- 读取 SDF 时使用 Chem.SDMolSupplier(input_mol, removeHs=False) 保留氢原子
修改后的docking_vina.py已经提供，替换掉原本的即可
