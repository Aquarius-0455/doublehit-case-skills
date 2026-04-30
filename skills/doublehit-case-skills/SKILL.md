---
name: doublehit-case-skills
description: >
  Generate high-coverage test cases from PRD, Story, feature lists, business rules, or requests such as 编写测试用例、设计测试场景、查缺补漏、补充异常逻辑、分析需求. Extract atomic requirements, choose suitable test design methods, output standardized JSON, and export to Excel with the bundled script.
---

# doublehit-case-skills 测试用例生成工具

当用户输入唤醒词 `双击666`，或请求根据需求文档、Story、功能列表、业务规则生成测试用例时，使用本 Skill。

核心原则：先完整覆盖需求中的明文功能点、判定分支、交互动作和审计要求，再补充必要的异常、边界与工程安全场景；不要无依据扩写，也不要遗漏显性需求。

## 1. 输入识别

收到输入后先判断类型，再决定是否生成。

| 输入特征 | 判定 | 处理方式 |
|---|---|---|
| 功能列表、字段定义、状态码表、业务规则 | 需求文档 | 直接分析并生成 |
| Story 编号或结构化 Story 描述 | Story | 直接分析并生成 |
| 口语化描述，如“帮我写登录用例” | 随意输入 | 先整理为结构化需求，必要时请用户确认 |
| 仅一句话，缺少字段、规则、边界或场景 | 模糊输入 | 先追问场景、字段、规则、边界和期望结果 |

删除线规则：需求文本中 `~~内容~~` 表示已取消功能或字段。必须列出删除线内容，且不得为这些内容生成用例。

## 2. 需求分析流程

生成用例前必须先输出分析预览，包含以下内容：

1. 删除线扫描结果：列出已取消内容；没有则写“无”。
2. 原子节点提取：将需求拆成可验证的原子约束。
   - 名词：业务参数、字段、状态、角色、系统对象。
   - 动词：新增、编辑、删除、提交、审批、跳过、回写、通知等动作。
   - 判定：if/else、AND/OR、阈值、范围、状态流转、权限限制。
   - 时序：生成前后、提交前后、异步回调、审计落库等顺序要求。
3. 需求溯源矩阵：每条需求原文或原子约束都要映射到覆盖点、设计方法和层级。

溯源矩阵格式：

| 需求原文/原子约束 | 覆盖点说明 | 设计方法 | 层级 |
|---|---|---|---|
| 需求文本原文 A | 对应测试关注点 | 所选设计方法 | L1/L2/L3 |

## 3. 设计方法选择

根据需求特征选择方法，不要固定套模板。

| 需求特征 | 优先方法 |
|---|---|
| 输入字段有范围限制，如数值、日期、金额 | 边界值分析 + 等价类 |
| 多个输入条件共同决定输出 | 因果图 -> 判定表 |
| 多条件独立且全排列用例爆炸 | 正交实验法 |
| 业务流程有明确状态变化 | 状态迁移法 |
| 用户完整操作路径型业务 | 场景法 |
| 历史 Bug 集中或经验高风险点 | 错误猜测法 |
| 多业务规则组合产生不同结论 | 判定表法 |
| 输入域可分合法、非法、边界类别 | 等价类划分 |

方法使用要点：

- 等价类：有效类和无效类各取代表值；触发同一路径的重复值要合并。
- 边界值：优先取 `[min-1, min, min+1, max-1, max, max+1]`，并结合等价类去重。
- 因果图：列出原因和结果，建立 AND/OR/NOT 关系后转判定表。
- 判定表：枚举条件组合，合并相同动作列，每列对应一条核心用例。
- 状态迁移：覆盖合法迁移和非法迁移尝试。
- 错误猜测：聚焦第三方集成、写后读一致性、极端值、并发、权限和历史高风险点。
- 正交实验：用于多因子组合抽样，保证关键因子组合被覆盖。

## 4. 覆盖模型

### L1 业务层

覆盖正常路径和正向业务闭环，是用例主体。

### L2 稳定性层

覆盖异常跳过、非法输入、边界极值、接口超时、数据缺失、状态不一致等稳定性问题。

### L3 工程安全层

覆盖并发冲突、重复提交、权限越权、审计完整性、数据一致性、敏感操作防护等工程风险。

测试方向优先级：功能测试为主；接口、性能、安全、易用性只抽取最典型的薄弱点，不要喧宾夺主。

建议配比：正向约 40%，反向异常约 40%，边界稳定性约 20%。如需求本身更偏流程或安全，可按实际风险调整。

## 5. 用例生成规则

### 覆盖红线

- 显性需求必须 100% 颗粒化覆盖：字段、条件、路径、按钮、提示、状态回写、审计日志都要落到用例。
- 每个 Path、分支或状态迁移至少有独立用例，不要在主流程中“顺带验证”。
- 先完成全量功能覆盖，再补充少量代表性非功能场景。
- 同一代码路径或同一等价类的重复用例要合并。
- 步骤必须原子化，每步对应明确操作或数据投喂。
- 文案必须具象，禁止“输入大金额”“选择一个物流”等模糊描述；应写成“输入 99999 USD”“选择 Carrier=DHL”。

### 预期结果规则

每条 `预期结果` 必须包含以下三个前缀，缺一不可：

- `[UI反馈]`：弹窗、提示文案、按钮状态、页面跳转等用户可见反馈。
- `[状态回写]`：后端字段、状态值、数据落库或业务状态变化。
- `[审计记录]`：审计日志、操作人、时间、前后状态、关键链路记录。

### 生成前自检

输出 JSON 前检查：

1. `用例标题` 不超过 20 字。
2. `预期结果` 均包含 `[UI反馈]`、`[状态回写]`、`[审计记录]`。
3. 原子约束清单中的显性字段、微交互和审计要求均已覆盖。
4. 必要时补齐三类高风险场景：全部为空/未选拦截、网络超时或并发双击幂等、AND/OR 条件部分匹配降级。

## 6. JSON 字段规范

只使用下列中文字段名；不要主动输出英文 Key 或同义替换。`系统补录` 由导出脚本自动生成，生成 JSON 时不要填写。

| 字段名 | 必填 | 规则 |
|---|---|---|
| `用例编号` | 是 | `TS-CA-001` 格式，三位数字，连续递增 |
| `所属模块` | 是 | 模块名，不加数字序号前缀 |
| `用例标题` | 是 | 描述测试目的，20 字以内 |
| `测试类型` | 是 | `功能测试` / `接口测试` / `性能测试` / `安全测试` / `易用性测试` / `边界测试` / `异常测试` |
| `前置条件` | 是 | 执行前系统必须满足的状态 |
| `测试步骤` | 是 | 使用 `\n` 分隔，格式为 `1. 操作\n2. 操作` |
| `预期结果` | 是 | 必含 `[UI反馈]`、`[状态回写]`、`[审计记录]` |
| `优先级` | 是 | `P0` / `P1` / `P2` |
| `备注` | 否 | 设计意图、风险说明或补充信息 |

JSON 必须是数组，结构字符使用半角引号、冒号、逗号和中括号。不要手动拼接不合法 JSON。

## 7. 文件与导出流程

默认缓冲文件：`skills/doublehit-case-skills/output/cases_buffer.json`
默认导出脚本：`skills/doublehit-case-skills/scripts/case_exporter.py`
默认导出文件：`skills/doublehit-case-skills/output/TestCases.xlsx`

执行流程：

1. 生成标准 JSON 数组。
2. 写入前先做轻量写入探测，确认当前环境可用的写入方式；不要直接把大段 JSON 交给未经验证的写入工具。

推荐探测顺序：

- 首选 `Read + Write`：先读取目标文件；如果不存在，先尝试读取并接受“文件不存在”结果，再使用 `Write` 写入正式 JSON。
- 如果 `Write` 返回 `File has not been read yet`、权限限制或其他工具错误，改用 Bash 写入探测：

```bash
mkdir -p skills/doublehit-case-skills/output && printf '[]' > skills/doublehit-case-skills/output/.write_probe.json && python -c "import json; json.load(open('skills/doublehit-case-skills/output/.write_probe.json', encoding='utf-8'))"
```

- 如果 Bash 写入不可用，再用 Python 写入探测：

```bash
python -c "from pathlib import Path; p=Path('skills/doublehit-case-skills/output/.write_probe.json'); p.parent.mkdir(parents=True, exist_ok=True); p.write_text('[]', encoding='utf-8')"
```

3. 使用探测通过的方式写入 `skills/doublehit-case-skills/output/cases_buffer.json`。
4. 校验 JSON 合法性：

```bash
python -c "import json; json.load(open('skills/doublehit-case-skills/output/cases_buffer.json', encoding='utf-8'))"
```

5. 第一次导出或需要覆盖旧文件时使用 `new`：

```bash
python skills/doublehit-case-skills/scripts/case_exporter.py skills/doublehit-case-skills/output/cases_buffer.json skills/doublehit-case-skills/output/TestCases.xlsx new
```

6. 分批追加导出时使用 `append`：

```bash
python skills/doublehit-case-skills/scripts/case_exporter.py skills/doublehit-case-skills/output/cases_buffer.json skills/doublehit-case-skills/output/TestCases.xlsx append
```

7. 确认导出命令成功后，向用户提供 Excel 文件路径。
8. 导出完成后可删除缓冲文件，或将其重置为 `[]`，避免下次生成混入旧数据。

如果预计用例超过 15 条，建议分批生成并用 `append` 追加，降低单次输出过长导致的失败概率。
