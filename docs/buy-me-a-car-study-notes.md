# buy-me-a-car 学习笔记

来源：https://github.com/DaizeDong/buy-me-a-car （MIT，v0.2.2，阅读日期 2026-11-04）

## 1. 它是什么

一个面向美国购车流程的 Claude Code 技能包（plugin）：1 个总控 `orchestrator`（技能名 `buy-me-a-car`）+ 15 个可单独调用的窄技能，
覆盖从需求、找车、比价、和 dealer 来回砍价，到验车、保险、签约提车的全过程。

## 2. 五条设计原则（核心思想）

1. **比较全部成本**：先确认预算指的是车价还是 OTD（out-the-door 落地价）；税、费、运费分项列出，未知项保持"未知"而不是乱填。
2. **把研究交付完**：复用已知需求，缺的问题一次性打包问，同时继续做不依赖答案的研究；第一轮研究后就直接出对比表 + HTML/PDF 报告。
3. **买家掌控**：买家心里的最高价（private ceiling / walk-away）与对外报价（authorized offer）严格分开，**绝不**在给 dealer 的邮件里透露上限或由它推出的区间；发邮件、预约、付定金都要逐项授权。
4. **证据可追溯**：保留来源 URL、观察日期、原始文件和 SHA-256；网页挂牌价 ≠ 书面 OTD 报价；社交媒体帖子只算"自述"。
5. **购车记录私有**：公开仓库只是工具；真实需求、报价、报告放在买家自己的私有 companion 仓库（`BUY_ME_A_CAR_CONFIG`）。

## 3. 九阶段工作流

| 阶段 | 做什么 |
|---|---|
| 1 需求 | 确认 OTD/车价预算、上牌州、车型约束、付款方式、置换车、卖家类型、时间线，存 criteria |
| 2 市场基线 | 抓当前挂牌 + 厂家优惠，开始写研究报告 |
| 3 库存 | 按 VIN 去重（保留不一致信息），产出 `master_comparison.md` + `buyer_research.html/pdf` |
| 4 外联 | 准备给具体 dealer 的邮件，经授权才发送 |
| 5 收件箱 | 分类：真回复 / 自动回复 / CRM 模板 / 垃圾 / 营销 |
| 6 谈判 | 用 reply drafter 生成受约束的回信，强制不泄露上限 |
| 7 证据审核 | 通读报价单、CARFAX、保修、PPI 文件，标出矛盾与缺页 |
| 8 决策档案 | 用新证据更新报告；对外 proposal 需要完整报价 + 授权条款 |
| 9 试驾与签约 | 用 close-day checklist 复核库存、验车、合同总价、付款方式、title、保险 |

需求有重大变化（州、车型、付款、预算等）→ 新建 criteria 版本，按 pivot protocol 标记哪些报价/计算过期。

## 4. 16 个技能一览

| 类别 | 技能 |
|---|---|
| 研究与证据 | orchestrator、quote-evidence-collector |
| 成本 | otd-calculator、state-fee-lookup、lease-vs-cash-analyzer、trade-in-valuator |
| 车辆核查 | carfax-pdf-review、cpo-eligibility（12 个品牌 CPO 规则）、ev-buyer-helper |
| 回信与报告 | dealer-reply-drafter、inbox-triage、dossier-builder |
| 付款与提车 | payment-method-decider、insurance-shopper、ppi-scheduler、close-day-checklist |

## 5. 谈判手册要点（negotiation_playbook）

- 三个数严格区分：dealer 当前书面 OTD、买家私有 walk-away、买家授权的对外 offer。
- **第一轮**：只要该 VIN 的完整 OTD 分项（车价、税、doc fee、title、注册、其他费用、有条件折扣），不透露融资能力、月供上限、目标区间。
- **可用锚点**只有：确认存在的当前挂牌、或竞品的书面 OTD（记录卖家、车、新旧、金额、日期）。不要编故事、不用无来源的"全国均价"。
- 每封回信 1–3 个请求 + 最多 1 句锚点 + 1 句离场语，约 10 行以内。
- 现金不等于折扣（dealer 可能有融资返利）；不要用"贷款"去换"去掉某个费用"。
- 离场理由：拒绝书面报价、强制加装、拒绝第三方验车（PPI）、车史矛盾、超过私有上限。离场语可以简单到："If that does not work, I will continue my search."

## 6. 值得记住的"坑"（gotchas）

- 给 dealer 的邮件用纯 ASCII，不要 markdown、破折号。
- dealer 邮件常进 Promotions/Spam；每天做一次 `newer_than:3d is:unread` 全量扫描。
- 带具体 VIN/价格的 CRM 模板邮件也要处理。
- 不要把 dealer 自己的网价截图发回给它；第三方平台（CarGurus/Cars.com/Edmunds）截图或竞品报价可以。
- "一口价"型 dealer（如 Enterprise）别浪费轮次，尽快转向。
- "72 小时内有效"可能是真也可能是施压，问清楚 hold/定金机制。

## 7. 德州（TX）相关（来自其 state_fees 表）

- 机动车销售税 6.25%，同一交易中合格的置换车全额抵扣（按车辆 allowance 计，不是 equity）。
- 单独列出的 documentary fee 不计入机动车税基。
- 表中 TX 的 doc fee 上限、title、注册费等标为 *Unverified*，用前需查官方来源（Texas Comptroller / TxDMV）。

## 8. 对本仓库（Cars-hunting）的启发 / 可借鉴的东西

- 把"预算 = OTD"作为第一个必问问题；所有比较都在同一假设下做。
- 每辆候选车记录：来源 URL、抓取日期、VIN、挂牌价、书面 OTD（如有）、证据文件。
- 回信模板：Round 1 只要完整 OTD 分项；后续回合 ≤10 行。
- 签约前清单：合同总价 vs 书面 OTD、拒绝 F&I 加装、付款方式与限额、title、保险 binder。

## 9. 注意

- 原项目安装依赖作者的 `fleet-guards`、`fleet-style`、`llmcall` 等配套工具和私有 companion 仓库，直接整套搬过来跑不起来，适合"借鉴思路 + 选择性移植"。
- 研究报告渲染器目前输出中文报告。
