# 多人在线实时拼图游戏

一个基于 WebSocket 的多人在线实时拼图游戏，支持多人同时竞技。

## 功能特点

- 管理员上传图片，生成二维码供玩家扫码加入
- 玩家自动生成随机唯一名称
- 九宫格旋转拼图（3x3），点击小块顺时针旋转90度
- 实时计时和步数统计
- 实时排行榜，金银铜牌显示
- 完成后有庆祝动画效果

## 技术栈

- 后端：Node.js + Express + WebSocket
- 前端：原生 HTML + CSS + JavaScript
- 其他：QRCode.js（生成二维码）、Multer（文件上传）

## 项目结构

```
puzzle0128/
├── server/
│   └── server.js          # 后端服务器
├── admin/
│   └── admin.html         # 管理员端界面
├── public/
│   └── player.html        # 玩家端界面
├── uploads/               # 上传的图片存储目录
├── package.json           # 项目配置
└── README.md              # 说明文档
```

## 本地开发部署

### 前置要求

- Node.js 14+ 安装在系统上

### 安装步骤

1. 进入项目目录
```bash
cd E:/Works/AI_project/puzzle0128
```

2. 安装依赖
```bash
npm install
```

3. 启动服务器
```bash
npm start
```

4. 访问应用
- 管理员端：http://localhost:3000/admin.html
- 玩家端：扫描管理员端显示的二维码

### 开发模式（自动重启）

```bash
npm run dev
```

## 生产环境部署

### 方式一：直接使用 PM2 部署

1. 全局安装 PM2
```bash
npm install -g pm2
```

2. 启动应用
```bash
cd E:/Works/AI_project/puzzle0128
pm2 start server/server.js --name puzzle-game
```

3. 设置开机自启
```bash
pm2 startup
pm2 save
```

4. 常用命令
```bash
pm2 status          # 查看状态
pm2 logs puzzle-game    # 查看日志
pm2 restart puzzle-game # 重启
pm2 stop puzzle-game    # 停止
pm2 delete puzzle-game  # 删除
```

### 方式二：使用 Nginx 反向代理

1. 安装 Nginx

2. 配置 Nginx（创建配置文件 `/etc/nginx/sites-available/puzzle-game`）
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

3. 启用配置
```bash
sudo ln -s /etc/nginx/sites-available/puzzle-game /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 方式三：Docker 部署

1. 创建 Dockerfile
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --production

COPY . .

EXPOSE 3000

CMD ["node", "server/server.js"]
```

2. 构建镜像
```bash
docker build -t puzzle-game .
```

3. 运行容器
```bash
docker run -d -p 3000:3000 --name puzzle-game puzzle-game
```

### 方式四：云平台部署（推荐，跨网络访问）

#### 部署到 Railway（推荐，免费额度）

Railway 是一个易用的云平台，提供免费额度，完美支持 WebSocket。

**步骤一：准备代码仓库**

1. 将项目推送到 GitHub
```bash
git init
git add .
git commit -m "Initial commit"
# 在 GitHub 创建新仓库后执行
git remote add origin https://github.com/你的用户名/puzzle0128.git
git push -u origin main
```

**步骤二：部署到 Railway**

1. 访问 [railway.app](https://railway.app/)
2. 使用 GitHub 账号登录
3. 点击 "New Project" → "Deploy from GitHub repo"
4. 选择你的 `puzzle0128` 仓库
5. Railway 会自动检测 Node.js 项目并部署

**步骤三：配置环境变量**

在 Railway 项目设置中添加环境变量：
- `NODE_ENV` = `production`

部署完成后，Railway 会自动分配一个公网域名，如 `https://your-app.railway.app`

**步骤四：获取公网地址**

获取部署后的域名，在 Railway 设置中添加环境变量：
- `PUBLIC_URL` = `https://your-app.railway.app`（你的 Railway 域名）

**访问应用：**
- 管理员端：`https://your-app.railway.app/admin.html`
- 玩家端：扫描二维码即可

**免费额度说明：**
- 每月 $5 免费额度
- 512MB RAM
- 支持自定义域名

---

#### 部署到 Render（备选）

1. 访问 [render.com](https://render.com/)
2. 使用 GitHub 账号登录
3. 点击 "New" → "Web Service"
4. 连接你的 GitHub 仓库
5. 配置：
   - Name: `puzzle-game`
   - Runtime: `Node`
   - Build Command: `npm install`
   - Start Command: `npm start`
   - 实例类型: `Free`

6. 添加环境变量 `PUBLIC_URL` = `https://your-app.onrender.com`

---

#### 部署到 Heroku（已取消免费额度）

Heroku 不再提供免费额度，仅适合付费使用。

## 环境变量配置

创建 `.env` 文件（可选）：
```env
PORT=3000
NODE_ENV=production
```

修改 `server/server.js` 顶部添加：
```javascript
require('dotenv').config();
const PORT = process.env.PORT || 3000;
```

## 防火墙配置

确保开放 3000 端口：
```bash
# Ubuntu/Debian
sudo ufw allow 3000

# CentOS/RHEL
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```

## 使用说明

### 管理员端操作流程

1. 打开管理员端页面（默认：http://localhost:3000/admin.html）
2. 点击上传区域或拖拽图片上传
3. 等待玩家扫码加入
4. 点击"开始游戏"按钮
5. 查看实时排行榜
6. 游戏结束后可点击"重置游戏"开始新一局

### 玩家端操作流程

1. 用手机扫描管理员端的二维码
2. 自动分配随机名称
3. 等待管理员开始游戏
4. 点击旋转小块，使所有图片块回到正确角度
5. 完成拼图后查看成绩

## 游戏规则

- 拼图为 3x3 九宫格，每个小块被随机旋转（0°/90°/180°/270°）
- 点击小块可将其顺时针旋转90度
- 将所有小块都旋转回正确角度（0度）即可完成
- 用时越少、步数越少，排名越高
- 前三名显示金银铜牌

## 故障排除

### 端口被占用
修改 `server/server.js` 中的端口号：
```javascript
const PORT = process.env.PORT || 3001; // 改为其他端口
```

### WebSocket 连接失败
检查防火墙设置，确保 WebSocket 端口可访问。

### 图片上传失败
检查 `uploads` 目录权限：
```bash
chmod 755 uploads/
```

### 二维码无法加载
确保服务器正常运行，检查网络连接。

## 注意事项

1. 建议使用正方形图片，效果最佳
2. 图片大小建议控制在 2MB 以内
3. 支持的图片格式：JPG、PNG、GIF
4. 游戏过程中请保持网络连接
5. 建议使用 Chrome 或 Safari 浏览器获得最佳体验

## 许可证

MIT License
