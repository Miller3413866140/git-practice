# Git & GitHub 实操速查表

> 本机环境已确认（2026-11-01）：
> - git 2.55.0.windows.3
> - 全局身份：`Dust` / `3413866140@qq.com`
> - **GitHub 账号：`Miller3413866140`** —— 不是 `Miller`（被别人占用），也不是 `MILLER112019`（不存在）
> - **SSH 已打通**：`ssh -T git@github.com` → `Hi Miller3413866140!`
>   指纹 `SHA256:Y+hU9pv8QcrOJqiUVxlG94SdK+CsEPfoVHvpag4Jk/E`
> - 已配 `core.quotepath false`（中文名不转义）、`core.autocrlf false`（关 CRLF 警告）
> - ⚠️ **`github.com` 的 443 端口被 RST 阻断**：网页打不开、`git clone https://...` 失败、
>   浏览器报 `ERR_CONNECTION_RESET`。但 **`api.github.com` 正常**（0.5 秒响应），**SSH 22 端口正常**。
>   → **结论：仓库操作用 SSH 地址，别用 HTTPS。** 网页访问需要加速工具或手机流量。

---

## 一、拿到代码：三种方式

```bash
# 1. 克隆整个仓库（含全部历史）—— 最常用
git clone https://github.com/用户名/仓库名.git

# 2. 只克隆最近 1 次提交（仓库大、只要代码时用，快很多）
git clone --depth 1 https://github.com/用户名/仓库名.git

# 3. 克隆到指定文件夹名
git clone <url> 我的文件夹名
```

**clone 之后你得到什么**：一个工作目录 + 一个 `.git/` 隐藏目录。
`.git/` 就是整个仓库的数据库，**删掉它，你的代码就变成一堆没有历史的散文件**。

---

## 二、四层结构与对应命令

```
工作区  --add-->  暂存区  --commit-->  本地仓库  --push-->  远程仓库(GitHub)
                                 <--pull/fetch--
```

| 层 | 是什么 | 命令 |
|---|---|---|
| 工作区 | 你能看到、能编辑的文件 | 直接改 |
| 暂存区 | 已挑选、等待提交的改动 | `git add` |
| 本地仓库 | `.git` 里的提交历史 | `git commit` |
| 远程仓库 | GitHub 服务器上的副本 | `git push` |

---

## 三、日常开发循环（80% 的时间只用这五个）

```bash
git status              # 我现在处于什么状态？（最高频，随时敲）
git diff                # 我具体改了哪几行？
git add .               # 把所有改动加入暂存区（或 git add 文件名）
git commit -m "说明"    # 存档，必须写清楚做了什么
git push                # 上传到 GitHub
```

### 提交说明的写法（推荐 Conventional Commits）

```
feat: 添加相机识别功能      # 新功能
fix: 修复登录崩溃           # 修 bug
docs: 更新 README           # 文档
refactor: 重构支付模块      # 重构（不改行为）
chore: 升级依赖             # 杂活
```

---

## 四、看状态：`git status --short` 怎么读

```
 M README      ← 前面空格、M 在右边 = 改了但没 add（未暂存）
M  README      ← M 在左边 = 已经 add 进暂存区
MM README      ← 两边都有 = add 之后又改了
?? newfile.txt ← ?? = 全新文件，Git 还没管它
A  newfile.txt ← 已暂存的新文件
D  oldfile.txt ← 已暂存的文件删除
```

**记住：靠左 = 已暂存，靠右 = 未暂存。**

---

## 五、改错了怎么办（保命技能）

```bash
# 改了但没 add —— 直接丢弃，恢复成上次提交的样子
git restore 文件名

# 已经 add 了 —— 先撤出暂存区（改动还在），再决定
git reset HEAD 文件名
git restore 文件名

# 已经 commit 了，想撤销这次提交但保留改动
git reset --soft HEAD~1

# 已经 commit 了，连改动一起彻底丢弃（危险）
git reset --hard HEAD~1

# 只是想看看某个历史版本的文件长什么样，别乱改
git show 提交ID:文件名
```

> `--hard` 会真的删掉你的改动，用之前先 `git stash` 或确认已推送。

---

## 六、分支：并行开发的核心

```bash
git branch                  # 列出本地分支（* 号是当前分支）
git branch -r               # 列出远程分支
git branch -a               # 全部
git switch -c feature/xxx   # 新建并切换（推荐）
git switch master           # 切回主线
git switch -                # 切回上一个分支
git merge feature/xxx       # 站在当前分支上，把 xxx 合并进来
git branch -d feature/xxx   # 合并完删掉分支
```

**关键实验（亲测有效）**：在 `feature/note` 里加一行 → 切回 `master` → 那行消失了。
分支不是"复制文件夹"，而是**切换整套文件的状态**。功能并没有丢，它在另一条时间线上。

### 合并的两种结果

- `Fast-forward`：主线没动过，直接把指针快进过去，无冲突
- `Merge made by the 'ort' strategy`：双方都改过，Git 自动生成合并提交
- `CONFLICT`：同一行被两边改过。手动编辑冲突文件，删掉 `<<<<<<<` `=======` `>>>>>>>` 标记，然后 `git add` + `git commit`

---

## 七、远程仓库管理

```bash
git remote -v                          # 看当前远程地址（默认别名 origin）
git remote add origin <url>            # 给本地仓库关联远程
git remote set-url origin <新url>      # 改地址（比如 HTTPS 换 SSH）
git remote remove origin               # 解除关联
git remote show origin                 # 远程仓库全貌（分支追踪关系）
```

### fetch 和 pull 的区别（重要）

| 命令 | 做了什么 | 风险 |
|---|---|---|
| `git fetch` | **只下载**远程更新到本地缓存，不动你的文件和分支 | 零风险，可随时敲 |
| `git pull` | fetch + **自动合并**进当前分支 | 可能有冲突 |

> 习惯做法：先 `git fetch` 看一眼有没有更新，再决定 `git pull`。
> `git pull` 前确保工作区干净（`git status` 没有未提交改动），否则容易乱。

### 推送

```bash
git push                                    # 推当前分支到 origin 同名分支
git push -u origin master                   # 首次推送，-u 建立追踪关系，以后直接 git push
git push -u origin feature/xxx              # 推新分支上去
```

---

## 八、认证：两条路

### 路线 A：SSH（推荐，一次配好永久免密）

本机公钥（**可公开**）：
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJfE8JtsF+l6JN6iKPtBH9PanYbhzkqsHJ/LWHFL9JPL 3413866140@qq.com
```

私钥在 `~/.ssh/id_ed25519`，**绝不外传、绝不提交进仓库**。

配置步骤：
1. 打开 https://github.com/settings/keys
2. 点 **New SSH key**
3. Title 填 `Dust-Windows`（随便起）
4. Key type 选 **Authentication Key**
5. 粘贴上面整行公钥
6. 点 **Add SSH key**
7. 回来验证：`ssh -T git@github.com`
   成功会显示 `Hi 用户名! You've successfully authenticated...`

克隆/推送时用 SSH 地址：
```bash
git clone git@github.com:用户名/仓库名.git
git remote set-url origin git@github.com:用户名/仓库名.git   # 已有仓库切过去
```

> **本机已于 2026-11-01 配置成功**，`ssh -T git@github.com` 返回
> `Hi Miller3413866140! You've successfully authenticated...`。
>
> 若以后又报 `Permission denied (publickey)`，按这个顺序排查：
> 1. `ssh -vT git@github.com | grep -i offering` —— 看 SSH 提交的是哪把钥匙
> 2. `ssh-keygen -lf ~/.ssh/id_ed25519.pub` —— 拿到本地指纹
> 3. 打开 https://github.com/settings/keys —— 对比页面上的指纹是否一致
> 4. 不一致 = key 没加上；一致但仍失败 = Key type 可能选成了 Signing Key（要选 Authentication Key）
>
> ⚠️ 加 key 时**别把公钥粘进 GitHub 搜索框**（会返回 404 / 0 结果），
> 必须走 Settings → SSH and GPG keys → New SSH key。

### 路线 B：HTTPS + Personal Access Token

GitHub 早就不支持密码推送了，必须用 Token：
1. https://github.com/settings/tokens → **Generate new token (classic)**
2. 勾选 `repo` 权限
3. 生成后**立刻复制**（只显示一次）
4. 推送时用户名填 GitHub 用户名，密码位置粘 Token

想免输入：
```bash
git config --global credential.helper manager
```

---

## 九、本机特有的坑与对策

| 现象 | 原因 | 对策 |
|---|---|---|
| `warning: LF will be replaced by CRLF` | Windows 换行符差异 | 无害。想彻底关掉：`git config --global core.autocrlf false` |
| `Permission denied (publickey)` | 公钥没登记到 GitHub | 见上方路线 A |
| `expected flush after ref listing` | HTTPS 传输被中途打断 | 改用 SSH 地址。本机 443 被 RST，重试意义不大 |
| 浏览器 `ERR_CONNECTION_RESET` | 443 直连被阻断（网页打不开） | 装 Watt Toolkit 加速，或手机流量应急加 key |
| `fatal: destination path already exists` | 目录已存在 | 换个名字，或先删掉旧目录 |
| `rejected: non-fast-forward` | 远程有你本地没有的提交 | 先 `git pull` 再 `git push` |
| 推送别人的仓库 403 | 你不是协作者，没写权限 | 先 **Fork** 到自己账号，再推到自己的 fork |

---

## 十、常用配置（一次性）

```bash
git config --global user.name "Dust"
git config --global user.email "3413866140@qq.com"
git config --global init.defaultBranch main    # 新仓库默认用 main
git config --global core.autocrlf false        # 关掉换行符警告
git config --global core.quotepath false       # 中文文件名正常显示
git config --global --list                     # 查看所有配置
```

### `.gitignore`（必须会）

在仓库根目录建 `.gitignore`，写进去的文件 Git 永远不管：

```gitignore
node_modules/
dist/
.env
*.log
.DS_Store
```

> 已经提交过的文件再加进 `.gitignore` 是**没用的**，要先：
> `git rm --cached 文件名` 再提交。

---

## 十一、从零建自己的仓库

```bash
mkdir my-project && cd my-project
git init                          # 初始化，生成 .git
git add .
git commit -m "chore: 初始化项目"
# 然后到 GitHub 网页建一个空仓库，拿到地址
git remote add origin git@github.com:用户名/my-project.git
git push -u origin main
```

> **建空仓库时不要勾选** README / .gitignore / license，
> 否则远程会先有一条提交，你本地推上去就会报 `non-fast-forward`。

---

## 十二、这次的练习成果

本地目录：`C:\Users\Dust\WorkBuddy\2026-11-01-15-44-23\git-practice\`

```
git-practice/            ← 自己的仓库，已推送到 GitHub
├── .gitignore
├── README.md
├── Git速查表.md
└── demo-hello/          ← 克隆别人的练习仓库（已被 .gitignore 排除）
    └── README
```

远程地址：`git@github.com:Miller3413866140/git-practice.git`
网页：https://github.com/Miller3413866140/git-practice

### 已完成的操作

| # | 操作 | 结果 |
|---|---|---|
| 1 | `git clone` 克隆 octocat/Hello-World | 拿到全部 3 条历史 |
| 2 | 改文件 → `status` → `diff` → `add` → `commit` | 提交 `0718356` |
| 3 | 乱写一行 → `git restore` | 干净回退 |
| 4 | `switch -c feature/note` → 提交 → 切回 master | 文件自动还原，功能没丢 |
| 5 | `merge feature/note` | `Fast-forward` 合并成功 |
| 6 | `remote add origin`（SSH 地址） | 关联成功 |
| 7 | `push -u origin main` | **首次推送成功** |
| 8 | 改 README → `commit` → `push` | **增量推送 `0718356..3c6a4fb`** |

### 关键教训

- **`git clone https://...` 在本机会失败**（443 被 RST）。克隆也要用 SSH 地址：
  `git clone git@github.com:用户名/仓库名.git`
- 建 GitHub 仓库时**三个勾选项全部别勾**，否则远程先有一条提交，本地推上去会 `non-fast-forward`
- 别人的仓库 push 必然 403，要贡献代码得先 **Fork**
