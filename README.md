# Spectrum MCP Server

这个项目将来自 ESP32 上 AS7341 光谱传感器的数据通过 MQTT 发布，并在本地运行一个 FastMCP 服务器将最新光谱封装为 MCP 资源与工具接口，方便其它程序/服务调用。

目录结构（相关文件）:

- `server.py` — 本地 FastMCP 服务，订阅 MQTT 主题并暴露 `spectrum://latest` 资源。
- `esp32/main.txt` — ESP32 上的 MicroPython 主程序（请在设备上保存为 `main.py` 并按需修改 WiFi/MQTT 配置）。
- `esp32/as7341.txt` — AS7341 驱动（MicroPython）。

注意：ESP32 代码以 `.txt` 附件形式保存在仓库以便在 PC 上编辑；部署到设备时应保存为 `main.py` 和 `as7341.py`。

## 快速开始（在开发机器）

1. 创建并激活虚拟环境（可选但推荐）:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

2. 安装依赖:

```powershell
python -m pip install "mcp[cli]>=1.26.0" paho-mqtt>=1.6.1
```

3. 运行 MCP 服务器:

```powershell
python server.py
```

服务器启动后会连接到配置的 MQTT Broker（默认：`meimefarm/Spectrum`），并在收到最新光谱消息时保存到内存中。

## MCP 接口

- 资源 `spectrum://latest` — 返回最新的光谱数据（字典，键为 `channel1`..`channel11`）。
- 工具 `get_channel(name)` — 传入通道名（例如 `channel1`）返回整数值。

示例：使用 Python 客户端调用（示例使用 `mcp` 包内的客户端 API）：

```python
from mcp.client import MCPClient

c = MCPClient.as_stdio()  # 依据运行方式选择合适 transport
print(c.resource('spectrum://latest')())
print(c.tool('get_channel')('channel1'))
```

注：如果使用 `mcp[cli]` 提供的命令行工具，参阅 `mcp` 文档以通过 CLI 调用资源/工具。

## 在 ESP32 上部署

1. 修改 `esp32/main.txt` 中的 WiFi 和 MQTT 配置（如 `WIFI_SSID`, `WIFI_PASS`, `MQTT_BROKER`, `MQTT_USER`, `MQTT_PASSWORD`）。
2. 将 `esp32/as7341.txt` 重命名为 `as7341.py`，将 `esp32/main.txt` 重命名为 `main.py`。
3. 使用 `mpremote` / `ampy` / `rshell` 等工具上传文件到设备并重启：

```bash
# 使用 mpremote 示例
mpremote connect auto fs cp esp32/as7341.py :
mpremote connect auto fs cp esp32/main.py :
mpremote connect auto run :
```

设备上运行的 `main.py` 会连接 WiFi、读取 AS7341 光谱并通过 MQTT 向 `meimefarm/Spectrum` 主题发布 JSON 消息，例如：

```json
{
  "channel1": 123,
  "channel2": 456,
  ...
}
```

## 自定义与调试

- 若要改为异步 MQTT 客户端，可在 `server.py` 中替换为 `asyncio-mqtt`。
- 若 MQTT 需要自定义 CA 或证书，请在 `server.py` 的 `start_mqtt()` 中调用 `client.tls_set(ca_certs=..., certfile=..., keyfile=...)` 并根据需要设置 `tls_insecure_set()`。
- 若无法收到消息，检查设备是否能连上 WiFi，并确认 Broker/Topic 与 `server.py` 中的一致。

## 常见问题

- 为什么 ESP32 文件以 `.txt` 結尾？
  - 为了方便在桌面环境编辑并避免直接覆盖开发环境，发布库内保存为 `.txt`；部署到设备时请改名为 `.py`。

- 如何保证数据持久化？
  - 当前实现仅保存在内存 `latest_spectrum`，若需要持久化可把收到的消息写入数据库或文件。

## 后续建议

- 添加一个小型 web 前端或 HTTP API 以便浏览器/其他服务直接读取最新光谱。
- 将 MCP 服务改为使用持久后端（Redis/SQLite）以支持历史查询与持久化。

---

如果你愿意，我可以：

- 把 `esp32/main.txt` 转为 `esp32/main.py`（直接写文件），并生成一个 `requirements.txt` 或 `pyproject` 更新示例；
- 或把 `server.py` 改写为异步版本并使用 `asyncio-mqtt`。

告诉我你下一步想要哪个，我会继续实现。 
