# 远程求职向世界 · Agent 接手简报

> **这是给接手 Agent 看的。** 目标：把现有内容资产做成「全电脑能用」的产品。
> **状态**：2026-08-13，内容已开源、持续共建中。
> **公开仓库**：`https://github.com/gengyueworks/remote-jobhunt-guide`（CC0）

---

## ⏱ 30 秒速览（给接手 Agent）

- **这是什么**：一个开源（CC0）的「如何面向全世界 remote 求职」方法论手册。现有形态是 **静态 HTML + Markdown 内容库**，核心价值在"教人怎么干"，不是职位板。
- **你要干的事**：把"GitHub 上的笔记"升级成"任意电脑（mac/win/linux）能安装/运行/交互"的产品，并把两条已验证的**流水线引擎**（小红书转写、简历改写）包进产品。
- **最关键约束**：`get-job.skill` 是 **CC-BY-NC-ND-4.0（非商业 + 禁止演绎）**，产品若商业化必须自研等价简历引擎，不能依赖它本体。
- **协作纪律**：动本仓库前先 `git fetch` + `GIT_EDITOR=true git rebase origin/main`，否则会被拒或冲突。

---

## 1. 项目定位

- **核心诉求**：「如何面向全世界 remote 求职」的方法论，差异化在"教人怎么干"，不是再建一个职位板。
- **参考标杆**：levelsio（Remote OK 创始人）+ awesome-remote-job 列表。
- **增长打法**：awesome-list 飞轮 = GitHub 高星 → Google 排名 → 免费长期流量 → 社区 PR 共建。内容即获客。

---

## 2. 现有交付物（仓库真实结构）

```
remote-jobhunt-guide/
├── index.html                     # 落地页（卡片导航）
├── playbook.html                  # 实战手册：按角色选平台/竞争力三件套/打法/避坑/冷启动
├── analysis.html                  # 平台借鉴分析：10 个主流平台定位 + 可借鉴打法
├── tips.html                      # 求职实战 tips：小红书案例转写 + 共性总结
├── README.md                     # 内容清单 + 本地查看 + 贡献说明
├── CONTRIBUTING.md
├── LICENSE                       # CC0 1.0
└── 参考资料/
    ├── media-reports/            # HBR 等远程工作研究报道（中英双语）
    ├── upwork专题/               # Upwork 出海接单实战（小红书 OCR 转写）
    ├── 中文远程岗位/             # 从小红书筛的远程/自由职业岗位（来源+日期+申请方式）
    ├── 远程求职故事/             # 真实上岸经历（Reddit 10 个月 / 荷兰产品经理 2 个月）
    ├── 简历专题/                 # AI 简历 Skill 锐评 + 反幻觉工作流 + 红线清单
    ├── linkedin-profile-4changes.md   # LinkedIn 主页优化（中英双语）
    └── 个人品牌与定位/           # AI 时代自我定位认知框架（战略层）
```

**10 类内容板块**：实战手册 / 平台借鉴分析 / 求职实战 tips / 媒体报道 / Upwork 专题 / 中文远程岗位 / 远程求职故事 / 简历专题 / LinkedIn 优化（中英双语）/ 个人品牌与定位。

---

## 3. 两条可复用流水线（引擎 · 重点）

### 3.1 小红书笔记转写流水线（已验证）
输入 `xhslink.cn/o/<code>` 短链 → 输出结构化 Markdown 进对应板块。

1. 解析短链：`curl -A 'Mozilla/5.0...' https://xhslink.cn/o/<code>` → 从返回 HTML 提取 `<a href="https://www.xiaohongshu.com/discovery/item/<note_id>?...&xsec_token=<token>">`。
2. 抓 explore 页：`curl -A ... "https://www.xiaohongshu.com/explore/<note_id>?xsec_token=<token>&xsec_source=app_share" -o xhs.html`（约 700KB，HTTP 200）。
3. 解析 `window.__INITIAL_STATE__`：形态 A 引号包裹字符串（`\"`→`"` 还原）；形态 B 裸 `{...}`（brace-depth 匹配截取）。都先 `undefined`→`null` 再 `json.loads`。
4. **视频笔记**：`note.noteDetailMap[<id>].note.video` → 双重转义 JSON → `subtitles` 块 → 正则抓 `\\"url\\":\\"(https:\\u002F\\u002F[^\"]+?\.srt)\\",\\"language\\":\\"([a-zA-Z\-]+)\\"` → `urllib` + `Referer: https://www.xiaohongshu.com/` 下载 `.srt`（视频裸直链会 403，必须用品幕签名 URL）→ 按空行切 cue、丢序号行与时间轴、合并文本得逐字稿。
5. **图文笔记**：`note.imageList[].urlDefault` → 下载 xhscdn JPEG → `tesseract` OCR（`-l chi_sim+eng --psm 6`）。正文多在 `note.desc`，配图 OCR 作补充/校验。
6. 归类进板块（见第 5 节），写 Markdown + 更新 README/index.html。

**已知坑**
- 热门笔记触发 XHS「扫码查看」风控墙 → web 路线走不通，回退 App 导出图片 OCR 或用户截长图。
- ASR 误识（荷兰→"河南"、面试率→"音乐录取率"）必须结合标题/上下文人工校正。
- 细体灰字 OCR 可能空 → 先 PIL 放大 3 倍 + autocontrast + Otsu 二值化再 OCR。

### 3.2 get-job.skill（已审计 · 重要）
排第一的简历改写 skill。
- **源码**：`https://github.com/agentenatalie/get-job.skill`（displayName「实习.skill」）
- **许可证**：**CC-BY-NC-ND-4.0** ⚠️ 非商业 + 禁止演绎。只能原样个人使用，不能改、不能商用、不能重新分发改版。
- **审计结论**：安全。SKILL.md 有完整 P0 红线（造假/改背调硬信息/来源冒充事实/轮次写死/bullet 漏项/标签泄漏/过度包装）；`scripts/quality_check.py` 纯本地只读 lint；`scripts/generate_resume.py` 仅 python-docx 本地出 docx，无网络、无外传。
- **质量闸门**：`python3 scripts/quality_check.py examples/` → `Quality check passed.`
- **自带案例**：`examples/` 下 3 个完整真实产出（文科生投运营 / 非科班投 AI 产品 / 文科投 AICoding）。

---

## 4. 协作铁律（必读）

- 提交前先 `git fetch` + `GIT_EDITOR=true git rebase origin/main`：本仓库多会话并行维护，本地与远程常分叉。
- 冲突只在 `README.md` 的「内容」区块，手动并入保留双方条目即可。
- 不要跳过 rebase 直接 push（会被 non-fast-forward 拒）。
- 仓库是 **CC0**，但 get-job.skill 是 CC-BY-NC-ND，混用时后者约束优先。
- 提交信息风格：`feat(板块): 简述`（中文）。

---

## 5. 内容分类法

- **战术层（怎么落地）**：简历专题、LinkedIn 优化、tips、中文远程岗位、Upwork 专题、求职故事。
- **战略层（认知框架）**：个人品牌与定位（裸能力 < 公开 Claim ≤ AI 增强交付力；竞争力 = Capability ✖️ Leverage ✖️ Narrative ✖️ Distribution）。
- **情报层（外部证据）**：media-reports（常做中英双语）。
- 新笔记先判层级/主题，再落 `参考资料/<板块>/` 或进 `tips.html`。

---

## 6. 红线（不能碰）

1. **不编造**：公开定位上限 =「AI 增强后的真实交付力」，与简历红线「写真实可证明结果」不冲突（前者定上限、后者定底线）。
2. **get-job.skill 许可证**（最关键）：CC-BY-NC-ND。商业化产品必须自研等价简历引擎（可借鉴其"证据分级/质量闸门/可靠性协议"设计，但不复制代码）。
3. **小红书合规**：转写仅作个人学习整理，注意平台 ToS；不硬刚风控墙；不做大规模自动抓取。
4. **转写校验**：所有 ASR/OCR 稿须二次校验人名、地名、数字、术语。

---

## 7. 目标：做成「全电脑能用」的产品

### 7.1 目标定义
把"GitHub 上的 HTML+Markdown 笔记"升级为任意电脑（mac/win/linux）能安装/运行/交互的产品，同时保留 awesome-list 飞轮（公开静态站承接 SEO/流量，桌面端承接深度使用）。

### 7.2 建议形态
- **推荐：Tauri 桌面应用**（Rust 壳，轻量、原生、跨平台）
  - 模块 A「手册」：内嵌 `playbook.html / analysis.html / tips.html` + 参考资料，左侧目录导航。
  - 模块 B「小红书转写器」：贴 xhslink → 调 3.1 流水线 → 自动归类 → 落盘对应板块（可选一键 PR）。
  - 模块 C「简历工坊」：上传简历 + JD → 调自研/等价改写引擎（注意 6.2 许可）→ 出 docx + 质量闸门报告。
- **备选（更轻）：本地 Web 服务 + CLI**。Python/Node 起本地服务，浏览器即用，无需安装；CLI 负责两条流水线。

### 7.3 功能模块 ↔ 现有内容
| 产品模块 | 复用现有 | 需新建 |
|---|---|---|
| 手册阅读 | playbook/analysis/tips + 参考资料 | 导航/搜索 |
| 小红书转写器 | 3.1 流水线 | UI 包裹、自动归类、PR 提交 |
| 简历工坊 | get-job 思路（需自研等价实现） | 上传/解析/出 docx/闸门 |
| 岗位聚合入口 | 中文远程岗位 / 用户已有聚合 demo | 接入与去重 |

### 7.4 技术建议
- 跨平台优先 Tauri；求最快落地可先静态站 + Python 本地服务验证体验。
- 转写流水线依赖：Python + tesseract（`chi_sim+eng`）+ Pillow 预处理。
- 简历引擎自研时复用 get-job 的设计（仅借鉴，不复制代码，规避 ND）。

### 7.5 待你确认（决策点）
1. 外壳：Tauri 桌面端 vs 本地 Web+CLI？
2. 简历模块是否商业化？否→可考虑 get-job 副本；是→必须自研等价实现。
3. 是否接用户已有「聚合 demo」作岗位流量入口？
4. 跨平台最低支持：mac/win/linux 全要，还是先做 mac？

---

## 8. 下一步清单（TODO）

- [ ] 确认 7.5 四个决策点（优先外壳 + 简历许可）。
- [ ] 搭产品骨架：内嵌现有 3 张 HTML + 参考资料导航。
- [ ] 把 3.1 小红书转写流水线封装为可调用的转写器模块。
- [ ] 简历工坊：自研或等价实现改写引擎（避开 get-job ND），接入质量闸门。
- [ ] 跨平台构建验证（mac/win/linux 至少能跑）。
- [ ] 保留 awesome-list 飞轮：公开静态站 + 桌面端进阶工具双层结构。
- [ ] 更新 README/落地页，说明「网页版 vs 桌面端」分工。

---

## 9. 路径速查（仓库相对）

| 用途 | 路径 |
|---|---|
| 实战手册 | `playbook.html` |
| 平台分析 | `analysis.html` |
| 求职 tips | `tips.html` |
| 简历 Skill 锐评 | `参考资料/简历专题/简历skill-锐评-从夯到拉.md` |
| 简历红线清单 | `参考资料/简历专题/README.md` |
| 个人品牌框架 | `参考资料/个人品牌与定位/不要只按真实水平宣传自己.md` |
| LinkedIn 优化 | `参考资料/linkedin-profile-4changes.md` |
| get-job.skill 源码 | `github.com/agentenatalie/get-job.skill`（CC-BY-NC-ND） |

---

## 10. 参考
- 仓库 README.md —— 内容清单与本地查看方式。
- 本简报对应完整版内部交接见历史记录（含本机路径与详细审计日志）。
