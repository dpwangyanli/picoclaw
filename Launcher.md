# 启动服务
简单说：

- Launcher 是“管理后台”
- Gateway 是“真正运行 AI 的服务”

| 组件 | 端口 | 作用 |
|---|---:|---|
| Launcher | `18800` | 提供网页管理界面，用来修改配置、查看日志、启动/停止 Gateway |
| Gateway | `18790` | 处理聊天请求、调用 Codex、执行 MCP 工具、连接各种频道 |

它们的关系是：

```text
浏览器
  ↓
Launcher（18800）
  ↓ 启动/停止/管理
Gateway（18790）
  ↓
Codex + MCP（18080）
```

你打开：

```text
http://localhost:18800
```

看到的是 Launcher。你在页面上点击“启动服务”后，Launcher 会启动这个子进程：

```bash
./build/picoclaw gateway -E
```

真正收到你的问题、调用 `mcp_go-rag-mcp-client_retrieve`、再让 Codex 回答的是 Gateway。

可以分别运行：

只启动 Gateway，不需要网页后台：

```bash
cd /Users/kenny/workspace/src/github.com/dpwangyanli/picoclaw
./build/picoclaw gateway -E
```

启动 Launcher，通过网页管理 Gateway：

```bash
cd /Users/kenny/workspace/src/github.com/dpwangyanli/picoclaw
./build/picoclaw-launcher -console -lang zh -no-browser
```

通常建议使用 Launcher，因为启动、停止、配置和看日志更方便。Launcher 停止后，它管理的 Gateway 通常也会一起停止；但独立启动的 MCP 服务不会受到影响。

# 停止服务
如果程序正在当前终端前台运行，直接按：

```text
Ctrl+C
```

这是最安全的停止方式。

分别停止可以这样操作：

- MCP 服务终端：按 `Ctrl+C`
- PicoClaw Launcher 终端：按 `Ctrl+C`
- PicoClaw Gateway：在 <http://localhost:18800> 点击顶部“停止服务”

如果找不到原来的终端，可以按端口停止。

停止 MCP（18080）：

```bash
kill $(lsof -tiTCP:18080 -sTCP:LISTEN)
```

停止监控面板（9090，通常与 MCP 是同一进程）：

```bash
kill $(lsof -tiTCP:9090 -sTCP:LISTEN)
```

停止 PicoClaw Gateway（18790）：

```bash
kill $(lsof -tiTCP:18790 -sTCP:LISTEN)
```

停止 Launcher（18800）：

```bash
kill $(lsof -tiTCP:18800 -sTCP:LISTEN)
```

由于 MCP 和监控面板属于同一个进程，停止 `18080` 后，`9090` 通常也会一起停止。

检查是否已停止：

```bash
lsof -i :18080
lsof -i :18790
lsof -i :18800
lsof -i :9090
```

没有输出就表示对应端口已经停止。