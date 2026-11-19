## 前排劝退
**不回答“怎么用”这类问题；无前端、无订阅，专注代理本身：极致直连 + 多落地协议。**  
**仅适合对 CF 节点有一定基础的同学，至少会用节点模板修改节点信息。订阅可自行搭配 [EDT](https://github.com/cmliu/edgetunnel) 或订阅器实现。**

---
## 文件说明

| 文件 | 传输 | 协议 | 落地支持 | 备注 |
| --- | --- | --- | --- | --- |
| **worker.js** | ws / xhttp | vless / trojan / ss | `!txt + socks5 + http + https + sstp + turn + turns + global + auto` | 全功能，https / turns 为完全体 |
| **snippet.js** | ws / xhttp | vless / trojan / ss | `!txt + socks5 + http + https + sstp + turn + turns + global + auto` | 同 worker.js，但 https / turns 为非完全体 |
| **https.js** | ws / xhttp | vless / ss | `!txt + https + global + auto` | https 为完全体；无 trojan，无其它落地代理 |
| **lite.js** | ws / xhttp | vless / trojan / ss | `!txt + auto` | 精简版，仅 直连 + proxyip，无 global、无落地代理，冷启快，性能预期略好于其它三个 |

_建议：ss 用 notls。_  
_注1：ss 仅 ws，无 xhttp。_  
_注2：代码验证基于 Pro 计划 snippet，worker free。_

---
## 功能说明

1. **!txt：** 域名加 `!txt` 后缀时，取其 TXT 记录值作为 proxyip 或协议代理（多个值以 `,` 或换行分隔，随机取一条）；普通 A 记录域名无需加。**四个文件均支持。**
2. **socks5：** `socks5://[user:pass@]host:port`（`socks://` 同样接受）。**worker.js / snippet.js。**
3. **http：** `http://[user:pass@]host:port`。**worker.js / snippet.js。**
4. **https：** 完全体 `https://domain:port` 走 CF secureTransport，`https://ip:port` 与 `https://host:port!ip` 走内置 TlsClient；非完全体仅支持 `https://domain:port`。**worker.js / https.js 为完全体，snippet.js 为非完全体。**
5. **sstp：** `sstp://[user:pass@]host:port`，默认用户密码 `vpn`。**worker.js / snippet.js。**
6. **turn：** `turn://host:port`，默认端口 3478。**worker.js / snippet.js。**
7. **turns：** turn over tls，默认端口 5349；完全体支持 `turns://domain:port`、`turns://ip:port` 与 `!ip` 后缀，非完全体仅 `turns://domain:port`。**worker.js 为完全体，snippet.js 为非完全体。**
8. **global：** 协议代理（socks5 等）默认“先试直连、失败再走代理”的回落模式，`?global=1` 改为**跳过直连、全局直接走路径代理**：优先于 `auto` 与 AI/SNI 判定，`p` 为空时回落 `zj`。路径内嵌 query（`/fdip={proxy}?global=1`）与 header `global: 1` 同样识别，兼容 sing-box 把 path 里的 `?` 转义成 `%3F`。**worker.js / snippet.js / https.js。**
9. **auto：** ZJ 自适应 cf 官方 proxyip 服务，`zj` 按 colo 生成：colo 为 `HKG` 时 AI 域名回落 `p`（`p` 为空则 `zj`），其它（含非 HKG 的 AI）回落 `zj`；`auto=0` 关闭 `zj`，强制走 `p`（`p` 为空则 `zj`）。**四个文件均支持。**

**说明：** `p` = 路径指定的代理，`zj` = 按 colo 生成的 ZJ 官方 proxyip；AI 域名 = 代码里的 `AI` 常量，目标以 IP 形式送入时用 TLS SNI 判定。  
**注：** `auto=1` / `auto=2` 不再特殊处理，与不设 auto 等价；目标为 IP 时最多等首块数据 200 ms 做 SNI 判定，超时按 `zj`；`global=1` 时不做 SNI 判定、也不等首块数据（直接走 `p`）；AI 域名任意 colo 跳过直连直接回落，直连失败的主机记忆 60 s（上限 200 条，随 isolate 重置），期间同样跳过直连。  
**注：** TXT 内容以 `,` 分隔、换行或两者混用；这些功能解决的是 CF 节点的落地问题。

---
## 路径示例

路径统一为 `/{任意字母数字}=代理`（如 `/fdip=...`，`fdip` 可换成任意字母数字组合），`ed=2560` 放最后。

| 用途 | 路径 | worker.js | snippet.js | https.js | lite.js |
| --- | --- | :---: | :---: | :---: | :---: |
| 直连 / proxyip | `/fdip=1.2.3.4:443?ed=2560` | ✓ | ✓ | ✓ | ✓ |
| TXT 记录 | `/fdip=domain!txt?ed=2560` | ✓ | ✓ | ✓ | ✓ |
| socks5 | `/fdip=socks5://host:port?ed=2560` | ✓ | ✓ | — | — |
| http | `/fdip=http://host:port?ed=2560` | ✓ | ✓ | — | — |
| https（域名） | `/fdip=https://domain:port?ed=2560` | ✓ | ✓ | ✓ | — |
| https（IP） | `/fdip=https://ip:port?ed=2560` | ✓ | — | ✓ | — |
| https（强制 TlsClient） | `/fdip=https://host:port!ip?ed=2560` | ✓ | — | ✓ | — |
| sstp | `/fdip=sstp://host:port?ed=2560` | ✓ | ✓ | — | — |
| turn | `/fdip=turn://host:port?ed=2560` | ✓ | ✓ | — | — |
| turns（域名） | `/fdip=turns://domain:port?ed=2560` | ✓ | ✓ | — | — |
| turns（IP / `!ip`） | `/fdip=turns://ip:port?ed=2560` | ✓ | — | — | — |
| global | `/fdip={proxy}?global=1&ed=2560` | ✓ | ✓ | ✓ | — |
| auto | `/?auto=0&ed=2560`、`/fdip={proxy}?auto=0&ed=2560` | ✓ | ✓ | ✓ | ✓ |

---
## 节点示例

**Vless ws**
```ws
vless://495c7195-85b8-498a-bf20-2ea9ce9175b5@www.shopify.com:443?path=%2Ffdip%3D1.2.3.4%3A443%3Fed%3D2560&security=tls&encryption=none&insecure=0&host=vless.snippets.cf&fp=chrome&type=ws&allowInsecure=0&sni=vless.snippets.cf#ws
```
**Vless xhttp**
```xhttp
vless://495c7195-85b8-498a-bf20-2ea9ce9175b5@www.shopify.com:443?mode=stream-one&path=%2Ffdip%3D1.2.3.4%3A443%3Fed%3D2560&security=tls&alpn=h2&encryption=none&insecure=0&host=vless.snippets.cf&fp=chrome&type=xhttp&allowInsecure=0&sni=vless.snippets.cf#xhttp
```
**Trojan ws**
```ws
trojan://495c7195-85b8-498a-bf20-2ea9ce9175b5@www.shopify.com:443?path=%2Ffdip%3D1.2.3.4.%3A443%3Fed%3D2560&security=tls&insecure=0&host=trojan.snippet.cf&fp=chrome&type=ws&allowInsecure=0&sni=trojan.snippet.cf#ws
```
**Trojan xhttp**
```xhttp
trojan://495c7195-85b8-498a-bf20-2ea9ce9175b5@www.shopify.com:443?mode=stream-one&path=%2Ffdip%3D1.2.3.4.%3A443%3Fed%3D2560&security=tls&alpn=h2&insecure=0&host=trojan.snippet.cf&fp=chrome&type=xhttp&allowInsecure=0&sni=trojan.snippet.cf#xhttp
```
**SS(notls) ws**
```ws
ss://YWVzLTEyOC1nY206NDk1YzcxOTUtODViOC00OThhLWJmMjAtMmVhOWNlOTE3NWI1@www.shopify.com:80?plugin=v2ray-plugin%3Bmode%3Dwebsocket%3Bhost%3Dss.snippets.cf%3Bpath%3D%2Ffdip%3D1.2.3.4%3A443%3Fed%3D2560%3Bmux%3D0#ws
```
<details>
<summary>xhttp extra（留空 或 填入以下内容 或 自行配置，效果自测）</summary>

```json
{
"extra": {
  "noGRPCHeader": true,
  "headers": {
    "Content-Type": "application/octet-stream"
  },
  "xPaddingBytes": "100-1000",
  "xPaddingObfsMode": true,
  "xPaddingMethod": "tokenish",
  "xPaddingPlacement": "queryInHeader",
  "xPaddingHeader": "X-Cache",
  "xPaddingKey": "_dc"
}
}
```
</details>

---
## 特别提醒
**若 1101 请全删旧片段再部署，已有正常运行中的片段需谨慎，部署新片段会触发全部片段代码检测。**  
**有问题请开 issue 或联系 [tg bot](https://t.me/meindmBot) 直奔主题，欢迎反馈，欢迎 PR。**

---
## 鸣谢
**[AK](https://github.com/ToiCF)、[CM](https://github.com/cmliu)、[ZJ](https://github.com/1345695)、AI**

---
## 开源协议
**[MIT](LICENSE) © 2026 gabou** —— 自由使用、修改、分发（含商用），保留版权与许可声明即可；软件按「原样」提供，不附带任何担保。

---
## Star History
[![Star History Chart](https://api.star-history.com/svg?repos=jacobax/snippets&type=Date)](https://star-history.com/#jacobax/snippets&Date)

> 如有帮助，欢迎点 ⭐，权当鼓励。
