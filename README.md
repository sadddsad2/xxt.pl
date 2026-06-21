# EpicHost 保活脚本

基于 GitHub Actions 的定时续期脚本,用于自动续期 EpicHost 免费服务器,支持同时管理多个服务器。

## 功能

- 定时(默认每 6 小时)自动调用续期接口
- 支持多个服务器,URL 与 API Key 按数组下标一一对应
- 单个服务器续期失败自动重试 3 次
- 某个服务器失败不影响其他服务器继续续期
- 支持手动触发(workflow_dispatch),方便测试

## 使用方法

### 1. 放置 workflow 文件

将 `keepalive.yml` 放到仓库的 `.github/workflows/keepalive.yml`

### 2. 配置 Secrets

进入仓库 `Settings → Secrets and variables → Actions → New repository secret`,新增以下两个:

**`EPICHOST_BASE_URLS`**(JSON 数组,每个服务器的续期接口地址)

```json
[
  "https://panel.epichost.pl/api/client/freeservers/7f8e936f-9b70-4041-ad83-5911155d642e/renew",
  "https://panel.epichost.pl/api/client/freeservers/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/renew"
]
```

**`EPICHOST_API_KEYS`**(JSON 数组,与上面 URL 按下标一一对应的 API Key)

```json
[
  "key_for_server_1",
  "key_for_server_2"
]
```

> 两个数组长度必须一致,下标 `i` 的 URL 对应下标 `i` 的 Key。脚本启动时会自动校验数量是否匹配,不匹配会直接报错退出。

### 3. 调整执行频率(可选)

默认 cron 为:

```yaml
schedule:
  - cron: '0 */6 * * *'   # 每 6 小时一次,UTC 时间
```

如果服务器要求 24 小时内续期一次,建议设置为 8~12 小时一次,留出失败重试的冗余时间。

### 4. 手动测试

仓库页面 → `Actions` → 选择 `EpicHost Keepalive` → `Run workflow`,可立即触发一次执行,查看日志确认是否成功。

## 新增/删除服务器

无需修改 workflow 文件,只需要同步编辑 `EPICHOST_BASE_URLS` 和 `EPICHOST_API_KEYS` 两个 Secrets,新增对应下标的条目即可。

## 日志说明

每次运行会在 Actions 日志中打印:

- 当前续期的 URL
- 每次尝试的 HTTP 状态码
- 接口返回的响应内容(JSON)

如果某台服务器最终续期失败,整个 job 会标记为失败(红色),便于第一时间发现问题,但不会中断其他服务器的续期流程。

## 常见问题

**Q: API Key 过期了怎么办?**
A: 重新在面板生成 Key,更新对应下标的 `EPICHOST_API_KEYS` 即可,无需改动代码。

**Q: 想给不同服务器设置不同的重试次数/间隔?**
A: 当前版本所有服务器共用同一套重试策略(3 次,间隔 10 秒)。如有差异化需求需要扩展脚本逻辑,可以再提出来。

**Q: cron 时间是哪个时区?**
A: GitHub Actions 的 cron 固定为 UTC 时间,需要换算成你所在时区对应的时间。
