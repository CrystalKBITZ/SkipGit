# SkipGit

**纯 Java 构建的跨平台桌面 Git 客户端**。Git 与 Git LFS 引擎全部内置，不依赖系统 `git` / `git-lfs`，开箱即用。

界面采用 macOS 风格的无边框设计：圆角、投影、磨砂玻璃质感与柔光流动背景，全部交互带缓动动画。

---

## 设计取向

系统自带 git 要在命令行敲一堆命令；现成的 GUI 客户端要么依赖命令行 git，要么只支持 GitHub 一家。

SkipGit 的目标是：**新手拖个文件进来就能提交上传，老手该有的操作一个不少**。

- 引擎完全进程内运行（JGit + 自研 LFS 客户端），无需外部二进制
- 单一界面统一管理 6 个托管平台：GitHub、GitLab、Gitee、Bitbucket、Codeberg、Gitea
- 错误信息本地化：`non-fast-forward` 会翻译成「请先拉取再推送」这类可执行建议

---

## 功能

### 仓库
- 打开本地仓库、本地初始化、从远程克隆
- 云端新建仓库（自动关联 origin 并推送）
- 启动自动恢复上次打开的仓库

### 变更与提交
- 未暂存 / 已暂存分列，文本 diff 预览
- 单文件暂存、全部暂存、丢弃未暂存
- **按块暂存**：只提交文件里的部分改动（等价 `git add -p`）
- **一键上传**：暂存全部 → 提交 → 推送，一步到位
- 提交说明空着也能提交，自动生成 `chore: 更新 MM-dd HH:mm`

### 历史与回滚
- 提交列表带轻量分支图（合并提交用紫色节点标出）
- 历史搜索：按提交信息、作者、哈希过滤
- **撤销到此提交**（保留工作区）、**强制重置**（丢弃改动）、**回滚本次提交**（生成反向提交，可安全推送）

### 冲突处理
- 拉取/合并产生冲突时，变更页顶部出现提示条
- 每个冲突文件可选：**用我的版本**、**用远端版本**、打开文件手改、标记已解决

### 分支
- 列表（含远程分支）、新建并切换、切换、合并、删除

### 远程与多平台
- 推送、拉取、刷新
- 修改 origin 地址（地址配错时的救命入口）
- 把账号下的云端仓库一键设为 origin
- 跨平台同步：GitHub ↔ GitLab ↔ Gitee ↔ Bitbucket ↔ Codeberg ↔ Gitea 镜像

### 云端文件直改
- **不克隆也能改远端文件**：浏览目录 → 点文件 → 改 → 保存直接提交到云端
- 支持 GitHub / Gitee / GitLab / Gitea / Codeberg / Bitbucket

### Git LFS
- `.gitattributes` 规则管理
- 提交时大文件自动替换为指针并上传，拉取后自动还原

### AI 助手
- 左侧导航独立页面，聊天式交互
- **能执行操作**：说「帮我上传」「建个 feature/login 分支」直接执行
- **能解答用法**：问「怎么改 origin」会给分步说明
- **能看文件**：点「附件」多选文件，AI 读内容后给分析和建议（超 50KB 自动分块汇总）
- 会带上当前软件状态（仓库、分支、未提交改动、origin）来判断你的意图
- 只发送你输入的文字和主动选中的文件内容，绝不上传令牌与密码

### 云端账号
- 邮箱验证码注册 / 登录 / 重置密码（Supabase 承载）
- 令牌与敏感配置用登录密码加密后同步到云端，换设备登录自动取回
- 异地登录会发安全邮件验证

### 其他
- 中英文界面切换
- 自动更新检查（发现新版本可直接下载到桌面）
- 明暗背景、不透明度、动态背景
- 鼠标悬停有功能说明（tooltip）
- 所有 Git 报错翻译成人话 + 处理建议

---

## 界面预览

<p align="center">
  <img src="shot-changes.png" width="90%" alt="变更页：暂存、提交、一键上传">
</p>

<p align="center"><i>变更页 —— 未暂存 / 已暂存分列、diff 预览、一键上传</i></p>

<p align="center">
  <img src="shot-assistant.png" width="90%" alt="AI 助手：能操作软件，也能解答用法">
</p>

<p align="center"><i>AI 助手 —— 说一句话直接执行，也能问怎么用</i></p>

<p align="center">
  <img src="shot-remote.png" width="90%" alt="远程与同步：六平台统一管理">
</p>

<p align="center"><i>远程与同步 —— origin 管理、跨平台镜像</i></p>

## 运行要求

- **JDK 21**（Microsoft Build of OpenJDK 或任意 21 版本）
- 无需安装 git、git-lfs
- Windows 10/11（其他平台未测试）

## 构建

```bash
./build.sh
```

构建流程：`javac` 编译 → 打包 `target/skipgit.jar` → 收集依赖到 `build/lib` → 生成 CDS 启动加速包 `build/skipgit.jsa`。

## 运行

```bash
./run.bat
```

等价命令：

```bash
java --module-path build/lib/fx --add-modules javafx.controls \
     -Djdk.tls.client.protocols=TLSv1.2 -Dprism.order=d3d,sw \
     -cp "target/skipgit.jar;build/lib/jgit.jar;build/lib/ewah.jar;build/lib/slf4j.jar;build/lib/codec.jar;build/lib/gson.jar" \
     com.skipgit.Launcher
```

常用参数：

| 参数 | 作用 |
| --- | --- |
| `--repo <path>` | 启动时打开指定仓库 |
| `--page <name>` | 启动后切到指定页面（changes/files/history/branches/remote/assistant） |
| `--shot <file>` | 渲染一帧并保存 PNG（离屏截图） |

---

## 技术栈

| 部分 | 选择 |
| --- | --- |
| 语言 / 运行时 | Java 21 |
| 界面 | JavaFX 21（controls + graphics） |
| Git 引擎 | JGit 6.10（纯 Java） |
| LFS | 自研 HTTP 客户端（batch API） |
| JSON | Gson 2.11 |
| 云端账号 | Supabase（PostgREST + Auth） |

**没有** WebView、没有 Electron、没有外挂命令行。

## 目录结构

```
src/main/java/com/skipgit/
├── App.java             启动入口、自动更新、启动时序
├── Launcher.java        无模块依赖的引导类
├── core/                引擎层
│   ├── GitService       Git 操作（状态、提交、推送、撤销、按块暂存）
│   ├── LfsClient        LFS batch API
│   ├── LfsManager       .gitattributes 规则与指针处理
│   ├── HostClient       六平台统一接口
│   ├── Supabase*        云账号、会话、同步
│   ├── AiClient         AI 助手（智谱 GLM）
│   ├── CommandAssistant 动作定义
│   ├── UpdateChecker    版本检查
│   └── Updater          新版本下载
└── ui/                  界面层
    ├── GlassWindow      无边框毛玻璃窗口
    ├── MainView         主框架与页面路由
    ├── ChangesView      变更页（含冲突条、按块暂存）
    ├── HistoryView      历史页（分支图、搜索、回滚）
    ├── RemoteView       远程与同步
    ├── AssistantView    AI 助手聊天页
    ├── RemoteEditDialog 云端文件直改
    └── Widgets          自绘控件库
```

---

## 常见问题

**提示「认证失败」**
令牌无效或过期。到「设置 → 账号与令牌」重填，注意勾选仓库读写权限。

**提示「推送被拒绝」**
远端有你本地没有的提交。先点「拉取」，处理完差异再推送。

**大文件传不上去**
在「远程与同步」里添加 LFS 规则（如 `*.psd`），提交时会自动转成指针。

**启动比较慢**
启动时会加载背景图与云端会话。各阶段耗时记录在 `~/.skipgit/startup.log`。

**换电脑后令牌没了**
登录同一云账号即可自动取回（需要用同一个登录密码解密）。

---

## 开源协议

[MIT](LICENSE) © 2026 CrystalKBITZ
