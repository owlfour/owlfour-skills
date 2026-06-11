---
name: oas-commit
description: 英文 commit：分析暂存区改动，生成 conventional commit 格式的提交信息
allowed-tools: Bash, Read, Grep
---
# OAS Commit Skill
生成符合仓库规范的英文 git commit message 并提交。
## 输出格式规范

每次输出的时候按照一下结构输出，

```markdown
### 改动分析
- deploy/config.py: 做了什么（中文简述）                                                     
- deploy/installer.py: 做了什么（中文简述）
### 判定依据
- type = fix，因为修改了已有逻辑的 bug / 行为                                                
- scope = Deploy，因为改动文件在 deploy/ 目录                                                
- 复杂改动：是/否，理由：涉及 2 个文件 / 单文件单逻辑 / ... 
### 提案
type(Scope): summary  #123
- change point 1
- change point 2
```
- type(Scope): 动词开头的英文概述，末尾附 issue 号（如有）
**body 格式**（仅复杂改动需要，简单改动只有一行 subject）：
## 仓库风格参考
基于本仓库实际 commit 归纳：
**type**: `fix`（最常用）/ `feat` / `refactor` / `chore` / `docs` / `style` / `test` / `perf`
**scope**: PascalCase 模块名，从改动文件路径推断：
`(Exploration)` `(Guild)` `(WantedQuests)` `(Orochi)` `(Sougenbi)` `(Handle)` `(SwitchSoul)` `(CostumeShikigami)` `(TrueOrochi)` `(WeeklyTrifles)` `(SixRealms)` `(workflow)` `(device)` 等
**真实范例**：
简单改动（无 body）：

```
fix(Guild): Close chat window popup when entering guild page  #1571
fix(Exploration): Normalize misrecognized level numbers by replacing similar characters  #1540
fix(Sougenbi): Delete redundant image files
```
复杂改动（带 body）：
```
fix(Handle): Add retry mechanism for handle tree construction and optimize logging
- fix IndexError when accessing empty children of screenshot_handle_num
- retry up to 10 times with 1s interval when handle_tree children is empty
```

```  
feat(DemonEncounter): Remove duplicate soul config for Demon Encounter boss
- merge switch_soul_config_1 and switch_soul_config_2 into single config
- update task script references to use the new unified config
- remove duplicate soul switching logic
```

**issue 号**：` #数字` 放在 subject 行末尾，前面留一个空格。

## 参数

用户可通过 `/oas-commit <额外信息>` 传入补充说明，例如：
- `/oas-commit fix #242`
- `/oas-commit fix(Exploration): adjust team creation ROI coordinates`
- `/oas-commit 修了登录`

若参数中包含 `#数字`，自动提取并附加到 subject 末尾。若用户已给出完整的 `type(Scope): summary`，直接沿用。

**重要**：当用户参数中带有 `#数字` 时，表示本次改动关联某个 issue 或 PR。在分析代码变更之前，必须先执行 `gh issue view <number>` 或 `gh pr view <number>` 获取该 issue/PR 的标题、描述、讨论内容，理解改动背景和要解决的问题，再结合 `git diff` 的代码变更，综合判断改动类型（fix/feat/chore 等）和 scope，生成准确的 commit message。

## 工作流程

1. 若用户参数中包含 `#数字`，先运行 `gh issue view <number>` / `gh pr view <number>` 获取上下文
2. 运行 `git diff` 仔细阅读代码变更
3. 综合 issue/PR 背景和代码变更，按格式规范生成英文 commit message，同时输出中文改动分析，展示给用户确认
4. 用户确认后，用 heredoc 格式执行 git commit；除非用户明确要求，否则不执行 `git push`
