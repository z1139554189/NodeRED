# 项目 MEMORY.md — OPC-UA-Bridge + Node-RED Dashboard

## 成功节点 ✅

| # | 节点 | 说明 |
|---|------|------|
| 1 | Node-RED v4.1.10 安装 | NSSM 注册为 Windows 永久服务（开机自启），admin/Admin_00 |
| 2 | Admin API 认证 | settings.js adminAuth + bcryptjs 密码哈希 |
| 3 | HTTP 端点方案 | `http in` + `file in` + `http response` 独立看板，绕开 Dashboard 2.0 ui-template bug |
| 4 | API Proxy 架构 | 5 个代理端点转发到 Bridge 8000：health / batch-read / query / export / cache-stats |
| 5 | httpNodeAuth:false | HTTP In 公开访问 + 编辑器 adminAuth 保护，两不误 |
| 6 | 中文节点名修复 | UTF-8 编码显式指定，解决 PowerShell 部署乱码 |
| 7 | Excel 采样逻辑修复 | 从"桶内最后一条"改为"离桶起始最近的值" |
| 8 | FIQ 单位显示 | Excel/卡片/tooltip 三处统一加 `kg` |
| 9 | Dashboard 断开修复 | fetch 路径从 Bridge API → Proxy API，flows.json 去掉 function 节点 |
| 10 | Node-RED v4 兼容 | http request URL 直写节点配置，不通过 msg 传入 |
| 11 | Dashboard 断开修复（第二次） | dashboard.html 更新后 fetch 路径回退到 Bridge API，全部改回 PROXY 路径 |
| 12 | GitHub 推送 | 清理无用文件（.config.*、.sessions.json），推送到 z1139554189/Node-RED- |

## API Proxy 架构

| 前端请求 | 转发到 Bridge |
|---|---|
| GET /api/proxy/health | GET 127.0.0.1:8000/health |
| GET /api/proxy/cache-stats | GET /api/v1/cache/stats |
| POST /api/proxy/batch-read | POST /api/v1/nodes/batch-read |
| POST /api/proxy/query | POST /api/v1/history/query |
| POST /api/proxy/export | POST /api/v1/history/export |

## 关键文件

| 文件 | 用途 |
|------|------|
| `C:\Users\Administrator\.node-red\dashboard.html` | 看板 HTML（含 Chart.js） |
| `C:\Users\Administrator\.node-red\flows.json` | Node-RED 部署的 flow（源文件 opcua-dashboard-flows.json） |
| `C:\Users\Administrator\.node-red\settings.js` | adminAuth + httpNodeAuth:false |
| `opcua_api_bridge/src/api/main.py` | Bridge API（采样逻辑、Excel 导出） |

## 经验教训

| # | 教训 | 场景 |
|---|------|------|
| 1 | Dashboard 2.0 ui-template v1.30.2 Windows 有 `constrained.includes` bug | 放弃 ui-template，改用 HTTP 端点方案 |
| 2 | `file in` 节点读 UTF-8 HTML 比 function 内嵌 HTML 更干净 | 看板 HTML 用 file in 节点读取 |
| 3 | `localhost` → Node.js 18+ 优先 IPv6 `::1`，必须用 `127.0.0.1` | 所有代理转发用 127.0.0.1 |
| 4 | `httpNodeAuth: false` 只影响 HTTP In，不影响编辑器 adminAuth | 看板公开访问 + 编辑器受保护 |
| 5 | Node-RED v4：msg.url/method/headers 不能覆盖 http request 节点属性 | URL 直接写在 http request 节点配置中 |
| 6 | Dashboard JS fetch 路径必须和代理路径一致，否则 404 | 改为 `/api/proxy/...` 路径 |
| 7 | PowerShell 部署 JSON 含中文必须 `Get-Content -Encoding UTF8` + `charset=utf-8` | 否则中文变 `??` |
| 8 | 采样取离桶起始最近的值，不是最后一条也不是平均值 | 用户明确的业务需求 |
| 9 | dashboard.html 更新后 fetch 路径容易回退 | 每次编辑 HTML 必须检查所有 fetch 是否用 PROXY 而非 BASE + '/api/v1/...' |

## 访问地址

| 地址 | 说明 |
|------|------|
| http://localhost:1880 | Node-RED 编辑器（需登录 admin/Admin_00） |
| http://localhost:1880/dashboard | OPC UA Dashboard（无需登录） |
| http://localhost:8000/docs | Bridge API 文档 |
