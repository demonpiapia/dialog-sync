# Subject Process Guideline — General v0.5

> **类型**：**Guideline（操作规程）**——规定谁在何时做什么、产出什么、过什么门；原则论证见上游理论文件
> **版本**：v0.5（Draft / Review Candidate · 二轮复核遗留清零，待三轮确认闭环）
> **日期**：2026-09-14
> **适用**：外部探测 / 逆向分析 / 自动化集成 / Patch / 协议复现 / 复杂工程交付类 Agent 项目
> **上游文件**：`通用项目流程spec-v0.1.md`（流程框架）＋ `通用项目流程 Spec v0.2.md`（原则体系，**规范基准**）；v0.3 将二者合并并文体从论述转为规程；v0.4 针对 v0.3 审计发现的原则缺口做定点修复与自述对齐；v0.5 清零二轮复核遗留的登记级问题（详见 §9/§10）
> **核心立场**（v0.2 §26 原文，全文效力最高）：
> **Agent 可以猜方法，但不能猜成事实；可以失败，但不能把失败包装成结论；可以带着不确定性前进，但必须显式标记不确定性；可以继承先例，但不能继承先例的当前事实；可以修改计划，但核心机制变化时必须重新验证。**

---

## 1. 流程总览

```
S0 Scope/Authority → S1 P1 摸底 → S2 对齐 → S3 P2 全面探测 → S4 Spec → S5 计划 → S6 执行⇄P3 定向 → S7 交付
                                                      ↑__________________ 回退环 __________________|
任意阶段触发升级条件（§3.7）→ Full-Audit → 重建事实与证据链 → 回 P2 / P1 / S0
```

| 阶段 | 一句话职责 | 关键产出 | 过门条件 |
|---|---|---|---|
| S0 | 定边界与权限，不定机制 | Goal / Scope / Authority / 动作分级表 | §2.1 门 |
| S1 P1 | 建立基线，不下机制结论 | Target/Entry/Environment Map + Known Evidence + 假设清单 | §2.2 门 |
| S2 | 与决策者确认要证明什么 | 对齐记录（语义以决策者引用为准） | §2.3 门 |
| S3 P2 | 声明范围内穷尽假设裁决 | 机制规格 + 假设裁决表 + 对抗复核报告 | §2.4 门 |
| S4 | 核心机制结论须有 L1/L2 证据链；L3/L4 单独支撑的条目以 Unresolved 登记 | Mechanism Spec + Evidence Matrix | §2.5 门 |
| S5 | 只依赖 Confirmed 机制排计划 | Execution Plan（步骤映射 DD） | §2.6 门 |
| S6 | 执行 + 按需 P3 定向探测 | 交付物 + P3 报告（新证据四分类处置） | §2.7 门 |
| S7 | 闭环交付 | Closure 报告（七问全答） | §2.8 门 |

---

## 2. 阶段操作规程

> 每阶段固定五要素：**职责 / 必做 / 产出 / 过门（Gate） / 禁止**。Gate 未全勾不得进入下一阶段；确实无法满足时按 §3.7 升级，不得静默跳过。

### 2.1 S0 — Scope / Authority

- **职责**：建立项目边界与动作权限；此阶段**不产生任何机制事实**。
- **必做**：
  - [ ] Goal：目标 / 交付物 / 成功标准 / 明确排除项
  - [ ] Scope：目标系统与版本 / 输入输出边界 / 允许观察对象 / 允许修改对象 / 禁触范围
  - [ ] Authority：Agent 可独立执行的动作 / 需授权动作 / 始终禁止动作 / 仅限测试环境动作
  - [ ] **动作分级**：计划中每个动作标注类别 —— `READ / OBSERVE / ANALYZE / MODIFY / EXECUTE / EXTERNAL-ACTION`；高风险动作（不可逆、消耗性、外部副作用）执行前**必须再次授权**
- **产出**：S0 边界文档（含动作分级表）
- **Gate-A0**：Goal/Scope/Authority/动作分级 四表齐全；决策者已确认
- **禁止**：在本阶段写入任何机制判断

### 2.2 S1 — P1 Baseline Probe

- **职责**：建立系统基线（组成/入口/环境/先例/假设），**不下机制结论**。
- **必做**：
  - [ ] **先例盘点（第 0 步）**：查找并精读已验证先例；每个先例登记三分法：
    | 分法 | 当前项目中的地位 |
    |---|---|
    | Precedent **Fact**（先例中观察到的事实） | 交叉参考（L4） |
    | Precedent **Method**（先例的方法） | **可继承为候选方法** |
    | Precedent **Mechanism**（先例确认的机制） | **仅候选假设，不得继承为当前事实** |
  - [ ] Target Map：组成 / 组件 / 运行环境 / 可观察接口 / 依赖
  - [ ] Entry Map：按 8 类枚举——`UI / CLI / API / Runtime / Network / File / Process / State`；**允许写 Unknown，禁止为地图完整而强行补齐**
  - [ ] Environment Map：版本 / 路径 / 关键文件哈希基线
  - [ ] **Known Evidence**：目前已知的证据（标注 L3/L4/U 级，不可作为机制结论依据）
  - [ ] Initial Hypothesis Space + Open Questions 登记（格式见 §3.3）
- **产出**：P1 基线报告（全条目标注 L3/L4/U 级）
- **Gate-A1**：先例盘点完成；三类 Map 建立；Known Evidence 登记；假设已登记；**时间盒已设定且未超**
- **禁止**：机制结论；深层手段开局（抓包/逆向加密/网络栈指纹）；任何写操作与真实动作

### 2.3 S2 — Alignment

- **职责**：与决策者确认"本项目到底要证明什么"。
- **必做**：确认 Goal / Scope / Success Criteria / Known Facts / Open Questions / Hypothesis Space / 风险 / 执行前必须解决的问题清单
- **产出**：对齐记录
- **Gate-A2**：决策者逐项确认；**语义以决策者引用的先例原文为准**，agent 不得用自己的框架解释决策者概念（纠正 = 语义对齐信号，逐条复述）
- **禁止**：在本阶段重新探测或修改事实

### 2.4 S3 — P2 Full Probe

- **职责**：在**已声明范围内**对关键 Entry 与关键假设做系统性验证。
- **"全面"的可验收定义**：Entry Set × Hypothesis Set 每项均处于 `Confirmed / Rejected / Unresolved / Not-Tested` 之一（详见 §3.3）。**不存在"agent 漏了假设然后宣布完成"**。
- **必做**：
  - [ ] **实验矩阵**：每个假设按 `Hypothesis → Expected Observation → Experiment → Observed Result → Evidence → Verdict` 登记一条链
  - [ ] **区分性实验优先**：按 `高信息增益 > 低成本 > 低风险 > 可重复 > 可观测` 排序；禁止无区分性的"多做几次看看"
  - [ ] **手段梯度**：运行时拦截（hook fetch/XHR/关键函数）→ 调试器真值 → （仅当前者穷尽且确需鉴权/实现层分析时）深层手段；静态分析仅作辅助，**不得在动态不确定时凌驾 L1/L2**
  - [ ] **对抗性复核**：P2 产物交独立 agent/审计者——**验证据，不验叙事**（重新检查关键 Evidence / 复做关键 Experiment / 独立验证关键 Claim）
- **产出**：机制规格 + 假设裁决表 + 对抗复核报告
- **Gate-A3**：§4 Gate-A 检查单全勾（Probe→Spec 门）
- **禁止**：用 agent 主观"探索完毕感"代替裁决表；隐藏 Unresolved

### 2.5 S4 — Mechanism Spec

- **职责**：把 Confirmed 机制固化为规格；**机制不明不动手**（例外见 Gate-B 注）。
- **必做**：
  - [ ] Spec 十节结构：`Goal / Scope / Known Facts / Observations / Mechanism Claims / Evidence Matrix / Unresolved Issues / Design Decisions / Constraints / Risks`
  - [ ] **Evidence Matrix**（必备表）：

    | Claim | Evidence | Strength | Status |
    |---|---|---|---|
    | MC-001 | OBS-001, E-002 | L1+L2 | Confirmed |
    | MC-003 | Static only | L3 | Unresolved |

  - [ ] 设计决策与机制事实**分区书写**，不得混写（§3.2）
- **产出**：Spec（交独立审计）
- **Gate-A4**：核心机制全部 Confirmed；**例外**——机制 Unresolved 但同时满足（风险低 + 动作可逆 + 不把假设固化为事实 + 已获授权）方可放行，且必须在 Spec 中显式登记
- **禁止**：L3/L4 证据**单独**支撑核心机制结论（交叉参考与辅助解释允许）；把设计决策写成观察事实

### 2.6 S5 — Execution Plan

- **职责**：只依赖三种东西排计划——`Confirmed 机制 + 显式接受的 Unresolved + 已批准的 Design Decisions`。
- **必做**：
  - [ ] 每个关键步骤映射到 `DD-xxx`
  - [ ] 每个 Unresolved 写四件套：`Impact / Mitigation / Fallback / Failure Trigger`（Fallback 触发即进入 P3 定向探测）
- **产出**：Execution Plan + 测试矩阵（含合规自检项）+ 备份/回滚方案
- **Gate-A5**：Plan 与 Spec 一致；高风险动作已授权；Unresolved 四件套齐全
- **禁止**：Plan 引用无证据链的机制

### 2.7 S6 — Execution ⇄ P3 Directed Probe

- **职责**：执行交付；P3 用于执行中验证假设、核对 Spec 与实际行为、定向解决新问题（**不是重跑 P2**）。
- **必做**：
  - [ ] 执行中出现新证据时，逐条四分类并处置：
    | 分类 | 处置 |
    |---|---|
    | **Confirmed**（与模型一致） | 继续 |
    | **Modified**（机制部分成立，边界变化） | 更新 Spec/Plan 后继续 |
    | **Invalidated**（核心机制被证伪） | **必须回退**（§3.7） |
    | **Newly Discovered**（新入口/状态/机制） | 评估对 Spec/Plan 影响，必要时回退 |
  - [ ] P3 探测若涉及真实动作（写/消耗性），先过授权门
- **产出**：交付物 + P3 定向报告（带证据级别）
- **Gate-A6**：验证通过（只读验证 → 真实动作验证，见 §2.8）；无未处置的 Invalidated/Newly Discovered
- **禁止**：执行中静默修改机制认知（必须走回环更新 Spec）；重复同类失败时继续局部修补（→ §3.7 Full-Audit）

### 2.8 S7 — Delivery Closure

- **职责**：闭环交付，回答七问：`目标是否完成？交付物在哪？如何验证？实际观察到什么？哪些限制？哪些 Unresolved？仓库/工作区状态？`
- **必做（Core Closure）**：
  - [ ] 只读验证：lint / typecheck / static check / dry-run / read-only test / artifact inspection / 配置校验（按项目选用）
  - [ ] 真实动作验证（如需）七要素齐备：`Action / Scope / Risk / Authority / Expected / Observed / Rollback`；**不可逆动作单独授权**
  - [ ] Project-specific Closure Profile：`Idempotence / Storage Integrity（哈希对比） / Commit / External Action` 按项目类型启用（示例见 §6）
  - [ ] Commit 前顺序：验证 → 检查 diff → 确认交付物 → commit。**Commit 只是版本状态，不是完成证明**
  - [ ] 记忆留存（§3.8）
- **产出**：Closure 报告（七问 + 检查单存档）
- **Gate-A7**：§4 Gate-C 检查单全勾
- **禁止**：用 commit 代替验证；沉默跳过未完成项

---

## 3. 全局工作规则（跨阶段，随时生效）

### 3.0 基本原则：先事实，后机制，后设计，后执行

**流程链条（v0.2 §0.1）**：

```text
Scope / Authority
    ↓
Observation
    ↓
Evidence
    ↓
Mechanism Claim
    ↓
Design Decision
    ↓
Plan
    ↓
Execution
    ↓
Verification
```

**禁止跳跃清单**——不得直接从：`目标描述 / 先例 / Agent 经验 / 推测` 跳到：`机制结论 / 实现方案 / 最终结论`。

### 3.1 证据分级与采信

| 级别 | 类型 | 采信规则 |
|---|---|---|
| **L1** | 直接观测：实际请求/响应/调用/状态变化/运行行为/文件与进程网络行为 | **最高等级证据**；机制判定的首要依据 |
| **L2** | 可复现实验：条件明确、操作明确、结果可记录、能重复 | 高；机制判定依据 |
| L3 | 静态分析：源码/二进制/配置/bundle/AST/文档化结构 | 定位与假设用；**动态不确定时不得凌驾 L1/L2** |
| L4 | 历史产物：历史 commit/旧实现/社区资料/过去报告 | 候选机制与方法；**不得自动成为当前事实** |
| **U** | 用户实证：用户明确提供并可引用的经验事实 | 直接进入调查范围；条件允许时用 L1/L2 验证 |

冲突规则：L3/L4 与 L1/L2 存在明确冲突时以 L1/L2 为准。

### 3.2 三类结论登记（编号强制）

| 类型 | 编号 | 单行模板 | 铁律 |
|---|---|---|---|
| Observation | `OBS-xxx` | `OBS-017｜Action: 触发请求｜Observed: 捕获 POST /v1/chat/completions｜Evidence: L1｜Status: Confirmed` | 只记"实际发生了什么"，不得夹带机制解释 |
| Mechanism Claim | `MC-xxx` | `MC-004｜Claim: 请求经 runtime fetch wrapper｜Based On: OBS-017+OBS-021+EXP-008｜Evidence: L1+L2｜Status: Confirmed` | **必须引用依据编号** |
| Design Decision | `DD-xxx` | `DD-003｜Decision: 采用 fetch interception 而非 DOM automation｜Reason: MC-004 已确认 + 成本更低` | **不得伪装成 Observation** |

### 3.3 假设空间登记（Hypothesis Register）

| 状态 | 判据 |
|---|---|
| **Confirmed** | 存在足够证据支持该机制判断 |
| **Rejected** | 存在能有效区分并否定该假设的证据 |
| **Unresolved** | 已调查，现有证据不足以确认或否定 |
| **Not-Tested** | 尚未获得有效测试 |

格式：`H-03｜假设: 请求由 native bridge 发出｜Status: Confirmed｜Evidence: E-07 (L1/L2)`
"没有证明" ≠ "证明不存在"；未发现 ≠ 不存在；无法定性时一律写 **Unresolved**，不写 Rejected。

### 3.4 Unresolved 显式化

- Unresolved **不得隐藏**，必须标明影响范围；
- 它可以随项目前进，但必须挂四件套（Impact/Mitigation/Fallback/Failure Trigger）；
- 任何报告/Spec/交付中，Unresolved 是独立章节，不并入"风险"或"备注"。

### 3.5 失败优先检查（v0.2 §0.2）

失败不是结论。单次失败只证明"当前方法在当前条件下未达预期"，不得直接推导"目标不可行 / 机制不存在 / 方向错误 / 项目无法完成"。

**失败发生后，Agent 应优先检查以下六项**：
1. 输入或环境是否正确（版本/路径/配置/凭证）；
2. 当前事实模型是否完整（是否遗漏关键 OBS）；
3. 当前假设是否成立（H-xx 是否需要更新状态）；
4. 当前实验是否具有区分性（能否区分多个候选机制）；
5. 是否遗漏入口、路径、状态或依赖；
6. 当前失败是否可能只是局部方法失败（换个手段可能就通）。

### 3.6 失败报告标准格式（agent 无法继续时必须按此输出，禁止只写"无法完成"）

```text
当前方法失败。
已确认：<Known 列表>
尚未确认：<Unknown 列表>
证据清单：<Evidence 列表，标注 L1/L2/L3/L4 级>
当前假设：<Hypotheses + 状态>
已尝试方法：<Attempted Methods 列表 + 结果简述>
已排除路径：<Rejected Paths 列表 + 排除依据>
失败只证明：<当前方法在当前条件下未达预期>
尚不能证明：<不可行 / 机制不存在 / 方向错误>
下一步最有价值的验证：<Next Best Probe>
```

### 3.7 回退与 Full-Audit

**回退触发表**：

| 触发事件 | 动作 |
|---|---|
| Core Mechanism Invalidated | S6 → S3 (P2) |
| Core Mechanism Modified | 更新 Spec/Plan；评估后决定是否回 S3 |
| Critical New Discovery | 评估后决定是否回 S3 |
| Evidence Conflict | 回 S3 重建证据链 |
| Unresolved 变为 Blocking | 回 S3 |
| 同类失败重复 + 仅靠局部修补 | **Full-Audit** |

**Full-Audit（治理升级，不是简单重跑）**——触发条件（任一）：
A. 用户/Agent/Spec/实验间出现重大事实矛盾；B. 核心 MC 进入 Invalidated；C. 关键事实缺失无法判断后续；D. 同类失败重复且仅局部修补；E. Agent 无法回答"现在知道什么/不知道什么/哪步是假设/哪步是事实"；F. 先例与当前事实严重冲突。

**Full-Audit 必查八项**：`Scope / Authority / Evidence / Observations / Hypothesis Space / Mechanism Claims / Failure History / Design Decisions`。**禁止跳过事实复核直接从原 Spec 修补**。审计后按结论回退：S3 → 必要时 S1 → 必要时 S0。

### 3.8 记忆留存

必须保存：`Confirmed Facts / Validated Methods / Rejected Approaches / 重要失败模式 / 可复用工具 / Known Constraints`。
**尤其保存"之前的方法为什么失败"，而非只保存"最后怎么成功的"。**

---

## 4. 三道门检查单（直接勾选使用）

### Gate-A：Probe → Spec
```
[ ] Scope 已确认            [ ] Authority 已确认
[ ] Entry Set 已建立        [ ] Hypothesis Space 已建立
[ ] 关键 OBS 已记录         [ ] 关键 MC 已建立
[ ] MC 均有证据链           [ ] Unresolved 已显式列出
[ ] 没有把失败写成结论      [ ] 没有把先例当当前事实
```

### Gate-B：Spec → Execution
```
[ ] 核心 Mechanism 已 Confirmed（或满足 2.5 例外且已授权）
[ ] Design Decisions 可追溯（DD 编号）
[ ] Unresolved 影响已评估（四件套）
[ ] 高风险动作已授权
[ ] Plan 与 Spec 一致
```

### Gate-C：Delivery Closure
```
[ ] 交付物存在             [ ] 验收标准已执行
[ ] 实际结果已观察         [ ] 限制已披露
[ ] Unresolved 已披露      [ ] 工作区/仓库状态已确认
```

---

## 5. 禁止清单（流程违规行为）

| # | 违规 | 纠正动作 |
|---|---|---|
| 1 | 未探测直接宣称机制 | 回 S3 补证据链 |
| 2 | 把失败等价为不可行 | 按 §3.5 六项检查 + §3.6 重写失败报告 |
| 3 | 把"未发现"写成"不存在" | 改标 Unresolved |
| 4 | 把先例机制当当前机制 | 改标候选假设（H-xx） |
| 5 | 把静态推断伪装成动态事实 | 降级 L3，重走动态验证 |
| 6 | 把自己计划的结果当验证结果 | 独立验证或 L1 观测 |
| 7 | 用第二个 agent 的主观认同替代独立证据 | 验证据不验叙事 |
| 8 | 假设未登记就宣称"全面完成" | 补 Hypothesis Register 再验收 |
| 9 | 核心机制被推翻后只局部修补 | 触发 Full-Audit |
| 10 | 隐藏 Unresolved | 独立章节显式列出 |

---

## 6. Closure Profile 示例（Core + 项目特化）

**Core Closure**（所有项目必做）：§2.8 前两项 + 七问。
**Profile 示例（外部自动化类，如 checkin 项目）**：
`Idempotence（同日二次→已签到）` + `Storage Integrity（目标存储文件哈希对比）` + `External Action（单次真实签到需授权）` + `Commit（等指令后本地 commit）`。
**Profile 由 S0 阶段按项目类型选定，写入 Execution Plan。**

---

## 7. 最小合规自检（agent 每轮自问）

> 现在到底知道什么、不知道什么、哪一步是假设、哪一步是事实？
> 回答不出 = 触发 Full-Audit 条件 E。

---

## 8. 与上游文件的关系

| 文件 | 角色 |
|---|---|
| `通用项目流程spec-v0.1.md` | 流程框架起源（本仓库）；v0.3～v0.5 从中合并了手段梯度、时间盒、深层手段禁止、语义对齐等条款 |
| `通用项目流程 Spec v0.2.md` | **原则体系（规范基准）**——v0.3～v0.5 的原则正确性以 v0.2 为最终依据 |
| **本文件 v0.5** | **执行规程**——v0.2 原则的 guideline 化，同时吸收 v0.1 中与 v0.2 相容的增强条款 |
| Teamwork-Guideline（general / 项目特化） | 工程约定层（目录分层/备份/Git 策略）；本规程与之并行，修改留待验证后 |

---

## 9. 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-09-14 | 本地 Agent（TRAE IDE） | Probe 三阶段前置流程初版 |
| v0.2 | 2026-09-14 | GPT Agent | 新增 Scope/Authority、Hypothesis Space、OBS/MC/DD 三层、Evidence Matrix、Unresolved 显式化、Modified/Invalidated/Newly Discovered、Full-Audit、证据型独立复核、Closure Profile；调整"不预设方向"为"不预设结论允许方法假设"等六处 |
| v0.3 | 2026-09-14 | 本地 Agent（TRAE IDE） | 文体重构；将 v0.1 与 v0.2 合并为统一规程；每阶段固定"职责/必做/产出/过门/禁止"五要素；三道门可勾选检查单；禁止清单补纠正动作列；回退触发表 + Full-Audit 触发条件表格式化；失败报告模板化；Closure Profile 示例化；新增最小合规自检（§7）。**同时将 v0.2 多处"建议"收紧为规程化必做（实验矩阵、DD 映射编号、OBS/MC/DD 编号强制、Spec 十节结构必做），并从 v0.1 吸收时间盒、深层手段禁止开局、手段梯度、语义对齐等增强条款** |
| v0.4 | 2026-09-14 | 本地 Agent（TRAE IDE） | **针对 codebuddy 审计 v0.3 的 12 项发现做定点修复**：①恢复 P2 完成标准的 Not-Tested 状态（v0.2 §9.4）；②失败报告模板补 Evidence/Attempted Methods/Rejected Paths 三项 + 失败后六项优先检查清单（v0.2 §22 + §0.2）；③§1 表 S4 职责修正为"核心机制结论须有 L1/L2 证据链；L3/L4 单独支撑的条目以 Unresolved 登记"，与 v0.2 §11.1 示例对齐；④§2.5 禁止条款限定为"单独支撑"，消除 §1 表与正文矛盾；⑤L1 标注"唯一推论基准"→"最高等级证据"（避免排除 L2 效力）；⑥回退表 Core Mechanism Modified 改为"评估后决定是否回 S3"，删除 Unresolved Blocking 的"授权替代回退"路径；⑦§3 增补 §0.1 先事实后机制链条 + 禁止跳跃清单；⑧P1 产出补 Known Evidence 项；⑨头部与 §9/§10 自述对齐（承认合并 v0.1 内容与收紧行为）；**保留经独立判断认为合理的 v0.3 收紧与 v0.1 增强条款**（冲突规则保留并加"存在明确冲突"限定、"建议→必做"收紧） |
| **v0.5** | 2026-09-14 | 本地 Agent（TRAE IDE） | **二轮复核（codebuddy 对 v0.4 的复核）遗留清零**：①删除本表 v0.4 行悬空引用"（§0）"（V-01）；②§1 表 S4 行补"单独"二字，与 §2.5 禁止条款对称，消除"凡涉 L3/L4 即 Unresolved"的过宽读法（V-02，本表 v0.4 行③引文同步更正）；③冲突规则登记由"保留 v0.3 表述"更正为"保留并加'存在明确冲突'限定"（V-03，消除登记与文件实况不一致；F-05 按"保留+限定+登记"结案，该条定位为 v0.1 系增强条款的规程化保留，非与 v0.2 逐字等价）；④v0.3 行收紧登记补"Spec 十节结构必做"（V-04，4/4 补齐）；⑤§10 §22 行去向更正为"§3.6 失败报告模板 + §7 自检"（独立发现：原含"§3.5 失败优先检查"属过度映射，§3.5 是 v0.2 §0.2 六项检查的落点）；⑥版本号线性迭代 v0.4→v0.5（未采用审计建议的"v0.4-r1"命名，保持本仓库线性版本序列）。V-05（总结引用计数更正）为对话层更正，无文件改动。**本轮不自行标记"审计闭环"**——闭环状态应由三轮独立复核确认，而非修订者自述 |

## 10. v0.2 → v0.5 内容映射（含审计修复标注）

| v0.2 章节 | 去向（v0.5） | 备注 |
|---|---|---|
| §0.1 先事实后机制链条 | §3.0（新增显式条款） | v0.3 仅结构性隐含，v0.4 显式化（审计修复 F-08） |
| §0.2 失败不是结论 + 六项检查 | §3.5（新增失败优先检查） | v0.3 仅保留报告格式，检查动作缺失（审计修复 F-03） |
| §0.3 不预设结论允许方法假设 | 头部核心立场 + §3.1 + §2.4 手段梯度 | 本质保留（允许方法选择但不得提前成为机制事实） |
| §0.4 OBS/MC/DD 可区分 | §3.2 | v0.3 收紧为编号强制（合理收紧） |
| §0.5 Unresolved ≠ Rejected | §3.4 | 一致 |
| §0.6 先例是候选路径 | §2.2 | 一致 |
| §1 总流程 / §25 状态机 | §1 流程总览 | 一致 |
| §2 S0 | §2.1 | v0.3 收紧（合理） |
| §3 S1 P1 七项产出 | §2.2 | v0.3 缺 Known Evidence，v0.4 补齐（审计修复 F-09） |
| §4 S2 Alignment | §2.3 | 一致 |
| §5 Hypothesis Space | §3.3 | 一致（四状态全保留） |
| §6 Evidence Model | §3.1 | v0.3 的"唯一推论基准"→v0.4 改为"最高等级证据"（审计修复 F-06）；冲突规则保留 v0.3 规程化表述并加"存在明确冲突"限定（F-05 结案口径：保留+限定+登记；文本链为 v0.1 无条件句 → v0.2 条件句 → v0.3/v0.4 无条件+限定，本条定位为 v0.1 系增强条款的规程化保留，非与 v0.2 逐字等价） |
| §7 OBS/MC/DD | §3.2 | v0.3 收紧为编号强制（合理） |
| §8–9 P2 + 完成标准 | §2.4 + Gate-A | v0.3 排除 Not-Tested，v0.4 恢复（审计修复 F-01）；"建议实验矩阵"→v0.3 必做（合理收紧） |
| §10 机制不明不动手 | §2.5 Gate-A4（含例外条件） | 一致 |
| §11 S4 Spec + Evidence Matrix | §2.5 | v0.3 §1 表"只用 L1/L2"与 §2.5 示例矛盾，v0.4 §1 表修正 + §2.5 禁止条款限定"单独支撑"（审计修复 F-04）；v0.5 §1 表补"单独"保持对称（V-02） |
| §12 S5 Plan + Unresolved 规则 | §2.6 + §3.4 | v0.3 "尽可能映射"→"每个映射"（合理收紧） |
| §13–14 S6/P3 + 新证据四分类 | §2.7 | 一致 |
| §15 回退规则 | §3.7 回退触发表 | Core Mechanism Modified 改为"评估后决定是否回"（审计修复 F-07(a)）；删除 Unresolved Blocking 的授权替代路径（审计修复 F-07(b)） |
| §16 Full-Audit | §3.7 | 一致 |
| §17 独立复核 | §2.4（对抗性复核条目） | 一致 |
| §18–19 S7 Closure | §2.8 + §6 | 一致 |
| §20 Git/Commit | §2.8（commit 只是版本状态） | 一致 |
| §21 Memory | §3.8 | 一致 |
| §22 Agent 状态表达 | §3.6 失败报告模板 + §7 自检 | v0.3 缺 3 项最低输出，v0.4 补齐 Evidence/Attempted Methods/Rejected Paths（审计修复 F-02）；v0.5 更正去向（v0.4 登记误含"§3.5 失败优先检查"，该节属 v0.2 §0.2 六项检查的落点，见 §0.2 行） |
| §23 禁止事项 | §5（补纠正动作） | 一致 |
| §24 最低合规检查 | §4 三道门检查单 | 21/21 一致 |
| §26 一句话版本 | 头部核心立场（原文保留） | 逐字一致 |
