# 配置自己的飞书多维表

Excel 为默认目标。飞书需要使用者自己准备多维表、快照表、应用权限和字段。

1. 在自己的多维表中建快照表，用 SHP Handle Name 作主文本列。按 examples/base-schema.csv 创建其余字段（名称须完全相同）。Creator 关联字段可选，未配置 creator_record_ids 时不写入。
2. 开通应用身份的 base:record:read、base:record:create、base:record:update，以及 docs:document.media:upload。将该应用添加为目标多维表可编辑的文档应用。权限是否可用以实际接口结果为准。
3. 从自己的表获取 Base token、快照表 ID、Screenshot 附件字段 ID，在界面的输出设置中填写 JSON：

```json
{
  "output": "feishu",
  "api_origin": "https://open.feishu.cn",
  "base_token": "YOUR_BASE_TOKEN",
  "snapshot_table_id": "YOUR_SNAPSHOT_TABLE_ID",
  "field_ids": {"Screenshot": "YOUR_SCREENSHOT_FIELD_ID"},
  "screenshot_upload_mode": "app"
}
```

国际 Lark 可使用 https://open.larksuite.com。仅允许这两个官方 API origin。当前适配器使用 Base v3 API，权限与表结构不兼容时会保留本地数据并明确报错；不会偷偷改写另一张表。

4. 在应用设置中输入自己的 App ID 和 Secret，保存到本机钥匙串。不要把凭据写入示例、README 或提交 Git。已有 Creator 关联时，可以添加 `creator_record_ids`，键为 `ID:达人数字ID`，值为自己达人名单表的 record_id。
5. 第一次先用一个获准访问的达人验证。数值写入后会读回核验；附件上传也会核对文件名。不同输出目标分别核验，切换到新表不表示旧记录已经迁移。

截图通过应用身份直接上传，无需个人 CLI 登录。Record Key 等字段供去重与恢复使用，可以在日常视图隐藏；不要删除这些字段。字段清单仅为模板，不含任何真实表或账号配置。
