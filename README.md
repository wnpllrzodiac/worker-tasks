# 自动化任务

基于 `CloudFlare Workers` 的自动化**任务**。

## 当前支持

| 任务名称                                    | 任务简介                     | 说明                                                                        |
| :------------------------------------------ | :--------------------------- | :-------------------------------------------------------------------------- |
| [MegStudio](https://studio.brainpp.com)签到 | AI 免费算力                  | 账号与密码，需自建 [OCR API 服务](https://github.com/sml2h3/ocr_api_server) |
| [v2ex](https://v2ex.com)签到                | 开发者社交平台               | 获取网页的 Cookie                                                           |
| **域名可注册检测**                          | 查询单个或多个域名是否可注册 | 手动设置相关域名                                                            |

## 部署教程

### 通过 GitHub Actions 发布至 CloudFlare

1. 从 CloudFlare 获取 `CLOUDFLARE_API_TOKEN` 值，并设置到项目。

   > `https://github.com/<ORG>/<REPO>/settings/secrets/actions`

2. 可选）。设置`别名`。创建 `KV`、，并绑定到此 Workers 服务。
   - 2.1a 手动后台绑定，（`Settings` -> `Variables` -> `KV Namespace Bindings` -> `Add binding` -> `Variable name (data)`, `选择创建的 KV`）
   - 2.1b 通过命令行创建。按照**本地部署**的第 6 步，创建和保存 `KV`

### 本地部署到 CloudFlare

1. 注册 [CloudFlare 账号](https://www.cloudflare.com/)，并且设置 **Workers** 域名 (比如：`xxx.workers.dev`)

2. 安装 [Wrangler 命令行工具](https://developers.cloudflare.com/workers/wrangler/)。
   ```bash
    npm install -g wrangler
   ```
3. 登录 `Wrangler`（可能需要扶梯）：

   ```bash
   # 若登录不成功，可能需要使用代理。
   wrangler login
   ```

4. 拉取本项目,并进入该项目目录：

   ```bash
   git clone https://github.com/servless/worker-tasks.git

   cd worker-tasks
   ```

5. 修改 `wrangler.toml` 文件中的 `name`（proj）为服务名 `tasks`（访问域名为：`tasks.xxx.workers.dev`）

6. 创建 **Workers** 和 **KV**，并绑定 `KV` 到 `Workers`

   1. **创建 KV，并设置参数值**

      1. 创建名为 `data` 的 `namespace`（最终会在前缀加上*服务名*，即为 `tasks-data`）

         ```bash
            wrangler kv:namespace create data
         ```

         得到

         ```bash
            ⛅️ wrangler 2.15.1
            --------------------
            🌀 Creating namespace with title "tasks-data"
            ✨ Success!
            Add the following to your configuration file in your kv_namespaces array:
            { binding = "data", id = "8c7d7ee9b6bb4f8fa3ca9f30eaf8d897" }

         ```

         将上述命令得到的 `kv_namespaces` 保存到 `wrangler.toml` 中，即

         ```bash
            # 替换当前项目该文件内相关的数据，即只需要将 id 的值替换为上一步骤得到的值
            { binding = "data", id = "8c7d7ee9b6bb4f8fa3ca9f30eaf8d897" }
         ```

   2. 先通过后面的教程，获取到*对应服务*的 `cookie`、`tokens`和参数值，填充到 `tasks-data` 的 **KV** 中。

   3. 将*对应服务*的**数据**值保存到 `KV namespace`

      ```bash
         # V2ex
         ## 通过电脑浏览器提取 cookie
         # wrangler kv:key put --binding=data 'v2ex' '<COOKE_VALUE>' # not work
         npx wrangler --binding=taskdata kv key put 'v2ex' '<COOKIE_VALUE>'

         # MegStudio
         ## 使用账号和密码，需要自建 OCR API 服务：https://github.com/sml2h3/ocr_api_server。
         ## 支持多账号。每个用户名之间、每个密码之间使用分号(;)分隔。
         wrangler kv:key put --binding=data 'megstudio_username' 'USERNAME1;USERNAME2;USERNAME3'
         wrangler kv:key put --binding=data 'megstudio_password' 'PASSWORD1;PASSWORD2;PASSWORD3'
         wrangler kv:key put --binding=data 'ocr_url' "https://ocr.xx.com"

         # Find Domains
         ## 查询域名是否可注册。
         ## 支持多域名。域名之间使用分号(;)分隔。
         wrangler kv:key put --binding=data 'domains' "example1.com;example2.com;example3.com"

      ```

7. 修改定时任务相关信息

   ```bash
   # 按照 Linux 定时任务的格式修改
   #
   crons = ["* * * * *"]
   ```

   [crontab 文档](https://www.man7.org/linux/man-pages/man5/crontab.5.html)

8. 发布

   ```bash
    wrangler deploy
   ```

   发布成功将会显示对应的网址

   ```bash
    Proxy environment variables detected. We'll use your proxy for fetch requests.
   ⛅️ wrangler 2.13.0
        --------------------
        Total Upload: 0.66 KiB / gzip: 0.35 KiB
        Uploaded tasks (1.38 sec)
        Published tasks (4.55 sec)
                https://tasks.xxx.workers.dev
        Current Deployment ID:  xxxx.xxxx.xxxx.xxxx
   ```

## 选项

### 推送通知

1. [**Bark** (iOS 端)](https://bark.day.app/)

```bash
# 设置 brak token
wrangler kv:key put --binding=data 'bark' '<BARK_TOKEN>'
```

2. [**Lark**](https://open.larksuite.com/document/client-docs/bot-v3/add-custom-bot#756b882f)

```bash
# 设置 brak token
wrangler kv:key put --binding=data 'lark' '<LARK_TOKEN>'
```

3. [**飞书**](https://open.feishu.cn/document/client-docs/bot-v3/add-custom-bot#756b882f)

```bash
# 设置 brak token
wrangler kv:key put --binding=data 'feishu' '<FEISHU_TOKEN>'
```

若不需要通知，删除对应的 `key` 即可

```bash
# 以 bark 为例
wrangler kv:key delete --binding=data 'bark'
```

## 帮助

### 获取网页 `cookie` 的方法

1. 首先使用 chrome 浏览器打开网站（比如为 `xxx.com`）， 登录账号。
2. Windows / Linux 系统可按 `F12` 快捷键打开开发者工具；Mac 快捷键 `option + command + i`；Linux 还有另一个快捷键 `Ctrl + Shift + i`。笔记本电脑可能需要再加一个 `fn` 键。
3. 选择开发者工具 `Network`，刷新页面，选择第一个`xxx.com`, 找到 `Requests Headers` 里的 `Cookie`。

### 获取 App `cookie` 的方法

使用 **[Reqable](https://reqable.com/)** 软件抓包获取对应的 cookie 值。

### 调试

1. 创建预览环境

   ```bash
   wrangler kv:namespace create data --preview
   ```

   得到

   ```bash
   { binding = "data", preview_id = "d5d5f6d84098496ead8c89667dcea788" }
   ```

   将 `preview_id` 添加到 `warngler.toml`，即

   ```bash
   kv_namespaces = [
   { binding = "data", id = "c63f7dad63014a70847d96b900a4fc3f", preview_id = "d5d5f6d84098496ead8c89667dcea788"}
   ]
   ```

2. 将相关值保存到 `KV namespace`，即每条命令后均添加参数 `--preview`

   ```bash
      wrangler kv key put 'v2ex' '<COOKE_VALUE>' --binding=taskdata --preview
      wrangler kv key put 'bark' '<BARK_TOKEN>' --binding=taskdata --preview
   ```

3. 执行调试命令

   ```bash
   wrangler dev --test-scheduled
   ```

   显示

   ```bash
   ⛅️ wrangler 2.15.1
   --------------------
   Your worker has access to the following bindings:
   - KV Namespaces:
   - data: d5d5f6d84098496ead8c89667dcea788
   ⬣ Listening at http://0.0.0.0:8787
   - http://127.0.0.1:8787
   - http://192.168.33.66:8787
   ╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
   │ [b] open a browser, [d] open Devtools, [l] turn on local mode, [c] clear console, [x] to exit
   ```

   按 `l` 显示相关的调试数据
   http://localhost:8787/__scheduled?cron=*+*+*+*+*%22

## 仓库镜像

- https://git.jetsung.com/servless/worker-tasks
- https://framagit.org/servless/worker-tasks
- https://github.com/servless/worker-tasks
