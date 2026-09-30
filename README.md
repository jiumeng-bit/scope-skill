# SCOPE

**SCOPE** = **S**patial and single-**C**ell **O**mics **P**lanning & **E**xperimental design.

面向单细胞与空间转录组研究的、以科学问题为导向的生信研究设计 skill。它将疾病、候选基因、治疗响应或细胞状态等研究问题，转换为可执行的样本分组、分析流程、结果文件、可视化方案、验证路径和论文组图故事线。

## 主要能力

- 把研究目标拆成主问题、递进子问题与可检验假设。
- 设计生物学重复、配对关系、发现/验证队列及空间取样策略。
- 为具体分析模块给出 R/Python 工具、核心计算、结果文件和可视化类型。
- 设计单细胞—空间转录组联合分析与证据链。
- 按科学论证顺序规划 Figure 1–N，而不是按软件运行顺序堆图。
- 审核伪重复、批次混杂、过度因果表述和验证不足等常见风险。

## 安装

将本仓库克隆或复制到 Codex skills 目录，并确保目录名为 `scope`：

```powershell
git clone https://github.com/jiumeng-bit/scope-skill.git "$env:USERPROFILE\.codex\skills\scope"
```

重新启动或刷新 Codex 后，使用 `$scope` 调用。

## 示例

```text
使用 $scope：我想研究基因 X 在疾病 Y 中的作用，请设计单细胞与空间转录组的样本分组、分析模块和论文组图。
```

```text
使用 $scope：请展开“细胞通讯”模块，说明使用的 R/Python 包、核心计算、结果文件和可视化方案。
```

## 仓库结构

- `SKILL.md`：核心工作流与输出规范
- `references/`：问题框架、样本设计、分析菜单、方法目录、验证和组图模板
- `assets/`：可直接填写的研究设计模板
- `agents/openai.yaml`：Codex 展示与调用配置

## 适用范围

SCOPE 主要用于研究设计、分析规划与证据审计，不替代原始论文核验、统计功效分析、临床专业判断或实验验证。
