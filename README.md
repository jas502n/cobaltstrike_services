# 将 Cobalt Strike Teamserver 作为服务运行

这些脚本可以用作模板，用于将 teamserver 设置为服务并自动启动监听器。

这些脚本已在 Ubuntu 服务器上测试过，您需要根据自己的使用情况进行调整。

## 步骤

1. 更新服务文件以匹配您的环境：
   - `teamserver.service`
   - `listener.service`
   - `listener_service.cna`
2. 将服务文件复制到您的 teamserver：
   - `/etc/systemd/system/teamserver.service`
   - `/etc/systemd/system/listener.service`
   - `/etc/cobaltstrike/listener_service.cna`
3. 注册新服务：
   - 执行命令：`systemctl daemon-reload`
4. 启动服务：
   - 执行命令：`systemctl start teamserver.service`
   - 执行命令：`systemctl start listener.service`
5. 测试。

------

## Teamserver 服务配置

更新配置以匹配您的环境：

- **WorkingDirectory**: 设置为 Cobalt Strike 的目录。
- **ExecStart**: 根据您的实际情况填写值。

```bash
# teamserver.service
[Unit]
Description=Cobalt Strike Teamserver 服务
After=network.target
Wants=network.target

[Service]
Type=Simple
WorkingDirectory=/opt/cobaltstrike
ExecStart=/opt/cobaltstrike/teamserver <TEAMSERVER IP> <PASSWORD> <PATH TO C2 PROFILE>

# 示例
# ExecStart=/opt/cobaltstrike/teamserver `hostname -I` thisismypassword /opt/cobaltstrike/c2.profile

[Install]
WantedBy=multi-user.target
```

------

## Listener 服务配置

更新配置以匹配您的环境：

- **WorkingDirectory**: 设置为 Cobalt Strike 的目录。
- **ExecStart**: 根据您的实际情况填写值。

```bash
# listener.service
[Unit]
Description=Cobalt Strike Aggressor 服务
After=teamserver.service network.target
Wants=teamserver.service
StartLimitIntervalSec=33

[Service]
Restart=on-failure
RestartSec=10
WorkingDirectory=/opt/cobaltstrike
ExecStartPre=/bin/sleep 60
ExecStart=/bin/bash /opt/cobaltstrike/agscript 127.0.0.1 50050 <USER to LOGON TO COBALTSTRIKE> <TEAMSERVER PASSWORD> <PATH TO listener_service.cna>

# 示例
# ExecStart=/bin/bash /opt/cobaltstrike/agscript 127.0.0.1 50050 listener_service thisismypassword /opt/cobaltstrike/listener_service.cna

[Install]
WantedBy=multi-user.target
```

------

## 无头模式的 Aggressor 脚本

此脚本需要根据您的环境进行更新。

以下是一个示例 Aggressor 脚本，用于创建 HTTP、HTTPS 和 SMB 监听器，并包含所有必要的参数。

### 修改内容

- HTTP 监听器
  - Listenername
  - host
  - althost

- HTTPS 监听器
  - Listenername
  - host
  - althost

- SMB 监听器
  - ....

```javascript
println("
###################################################################
 CobaltStrike Aggressor 脚本          
 作者：      Joe Vest
 描述：      用于创建监听器的无头模式脚本
###################################################################");

println('加载 listener_service.cna...');

on ready {
    println('listener_service.cna: 正在创建 HTTP 监听器...');
    listener_create_ext("HTTP", "windows/beacon_http/reverse_http", %(host => "iheartredteams.com", port => 80, beacons => "iheartredteams.com", althost => "iheartredteams.com", bindto => 80));

    println('listener_service.cna: 正在创建 HTTPS 监听器...');
    listener_create_ext("HTTPS", "windows/beacon_https/reverse_https", %(host => "iheartredteams.com", port => 443, beacons => "iheartredteams.com", althost => "iheartredteams.com", bindto => 443));

    println('listener_service.cna: 正在创建 SMB 监听器...');
    listener_create_ext("SMB", "windows/beacon_bind_pipe", %(port => "mojo.5887.8051.34782273429370473##"));
    sleep(10000);
}
```

------

