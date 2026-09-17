# TYARCH Rhino 环境部署

面向**新电脑 / 新同事**的环境部署说明。目标：把工作模板与显示模式装进 Rhino 8，使新机器一开箱就是团队环境。

本文件可交给 AI harness（RhinoMCP 通道）直接执行，也可由人工按 GUI 步骤操作。

| 项目 | 内容 |
| --- | --- |
| 仓库地址 | <https://github.com/HelloJack9811/TYArch-Rhino-Templates> |
| 适用 Rhino | Rhino 8（Windows） |
| 部署内容 | 模板 `.3dm` ×1、显示模式 `.ini` ×5 |
| 前置条件 | Rhino 8 已安装；harness 通道可用（见「前置：打通 harness 通道」） |

---

## 1. 部署清单

源文件位于本仓库根目录。两个目录名已与 Rhino 对齐，便于辨认：

| 内容 | 仓库内路径 | 目标落点 | 部署方式 |
| --- | --- | --- | --- |
| 工作模板 | `Template Files\TYARCH-general template-4.0.3dm` | `%APPDATA%\McNeel\Rhinoceros\8.0\Localization\<语言>\Template Files\` | 复制文件 |
| 默认模板设置 | — | Rhino 设置项 `FileSettings.TemplateFile` | 写设置项 |
| 显示模式 ×5 | `Display Modes\*.ini` | **Rhino 无对应磁盘目录** | 调 API 或 GUI 导入 |

显示模式清单（团队共用，共 5 个）：

- `材质预览模式.ini`
- `线稿白模.ini`
- `阴影白模.ini`
- `带点线的材质预览模式.ini`
- `带阴影的材质预览模式.ini`

---

## 2. 部署机制

### 2.1 模板 `.3dm`：复制文件 + 写设置项

模板目录名直接取自 Rhino 自身（`Template Files`），因此这一步就是**把 `Template Files` 目录里的 `.3dm` 拷进 Rhino 的同名目录**，再把默认模板设置指向它。

### 2.2 显示模式 `.ini`：必须调 API，不能复制文件

**这是本部署唯一容易踩空的地方。** Rhino 8 不再把显示模式存成散装 `.ini` 文件，而是持久化在 `settings\settings-Scheme__Default.xml` 里；`.ini` 仅作为导入导出的交换格式。Rhino 的安装目录和用户配置目录下都**没有**任何显示模式文件夹可供拷入。

所以 `Display Modes` 这个目录名虽然沿用了 Rhino 的界面用词，但它只是本仓库的收纳目录，**不存在"拷进去就生效"的目标位置**。必须调用：

```
Rhino.Display.DisplayModeDescription.ImportFromFile(filename, interactive) -> Guid
```

- `interactive = False` 可抑制界面提示，适合自动化。
- 每个 `.ini` 内含一个显示模式定义（含自身 GUID 与 `Name=` 字段），逐个导入即可。

---

## 3. 部署步骤（AI harness）

### 前置：打通 harness 通道

Rhino 需处于运行状态，并在 Rhino 中执行命令 `mcpstart`，桥接开始监听 `127.0.0.1:1999`。之后 harness 即可通过 `execute_rhinoscript_python_code` 在 Rhino 内执行下面的脚本。

### 部署脚本

把 `<仓库路径>` 替换为本仓库根目录的绝对路径，整段送入 Rhino 执行：

```python
import Rhino
import os
import shutil

REPO = r"<仓库路径>"
TPL_DIR = os.path.join(REPO, "Template Files")
MODE_DIR = os.path.join(REPO, "Display Modes")
DEFAULT_TPL = "TYARCH-general template-4.0.3dm"

fs = Rhino.ApplicationSettings.FileSettings

# 1) 解析 Rhino 的模板目录：优先从现有默认模板推导，否则扫描语言子目录
target_dir = None
if fs.TemplateFile and os.path.isdir(os.path.dirname(fs.TemplateFile)):
    target_dir = os.path.dirname(fs.TemplateFile)
else:
    base = Rhino.RhinoApp.GetDataDirectory(True, False, "Localization")
    for lang in sorted(os.listdir(base)):
        cand = os.path.join(base, lang, "Template Files")
        if os.path.isdir(cand):
            target_dir = cand
            break
if not target_dir:
    raise RuntimeError("未能定位 Rhino 的 Template Files 目录，请手动确认语言子目录")
print("Rhino 模板目录:", target_dir)

# 2) 拷入模板
copied = []
for name in sorted(os.listdir(TPL_DIR)):
    if name.lower().endswith(".3dm"):
        shutil.copy2(os.path.join(TPL_DIR, name), os.path.join(target_dir, name))
        copied.append(name)
print("已拷入模板:", copied)

# 3) 设为默认模板
fs.TemplateFile = os.path.join(target_dir, DEFAULT_TPL)
print("默认模板已设置:", fs.TemplateFile)

# 4) 注册显示模式（必须走 API，复制文件无效）
for name in sorted(os.listdir(MODE_DIR)):
    if name.lower().endswith(".ini"):
        gid = Rhino.Display.DisplayModeDescription.ImportFromFile(
            os.path.join(MODE_DIR, name), False)
        print("显示模式:", name, "->", gid)
```

**目标目录为什么不写死**：路径中的 `<语言>` 子目录随 Rhino 界面语言变化（中文为 `zh-CN`，英文为 `en-US`），新机器上不可假定。脚本先尝试从 `FileSettings.TemplateFile` 反推，失败再扫描 `Localization` 下的语言子目录，两种新机器情形都能覆盖。

---

## 4. 部署步骤（人工 GUI）

没有 harness 时按下列步骤操作，效果相同：

1. **模板**：把 `Template Files` 目录下的 `.3dm` 复制到 Rhino 的模板目录（可在 `_Options` → 文件中找到「默认模板」位置，从该处进入目录）。
2. **默认模板**：在 `_Options` → 文件中，把「新建时使用的模板」指向刚复制进去的 `.3dm`。
3. **显示模式**：`_Options` → 视图 → 显示模式 → **导入**，依次导入 `Display Modes` 下的 5 个 `.ini`。

---

## 5. 部署后验证

在 Rhino 中执行下列代码，或人工对照：

```python
import Rhino
print("默认模板:", Rhino.ApplicationSettings.FileSettings.TemplateFile)
for m in Rhino.Display.DisplayModeDescription.GetDisplayModes():
    print(" ", m.EnglishName)
```

通过标准：

- `FileSettings.TemplateFile` 指向新复制进去的 `TYARCH-general template-4.0.3dm`。
- 显示模式列表中能找到 5 个名称：`材质预览模式`、`线稿白模`、`阴影白模`、`带点线的材质预览模式`、`带阴影的材质预览模式`。
- 用模板新建文件，确认图层结构与 `TYARCH-Rhino建模指引.md` 第 2 章一致（10 个一级分区、单位毫米、容差 0.01）。

---

## 6. 注意事项

- **显示模式必须走 API**：见 2.2。Rhino 8 没有显示模式文件夹，复制 `.ini` 到任何位置都无效。
- **`.ini` 是 UTF-16 LE + CRLF**：如需编辑，另存时保持原编码与换行；存成 UTF-8 可能导致导入失败。
- **设置回写时机**：显示模式在 Rhino 运行时注册；默认模板设置即时生效，但设置文件在 Rhino 退出时才写回。部署完成后建议正常退出 Rhino 再重开一次。
- **部署范围仅限团队共用项**：仓库只收录团队共用的 5 个显示模式。维护机上可能另有个人定制模式（例如快速预览之类），属于个人环境，不纳入对外部署，也不必补齐。
- **尚未实测**：`ImportFromFile` 的实际导入结果、以及重复导入同一 `.ini` 时是覆盖还是新增，本文档编写时未做实测（在已装好该环境的机器上执行会污染现有配置）。首次在全新机器上部署时请留意第 5 步的验证输出，并把结果回填到本节。

---

*本文件与 `TYARCH-Rhino建模指引.md` 配套使用：本文负责把环境装好，建模指引负责装好之后怎么建模。*
