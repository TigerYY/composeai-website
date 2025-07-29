# 构图大师 ComposeAI 官方网站

这是构图大师ComposeAI应用的官方网站，采用现代化的响应式设计，支持中英文双语切换。

## 🌟 网站特色

### 设计特点
- **现代化设计**：采用渐变背景、毛玻璃效果和流畅动画
- **响应式布局**：完美适配桌面、平板和手机设备
- **双语支持**：中英文无缝切换，本地存储语言偏好
- **交互动效**：丰富的CSS动画和JavaScript交互效果

### 功能亮点
- **智能导航**：固定导航栏，滚动时背景透明度变化
- **手机模拟器**：动态展示应用界面，包含AI分析动画
- **平滑滚动**：锚点导航平滑滚动效果
- **性能优化**：事件节流、懒加载等性能优化措施

## 📁 文件结构

```
composeai-website/
├── index.html          # 主页
├── privacy.html        # 隐私政策页面
├── terms.html         # 服务条款页面
├── styles.css         # 主样式文件
├── script.js          # JavaScript功能文件
└── README.md          # 说明文档
```

## 🎨 页面结构

### 主页 (index.html)
1. **导航栏**：品牌Logo、导航链接、语言切换
2. **英雄区域**：主标题、副标题、CTA按钮、手机模拟器
3. **功能介绍**：6个核心功能卡片展示
4. **工作原理**：3步使用流程说明
5. **下载区域**：App Store下载链接和应用信息
6. **联系我们**：联系方式和相关链接
7. **页脚**：版权信息

### 隐私政策页面 (privacy.html)
- 详细的隐私保护说明
- 数据收集和使用政策
- 用户权利说明
- 中英文双语支持

### 服务条款页面 (terms.html)
- 完整的服务条款
- 用户责任和权利
- 知识产权说明
- 中英文双语支持

## 🛠 技术实现

### CSS特性
- **CSS Grid & Flexbox**：现代布局技术
- **CSS变量**：主题色彩管理
- **媒体查询**：响应式断点设计
- **CSS动画**：关键帧动画和过渡效果
- **渐变背景**：多层次视觉效果

### JavaScript功能
- **语言切换**：动态内容替换和本地存储
- **移动端导航**：汉堡菜单交互
- **滚动动画**：Intersection Observer API
- **平滑滚动**：锚点导航优化
- **性能优化**：事件节流和防抖

### 响应式设计
- **桌面端**：1200px+ 双栏布局
- **平板端**：768px-1199px 自适应布局
- **手机端**：<768px 单栏堆叠布局

## 🚀 部署说明

### 本地预览
1. 直接在浏览器中打开 `index.html`
2. 或使用本地服务器（推荐）：
   ```bash
   # 使用Python
   python3 -m http.server 8000
   
   # 使用Node.js
   npx serve .
   
   # 使用PHP
   php -S localhost:8000
   ```

### 生产部署
网站为纯静态文件，可部署到任何静态托管服务：
- **GitHub Pages**
- **Netlify**
- **Vercel**
- **阿里云OSS**
- **腾讯云COS**
- **传统Web服务器**

### 域名配置
建议使用以下域名结构：
- 主域名：`composeai.app`
- 中文域名：`构图大师.com`
- 备用域名：`composeai.cn`

## 📱 移动端优化

### 触摸优化
- 按钮最小点击区域44px
- 适当的间距和边距
- 防止意外点击

### 性能优化
- 图片懒加载
- CSS和JS压缩
- 字体优化加载
- 减少HTTP请求

### 用户体验
- 快速加载动画
- 触觉反馈效果
- 易于导航的界面
- 清晰的视觉层次

## 🔧 自定义配置

### 颜色主题
在 `styles.css` 中修改CSS变量：
```css
:root {
  --primary-color: #2563eb;
  --secondary-color: #64748b;
  --accent-color: #667eea;
}
```

### 内容更新
- 修改 `index.html` 中的 `data-zh` 和 `data-en` 属性
- 更新联系邮箱和下载链接
- 替换应用截图和图标

### 添加新页面
1. 创建新的HTML文件
2. 引入相同的CSS和JS文件
3. 添加导航链接
4. 实现语言切换功能

## 📊 SEO优化

### 已实现的SEO特性
- 语义化HTML结构
- Meta标签优化
- 结构化数据标记
- 图片Alt属性
- 页面标题和描述
- 友好的URL结构

### 建议的SEO改进
- 添加网站地图(sitemap.xml)
- 配置robots.txt
- 实现Open Graph标签
- 添加JSON-LD结构化数据
- 配置Google Analytics
- 设置Google Search Console

## 🔒 安全考虑

- 所有外部链接使用 `target="_blank" rel="noopener"`
- 表单输入验证和清理
- HTTPS强制重定向
- 内容安全策略(CSP)头部
- 防止XSS攻击

## 📈 性能监控

建议集成以下工具：
- **Google Analytics**：用户行为分析
- **Google PageSpeed Insights**：性能监控
- **GTmetrix**：加载速度测试
- **Lighthouse**：综合性能评估

## 🤝 维护和更新

### 定期维护任务
- 检查外部链接有效性
- 更新应用下载链接
- 优化图片和资源
- 测试跨浏览器兼容性
- 更新内容和功能介绍

### 版本控制
建议使用Git进行版本控制：
```bash
git init
git add .
git commit -m "Initial website setup"
```

## 📞 技术支持

如有技术问题或改进建议，请联系：
- 邮箱：support@composeai.app
- 项目仓库：[GitHub链接]

---

**构图大师ComposeAI** - 用AI重新定义摄影构图

© 2024 ComposeAI. All rights reserved.

##同步网站：
rsync -avz --delete --rsync-path="sudo rsync" ./ yy@composeai.net:/var/www/composeai/
