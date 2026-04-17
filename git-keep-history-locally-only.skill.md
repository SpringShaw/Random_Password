# Skill: Git 初始化项目到远程 - 大文件/历史版本本地保留，不上传远程

## 工作流说明

场景：本地已有项目开发，包含多个历史版本文件，需要推送到远程仓库，但**远程只保留最新代码，本地保留所有历史版本文件以方便回退**，不占用远程存储空间。

### 执行步骤

1. **初始化本地仓库**
   ```bash
   git init
   ```

2. **创建 .gitignore** 添加需要本地保留但不上传远程的文件模式
   ```gitignore
   # Keep these locally only
   index-v*.html
   *-backup.html
   *-old.html
   ```

3. **添加所有文件并首次提交**
   ```bash
   git add .
   git commit -m "Initial commit: <项目描述>"
   ```

4. **创建分支结构**
   ```bash
   git checkout -b develop
   git checkout master
   ```

5. **添加远程仓库**
   ```bash
   git remote add origin <远程仓库URL>
   ```

6. **首次推送处理冲突（远程已有初始内容）**
   ```bash
   # 拉取远程内容，允许不相关历史
   git pull origin master --allow-unrelated-histories
   # 解决冲突后提交
   git add .
   git commit -m "Merge remote: keep local codebase"
   # 推送
   git push -u origin master
   git push -u origin develop
   # 如果有tag
   git push origin <tagname>
   ```

7. **如果已经提交到Git，现在需要从远程移除但本地保留**

   ```bash
   # 1. 从Git缓存移除，保留本地文件
   git rm --cached <要移除的文件模式>
   # 2. 更新 .gitignore，确保以后不会被提交
   echo "index-v*.html" >> .gitignore
   # 3. 提交变更
   git add .gitignore
   git commit -m "Cleanup: remove historical versions from remote, keep locally"
   # 4. 推送到远程
   git push origin master
   # 同步到develop
   git checkout develop
   git rebase master
   git push -f origin develop
   ```

8. **恢复本地文件（如果意外删除）**
   ```bash
   # 从Git历史中检出文件，恢复到本地磁盘
   git checkout <commit-hash> -- <文件模式>
   # 取消暂存（因为已经在.gitignore中，不会再被提交）
   git restore --staged <文件模式>
   ```

### 关键要点

- `git rm --cached`：只从Git索引移除，**不删除本地磁盘文件**
- `.gitignore`：添加匹配规则后，Git会忽略这些文件，今后不会自动提交到远程
- 从Git历史恢复：文件在之前的commit中存在，可以通过 `git checkout <commit> -- <file>` 恢复到本地

### 本次具体案例

项目：离线JSON解析工具箱
- 远程保留：`index.html` (最新版本) + `README.md` + `VERSION.md`
- 本地保留：所有 `index-v1.x.html` 历史版本方便回退
- 远程仓库大小从 ~600KB 减少到 ~60KB

### 使用方式

在新项目需要做类似处理时：
```
/skill git-keep-history-locally-only
```
