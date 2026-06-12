# jd-link-converter

将京东移动端分享短链接（3.cn / u.jd.com）和移动端商品链接（item.m.jd.com）自动解析并转换为 PC 端纯净商品链接（item.jd.com）。

## 功能特性

- 支持 `3.cn` 短链接解析（模拟京东 App UA）
- 支持 `u.jd.com` 短链接解析
- 移动端链接 `item.m.jd.com` → PC 端链接 `item.jd.com`
- 去除所有 UTM 追踪参数，输出纯净链接
- 支持单链接 / 批量转换
- 纯 Python 标准库实现，零第三方依赖

## 转换逻辑

```
短链接 (3.cn/xxx) ──HTTP 302──▶ 移动端长链接 (item.m.jd.com/product/{id}.html?utm_*=...)
                                         │
                                   正则提取商品ID: /(\d+)\.html/
                                         │
                                   拼接PC端链接: https://item.jd.com/{id}.html
```

## 安装

```bash
pip install jd-link-converter
```

或直接克隆：

```bash
git clone https://github.com/your-username/jd-link-converter.git
cd jd-link-converter
pip install -e .
```

## 使用方式

### 命令行

```bash
# 单链接转换
jd-convert "https://3.cn/2RKwn3-g"

# 批量转换
jd-convert "https://3.cn/2RKwn3-g" "https://3.cn/2RKw-Gjq" "https://3.cn/2RK-xz6V"

# 管道输入
echo "https://item.m.jd.com/product/100198170140.html?utm_source=iosapp" | jd-convert
```

输出示例：

```json
[
  {
    "original": "https://3.cn/2RKwn3-g",
    "resolved": "https://item.m.jd.com/product/100198170140.html?utm_source=iosapp&...",
    "product_id": "100198170140",
    "pc_url": "https://item.jd.com/100198170140.html",
    "error": null
  }
]

--- 简洁结果 ---
https://3.cn/2RKwn3-g
  -> https://item.jd.com/100198170140.html
```

### Python API

```python
from jd_link_converter import convert_jd_link, batch_convert

# 单链接转换
result = convert_jd_link("https://3.cn/2RKwn3-g")
print(result["pc_url"])  # https://item.jd.com/100198170140.html

# 批量转换
results = batch_convert([
    "https://3.cn/2RKwn3-g",
    "https://item.m.jd.com/product/100120810291.html?utm_source=share",
])
for r in results:
    if r["pc_url"]:
        print(f"{r['original']} -> {r['pc_url']}")
```

## 支持的 URL 格式

| 输入格式 | 示例 | 转换结果 |
|----------|------|----------|
| 3.cn 短链接 | `https://3.cn/2RKwn3-g` | HTTP 解析 → 提取 ID → PC 链接 |
| u.jd.com 短链接 | `https://u.jd.com/abc123` | HTTP 解析 → 提取 ID → PC 链接 |
| 移动端商品页 | `https://item.m.jd.com/product/100198170140.html?utm_*=...` | 直接提取 ID → PC 链接 |
| PC 端商品页(含参数) | `https://item.jd.com/100198170140.html?cu=true` | 提取 ID → 纯净 PC 链接 |
| 纯商品 ID | `100198170140` | 直接拼接 PC 链接 |

## 注意事项

- 3.cn 短链接解析需要模拟京东 App 的 User-Agent，否则会被重定向到京东首页
- 短链接有时效性，过期的短链接可能无法解析
- 商品 ID 为 6-12 位纯数字
- PC 端纯净链接不需要任何查询参数，UTM 参数全部丢弃

## 作为 WorkBuddy Skill 使用

本项目也可作为 [WorkBuddy](https://www.codebuddy.cn/) 技能使用：

```bash
clawhub install jd-link-converter
```

技能安装后，在对话中粘贴京东短链接时会自动触发转换。

## 开发

```bash
# 克隆仓库
git clone https://github.com/your-username/jd-link-converter.git
cd jd-link-converter

# 安装开发依赖
pip install -e ".[dev]"

# 运行测试
python -m pytest tests/
```

## License

MIT License
