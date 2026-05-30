# PDF 简历提取方法

pm-portfolio skill 处理 PDF 简历时的标准提取流程。

## 环境准备（一次性）

```bash
python3 -m venv /tmp/pypdf_venv
source /tmp/pypdf_venv/bin/activate
pip install PyPDF2 -q
```

## 提取命令

```bash
source /tmp/pypdf_venv/bin/activate && python3 -c "
from PyPDF2 import PdfReader
r = PdfReader('PDF文件路径')
for p in r.pages: print(p.extract_text())
"
```

## 注意事项

- venv 路径固定为 `/tmp/pypdf_venv`，避免重复创建
- PyPDF2 对中文 PDF 兼容性好，偶有字体警告可忽略
- 如果 PyPDF2 提取为空，说明是扫描件，需用 OCR（pymupdf marker 模式），告知用户
