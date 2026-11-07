# EudicSyncToMaiMemo

把欧路独立收藏的单词、短语自动增量添加到墨墨云词本。

Fork 自 [viazure/EudicSyncToMaiMemo](https://github.com/viazure/EudicSyncToMaiMemo)，保留原作者 viazure 的署名及 [MIT License](LICENSE)。本 Fork 增加严格标题筛选、安全增量合并、写后校验、OAuth 刷新和只读诊断。通知功能参考 [jeessy2/ddns-go](https://github.com/jeessy2/ddns-go)。

## 工作原理与边界
（增加完整的图示形式,完整的表达Github连接欧路和墨墨词库的技术图解，并解释如何完整的自动更新的定期任务）
- 欧路新增 → 墨墨追加；欧路取消收藏 → 墨墨保留历史词汇。
- 只接受完整、有效的独立收藏标题；不拆短语、不生成词形、不从笔记或 Markdown 猜标题。
- 完整标题精确去重，区分大小写；没有新增就不写入。
- 原有合法标题、章节、空行及换行完整保留。非法行、重复标题及非章节结构阻止写入。
- 写后重新 GET：正文必须与计划追加结果完全一致，标题集合等于“原有标题 ∪ 当前有效源标题”，再次模拟新增应为零。
- 不修改欧路，不自动清空或重建词本。
- 墨墨部分未收录短语可能显示“自编此单词”，仍需在 App 中补充提交；不会自动生成释义。

## 推荐部署：GitHub Actions
1，先安装必要的软件和环境，否则无法执行后续的操作，Node.js 22+对应的终端的命令npx ，Git 对应的终端的命令git，GitHub CLI对应的终端的命令，PowerShell/CMD
2，需要欧路和墨墨账号授权
3，首次授权在本机完成，随后云端每天自动化更新运行，电脑无需常开。

### 1. Fork 并克隆自己的仓库（Fork指的是创建一个源代码的独立分支到自己的仓库，自己是可以修改fork后的所有内容，也可以更新源代码的内容，也可将自己修改的内容给原作者，审批后可将自己修改的内容更新的源码中）

点击 GitHub 的 **Fork**，保留默认分支 main。克隆自己的 Fork：
```powershell
git clone https://github.com/ShalliChan/EudicSyncToMaiMemo
cd EudicSyncToMaiMemo
git remote get-url origin
```
git clone https://github.com/ShalliChan/EudicSyncToMaiMemo
这一步是将源代码，下载到本地，直接下载源代码也可以
cd EudicSyncToMaiMemo
这一步是将powershell的地址，定位到源代码中的EudicSyncToMaiMemo文件夹
git remote get-url origin
（这段命令行代码不太确定具体的原因）确认 origin 指向自己，不是上游。不要提交任何授权凭证。

### 2. 墨墨首次授权

```powershell
npx --yes --package=@maimemo/memo-api@1.3.1 memo-api auth login
gh auth login --hostname github.com --web --git-protocol https
pwsh -File ./scripts/import-maimemo-oauth.ps1
```
npx --yes --package=@maimemo/memo-api@1.3.1 memo-api auth login
这一段是请求授权命令，注意必须安装node.js,否则无法识别，输入完后。会出现一个链接，打开后，登录momo，会出现登录成功
<img width="1201" height="116" alt="image" src="https://github.com/user-attachments/assets/356ea564-35e4-4c90-9657-337cfea6d1e5" />
gh auth login --hostname github.com --web --git-protocol https
这一段是请求github的授权，这段会有多个问题
1，powershell无法执行的命令行的情况，解决方案使用CMD替换：Win+r，运行窗口中输入CMD,然后导航到EudicSyncToMaiMemo文件夹下 cd C:\Users\xxx\...\EudicSyncToMaiMemo>git remote get-url origin
2，无法识别gh 命令，需要安装GitHub CLI应用
3，网络问题，无法进入github网站，解决方案：使用VPN，并必须使用全局模式，如果是规则模式，需要设置网络环境（AI需要把网络IP地址的方法添加进来）
<img width="1617" height="123" alt="image" src="https://github.com/user-attachments/assets/28e79b73-ee9f-49bb-b589-45141af23126" />
如果一切顺利，会跳出一个登录GitHub网站界面，这时注意终端中的one-time code的八位验证码，输入到登录GIthub的界面中即完成授权
<img width="1143" height="175" alt="image" src="https://github.com/user-attachments/assets/f9e2e439-726d-4ef4-9256-b5f060693b58" />
pwsh -File ./scripts/import-maimemo-oauth.ps1
这一步通过运行/scripts文件夹下的程序import-maimemo-oauth.ps1，然后连接墨墨应用中的数据库
注意：如果使用的CMD，会需要无法识别pwsh命令，这时需要写完整的签名
powershell -ExecutionPolicy Bypass -File .\scripts\import-maimemo-oauth.ps1
<img width="1278" height="78" alt="image" src="https://github.com/user-attachments/assets/aaff0a58-e6d2-43c6-abfc-5afd5e4cfd78" />

（没有看懂这段的意思：在浏览器登录拥有词本的墨墨账号并授权。导入脚本显示当前 origin 仓库；检查后输入完整的“账号/仓库名”确认。脚本确认仓库写权限后，通过标准输入上传刷新凭证，不打印凭证或上传整个配置文件。）

（这里写的不准确：成功后自己的 Fork 会出现 MAIMEMO_OIDC_CREDENTIALS Secret。不要将本机凭证文件粘贴到聊天、Issue、仓库或构建产物。）
执行成功后，是在GitHub中的的settings-Secrets and varibles-Actions中Respository secrets中出现
<img width="760" height="254" alt="image" src="https://github.com/user-attachments/assets/ce2c0b31-08b6-48f6-b63f-f717fe2089f9" />
到这一步连接和授权的工作基本完成，后续才开始真正的打通墨墨-GitHub-欧路词典的交互

### 3. 配置自己的账号和词本
去到手机墨墨APP,在云词典出创建名称为 **欧路词典同步（严格）** 的词本，
正文不需要写具体的单词，只要用#号写一个字即可，可填：
```text
# 欧路词典同步
```
创建完后不要随便修改名字，每次创建或者修改可能会影响这个词本在墨墨app数据库的id；
（没懂：保留其他旧词本。）
现在需要询墨墨中每个词本的的完整 ID，还是接着CMD中输入，然后找到你词典的ID，保留备用：
```powershell
npx --yes --package=@maimemo/memo-api@1.3.1 memo-api notepad list
```
<img width="1136" height="60" alt="image" src="https://github.com/user-attachments/assets/17c68868-e129-4b3c-8eeb-dc9ffae56a3a" />

（没懂：不要使用截断 ID 或网页数字 ID。词本 ID 配置在 **Variables**，不是 Secrets。同步和诊断共用它，不需要编辑工作流。缺失或不一致时停止，不回退到示例或其他人的 ID。）

现在开始创建GitHub的自动执行环境，以及连接，读取，同步两个app个人词典库

回答Github中 **Settings → Secrets and variables → Actions** 配置：

| 类型 | 名称 | 内容 |
|---|---|---|
| Secret | MAIMEMO_OIDC_CREDENTIALS | 已经自动创建 |
| Secret | EUDIC_AUTHORIZATION | [欧路开放平台](https://my.eudic.net/OpenAPI/Authorization)提供的完整 Authorization |
| Secret | EUDIC_BOOK_ID | 默认生词本填 0 |
| Repository Variable | MAIMEMO_NOTEPAD_ID | 自己的完整 np- 开头云词本 ID |

创建新的repository Secret ，名字为EUDIC_AUTHORIZATION ，然后编辑，value的值填写欧路开放平台中的授权信息码（点击欧路开放平台的链接，找到获取授权，复制信息，粘贴EUDIC_AUTHORIZATION 的value栏中），这一步就是连接欧路词典的关键
<img width="644" height="473" alt="image" src="https://github.com/user-attachments/assets/e40ddff4-26dd-48b4-aa05-8531a0989c92" />
<img width="741" height="182" alt="image" src="https://github.com/user-attachments/assets/8507ef0d-5a8f-47f6-8466-a90c09b0668f" />

创建EUDIC_BOOK_ID，value这里一般都是默认的单词本，所以填写数字： 0 
在secret的同级，有一栏Variable，点击它并创建Repository Variable ，value值为我们刚刚在终端上获取的墨墨云词本的ID地址，np-开头的字符

完成后我们就实现了三个软件的所有连接和授权的设置，最后一步是执行把欧路词典同步到墨墨云词典中


### 4. 先做只读预检

回到Github的主仓库界面，找到setting同级的 **Actions** 栏，找到并运行指令 **MaiMemo Read-only Diagnostics → Run workflow → main**，保持 difference_audit=false。
执行完后可以看到没有错误，如果有错误就需要修改workflow file的内容
<img width="1059" height="729" alt="image" src="https://github.com/user-attachments/assets/34746eaa-5a3c-43b5-b39f-3eefae391e7b" />


```
name: Sync Words

on:
  schedule:
    # UTC 01:05 = 北京时间 09:05，避开整点。
    - cron: '5 1 * * *'
  workflow_dispatch:
    inputs:
      authentication:
        description: 'oauth normally; fixed only for explicit manual fallback'
        type: choice
        options:
          - oauth
          - fixed
        default: oauth

env:
  MAIMEMO_NOTEPAD_ID: ${{ vars.MAIMEMO_NOTEPAD_ID }}

permissions:
  contents: read

concurrency:
  group: maimemo-user-authorization
  cancel-in-progress: false

jobs:
  sync:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4 # 修正：当前稳定版本为 v4
        with:
          fetch-depth: 0

      - name: Setup .NET
        uses: actions/setup-dotnet@v4 # 修正：当前稳定版本为 v4
        with:
          dotnet-version: '10.0.x' # 注意：请确保 .NET 10 已正式发布，否则建议改为 '8.0.x' 或 '9.0.x'
          dotnet-quality: 'ga'

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          
      - name: Install pinned OAuth dependencies
        working-directory: scripts/maimemo-oauth
        run: npm ci --ignore-scripts --no-audit --no-fund
        
      - name: Run Sync
        shell: bash
        run: |
          node scripts/maimemo-oauth/verify.mjs --check-config
          if [ "$MAIMEMO_AUTH_MODE" = "oauth" ]; then
            node scripts/maimemo-oauth/run-sync.mjs
          elif [ "$MAIMEMO_AUTH_MODE" = "fixed" ]; then
            dotnet run --project src/EudicSyncToMaiMemo/EudicSyncToMaiMemo.csproj --configuration Release -- --sync-once
          else
            echo 'authentication_mode_invalid'
            exit 1
          fi
        env:
          MAIMEMO_AUTH_MODE: ${{ inputs.authentication || 'oauth' }}
          MAIMEMO_OAUTH_SYNC_CONFIRMED: 'true'
          MAIMEMO_OIDC_CREDENTIALS: ${{ inputs.authentication != 'fixed' && secrets.MAIMEMO_OIDC_CREDENTIALS || '' }}
          Eudic__Authorization: ${{ secrets.EUDIC_AUTHORIZATION }}
          Eudic__DefaultBookId: ${{ secrets.EUDIC_BOOK_ID }}
          MaiMemo__Provider: OpenApi
          MaiMemo__Token: ${{ inputs.authentication == 'fixed' && secrets.MAIMEMO_TOKEN || '' }}
          MaiMemo__DefaultNotepadId: ${{ vars.MAIMEMO_NOTEPAD_ID }}
          
          # ✅ 核心修复：增加空值兜底逻辑。如果 Secret 为空，则强制传入 'false'，防止 .NET 解析空字符串报错
          Notification__Enabled: ${{ secrets.NOTIFICATION_ENABLED != '' && secrets.NOTIFICATION_ENABLED || 'false' }}
          Notification__Url: ${{ secrets.NOTIFICATION_URL || '' }}
          Notification__RequestBody: ${{ secrets.NOTIFICATION_REQUEST_BODY || '' }}
          Notification__Headers: ${{ secrets.NOTIFICATION_HEADERS || '' }}
```

完成后执行后，可以看到GitHub的 run sycn 栏中显示了获取的欧路生词本的单词个数，并同步到墨墨
<img width="675" height="717" alt="image" src="https://github.com/user-attachments/assets/ef0432d0-247d-47da-9428-60fa30d25087" />

（没懂：查看 source=strict_preview：

- ready=1、prepared_union_equal=1、original_content_preserved=1。
- target_duplicates=0、invalid_lines=0、second_merge_additions=0。
- would_add 是预计新增数量；target_titles 是原有数量。
- extra 是合法历史标题数，允许大于零，不会删除。

以自己的数据为准，不套用他人的历史词数。数字被 GitHub 遮罩为 *** 时，不要关闭凭证遮罩。）

### 5. 首次手动同步与验收

通过后，最后找到Sync Words，运行 **Sync Words → Run workflow → main**，选择默认 oauth。查看 Run Sync 的刷新、目标读取、新增数量和写后校验。

随后再次运行只读诊断，确认 ready=1、missing=0、would_add=0，无重复或非法标题。不需要重复真实写入来测试去重。无新增的运行只验证读取和幂等，不代表实际测试了写权限。

### 6. 自动运行和维护

（没懂，在运行Sync Words后，不是启动了自动每天计划运行的计划吗，为啥这里还要手动计划，另外同步几天后，显示target_configuration_valid=1 oauth_sync_failed=oauth_client_rejected，这里应该需要重新填入墨墨的还是欧路的授权码？：计划每天 UTC 01:05，即北京时间 09:05；**GitHub 调度可能明显延迟，不保证准点**。Fork 定时任务可能需要在 Actions 手动启用。首次配置前可先禁用 Sync Words，验收后再启用。

每次先刷新 Access Token，再通过子进程环境交给原 .NET 程序。任务共用并发组。OAuth 出错即停止，不自动回退。

当前 CLI 按首次授权后约 **90 天**记录刷新凭证到期，撤销或服务端策略也可能提前使其失效。届时重复本机登录和导入，不是永久免维护授权。发现 Refresh Token 轮换会停止，不使用公开文件或产物持久化。

查看自动结果：**Actions → Sync Words → 最新运行 → Run Sync**。报错后先排查，不反复重跑。）

## 可选固定 Token 回退
（没懂固定Token的意义）
仅需要回退时设置 MAIMEMO_TOKEN Secret，手动运行时明确选择 authentication=fixed。定时仍使用 OAuth。固定 Token 不具备本项目的自动续期能力。

Prepare Strict MaiMemo Notebook 是可选手动建本工具，仍需要固定 Token，**会创建词本，不是只读检查**。推荐教程不依赖它，也不要在排障中反复创建。

## 本地运行与开发

原 .NET 控制台和 Windows 服务保留。直接运行这些程序需自行提供有效 Token；独立可执行文件和 Windows 服务不会自动运行 GitHub 的 OAuth 包装脚本。

配置示例见 src/EudicSyncToMaiMemo/appsettings.json。真实授权只保存在本机，不提交。使用 .NET 10 SDK：
```powershell
dotnet test src/EudicSyncToMaiMemo.slnx
npm ci --prefix scripts/maimemo-oauth --ignore-scripts
npm test --prefix scripts/maimemo-oauth
pwsh -File scripts/test-maimemo-diagnostics.ps1
pwsh -File scripts/test-provision-strict-notepad.ps1
pwsh -File scripts/test-import-maimemo-oauth.ps1
```
测试使用模拟数据，不会同步真实词库。打包见 .github/workflows/release.yml 的 v* 标签流程。上游发布包不代表本 Fork 的功能，请使用对应仓库的版本或自行构建。


更多说明：[OAuth](docs/maimemo-oauth.md) · [故障排查](docs/troubleshooting.md)
