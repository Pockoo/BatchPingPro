# BatchPingPro

Windows 下的批量 IP 测速小工具。把一堆 IP 丢进去，几秒钟就能看出哪些通、哪些快、各自在哪个国家和运营商，适合从 CDN 或代理节点的 IP 池里挑可用的那几个。

纯 C++ / Win32 编写，单个 exe，绿色免安装，没有第三方依赖。

## 能做什么

**导入 IP**

- 直接在左侧文本框粘贴，或者拖文件进窗口、点“导入文件”。
- 支持单个 IP、`IP:端口`、CIDR（`1.1.1.0/24`）、范围（`1.1.1.1-1.1.1.20`），后两种也可以带端口。
- 网页源码、JSON、CSV 里的 IP 也能提取，包括 HTML 表格里相邻单元格的 IP 和端口、`"ip":"…","port":…` 这类写法。
- “导入文件”按钮右键可以填一个网址，程序会下载该网页并提取里面的 IP。

**四种测试方式**

| 方式 | 做什么 |
|---|---|
| ICMPING | 普通 ping，只测延迟 |
| TCPING | 对 IP:端口 做 TCP 握手，耗时即延迟（如果ICPING超时而TCPING有延迟则说明服务器禁PING） |
| SNIPROXY | 直连 IP:端口，带上指定域名的 SNI / Host 请求，STATUS 列显示 HTTP 状态码，LOADING 列显示访问耗时（端口 80 走 HTTP，其余走 HTTPS，可用来测试 cloudflare 反代节点是否可用） |
| DNS | 把 IP 当 DNS 服务器，解析指定域名的 A / AAAA 记录；STATUS 列显示解析到的记录数，LOADING 列显示解析耗时 |

后三种会同时测 ICMPING，结果各占一列。不写端口时，TCPING 默认 80，SNIPROXY 默认 443，DNS 默认 53，也可以在端口框里统一指定。

**其他**

- 并发数 16–512、超时 500–5000 ms、每个 IP 发包 1–4 次，都可以选。
- 可以只测超时项、只测没测过的项，方便补测。
- 相同目标自动去重：同一个 IP 的 ICMP 只测一次，同一个“IP+端口”的 TCP 只测一次。
- 每列都能点表头排序，延迟按快慢着色。
- 双击某一行单独重测；SNIPROXY 会弹出响应头，DNS 会弹出解析到的所有地址。
- 右键菜单：复制 IP / IP+端口 / 整行、用浏览器打开（HTTP 或 HTTPS）、路由追踪。
- 导出 TXT（只导出可达的 IP）或 CSV（全部列）。
- 支持 SOCKS5 代理（无认证），用于 TCPING 和 SNIPROXY。

**归属地与 ASN**

- 离线：读取 `ip2region_v4.xdb` 和 `ip2asn-combined.tsv`（也可以是 `.gz`，边解压边读，内存占用很小）。
- 在线：没有离线库时自动使用 ip-api.com，失败时切换到 ipwho.is，不需要密钥。
- 设置里可以固定用在线或离线，默认自动选择。

**路由追踪**

右键选一个 IP 就能开，每一跳显示 IP、三次延迟、平均值、ASN、归属地和主机名。默认所有 TTL 同时发出，结果按跳数实时出现；也可以在设置里改成逐跳发送。可以同时开多个窗口。

## 使用

1. 到 Releases 下载压缩包，解压后运行 `BatchPingPro.exe`。
2. 想要离线归属地，把下面两个文件放在 exe 同目录（`ip2asn` 的 tsv 也可以放在 `data` 子目录或上一级目录）：
   - `ip2region_v4.xdb`，来自 [ip2region](https://github.com/lionsoul2014/ip2region)
   - `ip2asn-combined.tsv` 或 `ip2asn-combined.tsv.gz`，来自 [iptoasn.com](https://iptoasn.com/)
3. 粘贴或导入 IP，选测试方式，点“开始”。

不放这两个文件也能用，归属地会走在线接口。

程序产生的文件都在 exe 所在目录，不写注册表，删掉文件夹就是卸载：

| 文件 | 用途 |
|---|---|
| `BatchPingPro.ini` | 设置，退出时自动保存 |
| `port.txt` | 端口下拉列表，每行一个 |
| `snitest.txt` | SNIPROXY 的测试域名，每行一个，`#` 开头为注释 |
| `dnstest.txt` | DNS 测试域名，每行一个 |
| `crash.log` | 出现未处理异常时才会生成 |

三个 txt 缺失时会自动生成默认内容，可以按需修改。

## 一些限制

- 目前只支持 IPv4，CIDR 和范围只展开 1.0.0.0–223.255.255.255。
- SOCKS5 转发不了 ICMP 和 UDP，所以启用代理后只能用 TCPING 和 SNIPROXY，DNS 测试不可用。
- SNIPROXY 的 HTTPS 使用系统 SChannel，支持 TLS 1.0–1.2。
- 在线归属地接口有限流，一次导入很多 IP 时会分批查询，需要等一会儿。

## 自己编译

暂时未开源

## 反馈

使用中遇到问题或有想法，欢迎到 [Issues](https://github.com/Pockoo/BatchPingPro/issues) 留言。
