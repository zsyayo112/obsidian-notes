# 澳洲 Senior Software Engineer 学习路线图

> 基于 50 个 Sydney/Melbourne 真实岗位 JD 的 80% 共性技术栈 + AI 驱动的三层 Checklist 学习法

---

## 📑 目录

- [一、80% 共性技术栈总结](#一80-共性技术栈总结)
- [二、Claude Project 的完整 Prompt](#二claude-project-的完整-prompt)
- [三、学习清单（按优先级）](#三学习清单按优先级)
- [四、LeetCode + HackerRank 刷题策略](#四leetcode--hackerrank-刷题策略)
- [五、成为"大佬"的持续竞争力方法](#五成为大佬的持续竞争力方法)
- [六、下一步行动清单](#六下一步行动清单)

---

## 一、80% 共性技术栈总结

基于 50 个 Sydney/Melbourne 岗位的 JD 分析，澳洲 Senior Software Engineer 出现频率 ≥80% 的技术栈：

| 类别 | 核心技术（必学） | 出现频率 |
|---|---|---|
| **编程语言** | TypeScript（主线）+ Python（辅助） | ~90% |
| **前端框架** | React（含 Hooks、Context、React Query） | ~85% |
| **前端工程化** | Next.js、Vite、Tailwind CSS | ~70% |
| **后端框架** | Node.js + NestJS / Express | ~80% |
| **后端替代栈**（可选一条深挖） | Java + Spring Boot / C# + .NET / Python + Django | ~50% |
| **数据库** | PostgreSQL（主）+ DynamoDB/MongoDB（NoSQL） | ~90% |
| **API 设计** | RESTful API、GraphQL、OpenAPI/Swagger | ~85% |
| **云平台** | AWS（Lambda、S3、ECS/Fargate、RDS、API Gateway、SQS、CloudWatch） | ~90% |
| **容器化** | Docker + Docker Compose | ~80% |
| **编排** | Kubernetes（大厂） / ECS（中小厂） | ~55% |
| **IaC** | Terraform | ~50% |
| **CI/CD** | GitHub Actions、CircleCI | ~80% |
| **测试** | Jest、Vitest、Playwright、TDD | ~85% |
| **版本控制** | Git（工作流、rebase、cherry-pick） | 100% |
| **架构模式** | Microservices、Event-driven、CQRS | ~85% |
| **消息队列** | Kafka、SQS、EventBridge | ~60% |
| **缓存** | Redis | ~65% |
| **监控** | Grafana、Prometheus、Datadog、Sentry | ~70% |
| **认证/安全** | OAuth2、JWT、OWASP Top 10 | ~80% |
| **AI 工具** | Cursor、Claude、Copilot（协作写代码） | ~45% ↑ |
| **系统设计** | Load balancing、Scaling、Caching strategies | ~85% |

**学习优先级建议**：TypeScript + React + Node.js/NestJS + PostgreSQL + AWS + Docker + Git + System Design + Testing（这 9 项覆盖 80% 的 JD 要求）。

---

## 二、Claude Project 的完整 Prompt

### 使用说明

1. 打开 Claude → 左侧 **Projects** → **New Project**
2. 命名为：**AU Senior Engineer Roadmap**
3. 把下面整段 Prompt 粘贴到 **Custom Instructions** / **Project Knowledge**
4. 把你之前的 Excel（`Senior_Software_Developer_Jobs_Analysis.xlsx`）上传到 Project Knowledge

### Prompt 正文（复制以下内容）

````markdown
# 角色设定
你是一位资深的澳洲 Senior Software Engineer 导师，拥有 10+ 年全栈开发经验，
在 Canva、Atlassian、WiseTech 级别的公司带过团队，熟悉 Sydney/Melbourne 
就业市场对 Senior 级别工程师的真实要求（包括 5+ years commercial experience、
end-to-end ownership、system design、mentoring 等隐性标准）。

# 我的身份与目标
我是 Graduate（计算机相关专业，偏全栈方向），目标是在 3-12 个月内对标澳洲 
Senior Software Engineer 的技术水平，达到能独立负责 feature 端到端交付、
做技术选型、带 mid-level 工程师的能力。

# 三层 Checklist 学习法（严格遵守）

当我发送一个技术点（例如 "NestJS"、"AWS Lambda"、"React Hooks"、"PostgreSQL 索引"）时，
你必须按以下三层结构输出：

## 第一层：学习目录大纲（Level 1 - Outline）
按澳洲 Senior 工程师水平，列出该技术的 8-15 个核心 topic，每个 topic 包含：
- 序号 + 标题
- 难度标签（⭐基础 / ⭐⭐进阶 / ⭐⭐⭐高级 / ⭐⭐⭐⭐Senior 必备）
- 一句话说明为什么 Senior 需要掌握
- 学习路径按从易到难排序

输出第一层后停下，问我："要展开哪个 topic 的第二层？（可以说序号，或说 all 一次生成所有）"

## 第二层：每个 Topic 的知识点清单（Level 2 - Knowledge Points）
针对第一层指定的 topic，列出 5-10 个必须掌握的具体知识点（子 bullet 级别），每个知识点：
- 用简短标题命名
- 一句话说明该知识点解决什么问题

输出后问我："要展开哪个知识点的第三层？"

## 第三层：每个知识点的完整学习卡片（Level 3 - Learning Card）
针对第二层指定的知识点，生成以下 5 个部分（用清晰的 section 分隔）：

### 1. 📝 实例代码
- 可实际运行的代码（TypeScript 优先；后端代码用 Node.js/NestJS）
- 20-60 行，包含关键注释
- 在代码末尾标出 2-3 个常见易错点（用 ⚠️ 标注）

### 2. 🎯 使用场景
- 3 个真实业务场景（SaaS / Fintech / E-commerce / HealthTech 偏好）
- 每个场景 2-3 句，说明"为什么在这个场景下选这个技术"

### 3. 🔄 可替代的类似技术
- 列出 2-4 个替代方案
- 每个替代方案给出：
  - 优势（1-2 点）
  - 劣势（1-2 点）
  - 什么场景下应该选它
- 最后给出"选型决策树"（如果条件 A，选 X；如果条件 B，选 Y）

### 4. 💼 面试常考题目（2-3 题）
- 每题标注"澳洲 Senior 高频题"难度（🟢基础 / 🟡进阶 / 🔴高级）
- 提供：
  - 题目原文
  - 答题核心 points（关键词，不展开）
  - 2 个常见追问方向
  - 一个加分亮点回答（展示 Senior 思维的角度）

### 5. 🧪 小测验
输出以下两种之一：
- **代码块测验**：给一段有 bug 或待补全的代码，问我修复/补全，并说明原因
- **场景题**：给一个业务需求，让我做技术选型并说明理由

测验放在"点击展开答案"提示下方（不要直接给答案，让我先思考）。

---

## 额外要求

### A. 每个学习卡片末尾追加"Senior 思维彩蛋"
用 1 段话（50-100 字）告诉我：
- 一个 Mid-level 不会注意但 Senior 必会关注的点
- 或一个这个技术的"隐藏陷阱"/"生产事故常见原因"
- 或一个与这个技术相关的架构决策经验

### B. 与 LeetCode / HackerRank 关联
如果当前技术点能自然关联到算法题，追加：
- **相关算法主题**：（例如：React 性能优化 → 关联到"滑动窗口"、"虚拟列表"）
- **LeetCode 推荐 2-3 题**（附题号和难度）
- **HackerRank 推荐 1-2 题**（附主题）

### C. 进度追踪
- 当我说 "打卡 [topic名]"，你记录并回复"✅ 已打卡 [topic]，累计 N 个 topic。
  下一个建议学：[topic]"
- 当我说 "复习 [topic]"，你随机出 2 道该 topic 的面试题和 1 个场景题

### D. 综合 Case 考察（阶段性）
每当我打卡满 5 个 topic，主动出一道综合 case：
- 给一个完整的业务需求（如"设计一个 Pet Circle 的订单系统"）
- 让我用已学的技术点做架构设计
- 你扮演面试官追问

---

## 交互风格
- 输出结构清晰，多用 emoji 和 section 分隔
- 不冗长，每个 section 控制在合理长度
- 代码要能跑，不用伪代码
- 涉及 AWS/云服务时，给出真实的服务名和配置示例
- 涉及架构时，用文字画简单的 ASCII 图（或 Mermaid）

## 我的启动指令示例
- "开始 NestJS"（触发第一层）
- "展开 topic 3"（触发第二层）
- "深挖知识点 2"（触发第三层）
- "打卡 NestJS Dependency Injection"（进度追踪）
- "复习 React Hooks"（抽查）
- "给我一个综合 case"（阶段测验）
````

---

## 三、学习清单（按优先级）

### 🥇 Tier 1 - 必学核心（80%+ JD 都要求，3-4 个月内拿下）

- [ ] 1. TypeScript（深度）
- [ ] 2. React + React Hooks + React Query
- [ ] 3. Node.js + NestJS
- [ ] 4. PostgreSQL（含索引、事务、N+1 优化）
- [ ] 5. RESTful API 设计 + OpenAPI
- [ ] 6. Git 高阶工作流（rebase, cherry-pick, 冲突解决）
- [ ] 7. Docker + Docker Compose
- [ ] 8. AWS 核心服务（Lambda, S3, RDS, API Gateway, IAM, CloudWatch）
- [ ] 9. Jest / Vitest 单元测试 + Playwright E2E
- [ ] 10. System Design 基础（load balancing, caching, scaling）

### 🥈 Tier 2 - 高级加分项（区分 Mid vs Senior，2-3 个月内跟进）

- [ ] 11. Next.js（SSR/ISR/App Router）
- [ ] 12. GraphQL + Apollo
- [ ] 13. Redis（缓存策略、分布式锁）
- [ ] 14. Message Queue（SQS / Kafka）
- [ ] 15. Microservices 架构模式
- [ ] 16. Event-driven Architecture（EventBridge）
- [ ] 17. CI/CD（GitHub Actions 完整 pipeline）
- [ ] 18. Terraform（IaC 基础）
- [ ] 19. OAuth2 + JWT + OWASP Top 10
- [ ] 20. Monitoring（Grafana, Sentry, Datadog）

### 🥉 Tier 3 - 差异化竞争力（选性深入，让你简历闪光）

- [ ] 21. Kubernetes 基础
- [ ] 22. AI 集成（OpenAI/Claude API, RAG, Vector DB）
- [ ] 23. Performance Engineering（profiling, benchmarking）
- [ ] 24. Database 高级（分库分表、读写分离）
- [ ] 25. WebSocket / Server-Sent Events
- [ ] 26. 至少一个替代后端栈（Java Spring 或 C# .NET）
- [ ] 27. Domain-Driven Design（DDD）
- [ ] 28. CQRS + Event Sourcing

---

## 四、LeetCode + HackerRank 刷题策略

### 📅 刷题节奏

**每天 2 题**（1 道新题 + 1 道旧题复习），持续 3-6 个月，总量 300-500 题。

---

### 📚 LeetCode 路径

按主题分阶段，**不要按题号乱刷**。

#### Phase 1：基础数据结构与算法（4-6 周，约 80 题）

| 主题 | 推荐题目 | 目标数量 |
|---|---|---|
| Array / Hashing | Two Sum (1), Group Anagrams (49), Top K Frequent (347) | 15 题 |
| Two Pointers | Valid Palindrome (125), 3Sum (15), Container With Most Water (11) | 10 题 |
| Sliding Window | Longest Substring Without Repeating (3), Minimum Window Substring (76) | 10 题 |
| Stack | Valid Parentheses (20), Min Stack (155), Daily Temperatures (739) | 8 题 |
| Binary Search | Binary Search (704), Search Rotated Sorted Array (33) | 10 题 |
| Linked List | Reverse Linked List (206), Merge Two Sorted Lists (21), LRU Cache (146) | 12 题 |
| Trees | Invert Binary Tree (226), Max Depth (104), Lowest Common Ancestor (236) | 15 题 |

#### Phase 2：进阶算法（4-6 周，约 80 题）

| 主题 | 推荐题目 |
|---|---|
| Tries / Heap | Implement Trie (208), Kth Largest (703), Merge K Sorted Lists (23) |
| Backtracking | Subsets (78), Combination Sum (39), Word Search (79) |
| Graphs | Number of Islands (200), Clone Graph (133), Course Schedule (207) |
| Dynamic Programming | Climbing Stairs (70), House Robber (198), Coin Change (322), LIS (300) |
| Greedy | Jump Game (55), Gas Station (134) |
| Intervals | Merge Intervals (56), Insert Interval (57) |

#### Phase 3：高频面试 + Neetcode 150（持续，约 150 题）

直接按 **[Neetcode 150](https://neetcode.io/practice)** 清单刷（免费、分类清晰、大厂高频）。

#### LeetCode 刷题原则

- ⏱️ **每题限时**：Easy 15 分钟 / Medium 30 分钟 / Hard 45 分钟，超时看 solution
- 📝 **每题写 3 件事**：思路、时空复杂度、替代解法
- 🔁 **复习曲线**：第 1 天做 → 第 3 天复习 → 第 7 天复习 → 第 21 天复习（Ebbinghaus 遗忘曲线）

---

### 🛠️ HackerRank 路径（作为 LeetCode 的实战补充）

HackerRank 的特点是**接近真实工作场景**（SQL、Shell、系统设计小题），澳洲一些公司（尤其 recruiter 筛选轮）会用 HackerRank 做 OA。

#### 推荐刷题顺序

| 主题 | 说明 | 数量 |
|---|---|---|
| **SQL（Intermediate + Advanced）** | 最高优先级！澳洲 senior 面试必考 SQL | 60 题 |
| **Problem Solving（Medium + Hard）** | 算法题，和 LeetCode 互补 | 50 题 |
| **JavaScript / TypeScript** | 前端/全栈岗 OA 常用 | 30 题 |
| **REST API Test** | 模拟真实 API 工作 | 15 题 |
| **10 Days of Statistics** | 数据题面试常考 | 10 题 |
| **Interview Preparation Kit** | HackerRank 官方高频清单 | 69 题 |

---

### 🎯 Claude Project 中如何自动关联刷题

你在 Project 里学某个技术点时（比如学"React useMemo"），Claude 会按照 Prompt 里的 **B. 与 LeetCode / HackerRank 关联** 规则，给你配套题目推荐。例如：

> 学 `PostgreSQL 索引` → Claude 推荐：
> - LeetCode SQL：175, 176, 177, 185, 262
> - HackerRank SQL：Challenges Section 全部
> - 算法关联：B-Tree 题目（LC 108, 109）

---

## 五、成为"大佬"的持续竞争力方法

### 🌱 短期（0-1 年）：打牢地基

1. **深度 > 广度**：先把 Tier 1 的 10 个主题学到能上手做项目，再碰 Tier 2/3。样样通样样松是初级陷阱。
2. **写 3 个 end-to-end 项目**：前端 + 后端 + 数据库 + AWS 部署 + CI/CD + 测试 + 监控。这一套组合打完，简历就能进 senior JD 的门。
3. **每周刷 14 道题**（10 道 LeetCode + 4 道 HackerRank SQL），坚持 6 个月。
4. **读源码 + 官方文档**：React、NestJS、PostgreSQL 文档从头读一遍（不是 tutorial，是官方 docs）。

### 🚀 中期（1-3 年）：建立个人品牌

5. **写技术博客**（Dev.to / Medium / 个人站）：每月 1-2 篇，记录踩坑和架构决策。这是澳洲 senior 评估时的隐性加分项。
6. **贡献开源**：从给常用的库提 bug fix / docs PR 开始，逐步到小 feature。GitHub contribution 图是简历最硬的凭证。
7. **参加 meetup**：Sydney JS、Web Directions、AWS User Group Sydney、Melbourne Full Stack。澳洲工作机会 40% 通过人脉，不是招聘网站。
8. **讲清楚技术决策**：每个技术选择都能说出 trade-off。"我选了 A 而不是 B，因为 X 场景下 A 的 Y 特性更适合"——这是 Senior 和 Mid 的分水岭。
9. **刻意练习 System Design**：每周 1 个系统设计题（设计微信、设计 Uber、设计 Instagram）。推荐 ByteByteGo 频道。

### 🏔️ 长期（3-5 年+）：从 Senior 到 Staff/Principal

10. **专精一个方向**：纯全栈很难往上走。挑一个 specialty：
    - **Performance Engineering**（让系统跑更快）
    - **Platform Engineering**（做开发者工具）
    - **Security Engineering**（安全方向紧缺）
    - **AI/ML Engineering**（2026 最热，供不应求）
    - **Distributed Systems**（进大厂的硬通货）
11. **做 Tech Lead 经验**：带 2-3 人小团队交付项目。这是升 Staff 的必经之路。
12. **公开演讲**：在 meetup 或 conf 做 talk。一次 20 分钟的分享 = 500 次 LinkedIn post 的影响力。
13. **建立 mentor 关系**：找一个比你高 2-3 级的 mentor（LinkedIn 直接发消息约 coffee chat，澳洲工程师文化对这个很开放）。同时开始 mentor 比你低的人——教是最好的学。
14. **保持危机感**：每季度问自己："如果今天公司裁员，我 2 周内能拿到同级别 offer 吗？"如果不能，就是警报。

### 🧠 元认知层面：大佬的思维习惯

- **质量 > 数量**：刷 200 道精题 + 复习，胜过刷 1000 道不复习。
- **读源码 + 自己造轮子**：真正理解 useState？自己实现一个。真正理解 Express？自己写一个 mini 版。
- **教学反馈**：给别人解释清楚一个技术，是你真正掌握的标志。试试给家人解释"什么是 API"。
- **保持好奇**：每月花 10% 时间学"没用"的东西（Rust、Elixir、Nix、一门非主流数据库）。大佬的突破经常来自"跨界联想"。
- **健康 > 一切**：熬夜写代码 2 年看不出差别，5 年直接垮。每周固定 3 次运动，8 小时睡眠不能少。这是最长期的竞争力。

---

## 六、下一步行动清单

### ✅ 今天就做（30 分钟）

- [ ] 1. 打开 Claude，点左侧 **Projects** → **New Project**
- [ ] 2. 命名 `AU Senior Engineer Roadmap`
- [ ] 3. 把本文档第二部分的 Prompt 完整粘贴到 Custom Instructions
- [ ] 4. 把 `Senior_Software_Developer_Jobs_Analysis.xlsx` 上传到 Project Knowledge
- [ ] 5. 第一条消息发 `开始 TypeScript`，看 Claude 输出第一层大纲

### 📅 本周做（2-3 小时）

- [ ] 6. 在 LeetCode 开账号（免费就够用）
- [ ] 7. 在 HackerRank 开账号
- [ ] 8. 打印/保存本文档到 Notion/Obsidian 作为长期 reference
- [ ] 9. 在 Claude Project 里完成 TypeScript 第一层 → 展开 topic 1 → 深挖第一个知识点，走通三层流程
- [ ] 10. 刷第一道 LeetCode（Two Sum）+ 第一道 HackerRank SQL

### 🗓️ 本月做（持续）

- [ ] 11. 完成 Tier 1 的前 3 个主题（TypeScript、React、Node.js/NestJS）各自至少 3 个 topic 的第三层
- [ ] 12. LeetCode 累计 25 题 + HackerRank 累计 10 题
- [ ] 13. 启动第 1 个 end-to-end 项目（全栈 + AWS 部署 + CI/CD）
- [ ] 14. 参加 1 次 Sydney/Melbourne 的技术 meetup

### 🎯 3 个月目标

- [ ] 15. Tier 1 全部完成（10 个主题）
- [ ] 16. LeetCode 累计 100 题 + HackerRank 累计 40 题
- [ ] 17. 完成 2 个 end-to-end 项目，放上 GitHub
- [ ] 18. LinkedIn Profile 按 Senior JD 关键词优化完毕
- [ ] 19. 开始每周投递 10-15 个岗位

---

## 📎 附录：推荐资源

### 学习资源
- **Neetcode**: https://neetcode.io/practice（算法清单）
- **ByteByteGo**: https://bytebytego.com（系统设计）
- **Roadmap.sh**: https://roadmap.sh/backend（技术路线图）
- **Refactoring Guru**: https://refactoring.guru（设计模式）
- **AWS Skill Builder**: https://skillbuilder.aws（AWS 官方免费培训）

### 书籍
- *Designing Data-Intensive Applications* - Martin Kleppmann（Senior 必读）
- *Clean Code* - Robert Martin
- *The Pragmatic Programmer* - Dave Thomas
- *System Design Interview* Vol 1 & 2 - Alex Xu

### 澳洲 Meetup / 社区
- Sydney JS: https://www.meetup.com/sydney-javascript/
- Web Directions: https://webdirections.org/
- AWS User Group Sydney: https://www.meetup.com/aws-sydney/
- Melbourne Full Stack: https://www.meetup.com/melbourne-full-stack-meetup/

### 澳洲求职平台（参考你的 Excel Sheet 3）
- Seek, Indeed, LinkedIn, Built In Sydney, Wellfound, TopStartups.io

---

**最后一句话**：大佬不是学完所有东西才成为大佬，而是在**每个当下都比昨天的自己多懂一点、多做一点、多坚持一点**。坚持 3-5 年，你就在别人眼里是大佬了。

祝你早日拿到 offer！🚀
