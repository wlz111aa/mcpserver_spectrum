# Spectrum MCP Server — 使用与服务配置

本仓库包含一个将 ESP32 上 AS7341 光谱传感器数据通过 MQTT 收集并暴露为 MCP 资源的服务。该 README 重点说明如何在本地与平台上运行此服务，以及平台所需的启动配置说明。

核心文件
- `server.py` — FastMCP 服务实现，订阅 MQTT 主题并暴露资源 `spectrum://latest` 与工具 `get_channel()`。
- `mcp-config.json` — 平台启动配置（请务必包含在仓库根目录，平台据此知道如何安装并启动服务）。

快速开始（开发机器）
1. 创建并激活虚拟环境（可选但推荐）：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

2. 安装依赖：

```powershell
python -m pip install --upgrade pip
pip install -e .
# 或： pip install -r requirements.txt
```

3. 运行 MCP 服务器（开发）：

```powershell
python server.py
```

可选参数：`--no-mqtt`（不启动 MQTT 客户端，便于在无设备/无 Broker 场景测试）。

服务接口
- 资源 `spectrum://latest` — 返回最近一次接收到的光谱 JSON（字典，通常包含 `channel1`..`channel11`）。
- 工具 `get_channel(name)` — 返回指定通道的整数值。

检查与调试
- HTTP 健康/数据检查：`/api/latest` 返回当前 `latest_spectrum` 的 JSON 副本。
- 我们在响应中加入了诊断头 `X-Served-By: mcpserver`，平台请求返回该头则表示请求命中了本服务。
- 使用 curl 验证（检查 Content-Type 与自定义头）：

```bash
curl -I 'http://YOUR_HOST:8000/api/latest' -H 'Accept: application/json'
```

平台说明（重要）
平台不会自动猜测如何运行你的代码：请确保仓库根目录包含 `mcp-config.json`（或平台指定的等价文件），其中必须明确：
- 安装依赖命令（如 `pip install -r requirements.txt` 或 `pip install -e .`）
- 启动命令与参数（例如 `python server.py`，以及可选 `--no-mqtt`）
- 运行时环境变量（如 `HTTP_BIND`/`HTTP_PORT`/`MCP_TRANSPORT`）
- 服务监听端口与协议（例如 HTTP port 8000，health-check 路径 `/api/latest`）

仓库中已加入示例配置文件：`mcp-config.json`。平台应读取该文件并据此安装依赖、启动服务并进行健康检查。

部署建议
- 临时公开测试：使用 `ngrok` 暴露本地端口：

```bash
ngrok http 8000
# 将 ngrok 返回的 https://xxxxx.ngrok.io 指定为平台的服务 URL（/api/latest）
```

- 生产/稳定：部署到 VPS、Render、Railway、Fly 或 Heroku，并将平台指向部署后的 HTTPS URL（确保 /api/latest 可访问）。

常见问题
- 我把仓库放到 GitHub，平台却拿到 HTML：如果平台的 URL 指向 `https://github.com/...`（仓库页面），它会返回 HTML 页面而非运行中的 API。平台需要指向正在运行的服务地址（或 raw.githubusercontent.net 的静态文件 URL）。
- 平台请求时显示 HTML 登录页（例如 GitHub）：说明平台没有启动服务或 URL 指向了静态仓库页面。

后续可选操作
- 我可以帮你生成 `requirements.txt`（从 `pyproject.toml` 提取依赖）并提交到仓库；
- 我也可以添加一个小的 systemd/Procfile 或 GitHub Actions 示例以便自动部署到云平台；
- 如果你想快速测试，我可以给出 `ngrok` 的完整步骤并协助启动。

---

文件引用：启动配置位于仓库根目录的 `mcp-config.json`（请在平台上确认该文件被识别）。

