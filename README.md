# ST-WBM-Server v1.0

  > SillyTavern 世界书管理器 — 后端插件（Server Plugin）

  ST-WBM-Server 是 [WorldBook Manager](https://github.com/AliceSao/WorldBookManager) 的 SillyTavern 服务端插件，提供世界书 REST API 与双面板 Web 管理界面。

  ---

  ## 功能

  - **REST API**：直接读写 SillyTavern 世界书 JSON（绕过 CSRF，性能更高）
  - **条目管理**：增删查改、搜索（标题/关键字/内容）
  - **批量操作**：策略/位置/深度/Order/概率/关键字/递归控制/效果/启用禁用
  - **跨世界书复制**：将条目从一个世界书复制到另一个（自动重分配 UID）
  - **Web UI**：内置 Vue 3 双面板管理界面（可在浏览器单独访问）
  - **导入/导出**：标准 SillyTavern JSON 格式

  ---

  ## 系统要求

  - SillyTavern（最新版，支持 server plugin 格式）
  - Node.js 18+（SillyTavern 已内置）

  ---

  ## 安装

  ### 方式 A：直接使用编译版（推荐）

  ```bash
  # 将本仓库克隆到 SillyTavern 的 plugins 目录
  cd /path/to/SillyTavern/plugins
  git clone https://github.com/AliceSao/ST-WBM-Server.git wb-manager
  ```

  > **注意**：目录名必须为 `wb-manager`（插件路由前缀 `/api/plugins/wb-manager/`）

  无需执行 `npm install`，`dist/` 中已包含编译后的 JavaScript。

  ### 方式 B：从源码构建

  ```bash
  git clone https://github.com/AliceSao/ST-WBM-Server.git wb-manager
  cd wb-manager
  npm install
  npm run build   # 输出到 dist/
  ```

  ---

  ## 验证安装

  启动 SillyTavern 后访问：

  ```
  GET http://localhost:8000/api/plugins/wb-manager/ping
  ```

  返回 `{"success":true,"data":{"version":"1.0.0"}}` 则安装成功。

  ---

  ## Web 管理界面

  浏览器访问：

  ```
  http://localhost:8000/api/plugins/wb-manager/ui/
  ```

  或通过 [ST-WBM-UI](https://github.com/AliceSao/ST-WBM-UI) 扩展在 SillyTavern 内嵌打开。

  ---

  ## REST API 概览

  Base URL：`/api/plugins/wb-manager`

  ### 健康检查
  ```
  GET /ping
  ```

  ### 世界书
  | 方法 | 路径 | 说明 |
  |------|------|------|
  | GET | /worldbooks | 列出所有世界书 |
  | GET | /worldbooks/:name | 获取世界书条目（数组，按 uid 升序） |
  | POST | /worldbooks | 创建世界书 |
  | PUT | /worldbooks/:name | 覆盖保存世界书 |
  | DELETE | /worldbooks/:name | 删除世界书 |
  | GET | /worldbooks/:name/export | 下载世界书 JSON 文件 |
  | POST | /worldbooks/:name/import | 从标准 ST JSON 导入 |

  ### 条目
  | 方法 | 路径 | 说明 |
  |------|------|------|
  | GET | /worldbooks/:name/entries?q=<词> | 搜索条目 |
  | POST | /worldbooks/:name/entries | 批量添加条目 |
  | PUT | /worldbooks/:name/entries/:uid | 更新单条目 |
  | DELETE | /worldbooks/:name/entries | 批量删除条目 |

  ### 批量操作
  ```
  POST /worldbooks/:name/batch/strategy      # 激活策略
  POST /worldbooks/:name/batch/position      # 插入位置
  POST /worldbooks/:name/batch/depth         # 深度
  POST /worldbooks/:name/batch/order         # Order
  POST /worldbooks/:name/batch/probability   # 触发概率
  POST /worldbooks/:name/batch/name          # 条目标题
  POST /worldbooks/:name/batch/keys/set      # 替换关键字
  POST /worldbooks/:name/batch/keys/add      # 添加关键字
  POST /worldbooks/:name/batch/keys/clear    # 清空关键字
  POST /worldbooks/:name/batch/recursion     # 递归控制
  POST /worldbooks/:name/batch/effect        # 粘性/冷却/延迟效果
  POST /worldbooks/:name/batch/enabled       # 启用/禁用
  POST /worldbooks/:name/batch/group-weight  # 组权重
  POST /worldbooks/:name/batch/char-filter   # 角色绑定

  POST /worldbooks/:name/copy                # 跨世界书复制条目
  ```

  所有请求支持 `?user=<用户名>` 参数（默认 `default-user`）。

  ### 通用响应格式
  ```json
  { "success": true,  "message": "描述", "data": { ... } }
  { "success": false, "message": "错误描述", "data": null }
  ```

  ---

  ## 数据路径

  世界书文件读写路径：

  ```
  {ST根目录}/data/{用户名}/worlds/{世界书名}.json
  ```

  插件运行时以 SillyTavern 根目录（`process.cwd()`）为起点自动推导，无需配置。

  ---

  ## 目录结构

  ```
  wb-manager/
  ├── src/                    ← TypeScript 源代码
  │   ├── index.ts            ← 插件入口（SillyTavern plugin 格式）
  │   ├── routes/
  │   │   ├── worldbook.ts    ← 世界书 CRUD 路由
  │   │   ├── entry.ts        ← 条目管理路由
  │   │   ├── batch.ts        ← 批量操作路由
  │   │   └── web.ts          ← Web UI 静态服务路由
  │   └── services/
  │       ├── worldbook.ts    ← 文件读写服务
  │       ├── entry.ts        ← 条目操作服务
  │       └── batch.ts        ← 批量操作服务
  ├── dist/                   ← 编译后的 JavaScript（直接可用）
  ├── web/
  │   └── dist/               ← 编译后的 Vue 3 Web UI
  ├── package.json
  └── tsconfig.json
  ```

  ---

  ## 配合使用

  - **前端扩展**：[ST-WBM-UI](https://github.com/AliceSao/ST-WBM-UI)  
    SillyTavern 扩展，提供内嵌面板 + 23 条斜杠命令
  - **主仓库**：[WorldBookManager](https://github.com/AliceSao/WorldBookManager)  
    包含 Python CLI 工具与完整文档

  ---

  ## 作者

  AliceSao · MIT License
  