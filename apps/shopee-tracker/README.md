# Shopee Tracker · 本地自动追踪工具

纯代码的 macOS 印尼 Shopee 达人追踪工具。复用本人已登录的正式版 Chrome，按最近 7 天采集销售总览、直播与短视频，保存截图，默认生成本地 Excel，也可写入使用者自己的飞书多维表。运行时不调用 AI、不需要 Codex。

[查看模拟演示](https://nesealice.github.io/alice-ai-portfolio/apps/shopee-tracker/demo.html) · [下载源码包](https://github.com/nesealice/alice-ai-portfolio/raw/main/apps/shopee-tracker/Shopee-Tracker-Public-Source-v1.0.0.zip)

## 能做什么

- 自定义印尼达人名单，保留 Category、TTS 名称等元数据；没有预设私人达人名单。
- 每天 Mac 本地时间 10:00 自动执行；错过时间会在当天开机/唤醒后补跑。
- 每位达人完成采集后自动保存/回写；同日续跑只重试未完成渠道，不重复建记录。
- 本地 `tracking.xlsx`、JSON/CSV、SQLite 记录及 PNG 证据；飞书模式会上传截图附件并核对文件名。
- 提取、目标写入、截图分别显示状态；更换目标后重新核验，不继承旧目标的成功状态。

完整可运行目录（tool、mac、docs、examples 和测试）包含在源码 ZIP 中；下载解压后按下文运行。GitHub 页面展示说明、配置模板与演示。

## 安装与首次运行

M 系列 Mac、正式版 Google Chrome。源码运行需要 Python 3.10+；Mac 打包版自带 Python，不自带浏览器。当前发布包采用临时签名，未做 Apple Developer ID 公证。

源码用户：

```bash
cd apps/shopee-tracker
bash tool/Setup.command
bash Start.command
```

也可在 Python 虚拟环境中安装 `tool/requirements.txt` 后运行 `mac/app_launcher.py`。依赖固定版本；默认不会下载 Chrome for Testing。

1. 在界面保存自己的达人 CSV；默认输出为本地 Excel，无需任何飞书账号或密钥。
2. 在本人日常 Chrome 登录获准使用的 Shopee 账号，手动打开 `chrome://inspect/#remote-debugging` 并允许本机连接；在工具点“连接我的 Chrome / 检查登录”，按 Chrome 提示确认。
3. 点“开始采集并保存”。打包 Mac 版首次正常打开会启用登录后自启后台，每天 10:00 运行。源码 GUI 只在进程存活期间定时，不会自动安装后台服务。

数据目录为 `~/Library/Application Support/ShopeeTrackerPublic/`。与私人版本使用不同数据目录、钥匙串服务及后台标签。关闭界面不会关闭打包版后台。更换达人名单请在新一天首次采集前完成；同日已有任务时需要恢复原名单才能继续，避免混淆快照。

### 达人 CSV

模板：`examples/targets.example.csv`。示例 ID/Handle 为虚构，只展示格式，不自动执行。请替换成自己获准查看的印尼站达人详情页。

```csv
country,affiliate_id,shopee_handle,url,category,tts_link
ID,123456789,demo_creator,https://seller.shopee.co.id/portal/web-seller-affiliate/kol_marketplace/detail?affiliate_id=123456789,Demo category,demo_tts
```

CLI：设置相同数据目录后运行 `python -m shopee_tracker --data-dir <目录> daily`；`resume` 继续当天任务，`status` 查看状态，`export --output <文件前缀>` 导出明细，`writeback` 重新保存到当前目标。需从 `tool/` 下运行或设置 PYTHONPATH。

## 输出目标

默认生成数据目录下 `exports/tracking.xlsx`，可以用 Excel、Numbers 或兼容办公软件打开。文件是工具管理的快照工作簿，重复执行会更新同日记录；请勿在该文件中存放无关手工表格。若要自定义路径，可在本地 `base-config.json` 中设置 `excel_path`（`.xlsx`）。截图为本地文件，工作簿记录路径，不嵌入图片。

飞书为可选的**多维表**适配器，非普通在线电子表格适配器。选择飞书后，在界面填写自己的 Base 配置与应用凭据；完整字段清单及权限见 [飞书配置](feishu.md)。本项目没有默认 Base、表ID、应用ID、记录ID或业务链接。应用密钥仅保存到本机独立钥匙串，不进入源码。

## 指标口径

- 所有渠道固定为平台 `Last 7 Days`，截图保留平台更新时间；不是当日增量。
- 主表历史字段 `Average Page Views(PV)` 实际保存直播总 Views，既不除以场次，也不是去重观看人数。
- 场次、Views、GPM 的 change 为同一达人相邻两个已采集日期的滚动 7 天快照之差；**不能据此还原当天 GMV**，也不保证两个日期连续。
- 缺失、失败和平台未提供的渠道留空；真实 0 保留。区间/上限/下限不伪造精确数值，原文和解析状态保存在 Excel 的 Metric details、JSON 与 CSV 中。
- GMV、订单数等只保存官方页面实际可见指标，不推算未展示的数据。

## 运行边界

Mac 必须开机、登录系统并联网，Chrome 可连接。只补跑当天，不补造关机期间的历史。若启动时无网络或授权失效，处理后需手动重试；当前不承诺自动无限重试。平台改版、风控或缺少账号权限会使采集暂停并保留已取得的数据。

持续复用后台 Chrome 连接可减少重复确认，但不能保证永久免鉴权。退出 Chrome、重启或 Shopee 登录过期后可能需要账号持有人再次确认。程序不保存密码、不复制 Cookie、不处理验证码、不使用隐藏 Shopee API。

## 开发与验证

```bash
PYTHONPATH=tool python -m unittest discover -s tool/tests
PYTHONPATH=tool:mac python -m unittest discover -s mac/tests
# 已安装正式 Chrome 时运行渲染用例
RUN_DOM_TESTS=1 PYTHONPATH=tool python -m unittest discover -s tool/tests
# macOS arm64，隔离环境安装 PyInstaller 后构建
BUILD_PYTHON=/absolute/path/to/python bash mac/build.sh
```

测试与公开演示均为模拟数据。私有部署曾完成真实渠道采集及回写验证，这不代表任何新账号/站点都能直接采集成功；需自行完成访问授权。完整验证结果见 `verification.json`。本项目是独立工具，与 Shopee、Google、飞书或 Lark 无隶属关系。仅用于本人或明确授权的数据访问。
