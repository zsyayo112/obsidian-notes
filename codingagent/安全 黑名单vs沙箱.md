---
tags:
  - agent开发
  - 机制
  - repopilot
  - 安全
created: 2026-07-13
---

# 🛡️ 安全：黑名单 vs 沙箱

> [!important] 一句话
> **黑名单是"列举所有坏的"——你永远列不全，蓄意/跑偏只要撞到一个没列的就赢了，必输的博弈。沙箱是"限制爆炸半径"——不管里面跑什么都伤不到宿主，能赢的博弈。**

## RepoPilot 的两道闸（policy.py）
1. **危险命令黑名单**（`check_command`）：命令含 `rm -rf /`、`sudo`、`git commit/push` 等就拒绝。防**事故**。
2. **路径牢笼**（`jail`）：文件操作路径必须落在仓库内（`is_relative_to`），写操作禁碰 `.git`。防**越狱**。
```python
DANGEROUS = ["rm -rf /","sudo ",...,"git commit","git push","git reset"]
def check_command(cmd):
    for pat in DANGEROUS:
        if pat in cmd: return False, f"…匹配到 {pat}"
    return True, ""
```
`git commit/push` 被拉黑 = **"最后一步永远归人"**用代码钉死。

## 黑名单的根本局限（面试必答诚实这点）
子串黑名单能被轻易绕过：`rm -rf ~`、`find . -delete`、写个 `.py` 脚本再 `python3 x.py`……你**列不全**。所以黑名单只挡**手滑事故**，不挡蓄意。

## 真正的强隔离 = Docker 沙箱
把 agent 关进一个"就算它跑任意命令也伤不到宿主"的环境（非 root、CPU/内存限制、网络隔离、超时）。这是 Phase 5 的独立课题。

## MVP 的现实防线 = 三层叠加
黑名单（挡事故）+ [[git当撤销键|git 可回滚]]（改坏能还原）+ 权限确认（人否决）+ MAX_MODIFIED_FILES（改动规模上限）。**能说清天花板和下一步（Docker），比假装刀枪不入更专业。**

## 面试
"怎么保证 agent 执行命令的安全" —— 分层防御（黑名单+jail+权限+回滚），且诚实：黑名单是列举坏的（必输），强隔离要靠沙箱（限制爆炸半径）。

关联：[[RepoPilot MOC]] · [[硬约束vs软约束]] · [[git当撤销键]]
