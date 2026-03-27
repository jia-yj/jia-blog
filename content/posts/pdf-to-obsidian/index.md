---
title: "把 PDF 扔进文件夹，自动变成 Markdown"
date: 2026-03-19
categories: ["日志"]
---

*本文由 AI 生成。*

这是一个本地 PDF 自动解析工具，基于 MinerU API。将 PDF 文件放入指定文件夹，脚本自动完成上传、解析、提取，结果以 Markdown 格式存入 Obsidian 笔记库。

## 目标

```
pdf_inbox/        ← 放入 PDF
阅读材料/
├── 论文名.md     ← 解析结果
└── 论文名.pdf    ← 原文件
```

## 工具

[MinerU](https://mineru.net) 提供 PDF 解析 API，支持 PDF、Word、PPT，可识别公式和表格。免费额度每天 2000 页。

## 流程

本地文件没有公网 URL，不能直接提交解析任务，需走批量上传接口：

```
申请预签名上传链接（POST /file-urls/batch）
  → PUT 上传本地文件
  → 轮询解析状态（GET /extract-results/batch/{batch_id}）
  → 下载 ZIP 结果包
  → 提取 full.md
```

轮询间隔 10 秒，超时 10 分钟。本地脚本无公网地址，不使用 Callback。

## 监听文件夹

主循环每 3 秒扫描一次 `pdf_inbox/`，检测到 PDF 后等待写入完整再处理：

```python
size_before = os.path.getsize(pdf_path)
time.sleep(2)
size_after = os.path.getsize(pdf_path)
if size_before != size_after:
    continue  # 文件仍在写入
```

大文件拷贝需要时间，前后大小一致才认为写入完成，否则跳过本轮。

## 只提取 Markdown

ZIP 包含多种格式（json、content_list 等），只取 `full.md`，解压到临时目录后清理：

```python
for root, _, files in os.walk(tmp_dir):
    for f in files:
        if f == "full.md":
            shutil.copy2(os.path.join(root, f),
                         os.path.join(OUTPUT_DIR, f"{name}.md"))
shutil.rmtree(tmp_dir)
```

## 使用

```bash
pip3 install requests
python3 mineru_watch.py
```

运行后将 PDF 拖入 `~/Desktop/pdf_inbox/`，处理完成后 Markdown 和原 PDF 一并出现在 Obsidian 目录中，按 `Ctrl+C` 停止。
