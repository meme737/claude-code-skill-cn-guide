---
name: weekly-update-protocol
description: 跨会话更新个人档案的标准流程——每周拖入个人档案文件夹并执行更新
metadata: 
  node_type: memory
  type: project
  updated: 2026-06-15
  originSessionId: 99906df7-f2f3-4094-b7b5-571293334da1
---

## 触发
用户说"更新我的个人档案"或在任何会话中拖入 `C:\Users\tyr52\Desktop\个人档案\` 文件夹。

## 执行流程
1. 读取 `C:\Users\tyr52\Desktop\个人档案\` 下所有 .md 文件（了解当前状态）
2. 读取 `user-profile-comprehensive.md`（了解用户完整背景）
3. 请用户用几句话概括本周变化（新事件、身体变化、心理变化、新的自我发现等）
4. 根据用户的概括 + 本会话的聊天内容，更新相应文件：
   - `01-基础信息.md`：当前状态和日期
   - `04-身体数据.md`：体重、健身数据
   - `07-行动清单.md`：标记已完成、添加新目标
   - 其他文件如有新信息一并更新
5. 同步更新 `user-profile-comprehensive.md` 中的相关字段
6. 汇报本周变更摘要给用户确认

## 注意
- 不要编造信息——只更新用户明确告知或本会话中讨论过的内容
- 如果用户这周没聊什么新东西，如实说"本周无变更"
- 更新日期格式：YYYY-MM-DD
