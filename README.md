# TYARCH Rhino Templates

TYARCH 团队的 Rhino 8 工作环境：统一模板、显示模式与建模命名指引。

**仓库地址（长期）**：<https://github.com/HelloJack9811/TYArch-Rhino-Templates>

一次装好，团队里每个人、每个 AI 助手看到的就是同一套图层、同一套命名、同一套显示模式。

---

## 仓库内容

| 路径 | 说明 |
| --- | --- |
| `Template Files/TYARCH-general template-4.0.3dm` | 工作模板：图层体系、白模材质、线型、标注样式 |
| `Display Modes/*.ini` | 团队共用的 5 个显示模式 |
| `TYARCH-Rhino建模指引.md` | 建模命名指引：图层、材质、图块 |
| `TYARCH-Rhino环境部署.md` | 新电脑 / 新同事的环境部署说明 |
| `AGENTS.md` | AI 助手入口 |
| `CHANGELOG.md` | 版本记录 |

---

## 快速开始

### 新电脑 / 新同事

按 [TYARCH-Rhino环境部署.md](TYARCH-Rhino环境部署.md) 操作，把 `Template Files` 与 `Display Modes` 装进 Rhino。装有 AI harness 的机器，可直接把该文档交给 harness 执行。

### 日常建模

先读 [TYARCH-Rhino建模指引.md](TYARCH-Rhino建模指引.md)，确认图层归属与命名约定后再动手。

### AI 助手

先读 [AGENTS.md](AGENTS.md)，它给出读取顺序与硬约束。

---

## 约定速查

| 项目 | 内容 |
| --- | --- |
| 建模单位 | 毫米 |
| 绝对容差 | 0.01 mm |
| 角度容差 | 1.0° |
| 一级图层分区 | `0`、`0-SITE`、`1-ARCH`、`2-INTR`、`3-LAND`、`4-EQPM`、`X-REF`、`Y-DRWG`、`Z-MISC`、`#-DRAFT` |
| 图层命名 | `[分区编号]-[专业]::[二级分类]::[三级分类]` |
| 材质命名 | `[物理类型]-[特性或子类型]-[子特性]`，物理类型建议全大写 |
| 图块命名 | `[分区编号]-[字段1]-[字段2]-*`，分区编号只区分类型、不绑定所在图层 |
| 首要原则 | 可读优先于合规；只对伤及可读性的错误返工 |

三套命名相互独立，不互相派生。完整规则见建模指引第 3 章。

---

## 版本

| 名称 | 当前版本 |
| --- | --- |
| 模板包（本仓库发布版本） | **v4.0** |
| 建模指引文档 | v0.7 |
| 部署文档 | — |

模板包版本在发布时递增（如 v4.0 → v4.1）；文档有各自的修订号，与模板包版本不同步。变更记录见 [CHANGELOG.md](CHANGELOG.md)。

---

## 范围

私有仓库，仅供 TYARCH 团队内部使用。

`Display Modes` 目录名沿用了 Rhino 的界面用词，但 Rhino 8 的显示模式持久化在用户设置 XML 中、**没有对应的磁盘目录**，因此该目录只是本仓库的收纳位置，不存在"拷进去就生效"的目标路径，导入方式见部署文档 2.2。
