<!-- SCOPE project README -->
<div align="center">

# SCOPE

### 单细胞与空间组学研究设计

**S**patial and single-**C**ell **O**mics **P**lanning & **E**xperimental design

以科学问题为起点，把研究想法组织成可执行、可追溯的分析方案与论文图表故事线。

![Public repository](https://img.shields.io/badge/visibility-public-2ea44f)
![Focus](https://img.shields.io/badge/focus-single--cell%20%26%20spatial%20omics-3973ac)
![Language](https://img.shields.io/badge/language-Chinese-ef8354)

[快速开始](#快速开始) · [能做什么](#能做什么) · [工作流程](#工作流程) · [仓库内容](#仓库内容)

</div>

---

## SCOPE 是什么？

SCOPE 是一个面向**单细胞转录组与空间转录组研究设计**的 Codex skill。它帮助研究者把疾病表型、候选基因、治疗响应或细胞状态等问题，拆解为合理的样本设计、分析模块、证据链和论文组图。

核心思路是：

> 先明确每项分析要回答的问题，再决定样本怎么分、用什么方法、保存什么结果，以及结果如何展示。

## 能做什么？

| 研究环节 | SCOPE 帮你规划 |
| --- | --- |
| 科学问题 | 主问题、递进子问题、可检验假设与证据层级 |
| 样本设计 | 生物学重复、分组与配对、发现/验证队列、空间取样区域 |
| 分析方案 | 分析目的、输入数据、推荐 R/Python 包、核心计算与替代方法 |
| 结果交付 | 预期结果文件、关键字段、可复现的中间产物 |
| 可视化 | 每个分析适合的图形，以及主图和补充图的取舍 |
| 证据审计 | 伪重复、批次混杂、统计单位、因果表述和验证缺口 |
| 论文组图 | 按科学论证顺序设计 Figure 1–N 与各面板内容 |

对于具体分析模块，SCOPE 会尽量给出一张可执行的“分析卡片”：

**科学问题 → 样本/比较 → 工具与计算 → 结果文件 → 可视化 → 统计单位与局限**

## 工作流程

~~~mermaid
flowchart LR
    A["研究问题与假设"] --> B["样本分组与取样"]
    B --> C["单细胞分析"]
    B --> D["空间分析"]
    C --> E["跨组学证据整合"]
    D --> E
    E --> F["验证策略与证据审计"]
    F --> G["Figure 1–N 故事线"]
~~~

分析模块围绕子问题组织，而不是按软件运行顺序堆叠。单细胞用于识别细胞类型、状态和候选机制；空间数据用于检查组织位置、邻域和微环境关系；两者联合后再判断证据能支持到什么程度。

## 快速开始

克隆到 Codex skills 目录，并将目录命名为 <code>scope</code>：

**Windows PowerShell**

~~~powershell
git clone https://github.com/jiumeng-bit/scope-skill.git "$env:USERPROFILE\.codex\skills\scope"
~~~

**macOS / Linux**

~~~bash
git clone https://github.com/jiumeng-bit/scope-skill.git "$HOME/.codex/skills/scope"
~~~

重新加载 Codex 后，在对话中调用 <code>$scope</code>。

### 示例：从候选基因开始

~~~text
使用 $scope：我想研究基因 X 在疾病 Y 中的作用。
请重点设计单细胞与空间转录组的样本分组、
分析流程、验证路径和论文组图。
~~~

### 示例：展开一个分析模块

~~~text
使用 $scope：请展开“细胞通讯”分析，列出推荐的 R/Python 包、
核心计算、结果文件、可视化和统计注意事项。
~~~

也可以让 SCOPE 审核已有方案：

~~~text
使用 $scope：请检查这份单细胞与空间转录组分析方案，
重点审查生物学重复、批次混杂和结论是否超出数据支持范围。
~~~

## 设计原则

- **以问题为导向：** 每个分析模块都要说明它回答什么问题。
- **以样本为基础：** 患者、动物或独立组织通常才是生物学重复；细胞、spot、bin 和切片不能自动视作独立样本。
- **分层建立证据：** 发现、空间定位、临床关联与因果验证各自承担不同的证据角色。
- **方法与数据匹配：** 工具选择要考虑平台分辨率、样本量、配对结构和研究目标。
- **图表服务论证：** 主图按结论推进，补充图承担质控、稳健性和扩展分析。
- **明确结论边界：** 共表达、空间邻近或计算通讯本身不等于因果机制。

## 仓库内容

~~~text
scope-skill/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── assets/
│   └── study-design-template.md
└── references/
    ├── analysis-menu.md
    ├── evidence-validation.md
    ├── figure-storyboards.md
    ├── method-catalog.md
    ├── note-derived-patterns.md
    ├── problem-framework.md
    └── sample-design.md
~~~

- [SKILL.md](SKILL.md)：核心指令、工作流程与输出规范
- [method-catalog.md](references/method-catalog.md)：分析方法、工具与执行细节
- [sample-design.md](references/sample-design.md)：样本分组、配对和取样原则
- [figure-storyboards.md](references/figure-storyboards.md)：论文组图叙事结构
- [evidence-validation.md](references/evidence-validation.md)：证据验证与结论边界
- [study-design-template.md](assets/study-design-template.md)：可填写的研究设计模板

## 适用范围

SCOPE 用于**研究设计、分析规划和证据审阅**。方法建议应结合具体数据、统计设计和最新文献核实；它不能替代正式的统计功效评估、原始论文核验、临床判断或实验验证。

---

<div align="center">
如果 SCOPE 帮助你把研究问题变成更清晰的分析计划，欢迎使用、反馈并继续改进。
</div>
