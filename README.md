GoIndex 部署与配置指南 (KV 优化版)
本指南将引导您完成 GoIndex 优化版脚本的完整部署流程，包括启用 KV 缓存以获得最佳性能。

零、最简单的方式：一键部署
这种方法通过一个预设的模板，让您只需点击一个按钮即可完成代码和基础环境的部署，之后只需在网页上填写您的个人凭证即可。

操作流程:

点击下方按钮进行部署：

授权并创建项目：

在打开的 Cloudflare 页面，使用您的账户登录。

Cloudflare 会要求您授权从 GitHub 克隆仓库，请允许。

为您的新项目输入一个项目名称（例如 my-goindex-drive），然后点击 创建并部署 (Create and Deploy)。

Cloudflare 会自动为您完成部署，并创建和绑定所需的 KV 命名空间。

获取 Google API 凭证：

部署正在进行时，您可以开始参照下方的 “手动部署 - 第一步” 来获取您的 client_id、client_secret 和 refresh_token。

配置环境变量 (关键步骤):

部署完成后，进入您新创建的项目仪表板。

点击 设置 (Settings) -> 变量 (Variables) -> 添加变量 (Add variable)。

您需要添加以下 4 个必需的环境变量，将您获取的凭证填入其中：

CLIENT_ID: 您的 Google client_id。

CLIENT_SECRET: 您的 Google client_secret。

REFRESH_TOKEN: 您的 Google refresh_token。

ROOTS_CONFIG: 您的网盘配置。请注意，这是一个 JSON 字符串。

简单示例 (单个个人盘):
[{"id":"root","name":"My Drive","pass":""}]

复杂示例 (一个团队盘和一个带密码的文件夹):
[{"id":"TEAMDRIVE_ID_HERE","name":"Team Drive","pass":""},{"id":"FOLDER_ID_HERE","name":"Private Folder","pass":"123456"}]

添加完变量后，点击 保存 (Save)。

重新部署以应用变量：

回到项目的 部署 (Deployments) 选项卡，找到最新的部署记录，点击 重新部署 (Redeploy)。

等待部署完成后，您的网站就可以通过 “第四步：完成与访问” 中的方法正常访问了。

手动部署与配置
如果您希望对代码进行更深度的自定义，或者不想使用 GitHub 模板，可以采用以下手动部署流程。

手动部署 - 第一步：获取 Google Drive API 凭证
在配置脚本之前，您需要从 Google 获取 client_id、client_secret 和 refresh_token。

访问 Google Cloud Console: 前往 Google Cloud Console。

创建项目: 如果您还没有项目，请创建一个新项目。

启用 Google Drive API:

在顶部搜索框中搜索 "Google Drive API" 并进入。

点击 启用 (Enable)。

配置 OAuth 同意屏幕:

在左侧菜单，转到 API 与服务 -> OAuth 同意屏幕。

选择 外部 (External)，然后点击 创建 (Create)。

填写应用名称（例如 "My GoIndex"），并提供您的用户支持电子邮箱和开发者联系信息。

在“测试用户”步骤中，必须添加您自己的 Google 账户作为测试用户，否则授权会失败。

创建凭据 (client_id & client_secret):

在左侧菜单，转到 API 与服务 -> 凭据 (Credentials)。

点击 + 创建凭据 (+ CREATE CREDENTIALS) -> OAuth 客户端 ID (OAuth client ID)。

应用类型 (Application type): 选择 Web 应用 (Web application)。

已获授权的 JavaScript 来源 (Authorized JavaScript origins): 添加 https://developers.google.com。

已获授权的重定向 URI (Authorized redirect URIs): 添加 https://developers.google.com/oauthplayground。

点击 创建 (CREATE)。您将获得您的 客户端 ID (client_id) 和 客户端密钥 (client_secret)。请复制并妥善保管它们。

获取刷新令牌 (refresh_token):

访问 Google OAuth Playground。

点击右上角的齿轮图标 ⚙️，勾选 Use your own OAuth credentials，然后输入您上一步获取的 client_id 和 client_secret。

在左侧的 API 列表中，找到并展开 "Drive API v3"，然后勾选 https://www.googleapis.com/auth/drive。

点击蓝色的 Authorize APIs 按钮，并用您添加为测试用户的 Google 账户完成授权流程。

授权成功后，点击 Exchange authorization code for tokens。您将在右侧看到 Refresh token。这就是您需要的最后一个凭证。

手动部署 - 第二步：部署到 Cloudflare Workers
登录 Cloudflare: 访问 Cloudflare 官网 并登录。

创建 Worker: 在您的仪表板左侧菜单中，找到并点击 Workers & Pages -> 创建应用程序 (Create Application) -> 创建 Worker (Create Worker)。为您的 Worker 指定一个子域名，然后点击 部署 (Deploy)。

粘贴代码: 点击 编辑代码 (Edit code)，将上面提供的完整代码复制并粘贴到编辑器中，完全替换掉所有默认代码。

填写凭证: 在代码顶部的 authConfig 部分，准确填入您刚刚获取的 client_id, client_secret, 和 refresh_token。同时，根据您的需要配置 roots 盘符列表。

手动部署 - 第三步：配置 KV 缓存 (性能优化的核心)
这是整个优化流程中最关键的一步。通过绑定 KV 存储，您的 Worker 才能将数据缓存起来，实现性能的飞跃。

创建 KV 命名空间:

在 Cloudflare 仪表板中，转到 Workers & Pages -> KV。

点击 创建命名空间 (Create a namespace)。

输入一个易于识别的名称，例如 GOINDEX_CACHE，然后点击 添加 (Add)。

绑定 KV 到 Worker:

回到您的 Worker (Workers & Pages -> 点击您的 Worker 名称)。

点击 设置 (Settings) -> 变量 (Variables)。

向下滚动到 KV 命名空间绑定 (KV Namespace Bindings)，点击 添加绑定 (Add binding)。

变量名称 (Variable name): 至关重要！ 必须准确无误地填写 GD_INDEX_CACHE。

大小写必须完全一致。

不能有任何拼写错误或多余的空格。

KV 命名空间 (KV namespace): 从下拉列表中选择您刚刚创建的 GOINDEX_CACHE。

点击 保存并部署 (Save and deploy)。

验证绑定是否成功:

完成绑定和部署后，访问您的任何一个盘符链接（例如 .../0:/）。

刷新几次页面，然后回到 KV 菜单中查看您创建的 GOINDEX_CACHE 命名空间。

如果您能看到里面出现了新的键值对 (Key-value pairs)，则证明缓存已成功启用。如果为空，请仔细检查第二步中的“变量名称”是否完全正确。

手动部署 - 第四步：完成与访问
最终部署: 在 Worker 编辑器页面，点击右上角的 保存并部署 (Save and deploy)。

访问您的网站: 部署成功后，返回 Worker 的概览页面，您会看到您的网站 URL。要访问您配置的网盘，请使用以下格式：
https://<你的Worker地址>/<盘符序号>:/

<盘符序号> 对应 roots 数组中的位置，从 0 开始。

例如，要访问第一个配置的盘符，URL 为: https://your-worker.your-name.workers.dev/0:/

要访问第二个，URL 为: https://your-worker.your-name.workers.dev/1:/

现在，您的 GoIndex 网站已成功部署并启用了性能优化。

手动部署 - 第五步：【开发者专属】部署流程优化 (使用 Wrangler CLI)
对于熟悉命令行的开发者，可以通过 Wrangler CLI 实现自动化部署，无需手动操作网页界面。

准备环境:

确保您的电脑上已安装 Node.js (建议使用 LTS 版本)。

在本地创建一个项目文件夹，并将完整的 GoIndex 脚本代码保存为 index.js 文件。

安装并登录 Wrangler:

打开终端或命令提示符，运行以下命令来安装 Wrangler：

npm install -g wrangler

运行登录命令，它会打开浏览器让您授权 Wrangler 访问您的 Cloudflare 账户：

wrangler login

创建 wrangler.toml 配置文件:

在您的项目文件夹中，创建一个名为 wrangler.toml 的文件。

将以下内容复制到文件中，并根据注释修改为您自己的信息：

# wrangler.toml

# 您的 Worker 名称，可以自定义
name = "my-goindex-worker"

# 主入口脚本文件
main = "index.js"

# 兼容性日期，保持最新即可
compatibility_date = "2024-06-10"

# --- KV 命名空间绑定 ---
# 这是自动创建并绑定 KV 的核心配置
[[kv_namespaces]]
# 这里的 "GD_INDEX_CACHE" 必须与 index.js 代码中的变量名完全一致
binding = "GD_INDEX_CACHE"
# 您可以为将要创建的 KV 命名空间指定一个预览版和正式版的 ID
# Wrangler 会在首次部署时自动为您创建这个命名空间
id = "" # 将来部署后会自动填充
preview_id = "" # 将来部署后会自动填充

一键部署:

在您的项目文件夹中打开终端，运行部署命令：

wrangler deploy

Wrangler 会自动读取 wrangler.toml 文件，创建 GD_INDEX_CACHE 命名空间，并将其绑定到您的 Worker，最后上传您的 index.js 脚本。整个过程一步到位。

通过以上步骤，您可以实现完全通过代码和命令行来管理和部署您的 GoIndex 项目，极大地提高了效率和可维护性。

手动部署 - 第六步：【高级自动化】“一键部署” (关联 GitHub)
这种方法将您的 Worker 与一个 GitHub 仓库关联起来。之后，您对代码的所有更新都只需推送到 GitHub，Cloudflare 会自动完成后续的部署、绑定等所有操作。

准备 GitHub 仓库:

在 GitHub 上创建一个新的仓库 (可以是私有的)。

将您本地项目文件夹中的 index.js (即 GoIndex 完整脚本) 和 wrangler.toml (来自第五步) 这两个文件推送到该仓库。

连接 Cloudflare 与 GitHub:

在 Cloudflare 仪表板，进入您的 Worker (Workers & Pages -> 您的 Worker 名称)。

点击 部署 (Deployments) 选项卡。

在页面右侧找到并点击 连接到 Git (Connect to Git)。

授权并选择仓库:

在弹出的窗口中，选择 连接 GitHub (Connect GitHub) 并授权 Cloudflare 访问。

选择您的账户，然后选择 所有仓库 (All repositories) 或 仅选择仓库 (Only select repositories) -> 找到您刚创建的那个仓库。

点击 安装并授权 (Install & Authorize)。

配置自动部署:

回到 Cloudflare，您现在可以配置部署选项。

生产分支 (Production branch): 选择 main (或您仓库的主分支名称)。

项目根目录 (Root directory): 保持为 / 即可。

点击 保存并部署 (Save and Deploy)。

体验自动化流程:

Cloudflare 将立即根据您 GitHub 仓库中的代码和 wrangler.toml 配置进行首次部署。它会自动处理 KV 的创建和绑定。

从现在开始，您的工作流程变为：

在本地修改 index.js 文件 (例如，更改网站名称)。

使用 git commit 和 git push 命令将修改推送到 GitHub。

完成！ Cloudflare 会自动检测到更新，并在几分钟内完成所有部署工作。您无需再登录 Cloudflare 界面进行任何手动操作。
