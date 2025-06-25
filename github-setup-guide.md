# GitHub仓库创建和代码上传指南

本指南详细介绍如何在GitHub上创建新仓库并上传ComposeAI官网代码。

## 🚀 快速开始

### 前置准备
- GitHub账户
- Git已安装在本地
- 终端/命令行工具

## 📋 步骤一：在GitHub上创建新仓库

### 1. 登录GitHub
访问 [GitHub.com](https://github.com) 并登录您的账户。

### 2. 创建新仓库
1. 点击右上角的 **"+"** 按钮
2. 选择 **"New repository"**
3. 填写仓库信息：
   - **Repository name**: `composeai-website`
   - **Description**: `ComposeAI官方网站 - 构图大师AI应用官方网站`
   - **Visibility**: 选择 `Public`（公开）或 `Private`（私有）
   - **Initialize**: 暂时不要勾选任何选项（我们会手动上传代码）
4. 点击 **"Create repository"**

### 3. 记录仓库信息
创建完成后，GitHub会显示仓库URL，类似：
```
https://github.com/your-username/composeai-website.git
```

## 🔧 步骤二：本地Git配置

### 1. 检查Git安装
```bash
# 检查Git版本
git --version

# 如果未安装，在macOS上可以使用：
# xcode-select --install
```

### 2. 配置Git用户信息（如果未配置）
```bash
# 设置用户名
git config --global user.name "Your Name"

# 设置邮箱
git config --global user.email "your.email@example.com"

# 验证配置
git config --list
```

### 3. 配置SSH密钥（推荐）

#### 生成SSH密钥
```bash
# 生成新的SSH密钥
ssh-keygen -t ed25519 -C "your.email@example.com"

# 按Enter使用默认文件位置
# 设置密码（可选）
```

#### 添加SSH密钥到ssh-agent
```bash
# 启动ssh-agent
eval "$(ssh-agent -s)"

# 添加SSH私钥
ssh-add ~/.ssh/id_ed25519
```

#### 复制公钥到GitHub
```bash
# 复制公钥到剪贴板
pbcopy < ~/.ssh/id_ed25519.pub
```

然后在GitHub上：
1. 点击右上角头像 → **Settings**
2. 左侧菜单选择 **SSH and GPG keys**
3. 点击 **New SSH key**
4. 粘贴公钥内容
5. 点击 **Add SSH key**

#### 测试SSH连接
```bash
ssh -T git@github.com
```

## 📁 步骤三：初始化本地仓库

### 1. 进入网站目录
```bash
cd "/Users/yangyang/Library/Mobile Documents/com~apple~CloudDocs/YYcode/SNAPS/pppai/composeai-website"
```

### 2. 初始化Git仓库
```bash
# 初始化Git仓库
git init

# 设置默认分支名为main
git branch -M main
```

### 3. 添加远程仓库
```bash
# 使用SSH（推荐）
git remote add origin git@github.com:your-username/composeai-website.git

# 或使用HTTPS
# git remote add origin https://github.com/your-username/composeai-website.git

# 验证远程仓库
git remote -v
```

## 📤 步骤四：上传代码到GitHub

### 1. 创建.gitignore文件
```bash
cat > .gitignore << 'EOF'
# macOS
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# 编辑器
.vscode/
.idea/
*.swp
*.swo
*~

# 日志文件
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# 临时文件
*.tmp
*.temp

# 备份文件
*.bak
*.backup

# 缓存目录
node_modules/
.cache/

# 构建输出
dist/
build/
EOF
```

### 2. 添加所有文件
```bash
# 查看当前状态
git status

# 添加所有文件
git add .

# 查看暂存的文件
git status
```

### 3. 创建首次提交
```bash
# 创建提交
git commit -m "🎉 Initial commit: ComposeAI官网首次上传

- 添加响应式官网设计
- 支持中英文双语切换
- 包含隐私政策和服务条款
- 现代化UI设计和动画效果
- 移动端优化"
```

### 4. 推送到GitHub
```bash
# 推送到远程仓库
git push -u origin main
```

## 🔄 步骤五：验证上传结果

### 1. 检查GitHub仓库
访问您的GitHub仓库页面，确认所有文件已成功上传：
- `index.html`
- `styles.css`
- `script.js`
- `privacy.html`
- `terms.html`
- `README.md`
- `deployment-guide.md`
- `.gitignore`

### 2. 启用GitHub Pages（可选）
如果想通过GitHub Pages托管网站：

1. 在仓库页面点击 **Settings**
2. 左侧菜单选择 **Pages**
3. 在 **Source** 下选择 **Deploy from a branch**
4. 选择 **main** 分支
5. 点击 **Save**

几分钟后，网站将在以下地址可访问：
```
https://your-username.github.io/composeai-website
```

## 📝 步骤六：完善仓库信息

### 1. 编辑仓库描述
在GitHub仓库页面：
1. 点击仓库名称下方的 **Edit**（齿轮图标）
2. 添加描述：`ComposeAI官方网站 - 构图大师AI应用官方网站`
3. 添加网站链接（如果启用了GitHub Pages）
4. 添加标签：`website`, `composeai`, `ai`, `photography`, `responsive`
5. 点击 **Save changes**

### 2. 创建Release（可选）
```bash
# 创建标签
git tag -a v1.0.0 -m "ComposeAI官网 v1.0.0 - 首个正式版本"

# 推送标签
git push origin v1.0.0
```

然后在GitHub上：
1. 点击 **Releases**
2. 点击 **Create a new release**
3. 选择刚创建的标签 `v1.0.0`
4. 填写发布说明
5. 点击 **Publish release**

## 🔄 日常更新流程

### 更新网站代码
```bash
# 1. 修改文件后，查看变更
git status
git diff

# 2. 添加变更
git add .

# 3. 提交变更
git commit -m "✨ 功能描述或修复说明"

# 4. 推送到GitHub
git push origin main
```

### 常用Git命令
```bash
# 查看提交历史
git log --oneline

# 查看远程仓库信息
git remote -v

# 拉取最新代码
git pull origin main

# 查看分支
git branch -a

# 创建新分支
git checkout -b feature/new-feature

# 切换分支
git checkout main
```

## 🛠 故障排除

### 常见问题

#### 1. 推送被拒绝
```bash
# 如果远程有更新，先拉取
git pull origin main --rebase

# 然后重新推送
git push origin main
```

#### 2. SSH连接问题
```bash
# 测试SSH连接
ssh -T git@github.com

# 如果失败，检查SSH密钥
ls -la ~/.ssh

# 重新添加密钥
ssh-add ~/.ssh/id_ed25519
```

#### 3. 权限问题
确保您有仓库的写入权限，或者仓库URL正确。

#### 4. 文件过大
GitHub单个文件限制100MB，仓库建议不超过1GB。

### 获取帮助
- [GitHub官方文档](https://docs.github.com)
- [Git官方文档](https://git-scm.com/doc)
- [GitHub社区论坛](https://github.community)

## 📊 仓库管理最佳实践

### 1. 提交信息规范
使用语义化提交信息：
```
🎉 feat: 新功能
🐛 fix: 修复bug
📝 docs: 文档更新
💄 style: 样式调整
♻️ refactor: 代码重构
⚡ perf: 性能优化
✅ test: 测试相关
🔧 chore: 构建/工具相关
```

### 2. 分支管理
- `main`: 主分支，稳定版本
- `develop`: 开发分支
- `feature/*`: 功能分支
- `hotfix/*`: 紧急修复分支

### 3. 安全考虑
- 不要提交敏感信息（密码、API密钥等）
- 使用`.gitignore`忽略不必要的文件
- 定期更新依赖
- 启用分支保护规则

### 4. 协作流程
- 使用Pull Request进行代码审查
- 编写清晰的提交信息
- 保持代码整洁和文档更新
- 使用Issues跟踪问题和功能请求

---

**恭喜！您已成功将ComposeAI官网代码上传到GitHub！**

现在您可以：
- 通过GitHub管理代码版本
- 与团队协作开发
- 使用GitHub Pages托管网站
- 设置自动化部署流程

如需技术支持，请联系：support@composeai.app