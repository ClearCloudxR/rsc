# RSC Minecraft 启动脚本

红石社（RSC）整合包的第三方外置登录启动脚本。

使用第三方外置登录（Yggdrasil）时，**在无法访问境外服务器的网络环境下（例如校园网）**，本脚本可配合本地自带的 `authlib-injector.jar` 补全认证环节，让游戏正常启动，无需从境外源站拉取该文件。

---

## 背景：为什么需要这个脚本

Minecraft 正版登录依赖 Mojang 的官方认证服务器，而这些服务器在境外。校园网、部分企业内网等环境访问境外服务不稳定甚至完全不通，直接启动会卡在登录阶段。

第三方外置登录（Yggdrasil 协议，如 LittleSkin、自建皮肤站）把认证指向**国内可达的 API**，从而绕开这个问题。让原版客户端认识这套外置认证，需要一个 Java 代理：

```
authlib-injector.jar
```

它通过 `-javaagent` 注入，把客户端的认证请求重定向到你指定的 Yggdrasil API。

问题在于：`authlib-injector.jar` 的官方发布地址在境外（GitHub Releases）。**校园网等环境往往下载不到这个 jar，而它恰恰是外置登录能跑起来的前提。**

所以本仓库**直接内置了这个 jar**，随脚本一起分发。网络受限时不需要再去境外下载，脚本用本地这份文件就能完成注入，游戏可以正常启动。

---

## 文件说明

| 文件 | 说明 |
| --- | --- |
| `RSC Minecraft 启动脚本.py` | Python 源码，可自行查看、修改 |
| `RSC Minecraft 启动脚本.exe` | 由上述源码打包，双击即可运行，**无需安装 Python** |
| `authlib-injector.jar` | 外置登录注入代理，已内置，无需联网下载 |

---

## 使用前提

脚本会 `os.chdir('Redstone')`，并且按相对路径调用 Java 和游戏文件。因此：

1. 把脚本和 `authlib-injector.jar` 放到整合包的 **`Redstone` 目录下**。
2. 该目录下需存在 `.minecraft\Java25\bin\java.exe`（整合包自带的 Java 运行时）。
3. `.minecraft` 目录结构需完整（`libraries`、`versions`、`assets` 等）。

> 若不在 `Redstone` 目录下运行，脚本会提示「请在Redstone目录下运行Login.py」。这条提示文字沿用自原始脚本，未作改动。

---

## 使用方法

### 方式一：直接运行 exe（推荐，无需 Python）

双击 `RSC Minecraft 启动脚本.exe`，按提示依次输入。

### 方式二：运行 Python 源码

```bash
pip install requests
python "RSC Minecraft 启动脚本.py"
```

### 运行流程

脚本会依次询问：

1. **YGGDRASIL_API** — 你的皮肤站 Yggdrasil 接口地址。
   留空则默认使用 `https://skin.mxha.barrierslink.cn/api/yggdrasil`（MXHA 皮肤站）。
   其他示例：`https://littleskin.cn/api/yggdrasil`
2. **用户名 / 邮箱**
3. **密码**

随后脚本向 `<YGGDRASIL_API>/authserver/authenticate` 发起标准 Yggdrasil 认证请求，取回 `accessToken` 与角色 UUID，再以此参数启动游戏。

> 关于密码：脚本仅将你输入的凭据直接提交给**你自己填写的** Yggdrasil API 用于换取 token，不会保存、不会上传到其他任何地方。请自行确认所填 API 地址可信。

---

## 启动参数简述

脚本最终执行的大致命令如下（已简化）：

```
java.exe
  "-javaagent:authlib-injector.jar=<YGGDRASIL_API>"
  -Dauthlibinjector.side=client
  -cp "<整合包 libraries 依赖>"
  net.fabricmc.loader.impl.launch.knot.KnotClient
  --username <用户名>
  --uuid <角色 UUID>
  --accessToken <accessToken>
  --gameDir ".minecraft\versions\RSC(红石社)游戏优化整合"
  --versionType MXHA
```

关键点：

- `-javaagent:authlib-injector.jar=<API>` —— 注入外置登录代理，指向你的 Yggdrasil API。**这一步依赖本地已存在 `authlib-injector.jar`。**
- `-Dauthlibinjector.side=client` —— 声明客户端模式。
- `--accessToken` / `--uuid` —— 由 Yggdrasil 认证返回，替代正版登录凭据。
- `--versionType MXHA` —— 版本类型标识，随所用整合包而定。

---

## 常见问题

**Q：提示找不到 `authlib-injector.jar`？**
确认该文件与脚本位于同一目录（即 `Redstone` 目录）。本仓库已附带该文件，直接复制过去即可。

**Q：提示「请在Redstone目录下运行Login.py」？**
当前工作目录不对。把脚本放进整合包的 `Redstone` 目录后再运行。

**Q：登录失败，提示「该账号没有创建 Minecraft 角色」？**
先在皮肤站网页端创建角色，再运行脚本。

**Q：认证请求失败？**
检查 `YGGDRASIL_API` 是否填写正确、网络能否访问该地址、账号密码是否正确。部分皮肤站对请求频率有限制。

**Q：为什么脚本里 `verify=False`？**
原始脚本为兼容部分皮肤站的自签证书而关闭了 TLS 校验。这会降低连接安全性，若你的 API 使用正规证书，可自行从源码中移除该参数。

---

## 说明

- 脚本会以明文形式在终端回显认证接口的原始返回内容，其中包含 token。请勿在公共场合截图分享。
- 本仓库仅包含启动脚本与 `authlib-injector.jar`，**不含**游戏本体、整合包资源或任何 Mojang 版权文件。
- `authlib-injector` 版权归其原作者 [yushijinhun](https://github.com/yushijinhun/authlib-injector) 所有，遵循其原项目 AGPL-3.0 协议。

## 协议

本仓库中的启动脚本以 [MIT](LICENSE) 协议开源。
内置的 `authlib-injector.jar` 为第三方组件，遵循其自身协议（AGPL-3.0），不受 MIT 覆盖。
