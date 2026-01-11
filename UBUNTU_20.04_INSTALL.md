# Ubuntu 20.04 安装指南

## 兼容性说明

✅ **本项目完全支持 Ubuntu 20.04 LTS**

- 支持的架构：x86_64 (amd64), i386, ARM, ARM64
- 支持的 init 系统：systemd (Ubuntu 20.04 默认)
- 内核版本要求：无特殊要求（TUN 模式需要内核 5.0+ 支持）

## 依赖检查与安装

### 1. 检查必需依赖

运行以下命令检查系统中是否已安装必需的依赖：

```bash
# 检查所有必需命令
for cmd in xz pgrep curl tar unzip; do
    if command -v $cmd >/dev/null 2>&1; then
        echo "✓ $cmd 已安装"
    else
        echo "✗ $cmd 未安装"
    fi
done
```

### 2. 安装缺失的依赖

如果有依赖缺失，使用以下命令安装：

```bash
# 更新包列表
sudo apt update

# 安装所有必需依赖（一键安装）
sudo apt install -y curl tar unzip xz-utils procps

# 可选：安装 cron（用于自动更新订阅功能）
sudo apt install -y cron
```

### 3. 各依赖包说明

| 命令 | Ubuntu 包名 | 用途 | Ubuntu 20.04 默认状态 |
|------|------------|------|---------------------|
| `curl` | curl | 下载订阅配置、内核二进制文件 | ⚠️ 可能未安装 |
| `tar` | tar | 解压 yq、subconverter 压缩包 | ✅ 默认安装 |
| `unzip` | unzip | 解压 Web UI 文件 | ⚠️ 可能未安装 |
| `xz` | xz-utils | 解压内核二进制文件 | ✅ 默认安装 |
| `pgrep` | procps | 进程管理，检测内核运行状态 | ✅ 默认安装 |
| `crontab` | cron | 定时任务（可选） | ✅ 默认安装 |

## 完整安装步骤

### 方式一：标准安装（推荐）

```bash
# 1. 安装依赖
sudo apt update
sudo apt install -y curl tar unzip xz-utils procps git

# 2. 克隆仓库
git clone --branch master --depth 1 https://github.com/nelvko/clash-for-linux-install.git
cd clash-for-linux-install

# 3. 执行安装（默认安装 mihomo 内核）
bash install.sh

# 4. 安装完成后，重新加载 shell 配置
source ~/.bashrc
# 如果使用 zsh，则执行: source ~/.zshrc
```

### 方式二：一键安装（使用 GitHub 加速）

```bash
# 确保已安装必需依赖
sudo apt update && sudo apt install -y curl tar unzip xz-utils procps git

# 一键安装
git clone --branch master --depth 1 https://gh-proxy.org/https://github.com/nelvko/clash-for-linux-install.git \
  && cd clash-for-linux-install \
  && bash install.sh
```

### 方式三：指定内核版本

```bash
# 安装 mihomo 内核（推荐，功能更完善）
bash install.sh mihomo

# 或安装 clash 内核
bash install.sh clash
```

### 方式四：安装时指定订阅地址

```bash
bash install.sh mihomo "https://your-subscription-url"
```

## 验证安装

### 1. 验证命令可用性

```bash
# 检查命令是否可用
clashctl --help

# 应该显示帮助信息：
# Usage:
#   clashctl COMMAND [OPTIONS]
# Commands:
#   on                    开启代理
#   off                   关闭代理
#   ...
```

### 2. 检查服务状态

```bash
# 查看 mihomo/clash 服务状态
clashstatus

# 或使用 systemd 命令查看
sudo systemctl status mihomo
```

### 3. 添加订阅并测试

```bash
# 添加订阅（替换为你的订阅链接）
clashsub add "https://your-subscription-url"

# 使用订阅
clashsub use 1

# 开启代理
clashon

# 测试代理是否工作
curl -I https://www.google.com

# 查看当前代理设置
echo $http_proxy
```

### 4. 访问 Web 控制台

```bash
# 显示 Web 控制台地址
clashui

# 输出示例：
# ╔═══════════════════════════════════════════════╗
# ║                😼 Web 控制台                  ║
# ║═══════════════════════════════════════════════║
# ║     🔓 注意放行端口：9090                      ║
# ║     🏠 内网：http://192.168.x.x:9090/ui       ║
# ║     ...                                       ║
# ╚═══════════════════════════════════════════════╝

# 在浏览器中打开显示的地址
```

## 可能遇到的问题及解决方案

### 问题 1：curl 未安装

**错误信息：**
```
请先安装以下命令：curl
```

**解决方案：**
```bash
sudo apt update
sudo apt install -y curl
```

### 问题 2：unzip 未安装

**错误信息：**
```
请先安装以下命令：unzip
```

**解决方案：**
```bash
sudo apt install -y unzip
```

### 问题 3：Git 未安装（克隆仓库时）

**错误信息：**
```
bash: git: command not found
```

**解决方案：**
```bash
sudo apt update
sudo apt install -y git
```

### 问题 4：权限不足

**错误信息：**
```
Permission denied
```

**解决方案：**
- 如果安装到用户目录（如 `~/clashctl`），不需要 sudo
- 如果遇到权限问题，可以编辑 `.env` 文件修改 `CLASH_BASE_DIR` 到用户目录：
  ```bash
  # 编辑 .env 文件
  nano .env
  # 修改 CLASH_BASE_DIR=~/clashctl
  ```

### 问题 5：端口冲突

**错误信息：**
```
端口冲突：[mixed-port] 7890 🎲 随机分配 12345
```

**说明：**
这不是错误，脚本会自动检测端口占用并分配可用端口。如果想使用固定端口，可以在安装后编辑 mixin 配置：

```bash
clashmixin -e
# 修改 mixed-port: 7890 为其他端口
```

### 问题 6：systemd 服务启动失败

**检查方法：**
```bash
# 查看服务状态
sudo systemctl status mihomo

# 查看详细日志
sudo journalctl -u mihomo -n 50
```

**可能原因及解决：**

1. **配置文件错误**：
   ```bash
   # 验证配置文件
   ~/clashctl/bin/mihomo -t -f ~/clashctl/resources/runtime.yaml
   ```

2. **端口被占用**：
   ```bash
   # 检查端口占用
   sudo netstat -tlnp | grep -E '7890|9090'
   # 或
   sudo ss -tlnp | grep -E '7890|9090'
   ```

3. **权限问题（TUN 模式）**：
   ```bash
   # TUN 模式需要 root 权限
   # 确保使用 sudo 或 root 用户运行
   sudo clashon
   ```

### 问题 7：订阅下载失败

**错误信息：**
```
订阅无效，请检查
```

**解决方案：**

1. **检查网络连接**：
   ```bash
   curl -I https://www.google.com
   ```

2. **使用订阅转换**：
   ```bash
   clashsub update 1 --convert
   ```

3. **手动下载订阅**：
   ```bash
   # 下载订阅到本地
   curl -o ~/config.yaml "https://your-subscription-url"

   # 添加本地订阅
   clashsub add "file://$(realpath ~/config.yaml)"
   ```

### 问题 8：TUN 模式不支持

**错误信息：**
```
系统内核版本不支持 Tun 模式
```

**检查内核版本：**
```bash
uname -r
# Ubuntu 20.04 默认内核是 5.4+，应该支持 TUN
```

**解决方案：**
```bash
# 如果内核版本低于 5.0，需要升级内核
sudo apt update
sudo apt install --install-recommends linux-generic-hwe-20.04

# 重启系统
sudo reboot
```

### 问题 9：代理环境变量未生效

**问题描述：**
执行 `clashon` 后，`curl` 仍然无法通过代理访问

**解决方案：**

1. **检查代理变量**：
   ```bash
   env | grep -i proxy
   ```

2. **重新加载 shell 配置**：
   ```bash
   source ~/.bashrc  # 或 source ~/.zshrc
   ```

3. **手动设置代理**：
   ```bash
   clashproxy on
   ```

4. **新开终端窗口**：
   有时需要新开终端窗口才能生效

### 问题 10：Fish shell 不工作

**问题描述：**
使用 Fish shell 时命令无法执行

**解决方案：**
```bash
# Fish shell 的配置文件会自动安装到
# ~/.config/fish/conf.d/clashctl.fish

# 检查文件是否存在
ls ~/.config/fish/conf.d/clashctl.fish

# 重新加载 Fish 配置
source ~/.config/fish/conf.d/clashctl.fish
```

## Ubuntu 20.04 特定注意事项

### 1. 内核版本

Ubuntu 20.04 LTS 默认内核版本为 5.4+，完全支持 TUN 模式。如果使用更旧的内核，建议升级：

```bash
# 安装 HWE (Hardware Enablement) 内核
sudo apt install --install-recommends linux-generic-hwe-20.04
```

### 2. systemd 版本

Ubuntu 20.04 使用 systemd 245，完全兼容本项目的 systemd 服务配置。

### 3. DNS 解析

如果使用 TUN 模式，注意 Ubuntu 20.04 默认使用 systemd-resolved，可能与 Clash DNS 冲突。建议在 mixin.yaml 中配置正确的 DNS：

```yaml
dns:
  enable: true
  listen: 0.0.0.0:1053
  enhanced-mode: fake-ip
  nameserver:
    - 114.114.114.114
    - 8.8.8.8
```

### 4. 防火墙设置

如果启用了 ufw 防火墙，需要放行相关端口：

```bash
# 放行 Web 控制台端口
sudo ufw allow 9090/tcp

# 如果需要局域网访问
sudo ufw allow 7890/tcp
```

## 测试清单

安装完成后，建议按以下清单测试：

- [ ] 依赖检查通过
- [ ] 安装脚本执行成功
- [ ] `clashctl --help` 显示帮助信息
- [ ] 添加订阅成功
- [ ] `clashon` 开启代理成功
- [ ] `curl https://www.google.com` 可以访问
- [ ] Web 控制台可以访问（http://localhost:9090/ui）
- [ ] `clashoff` 关闭代理成功
- [ ] 重启系统后代理自动启动（如果启用）
- [ ] TUN 模式可以启用（可选）

## 卸载

如果需要卸载：

```bash
cd clash-for-linux-install
bash uninstall.sh
```

## 常见使用场景

### 场景 1：开发环境代理

```bash
# 开启代理
clashon

# 进行开发工作（git、npm、pip 等会自动使用代理）
git clone https://github.com/xxx/xxx.git
npm install

# 完成后关闭代理
clashoff
```

### 场景 2：服务器长期运行

```bash
# 安装并启用开机自启（systemd 会自动启用）
bash install.sh mihomo

# 添加订阅
clashsub add "https://your-subscription-url"
clashsub use 1

# 启动服务
sudo systemctl start mihomo
sudo systemctl enable mihomo

# 不需要设置系统代理，应用可直接配置使用 localhost:7890
```

### 场景 3：定期更新订阅

```bash
# 配置自动更新（每 2 天更新一次）
clashsub update --auto

# 查看 cron 任务
crontab -l

# 手动更新
clashsub update
```

## 获取帮助

如遇到其他问题：

1. 查看项目 Wiki：https://github.com/nelvko/clash-for-linux-install/wiki/FAQ
2. 提交 Issue：https://github.com/nelvko/clash-for-linux-install/issues
3. 查看日志：`clashlog` 或 `sudo journalctl -u mihomo -f`
