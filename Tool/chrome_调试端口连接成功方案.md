# Chrome 调试端口连接成功方案

# WSL2 远程调试 Windows Chrome

## 配置步骤

### 1. 添加 Windows 防火墙规则（管理员 PowerShell）
```powershell
New-NetFirewallRule -DisplayName "Chrome Debug Port" -Direction Inbound -LocalPort 9222 -Protocol TCP -Action Allow
```

### 2. 添加端口转发规则（管理员 PowerShell）
```powershell
netsh interface portproxy add v4tov4 listenport=9222 listenaddress=0.0.0.0 connectport=9222 connectaddress=127.0.0.1
```

### 3. 关闭所有 Chrome 进程
```powershell
taskkill /F /IM chrome.exe
```

### 4. 启动 Chrome 并开启远程调试
```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --remote-debugging-address=0.0.0.0 --remote-allow-origins=* --user-data-dir="C:\Users\<用户名>\chrome-debug-profile" "http://localhost:5173"
```

### 5. 验证监听状态
```powershell
netstat -ano | findstr ":9222" | findstr "LISTENING"
```

### 6. 从 WSL 测试连接
```bash
# 获取网关 IP
ip route | grep default | awk '{print $3}'

# 测试连接
curl -s http://<网关IP>:9222/json/version
curl -s http://<网关IP>:9222/json
```

## 关键参数
- `--remote-debugging-port=9222`：调试端口
- `--remote-debugging-address=0.0.0.0`：监听所有接口
- `--remote-allow-origins=*`：允许所有来源

## 常见问题
- 空回复：关闭所有 Chrome 进程重新启动
- 端口占用：检查 netstat 并关闭冲突进程

# Chrome 调试端口连接成功方案

# WSL2 远程调试 Windows Chrome

## 配置步骤

### 1. 检查 9222 端口占用情况（管理员 PowerShell）

```powershell
# 查看占用 9222 端口的进程
netstat -ano | findstr ":9222"

# 根据 PID 查看进程名称
tasklist /FI "PID eq <PID>"
```

**常见占用进程：**
- `chrome.exe`：已有的 Chrome 调试实例，需要先关闭
- `svchost.exe`：可能是端口转发服务占用，需要删除端口转发规则

### 2. 清理占用端口的进程（管理员 PowerShell）

```powershell
# 关闭 Chrome 进程
taskkill /F /IM chrome.exe

# 或按 PID 关闭
taskkill /F /PID <进程ID>

# 删除端口转发规则（如果有 svchost 占用）
netsh interface portproxy delete v4tov4 listenport=9222 listenaddress=172.17.208.1
netsh interface portproxy delete v4tov4 listenport=9222 listenaddress=0.0.0.0

# 确认端口已释放
netstat -ano | findstr ":9222"
```

### 3. 添加 Windows 防火墙规则（管理员 PowerShell）

```powershell
New-NetFirewallRule -DisplayName "Chrome Debug Port" -Direction Inbound -LocalPort 9222 -Protocol TCP -Action Allow
```

### 4. 添加端口转发规则（管理员 PowerShell）

```powershell
netsh interface portproxy add v4tov4 listenport=9222 listenaddress=0.0.0.0 connectport=9222 connectaddress=127.0.0.1
```

### 5. 启动 Chrome 并开启远程调试

```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --remote-debugging-address=0.0.0.0 --remote-allow-origins=* --user-data-dir="C:\Users\<用户名>\chrome-debug-profile" "http://localhost:5173"
```

### 6. 验证监听状态

```powershell
netstat -ano | findstr ":9222" | findstr "LISTENING"
```

**注意：** 确保只有一条 `0.0.0.0:9222` 的监听记录。

### 7. 测试 Chrome 调试端口（管理员 PowerShell）

```powershell
curl http://127.0.0.1:9222/json/version
```

### 8. 从 WSL 测试连接

```bash
# 获取网关 IP
ip route | grep default | awk '{print $3}'

# 测试连接
curl -s http://<网关IP>:9222/json/version

# 如果网关 IP 不通，尝试 nameserver IP
curl -s http://10.255.255.254:9222/json/version
```

## 关键参数
- `--remote-debugging-port=9222`：调试端口
- `--remote-debugging-address=0.0.0.0`：监听所有接口
- `--remote-allow-origins=*`：允许所有来源

## 常见问题
- 空回复：关闭所有 Chrome 进程重新启动
- 端口占用：检查 netstat 并关闭冲突进程
- svchost 占用：删除端口转发规则后重启 Chrome
# Chrome 调试端口连接成功方案

## WSL2 远程调试 Windows Chrome

## 配置步骤

### 1. 检查 9222 端口占用情况（管理员 PowerShell）

```powershell
# 查看占用 9222 端口的进程
netstat -ano | findstr ":9222"

# 根据 PID 查看进程名称
tasklist /FI "PID eq <PID>"
```

**常见占用进程：**
- `chrome.exe`：已有的 Chrome 调试实例，需要先关闭
- `svchost.exe`：可能是端口转发服务占用，需要删除端口转发规则

### 2. 清理占用端口的进程（管理员 PowerShell）

```powershell
# 关闭 Chrome 进程
taskkill /F /IM chrome.exe

# 或按 PID 关闭
taskkill /F /PID <进程ID>

# 删除端口转发规则（如果有 svchost 占用）
netsh interface portproxy delete v4tov4 listenport=9222 listenaddress=172.17.208.1
netsh interface portproxy delete v4tov4 listenport=9222 listenaddress=0.0.0.0

# 确认端口已释放
netstat -ano | findstr ":9222"
```

### 3. 添加 Windows 防火墙规则（管理员 PowerShell）

```powershell
New-NetFirewallRule -DisplayName "Chrome Debug Port" -Direction Inbound -LocalPort 9222 -Protocol TCP -Action Allow
```

### 4. 添加端口转发规则（管理员 PowerShell）

```powershell
netsh interface portproxy add v4tov4 listenport=9222 listenaddress=0.0.0.0 connectport=9222 connectaddress=127.0.0.1
```

### 5. 启动 Chrome 并开启远程调试

```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --remote-debugging-address=0.0.0.0 --remote-allow-origins=* --user-data-dir="C:\Users\<用户名>\chrome-debug-profile" "http://localhost:5173"
```

### 6. 验证监听状态

```powershell
netstat -ano | findstr ":9222" | findstr "LISTENING"
```

**注意：** 确保只有一条 `0.0.0.0:9222` 的监听记录。

### 7. 测试 Chrome 调试端口（管理员 PowerShell）

```powershell
curl http://127.0.0.1:9222/json/version
```

### 8. 从 WSL 测试连接

```bash
# 获取网关 IP
ip route | grep default | awk '{print $3}'

# 测试连接
curl -s http://<网关IP>:9222/json/version

# 如果网关 IP 不通，尝试 nameserver IP
curl -s http://10.255.255.254:9222/json/version
```

## 关键参数
- `--remote-debugging-port=9222`：调试端口
- `--remote-debugging-address=0.0.0.0`：监听所有接口
- `--remote-allow-origins=*`：允许所有来源

## 常见问题
- 空回复：关闭所有 Chrome 进程重新启动
- 端口占用：检查 netstat 并关闭冲突进程
- svchost 占用：删除端口转发规则后重启 Chrome
