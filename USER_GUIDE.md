# Clash for Linux 使用指南

## 目录

- [快速开始](#快速开始)
- [Web 控制台使用](#web-控制台使用)
- [命令行使用](#命令行使用)
- [日常使用场景](#日常使用场景)
- [高级配置](#高级配置)
- [故障排查](#故障排查)
- [实用技巧](#实用技巧)

## 快速开始

### 确认安装成功

```bash
# 查看内核运行状态
clashstatus
# 应该显示类似：4579 /home/user/clashctl/bin/mihomo ...

# 查看帮助信息
clashctl --help
```

### 添加机场订阅

```bash
# 方法1：直接添加
clashsub add "https://你的机场订阅链接"

# 方法2：交互式添加
clashsub add
# 然后输入订阅链接

# 查看已添加的订阅
clashsub ls

# 激活订阅（1 是订阅 ID）
clashsub use 1
```

### 开启代理

```bash
# 开启代理
clashon

# 验证代理是否工作
curl -I https://www.google.com
# 应该返回 HTTP/1.1 200 OK

# 查看代理环境变量
echo $http_proxy
# 应该显示：http://127.0.0.1:7890
```

### 访问 Web 控制台

```bash
# 查看控制台访问信息
clashui

# 输出示例：
# ╔═══════════════════════════════════════════════╗
# ║                😼 Web 控制台                  ║
# ║     🏠 内网：http://192.168.0.114:9090/ui     ║
# ║     密钥：eSg05N                              ║
# ╚═══════════════════════════════════════════════╝
```

在浏览器中打开 `http://localhost:9090/ui` 或显示的内网地址，使用密钥登录。

## Web 控制台使用

### 登录 Web 控制台

1. **打开浏览器**，访问 `http://localhost:9090/ui`

2. **输入连接信息**：
   - **主机地址**: `http://127.0.0.1:9090`（本地）或 `http://192.168.x.x:9090`（局域网）
   - **密钥**: 运行 `clashsecret` 查看

3. **点击添加**，保存配置

### Web 控制台功能

#### 1. 代理页面 - 选择节点

- **查看所有节点**：显示机场提供的所有代理节点
- **切换节点**：点击节点名称即可切换
- **延迟测试**：点击 "延迟测试" 按钮，测试所有节点速度
- **节点筛选**：按地区、类型筛选节点
- **代理模式**：
  - **规则模式**（推荐）：根据规则自动选择直连或代理
  - **全局模式**：所有流量走代理
  - **直连模式**：所有流量直连

#### 2. 代理组

- **自动选择**：根据延迟自动选择最快节点
- **故障转移**：当前节点失败时自动切换
- **负载均衡**：分散流量到多个节点
- **手动选择**：手动指定使用的节点

#### 3. 连接页面 - 查看活动连接

- **实时连接**：查看哪些程序正在使用代理
- **流量统计**：查看每个连接的上传/下载流量
- **关闭连接**：可以手动断开特定连接
- **连接详情**：查看目标地址、协议、链路等信息

#### 4. 规则页面 - 查看代理规则

- **规则列表**：显示所有代理规则
- **规则类型**：
  - `DOMAIN`: 域名匹配
  - `DOMAIN-SUFFIX`: 域名后缀匹配
  - `DOMAIN-KEYWORD`: 域名关键词匹配
  - `IP-CIDR`: IP 段匹配
  - `GEOIP`: 地理位置匹配
  - `MATCH`: 默认规则
- **规则策略**：PROXY（代理）、DIRECT（直连）、REJECT（拒绝）

#### 5. 日志页面 - 实时日志

- **查看实时日志**：监控代理运行状态
- **过滤日志**：按类型筛选日志信息
- **日志级别**：info、warning、error、debug

#### 6. 设置页面

- **模式切换**：全局/规则/直连
- **系统代理**：是否设置系统代理
- **启动项**：开机自启设置
- **语言切换**：中文/英文

### Web 控制台操作示例

**场景1：选择最快节点**

1. 进入 "代理" 页面
2. 点击 "延迟测试" 按钮
3. 等待测试完成，选择延迟最低的节点

**场景2：查看哪些应用在使用代理**

1. 进入 "连接" 页面
2. 查看活动连接列表
3. 可以看到每个连接的目标地址和流量

**场景3：添加自定义规则**

1. 通过命令行编辑 mixin 配置：`clashmixin -e`
2. 添加自定义规则，保存后会自动生效

## 命令行使用

### 基础命令

#### 代理控制

```bash
# 开启代理（启动内核 + 设置系统代理）
clashon

# 关闭代理（停止内核 + 取消系统代理）
clashoff

# 重启代理
clashrestart
```

#### 系统代理管理

```bash
# 查看系统代理状态
clashproxy

# 单独开启系统代理（不启动内核）
clashproxy on

# 单独关闭系统代理（不停止内核）
clashproxy off
```

#### 内核状态

```bash
# 查看内核运行状态
clashstatus

# 查看详细状态（如果使用 systemd）
sudo systemctl status mihomo

# 查看实时日志
clashlog

# 查看最近 50 行日志
clashlog -n 50

# 实时跟踪日志
clashlog -f
```

#### Web 控制台

```bash
# 显示 Web 控制台访问信息
clashui

# 查看当前密钥
clashsecret

# 修改密钥
clashsecret 新密码

# 例如：
clashsecret mypassword123
```

### 订阅管理

#### 添加订阅

```bash
# 添加机场订阅
clashsub add "https://your-subscription-url"

# 添加本地配置文件
clashsub add "file:///path/to/config.yaml"

# 交互式添加
clashsub add
# 然后输入订阅链接
```

#### 查看订阅

```bash
# 列出所有订阅
clashsub list
# 或
clashsub ls

# 输出示例：
# profiles:
#   - id: 1
#     path: /home/user/clashctl/resources/profiles/1.yaml
#     url: https://your-subscription-url
# use: 1
```

#### 切换订阅

```bash
# 使用指定 ID 的订阅
clashsub use 1

# 交互式选择
clashsub use
# 会先显示订阅列表，然后提示输入 ID
```

#### 更新订阅

```bash
# 更新当前使用的订阅
clashsub update

# 更新指定 ID 的订阅
clashsub update 1

# 强制使用订阅转换
clashsub update 1 --convert

# 配置自动更新（每 2 天更新一次）
clashsub update --auto
```

#### 删除订阅

```bash
# 删除指定 ID 的订阅
clashsub del 1

# 注意：正在使用的订阅无法删除，需要先切换到其他订阅
```

#### 查看订阅日志

```bash
# 查看订阅操作历史
clashsub log

# 查看最近 10 条记录
clashsub log -n 10

# 查看最近 20 条记录
clashsub log -n 20
```

### 配置管理

#### Mixin 配置

Mixin 配置是用户自定义配置，会与订阅配置深度合并，优先级最高。

```bash
# 查看 mixin 配置
clashmixin

# 编辑 mixin 配置
clashmixin -e

# 查看原始订阅配置
clashmixin -c

# 查看运行时配置（合并后的最终配置）
clashmixin -r
```

#### Mixin 配置示例

编辑 mixin 配置（`clashmixin -e`）：

```yaml
# 自定义配置
_custom:
  system-proxy:
    enable: true

# 代理端口
mixed-port: 7890

# Web 控制台
external-controller: "0.0.0.0:9090"
secret: your-secret

# 规则（prefix: 前置，suffix: 后置）
rules:
  prefix:
    - DOMAIN,api.github.com,PROXY
    - DOMAIN-SUFFIX,github.com,PROXY
  suffix:
    - MATCH,PROXY

# 代理节点（override: 根据 name 覆盖原节点）
proxies:
  prefix: []
  suffix: []
  override: []

# 代理组（override: 根据 name 覆盖原代理组）
proxy-groups:
  prefix: []
  suffix: []
  override: []

# DNS 配置
dns:
  enable: true
  listen: 0.0.0.0:1053
  enhanced-mode: fake-ip
  nameserver:
    - 114.114.114.114
    - 8.8.8.8

# TUN 配置
tun:
  enable: false
  stack: system
  auto-route: true
  auto-redirect: true
  dns-hijack:
    - any:53
```

### TUN 模式

TUN 模式可以代理所有流量，包括不支持代理的应用。

```bash
# 查看 TUN 状态
clashtun

# 开启 TUN 模式（需要 root 权限）
clashtun on

# 关闭 TUN 模式
clashtun off
```

**注意事项**：
- TUN 模式需要 root 权限
- 内核版本需要 5.0+
- 可能与 Docker 网络冲突，已在 mixin.yaml 中排除 docker0 接口

### 内核升级

```bash
# 升级内核到最新版本
clashupgrade

# 查看升级日志
clashupgrade -v

# 升级到稳定版
clashupgrade -r

# 升级到测试版
clashupgrade -a
```

## 日常使用场景

### 场景1：开发环境使用

```bash
# 开启代理
clashon

# 使用 Git
git clone https://github.com/xxx/xxx.git
git push origin main

# 使用 npm
npm install
npm install -g some-package

# 使用 pip
pip install requests
pip install -r requirements.txt

# 使用 Docker
docker pull nginx
docker pull mysql

# 完成后关闭代理
clashoff
```

### 场景2：服务器长期运行

```bash
# 安装时自动启用 systemd 服务（root 用户）
sudo bash install.sh

# 添加订阅
clashsub add "https://your-subscription-url"
clashsub use 1

# 启动并启用开机自启
sudo systemctl start mihomo
sudo systemctl enable mihomo

# 查看状态
sudo systemctl status mihomo

# 不需要运行 clashon，其他应用直接配置使用：
# HTTP/HTTPS 代理: http://127.0.0.1:7890
# SOCKS5 代理: socks5://127.0.0.1:7890
```

### 场景3：多个机场切换

```bash
# 添加多个机场订阅
clashsub add "https://机场1订阅链接"
clashsub add "https://机场2订阅链接"
clashsub add "https://机场3订阅链接"

# 查看所有订阅
clashsub ls

# 切换到机场2
clashsub use 2

# 切换到机场3
clashsub use 3

# 删除不需要的机场
clashsub del 1
```

### 场景4：定时更新订阅

```bash
# 设置自动更新（每 2 天凌晨更新）
clashsub update --auto

# 查看 cron 任务
crontab -l
# 输出：0 0 */2 * * /bin/bash -i -c 'clashsub update'

# 手动更新订阅
clashsub update

# 查看更新日志
clashsub log
```

### 场景5：临时测试不同节点

1. 在 Web 控制台打开节点列表
2. 点击 "延迟测试"
3. 选择延迟最低的节点
4. 测试访问速度：
   ```bash
   curl -w "@curl-format.txt" -o /dev/null -s https://www.google.com
   ```

### 场景6：特定应用使用代理

```bash
# 方法1：使用环境变量
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
curl https://www.google.com

# 方法2：直接指定代理
curl -x http://127.0.0.1:7890 https://www.google.com

# 方法3：应用内配置
# 在应用设置中配置：
# HTTP 代理: 127.0.0.1:7890
# SOCKS5 代理: 127.0.0.1:7890
```

### 场景7：查看代理使用情况

```bash
# 在 Web 控制台查看
# 1. 打开 http://localhost:9090/ui
# 2. 进入 "连接" 页面
# 3. 查看所有活动连接

# 或查看日志
clashlog -f
```

## 高级配置

### 自定义代理规则

编辑 mixin 配置添加自定义规则：

```bash
clashmixin -e
```

常用规则示例：

```yaml
rules:
  prefix:
    # 强制 GitHub 走代理
    - DOMAIN-SUFFIX,github.com,PROXY
    - DOMAIN-SUFFIX,githubusercontent.com,PROXY

    # 强制国内网站直连
    - DOMAIN-SUFFIX,baidu.com,DIRECT
    - DOMAIN-SUFFIX,taobao.com,DIRECT
    - DOMAIN-SUFFIX,qq.com,DIRECT

    # 局域网直连
    - IP-CIDR,192.168.0.0/16,DIRECT
    - IP-CIDR,10.0.0.0/8,DIRECT

    # 广告拦截
    - DOMAIN-SUFFIX,googleads.com,REJECT
    - DOMAIN-SUFFIX,doubleclick.net,REJECT

  suffix:
    # 默认规则放在最后
    - MATCH,PROXY
```

### 自定义 DNS

```yaml
dns:
  enable: true
  listen: 0.0.0.0:1053
  enhanced-mode: fake-ip

  # 国内 DNS
  nameserver:
    - 114.114.114.114
    - 223.5.5.5

  # 国外 DNS（走代理）
  fallback:
    - tls://8.8.8.8:853
    - https://1.1.1.1/dns-query

  # 域名分流
  nameserver-policy:
    'geosite:cn':
      - 114.114.114.114
      - 223.5.5.5
    'geosite:geolocation-!cn':
      - tls://8.8.8.8:853
```

### 配置端口转发

如果端口冲突，可以修改端口：

```bash
clashmixin -e
```

修改以下配置：

```yaml
mixed-port: 7890          # 混合端口（HTTP + SOCKS5）
port: 7891                # HTTP 端口
socks-port: 7892          # SOCKS5 端口
external-controller: "0.0.0.0:9090"  # Web 控制台端口
```

### 配置认证

为了安全，可以启用代理认证：

```yaml
authentication:
  - "username:password"

# 使用时需要指定用户名密码
# curl -x http://username:password@127.0.0.1:7890 https://www.google.com
```

### 配置局域网访问

允许局域网设备使用代理：

```bash
clashmixin -e
```

修改配置：

```yaml
allow-lan: true
bind-address: "0.0.0.0"

# 建议同时启用认证防止滥用
authentication:
  - "user:pass"
```

局域网设备配置代理：
- HTTP 代理: `http://192.168.x.x:7890`
- SOCKS5 代理: `socks5://192.168.x.x:7890`

### 配置日志级别

```yaml
log-level: info  # silent, error, warning, info, debug
```

### 流量统计

Web 控制台自动显示流量统计，也可以通过 API 查询：

```bash
# 查看总流量
curl -H "Authorization: Bearer your-secret" \
  http://127.0.0.1:9090/traffic

# 查看实时连接
curl -H "Authorization: Bearer your-secret" \
  http://127.0.0.1:9090/connections
```

## 故障排查

### 问题1：代理无法启动

**症状**：
```bash
clashon
# 启动失败
```

**排查步骤**：

```bash
# 1. 查看详细状态
clashstatus

# 2. 查看错误日志
clashlog

# 3. 检查配置文件
~/clashctl/bin/mihomo -t -f ~/clashctl/resources/runtime.yaml

# 4. 检查端口占用
sudo netstat -tlnp | grep -E '7890|9090'

# 5. 检查订阅配置
clashmixin -c
```

### 问题2：代理连接失败

**症状**：
```bash
curl https://www.google.com
# curl: (7) Failed to connect to 127.0.0.1 port 7890
```

**解决方案**：

```bash
# 1. 确认内核运行
clashstatus

# 2. 确认代理端口
netstat -tlnp | grep 7890

# 3. 测试端口连接
telnet 127.0.0.1 7890

# 4. 检查防火墙
sudo ufw status
```

### 问题3：节点无法使用

**症状**：代理开启但无法访问外网

**排查步骤**：

1. **在 Web 控制台测试节点延迟**
   - 打开 http://localhost:9090/ui
   - 点击 "延迟测试"
   - 选择延迟低的节点

2. **检查订阅是否过期**
   ```bash
   # 更新订阅
   clashsub update
   ```

3. **查看连接日志**
   ```bash
   clashlog -f
   # 查看是否有错误信息
   ```

4. **测试直接连接**
   ```bash
   curl -v -x http://127.0.0.1:7890 https://www.google.com
   ```

### 问题4：DNS 解析失败

**症状**：无法解析域名

**解决方案**：

```bash
# 1. 检查 DNS 配置
clashmixin -r | grep -A 10 "dns:"

# 2. 测试 DNS 解析
nslookup www.google.com 127.0.0.1 -port=1053

# 3. 修改 DNS 配置
clashmixin -e
# 修改 dns.nameserver 为可用的 DNS
```

### 问题5：TUN 模式无法启用

**症状**：
```bash
clashtun on
# 系统内核版本不支持 Tun 模式
```

**解决方案**：

```bash
# 1. 检查内核版本
uname -r
# 需要 5.0+ 版本

# 2. 升级内核（Ubuntu 20.04）
sudo apt install --install-recommends linux-generic-hwe-20.04
sudo reboot

# 3. 检查 TUN 模块
lsmod | grep tun

# 4. 加载 TUN 模块
sudo modprobe tun
```

### 问题6：订阅更新失败

**症状**：
```bash
clashsub update
# 订阅更新失败
```

**解决方案**：

```bash
# 1. 查看详细日志
clashsub log

# 2. 手动下载测试
curl -v "你的订阅链接"

# 3. 使用订阅转换
clashsub update 1 --convert

# 4. 检查订阅链接是否过期
# 重新从机场获取订阅链接

# 5. 查看转换日志
cat ~/clashctl/bin/subconverter/latest.log
```

### 问题7：系统代理未生效

**症状**：
```bash
clashon
echo $http_proxy
# 没有输出
```

**解决方案**：

```bash
# 1. 重新加载 shell 配置
source ~/.bashrc  # 或 source ~/.zshrc

# 2. 手动设置代理
clashproxy on

# 3. 检查配置文件
grep -A 5 "clashctl START" ~/.bashrc

# 4. 新开终端窗口测试

# 5. 手动导出环境变量
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
```

### 问题8：Web 控制台无法访问

**症状**：浏览器无法打开 http://localhost:9090/ui

**解决方案**：

```bash
# 1. 确认内核运行
clashstatus

# 2. 检查端口监听
netstat -tlnp | grep 9090

# 3. 检查防火墙
sudo ufw allow 9090/tcp

# 4. 查看控制台配置
clashmixin -r | grep external-controller

# 5. 尝试不同地址
# http://127.0.0.1:9090/ui
# http://localhost:9090/ui
# http://[你的IP]:9090/ui

# 6. 检查密钥
clashsecret
```

### 问题9：性能问题

**症状**：代理速度慢

**优化方案**：

1. **选择最快节点**
   - 在 Web 控制台测试延迟
   - 选择延迟最低的节点

2. **使用负载均衡**
   ```yaml
   proxy-groups:
     - name: 负载均衡
       type: load-balance
       proxies:
         - 节点1
         - 节点2
   ```

3. **优化 DNS**
   ```yaml
   dns:
     enable: true
     enhanced-mode: fake-ip  # 使用 fake-ip 模式提升性能
   ```

4. **检查系统资源**
   ```bash
   top
   # 查看 mihomo 进程的 CPU 和内存使用
   ```

### 问题10：进程卡死

**症状**：mihomo 进程无响应

**解决方案**：

```bash
# 1. 强制停止
pkill -9 mihomo

# 2. 清理配置
rm ~/clashctl/resources/runtime.yaml

# 3. 重新生成配置
clashsub use 1

# 4. 重启代理
clashon

# 5. 检查日志
clashlog -n 100
```

## 实用技巧

### 技巧1：快速切换代理模式

在 Web 控制台顶部可以快速切换：
- **Rule**（规则模式）：推荐日常使用
- **Global**（全局模式）：所有流量走代理
- **Direct**（直连模式）：所有流量直连

### 技巧2：配置 Git 代理

```bash
# 全局配置
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 取消配置
git config --global --unset http.proxy
git config --global --unset https.proxy

# 或者仅为 GitHub 配置
git config --global http.https://github.com.proxy http://127.0.0.1:7890
```

### 技巧3：配置 Docker 代理

创建或编辑 `/etc/docker/daemon.json`：

```json
{
  "proxies": {
    "http-proxy": "http://127.0.0.1:7890",
    "https-proxy": "http://127.0.0.1:7890",
    "no-proxy": "localhost,127.0.0.1"
  }
}
```

重启 Docker：
```bash
sudo systemctl restart docker
```

### 技巧4：配置 apt 代理

临时使用：
```bash
sudo apt -o Acquire::http::proxy="http://127.0.0.1:7890" update
```

永久配置（创建 `/etc/apt/apt.conf.d/proxy.conf`）：
```
Acquire::http::Proxy "http://127.0.0.1:7890";
Acquire::https::Proxy "http://127.0.0.1:7890";
```

### 技巧5：测试代理速度

```bash
# 测试下载速度
curl -x http://127.0.0.1:7890 -o /dev/null \
  https://speed.cloudflare.com/__down?bytes=100000000

# 测试延迟
curl -x http://127.0.0.1:7890 -w "Time: %{time_total}s\n" \
  -o /dev/null -s https://www.google.com
```

### 技巧6：导出配置

```bash
# 备份配置
cp ~/clashctl/resources/mixin.yaml ~/mixin-backup.yaml

# 备份订阅信息
cp ~/clashctl/resources/profiles.yaml ~/profiles-backup.yaml

# 恢复配置
cp ~/mixin-backup.yaml ~/clashctl/resources/mixin.yaml
clashrestart
```

### 技巧7：使用别名简化命令

在 `~/.bashrc` 或 `~/.zshrc` 中添加：

```bash
alias con='clashon'
alias coff='clashoff'
alias cst='clashstatus'
alias cui='clashui'
alias clog='clashlog -f'
```

重新加载：
```bash
source ~/.bashrc
```

### 技巧8：监控流量使用

```bash
# 使用 iftop 监控网络流量
sudo apt install iftop
sudo iftop -i any

# 或使用 nethogs 按进程监控
sudo apt install nethogs
sudo nethogs
```

### 技巧9：定期清理日志

```bash
# 清理 mihomo 日志
truncate -s 0 ~/clashctl/resources/mihomo.log

# 清理订阅日志
truncate -s 0 ~/clashctl/resources/profiles.log

# 或使用 logrotate 自动管理日志
```

### 技巧10：脚本自动化

创建自动切换脚本：

```bash
#!/bin/bash
# auto-proxy.sh

# 检查网络
if ping -c 1 google.com &>/dev/null; then
    echo "Direct connection available"
    clashoff
else
    echo "Need proxy"
    clashon
fi
```

## 常用命令速查表

| 功能 | 命令 |
|------|------|
| 开启代理 | `clashon` |
| 关闭代理 | `clashoff` |
| 查看状态 | `clashstatus` |
| 查看日志 | `clashlog` |
| Web 控制台 | `clashui` |
| 查看密钥 | `clashsecret` |
| 修改密钥 | `clashsecret 新密码` |
| 添加订阅 | `clashsub add "url"` |
| 查看订阅 | `clashsub ls` |
| 切换订阅 | `clashsub use 1` |
| 更新订阅 | `clashsub update` |
| 删除订阅 | `clashsub del 1` |
| 订阅日志 | `clashsub log` |
| 查看配置 | `clashmixin` |
| 编辑配置 | `clashmixin -e` |
| TUN 开启 | `clashtun on` |
| TUN 关闭 | `clashtun off` |
| 升级内核 | `clashupgrade` |
| 系统代理状态 | `clashproxy` |
| 开启系统代理 | `clashproxy on` |
| 关闭系统代理 | `clashproxy off` |

## 环境变量说明

代理相关环境变量：

```bash
# HTTP 代理
http_proxy=http://127.0.0.1:7890
HTTP_PROXY=http://127.0.0.1:7890

# HTTPS 代理
https_proxy=http://127.0.0.1:7890
HTTPS_PROXY=http://127.0.0.1:7890

# SOCKS5 代理
all_proxy=socks5h://127.0.0.1:7890
ALL_PROXY=socks5h://127.0.0.1:7890

# 不走代理的地址
no_proxy=localhost,127.0.0.1,::1
NO_PROXY=localhost,127.0.0.1,::1
```

## 配置文件路径

| 文件 | 路径 | 说明 |
|------|------|------|
| 安装目录 | `~/clashctl/` | 默认安装路径 |
| 配置目录 | `~/clashctl/resources/` | 配置文件目录 |
| 原始订阅 | `~/clashctl/resources/config.yaml` | 订阅配置 |
| Mixin 配置 | `~/clashctl/resources/mixin.yaml` | 用户自定义配置 |
| 运行时配置 | `~/clashctl/resources/runtime.yaml` | 合并后的最终配置 |
| 订阅管理 | `~/clashctl/resources/profiles.yaml` | 订阅元数据 |
| 订阅目录 | `~/clashctl/resources/profiles/` | 订阅配置文件 |
| 内核二进制 | `~/clashctl/bin/mihomo` | mihomo 内核 |
| yq 工具 | `~/clashctl/bin/yq` | YAML 处理工具 |
| 订阅转换器 | `~/clashctl/bin/subconverter/` | 本地订阅转换工具 |
| 日志文件 | `~/clashctl/resources/mihomo.log` | 内核日志（nohup 模式） |
| 订阅日志 | `~/clashctl/resources/profiles.log` | 订阅操作日志 |

## 获取帮助

- **查看命令帮助**：`clashctl --help`
- **子命令帮助**：`clashsub --help`、`clashtun --help` 等
- **项目 Wiki**：https://github.com/nelvko/clash-for-linux-install/wiki/FAQ
- **提交 Issue**：https://github.com/nelvko/clash-for-linux-install/issues
- **查看日志排查问题**：`clashlog` 或 `clashsub log`

## 更新日志

查看项目更新：
```bash
cd ~/clash-for-linux-install
git pull origin master
```

## 卸载

```bash
cd ~/clash-for-linux-install
bash uninstall.sh
```

卸载脚本会：
- 停止并删除服务
- 删除安装目录
- 清理 shell 配置
- 删除 cron 任务

---

**祝使用愉快！** 🚀

如有问题，请查看 [故障排查](#故障排查) 章节或提交 Issue。
