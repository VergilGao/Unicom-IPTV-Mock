# 长春联通 IPTV STB Mock

用 Python 模拟机顶盒登录鉴权，获取 IPTV 频道列表和节目单。

## 功能

- Web 管理界面（兼容手机）
- 自动完成 EDS/EPG 服务器登录鉴权
- 获取频道列表（组播地址 + 回看地址）
- 频道筛选、重命名、重编号
- 生成 `iptv.m3u` 播放列表
- 抓取 ESAAS 节目单 → `epg.xml`
- SQLite 缓存节目单，增量更新
- 定时自动运行
- 在线查看日志（保留最近7次运行记录）

## 使用方法

### 本地运行

```bash
pip install -r requirements.txt
DATA_DIR=./data python src/web.py
```

访问 `http://localhost:5000`，通过 Web 界面完成全部配置。

### Podman / Docker

确保容器网络能路由到 IPTV 内网服务器。

```bash
# 构建
docker build -t stbmock .

# 创建数据目录
mkdir -p /path/data

# HTTP 运行
docker run -d --name stbmock \
  -v /path/data:/data:Z \
  -p 5000:5000 \
  --restart unless-stopped \
  stbmock

# HTTPS 运行（PEM 证书文件）
docker run -d --name stbmock \
  -v /path/data:/data:Z \
  -v /path/cert.pem:/ssl/cert.pem:Z \
  -e SSL_CERT_KEY=/ssl/cert.pem \
  -e WEB_PORT=8443 \
  -p 8443:8443 \
  --restart unless-stopped \
  stbmock
```

首次启动后访问 Web 界面，在"配置"页面填入参数，在"频道"页面勾选需要的频道。

### 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `DATA_DIR` | `/data` | 数据持久化目录（配置、日志、输出文件） |
| `WEB_PORT` | `5000` | Web 端口 |
| `SSL_CERT_KEY` | 无 | 合并 PEM 证书文件路径（启用 HTTPS） |

## Web 界面

| 页面 | 功能 |
|------|------|
| 首页 | 运行状态、手动触发、文件下载 |
| 配置 | 编辑登录参数和 EPG 设置 |
| 频道 | 搜索、勾选频道，自定义名称和频道号 |
| 定时 | Cron 表达式配置自动运行 |
| 日志 | 查看最近7次运行日志，实时流式输出 |

## 输出文件

| 文件 | 说明 |
|------|------|
| `iptv.m3u` | 播放列表（Web 界面可直接下载） |
| `epg.xml` | XMLTV 格式电子节目单 |
| `epg.db` | SQLite 节目单缓存 |
| `channels.txt` | 全部频道原始名称参考列表 |

## License

GPL-3.0
