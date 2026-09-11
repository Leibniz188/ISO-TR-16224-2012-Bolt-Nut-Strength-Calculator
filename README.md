# ISO/TR 16224:2012 Bolt–Nut Strength Calculator

一个基于 **ISO/TR 16224:2012 — Technical aspects of nut design** 与 Alexander 理论编写的 Python 螺栓/螺母静拉失效计算工具。

项目当前版本可计算：

- 螺栓断裂载荷 `FBb`
- 外螺纹剥牙载荷 `FSb`
- 内螺纹剥牙载荷 `FSn`
- 三种失效模式中的控制载荷
- 安全系数处理后的允许载荷
- 参数化螺纹/螺母 CAD 风格 PDF 工程图
- 中文 Markdown 完整计算书

> 本项目用于工程计算、公式核验与设计辅助，不是 ISO 官方软件，也不能替代正式的试验验证、设计审查或适用标准要求。

---

## 1. 项目特点

### 1.1 三种静拉失效模式

程序按照当前人工校核后的计算流程，分别计算：

\[
F_{Bb}
\]

螺栓本体断裂；

\[
F_{Sb}
\]

外螺纹（螺栓螺纹）剥牙；

\[
F_{Sn}
\]

内螺纹（螺母/内螺纹件）剥牙。

最终自动比较：

\[
F_{\mathrm{control}}
=
\min(F_{Bb},F_{Sb},F_{Sn})
\]

并给出安全系数处理后的允许载荷：

\[
F_{\mathrm{allow}}
=
\frac{F_{\mathrm{control}}}{n}
\]

其中 `n` 为用户输入的安全系数。

---

### 1.2 参数化螺纹几何

用户输入：

```python
d = 60.0
P = 3.0
thread_angle = 60.0
```

程序根据当前牙型参数实时计算：

- `H`
- `d1`
- `d2`
- `d3`
- `D1`
- `D2`

基本三角形高度：

\[
H=
\frac{P}
{2\tan(\alpha/2)}
\]

其中：

- \(P\)：螺距
- \(\alpha\)：用户输入的螺纹夹角

牙型截短系数也可以自定义：

```python
k_flat = 1.0 / 8.0
k_external_root = 1.0 / 6.0
k_internal_crest = 1.0 / 4.0
```

分别表示：

- `k_flat`：平顶截短高度 / `H`
- `k_external_root`：外螺纹牙底截短高度 / `H`
- `k_internal_crest`：内螺纹牙顶截短高度 / `H`

默认值恢复常用 ISO 公制 60° 基本牙型关系。

> 当 `thread_angle != 60°` 或修改截短系数时，程序是在 ISO/TR 16224 计算框架基础上进行参数化几何推广，不应把这些非标准牙型结果理解为 ISO 公制基本牙型的标准尺寸。

---

### 1.3 螺母高度支持 `0.8D` 或自定义

默认采用比例模式：

```python
nut_height_mode = "ratio"
nut_height_ratio = 0.80
```

即：

\[
m=0.8D
\]

也可以改成自定义高度：

```python
nut_height_mode = "custom"
nut_height_custom = 30.0
```

> `0.8D` 是本程序提供的常用设计初值选项，并不是在这里声明为 ISO/TR 16224 的强制规定。

---

### 1.4 修正系数自动做适用范围检查

程序实现：

- `C1`：螺母胀大修正
- `C2`：外螺纹弯曲修正
- `C3`：内螺纹弯曲修正

程序不会对标准未给出的区间强行外推。

如果某个修正项超出标准给出的适用范围：

1. 该修正项不再使用；
2. 对应系数按 `1.0` 处理；
3. 控制台与计算书明确指出超限项目；
4. 提示计算适用性/准确性降低；
5. 建议设计人员根据工程风险考虑增大安全系数，并通过试验或其他设计方法复核。

例如：

```text
C2 修正超出适用范围
本计算不考虑 C2 修正（C2 = 1.0）
建议提高安全系数并进一步复核
```

---

## 2. 当前主要计算关系

### 2.1 螺栓断裂

当前项目经人工校核采用：

\[
A_s=
\frac{\pi}{4}
\left(
\frac{d_1+d_2}{2}
\right)^2
\]

然后：

\[
F_{Bb}=R_mA_s
\]

程序同时保留名义应力面积作为对照：

\[
A_{s,\mathrm{nom}}
=
\frac{\pi}{4}
\left(
\frac{d_2+d_3}{2}
\right)^2
\]

---

### 2.2 有效啮合高度

单端倒角：

\[
m_{\mathrm{eff}}
=
m-0.6h_c
\]

双端倒角：

\[
m_{\mathrm{eff}}
=
m-1.2h_c
\]

---

### 2.3 外螺纹剪切面积

在 60° 牙型下对应 ISO/TR 16224 Eq. (7)；程序对任意用户输入角度使用：

\[
\tan(\alpha/2)
\]

代替 60° 情况下的：

\[
\frac{1}{\sqrt 3}
=
\tan30^\circ
\]

程序计算：

\[
D_m=1.026D_1
\]

以及外螺纹剪切面积 \(A_{Sb}\)。

---

### 2.4 内螺纹剪切面积

程序同样按照 Eq. (7) 的几何结构计算：

\[
A_{Sn}
\]

并结合：

\[
R_s=
\frac{R_{mn}A_{Sn}}
{R_mA_{Sb}}
\]

计算修正系数 `C2`、`C3`。

---

### 2.5 剥牙载荷

外螺纹：

\[
F_{Sb}
=
0.6R_mA_{Sb}C_1C_2
\]

内螺纹：

\[
F_{Sn}
=
0.6R_{mn}A_{Sn}C_1C_3
\]

---

## 3. PDF 工程图

程序自动生成 CAD 风格 A3 横向 PDF。

PDF **只包含工程图，不包含计算正文**。

当前绘图包括：

- 螺母轴向剖视图
- 螺母高度 `m`
- 有效啮合高度 `meff`
- `D / D1 / D2`
- 内螺纹放大牙型
- 外螺纹放大牙型
- `d / d2 / d3`
- `P`
- `P/2`
- `H`
- 中径及辅助平行线
- 节距辅助线
- 用户输入的螺纹夹角
- 外螺纹牙底圆角

外螺纹牙底圆角按几何相切条件绘制，使：

- 圆弧与左右理论牙面相切；
- 圆弧最低点落在计算得到的 `d3` 径向位置。

绘图会随 `thread_angle` 实时变化。

---

## 4. 中文 Markdown 计算书

每次运行还会生成：

```text
<CASE_NAME>_calculation.md
```

其中包含中文完整计算过程，包括：

- 输入参数
- 螺纹基本几何
- `H`
- `d1 / d2 / d3`
- `D1 / D2`
- 螺母高度
- `meff`
- `As`
- `FBb`
- `Dm`
- `ASb`
- `ASn`
- `Rs`
- `C1 / C2 / C3`
- 修正项适用范围检查
- `FSb`
- `FSn`
- 控制失效模式
- 安全系数后的允许载荷
- 超范围警告

计算书不是只输出最终答案，而是保留：

**原始公式 → 参数代入 → 中间值 → 最终结果**

方便人工复核。

---

## 5. 环境要求

推荐：

```text
Python >= 3.7
```

第三方依赖：

```text
reportlab
```

安装：

```bash
pip install reportlab
```

其余依赖均为 Python 标准库：

- `pathlib`
- `math`
- `hashlib`

---

## 6. Python 3.7 / ReportLab 兼容

部分较新的 ReportLab 版本会调用：

```python
hashlib.md5(..., usedforsecurity=False)
```

而部分 Python 3.7 环境中的 `hashlib.md5()` 不接受该关键字，会出现：

```text
TypeError: openssl_md5() takes no keyword arguments
```

本项目已在导入 ReportLab 前加入兼容处理，因此可直接在 Python 3.7 环境下运行。

已针对以下形式的环境进行兼容处理：

```text
Python 3.7.9
```

---

## 7. 快速开始

克隆仓库：

```bash
git clone <YOUR_REPOSITORY_URL>
cd <YOUR_REPOSITORY_NAME>
```

安装依赖：

```bash
pip install reportlab
```

运行：

```bash
python ISO_TR_16224_calculator_v13_correct_fillet_cn_md.py
```

Windows PowerShell 示例：

```powershell
D:/Programs/Python/Python379/python.exe ISO_TR_16224_calculator_v13_correct_fillet_cn_md.py
```

---

## 8. 用户输入

程序开头集中放置了用户输入参数。

### 输出目录

```python
OUTPUT_BASE_DIR = Path(r"./ISO_TR_16224_outputs")
CASE_NAME = "M60x3_demo"
```

程序自动创建：

```text
ISO_TR_16224_outputs/
└── M60x3_demo/
    ├── M60x3_demo_drawing.pdf
    └── M60x3_demo_calculation.md
```

---

### 材料参数

```python
bolt_property_class = "10.9"
nut_property_class = "10"

Rm = 1000.0
Rmn = 800.0
```

注意：

`bolt_property_class` 和 `nut_property_class` 当前仅作为标签。

程序**没有材料数据库**，不会根据等级自动推导 `Rm` 或 `Rmn`。

---

### 安全系数

```python
safety_factor = 1.50
```

---

### 螺纹尺寸

```python
d = 60.0
P = 3.0
thread_angle = 60.0
```

---

### 牙型系数

```python
k_flat = 1.0 / 8.0
k_external_root = 1.0 / 6.0
k_internal_crest = 1.0 / 4.0
```

---

### 螺母对边

```python
s = 90.0
```

---

### 螺母高度

比例模式：

```python
nut_height_mode = "ratio"
nut_height_ratio = 0.80
```

自定义模式：

```python
nut_height_mode = "custom"
nut_height_custom = 30.0
```

---

### 倒角

```python
hc = 0.0
chamfer_ends = 1
```

其中：

```text
1 = 单端倒角
2 = 双端倒角
```

PDF 绘图使用的沉孔/倒角夹角：

```python
drawing_countersink_included_angle = 90.0
```

该参数仅用于绘图，不进入强度计算。

---

## 9. 示例

默认示例：

```text
M60 × 3
thread angle = 60°
Rm = 1000 MPa
Rmn = 800 MPa
m = 0.8D
```

运行后会生成：

```text
M60x3_demo_drawing.pdf
M60x3_demo_calculation.md
```

终端同时输出：

```text
FBb
FSb
FSn
Allowable load
Warnings
```

---

## 10. 项目结构建议

推荐 GitHub 仓库结构：

```text
.
├── ISO_TR_16224_calculator_v13_correct_fillet_cn_md.py
├── README.md
├── requirements.txt
├── examples/
│   ├── M60x3_demo_drawing.pdf
│   └── M60x3_demo_calculation.md
└── LICENSE
```

`requirements.txt` 可以写：

```text
reportlab
```

---

## 11. 标准与适用范围说明

本项目主要参考：

- ISO/TR 16224:2012 — *Technical aspects of nut design*
- ISO 68-1 — ISO metric screw thread basic profile
- ISO 724 — ISO metric thread basic dimensions
- ISO 898-1 — Mechanical properties of bolts, screws and studs
- ISO 898-2 — Mechanical properties of nuts

请注意：

1. ISO/TR 16224:2012 是 Technical Report；
2. 实际螺纹公差、材料状态、制造误差、啮合状态、自由螺纹长度等因素都会影响实际承载能力；
3. 程序目前不包含 ISO 公差等级数据库；
4. 程序目前不包含材料数据库；
5. 非 60° 牙型属于本项目的参数化几何推广；
6. 超出修正系数适用范围时，本项目选择“不应用该修正项”，并主动给出警告；
7. 正式设计应结合适用的最新版标准、企业规范、试验和工程审查。

---

## 12. 免责声明

本项目主要用于：

- 工程计算辅助
- 公式核验
- 参数敏感性研究
- 教学与研究
- 初步尺寸设计

作者不保证该程序适用于所有螺纹连接形式、材料体系、制造公差和载荷工况。

对于安全关键、承压、起重、航空航天、核电、车辆或其他高风险应用，应由具备相应资质的工程人员依据适用标准进行独立复核和验证。

---

## 13. 后续计划

可继续扩展：

- [ ] ISO 公制螺纹尺寸数据库
- [ ] 螺纹公差等级
- [ ] 材料/性能等级数据库
- [ ] 批量计算
- [ ] CSV / Excel 输入
- [ ] GUI
- [ ] 更多 CAD 标注风格
- [ ] DXF 输出
- [ ] 与 FEM / 试验结果对比
- [ ] 单元测试

---

## 14. Contributing

欢迎提交：

- Issue
- Pull Request
- 公式核验
- 标准适用范围修正
- 绘图改进
- Python 兼容性修复

如果发现计算结果与标准原文不一致，请在 Issue 中提供：

1. 标准版本；
2. 页码/公式编号；
3. 输入参数；
4. 期望结果；
5. 当前程序结果。

这样更方便复核。

---

## 15. License

仓库发布前请根据你的需求选择许可证，例如：

- MIT
- BSD-3-Clause
- Apache-2.0
- GPL-3.0

如果暂时不希望他人自由复制、修改和再发布代码，请不要直接添加开源许可证。
