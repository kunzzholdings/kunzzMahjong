# 📁 项目结构说明

## 完整目录树

```
mahjong-game/
├── 📄 server.js              # Node.js 后端服务器 (核心)
├── 📄 package.json           # 项目配置和依赖
├── 📄 README.md              # 完整说明文档
├── 📄 QUICKSTART.md          # 快速启动指南
├── 📄 SOCKET_EVENTS.md       # Socket 事件详细说明
├── 📄 PROJECT_STRUCTURE.md   # 本文档
│
└── 📂 public/                # 前端静态文件目录
    ├── 📄 index.html         # 游戏主界面 (HTML)
    ├── 📄 style.css          # 样式文件 (CSS)
    ├── 📄 client.js          # 前端逻辑 (JavaScript)
    │
    └── 📂 assets/            # 资源文件目录 (可选)
        └── 📄 README.md      # 资源使用说明
```

---

## 📄 文件详细说明

### 后端核心文件

#### `server.js` (580+ 行)
**功能**：Node.js + Express + Socket.IO 服务器

**主要内容**：
```javascript
// 1. 依赖导入
const express = require('express');
const socketIo = require('socket.io');

// 2. 麻将牌定义
const TILES = { WAN, TIAO, TONG, HONOR }

// 3. 房间类 (Room)
class Room {
  constructor()         // 初始化房间
  addPlayer()          // 添加玩家
  startGame()          // 开始游戏
  drawTile()           // 摸牌
  discardTile()        // 出牌
  canPong/Chow/Kong()  // 判断可否操作
  performPong/Chow/Kong() // 执行操作
  checkWin()           // 判断胡牌
  calculateFan()       // 计算番型
}

// 4. Socket.IO 事件处理
io.on('connection', (socket) => {
  socket.on('create_room')
  socket.on('join_room')
  socket.on('start_game')
  socket.on('draw_tile')
  socket.on('play_tile')
  socket.on('claim_pong/chow/kong')
  socket.on('declare_win')
  socket.on('pass')
  socket.on('disconnect')
})

// 5. 服务器启动
server.listen(PORT)
```

**核心算法**：
- 洗牌算法：Fisher-Yates
- 胡牌判断：递归回溯
- 番型计算：规则匹配

**数据结构**：
```javascript
Room {
  roomId: String,
  players: Array<Player>,
  deck: Array<String>,
  wall: Array<String>,
  currentPlayerIndex: Number,
  gameStarted: Boolean,
  lastDiscard: Object
}

Player {
  id: String,
  name: String,
  hand: Array<String>,
  discarded: Array<String>,
  melds: Array<Meld>,
  score: Number
}

Meld {
  type: 'pong' | 'chow' | 'kong',
  tiles: Array<String>
}
```

---

#### `package.json`
**功能**：项目配置文件

**关键依赖**：
```json
{
  "dependencies": {
    "express": "^4.18.2",    // Web 服务器
    "socket.io": "^4.6.1"    // WebSocket 通信
  },
  "devDependencies": {
    "nodemon": "^3.0.1"      // 开发热重载
  },
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

---

### 前端文件

#### `public/index.html` (200+ 行)
**功能**：游戏界面结构

**主要区块**：
```html
<!-- 1. 登录界面 -->
<div id="login-screen">
  - 游戏标题
  - 昵称输入
  - 创建/加入房间按钮
  - 游戏说明
</div>

<!-- 2. 等待界面 -->
<div id="waiting-screen">
  - 房间号显示
  - 玩家槽位 (4个)
  - 开始游戏按钮
</div>

<!-- 3. 游戏界面 -->
<div id="game-screen">
  - 游戏信息栏
  - 对手区域 (上、左、右)
  - 弃牌池
  - 玩家手牌区
  - 操作按钮
</div>

<!-- 4. 结算弹窗 -->
<div id="game-over-modal">
  - 胜利信息
  - 番型显示
  - 手牌展示
</div>

<!-- 5. 提示消息 -->
<div id="toast"></div>
```

**引入资源**：
```html
<link rel="stylesheet" href="style.css">
<script src="/socket.io/socket.io.js"></script>
<script src="client.js"></script>
```

---

#### `public/style.css` (800+ 行)
**功能**：深色主题样式

**设计系统**：
```css
:root {
  --bg-dark: #0a0a0a;        /* 主背景 */
  --bg-card: #252525;        /* 卡片背景 */
  --gold: #ffd700;           /* 金色强调 */
  --text-primary: #ffffff;   /* 主文字 */
}
```

**核心样式**：
- **布局**：Flexbox + Grid
- **动画**：
  - `fadeIn` - 淡入
  - `tileAppear` - 牌出现
  - `pulse` - 脉冲高亮
  - `winPulse` - 胡牌闪烁
- **响应式**：
  - 桌面：完整4人界面
  - 移动：简化为单列布局

**组件样式**：
```
.screen          → 页面容器
.tile            → 麻将牌
.btn             → 按钮
.modal           → 弹窗
.toast           → 提示
.opponent        → 对手区
.player-area     → 玩家区
```

---

#### `public/client.js` (600+ 行)
**功能**：前端逻辑和 Socket 通信

**全局状态**：
```javascript
gameState = {
  roomId: String,
  playerName: String,
  playerId: String,
  playerIndex: Number,
  hand: Array<String>,
  currentPlayerIndex: Number,
  players: Array,
  canClaim: Object,
  selectedTile: String
}
```

**核心函数**：
```javascript
// UI 工具
showScreen()           // 切换界面
showToast()            // 显示提示
generateRoomId()       // 生成房间号

// 渲染函数
renderHand()           // 渲染手牌
renderMelds()          // 渲染碰杠
createTileElement()    // 创建麻将牌元素
updateGameState()      // 更新游戏状态
updateOpponentDisplay() // 更新对手显示

// 交互函数
onTileClick()          // 点击手牌

// Socket 监听
socket.on('connect')
socket.on('room_created')
socket.on('game_started')
socket.on('tile_drawn')
socket.on('tile_played')
socket.on('can_claim')
socket.on('game_over')
... 共15个事件
```

**麻将牌映射**：
```javascript
TILE_DISPLAY = {
  '1w': '一萬', '2w': '二萬', ...
  '1t': '一条', '2t': '二条', ...
  '1b': '一筒', '2b': '二筒', ...
  'dong': '东', 'nan': '南', ...
}
```

---

### 文档文件

#### `README.md` (400+ 行)
**内容**：
1. 功能特色
2. 快速开始
3. 游戏玩法
4. 项目结构
5. Socket 事件说明
6. 线上部署指南
7. 开发说明
8. 未来扩展

#### `QUICKSTART.md` (200+ 行)
**内容**：
1. 一键启动 (3步)
2. 本地测试方法
3. 多设备联机
4. 常见问题解决
5. 游戏操作速查
6. 开发提示

#### `SOCKET_EVENTS.md` (500+ 行)
**内容**：
1. 客户端事件 (10个)
2. 服务器事件 (15个)
3. 数据结构详解
4. 典型流程示例
5. 错误处理
6. 最佳实践

#### `PROJECT_STRUCTURE.md` (本文档)
**内容**：
1. 完整目录树
2. 文件详细说明
3. 代码统计
4. 技术栈分析
5. 扩展指南

---

## 📊 代码统计

| 文件 | 行数 | 类型 | 说明 |
|------|------|------|------|
| `server.js` | ~580 | JavaScript | 后端核心 |
| `client.js` | ~600 | JavaScript | 前端逻辑 |
| `index.html` | ~200 | HTML | 界面结构 |
| `style.css` | ~800 | CSS | 样式设计 |
| `README.md` | ~400 | Markdown | 主文档 |
| `QUICKSTART.md` | ~200 | Markdown | 快速指南 |
| `SOCKET_EVENTS.md` | ~500 | Markdown | 事件文档 |
| **总计** | **~3,280** | - | - |

---

## 🏗️ 技术架构

### 后端架构
```
┌─────────────────────────────────────┐
│         Express Server              │
│  (静态文件服务 + API 路由)          │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│         Socket.IO Server            │
│  (WebSocket 实时通信)               │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│         Room Management             │
│  (房间管理 + 游戏逻辑)              │
└─────────────────────────────────────┘
```

### 前端架构
```
┌─────────────────────────────────────┐
│            HTML DOM                 │
│  (界面结构 + 元素)                  │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│          CSS Styling                │
│  (深色主题 + 响应式)                │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│       JavaScript Client             │
│  (状态管理 + Socket 通信)           │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│      Socket.IO Client               │
│  (WebSocket 连接)                   │
└─────────────────────────────────────┘
```

### 数据流向
```
用户操作
    ↓
前端 UI 事件
    ↓
Socket.IO Client 发送
    ↓
WebSocket 传输
    ↓
Socket.IO Server 接收
    ↓
Room 类处理逻辑
    ↓
验证 + 更新状态
    ↓
Socket.IO Server 广播
    ↓
WebSocket 传输
    ↓
Socket.IO Client 接收
    ↓
更新前端状态
    ↓
重新渲染 UI
```

---

## 🔧 扩展指南

### 添加新功能的步骤

#### 1. 后端扩展 (server.js)
```javascript
// 添加新的 Socket 事件
socket.on('new_feature', (data) => {
  // 1. 验证数据
  // 2. 处理逻辑
  // 3. 更新状态
  // 4. 广播结果
});

// 添加新的 Room 方法
class Room {
  newFeature() {
    // 实现新功能逻辑
  }
}
```

#### 2. 前端扩展 (client.js)
```javascript
// 添加新的事件监听
socket.on('new_feature_result', (data) => {
  // 更新 UI
});

// 添加新的 UI 函数
function handleNewFeature() {
  // 发送事件
  socket.emit('new_feature', { ... });
}
```

#### 3. 界面扩展 (index.html + style.css)
```html
<!-- 添加新的 UI 元素 -->
<div class="new-feature">
  <!-- 内容 -->
</div>
```

```css
/* 添加新的样式 */
.new-feature {
  /* 样式规则 */
}
```

---

### 示例：添加聊天功能

#### 1. 后端 (server.js)
```javascript
socket.on('send_message', (data) => {
  const { roomId, message } = data;
  io.to(roomId).emit('new_message', {
    playerId: socket.id,
    playerName: getPlayerName(socket.id),
    message,
    timestamp: Date.now()
  });
});
```

#### 2. 前端 (client.js)
```javascript
socket.on('new_message', (data) => {
  appendMessage(data);
});

function sendMessage() {
  const message = chatInput.value;
  socket.emit('send_message', {
    roomId: gameState.roomId,
    message
  });
}
```

#### 3. 界面 (index.html)
```html
<div class="chat-panel">
  <div class="messages" id="messages"></div>
  <input type="text" id="chat-input" />
  <button onclick="sendMessage()">发送</button>
</div>
```

---

## 🎯 开发建议

### 1. 代码规范
- 使用 ESLint 检查代码
- 保持函数单一职责
- 添加注释说明复杂逻辑
- 统一命名规范

### 2. 性能优化
- 避免频繁 DOM 操作
- 使用事件委托
- 缓存常用 DOM 元素
- 优化动画性能

### 3. 安全考虑
- 验证所有用户输入
- 防止 XSS 攻击
- 限制房间数量
- 添加操作频率限制

### 4. 测试建议
- 单元测试：核心算法
- 集成测试：Socket 事件
- 压力测试：多房间并发
- 兼容性测试：不同浏览器

---

## 📚 学习资源

### 相关技术文档
- [Node.js 官方文档](https://nodejs.org/docs/)
- [Express.js 文档](https://expressjs.com/)
- [Socket.IO 文档](https://socket.io/docs/)
- [MDN Web 文档](https://developer.mozilla.org/)

### 麻将规则
- 马来西亚麻将规则
- 番型计算方法
- 胡牌判定算法

---

**完整的项目结构说明，帮助你理解和扩展这个游戏！📁✨**