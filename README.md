# ObjectARX SDK（精简版 · .NET 托管程序集库）

> AutoCAD 二次开发 **.NET（C# / VB.NET）插件**所需的托管程序集（DLL + XML）精选库。
> 按 **.NET TFM** 只保留代表版本，覆盖 **AutoCAD 2006 – 2027**。

> ⚠️ 本仓库是上游 `HexTTX/ObjectARX-SDK` 的 **fork（`ldd1good/ObjectARX-SDK`）**，已大幅精简，仅保留 .NET 开发必需内容，与原仓库不一致，属预期分叉。

---

## 为什么精简

原始仓库含 26 个版本完整 SDK（C++ 头、`.lib`、samples、docs、utils…）约 2.3 GB，但 .NET 插件编译**只需要**每个版本 `inc/` 下的**托管 DLL + 同名 XML**。其余内容对 C# 编译毫无用处，已全部删除。

本库因此只保留：

- 每个代表版本 `inc/` 中按文件名筛选出的 **13 个托管程序集**（DLL）+ 同名 `.xml`（IntelliSense）
- 配套工具脚本（见 `tools/`）

> 这是 **.NET 库**，不是 C++ SDK。没有 `inc-x64` / `lib-*` / `samples` / `docs`，无法用于 ObjectARX(C++) 开发。

---

## 版本覆盖（5 个代表版本，按 .NET TFM 取最低一代）

| 目录 | AutoCAD 年份 | ObjectARX | .NET TFM | CLR | 覆盖年份 | 托管程序集 |
|------|------------|-----------|----------|-----|---------|-----------|
| `2006-R16.2` | 2006 | R16.2 | net2.0 | 2.0 | 2006–2012 | AcMgd + AcDbMgd（**2 个**，无 AcCoreMgd\*） |
| `2013-R19.0` | 2013 | R19.0 | net4.5 | 4.0 | 2013–2014 | 三件套 + 7 个（**10 个**） |
| `2015-R20.0` | 2015 | R20.0 | net48 | 4.0 | 2015–2024 | 同 2013（**10 个**），**主参考版本** |
| `2025-R25.0` | 2025 | R25.0 | net8.0 | 8 | 2025 | 全 13 个 + 13 xml |
| `2027-R26.0` | 2027 | R26.0 | net10.0 | 10 | 2026–2027 | 全 13 个 + 13 xml |

\* `AcCoreMgd` 自 2013 起才从 `AcMgd` / `AcDbMgd` 拆分出来，故 2006 仅有 2 个托管 DLL。

**13 个托管程序集 stem**：`AcCoreMgd, AcCui, AcDbMgd, AcDbMgdBrep, AcDx, AcMgd, AcMr, AcSeamless, AcTcMgd, AcWindows, AdUIMgd, AdUiPalettes, AdWindows`
（2006 含前 2 个；2013 / 2015 含前 10 个；2025 / 2027 含全 13 个。2013 / 2015 的 `inc/` 未带 `.xml`，IntelliSense 文档缺失属可接受的取舍。）

---

## ★ 关键约束：.NET 运行时决定可运行范围

.NET 插件绑的是 **.NET 运行时（CLR）**，不是 CAD 版本号：

- **同 CLR 代内"低版本编、高版本跑"成立**：用 `2006-R16.2`（net2.0 / CLR2.0）编译的插件，可在 2006–2012（同为 CLR2.0 宿主）任意版本加载。
- **跨 CLR 代不可混用**：`2015-R20.0`（net48 / CLR4.0）的插件**不能**在 2006–2012（CLR2.0）宿主加载，反之亦然。
- 因此 **不能**用 2015 的 net48 包去开发 2006–2012 的插件——必须回到 `2006-R16.2`（net2.0）。
- 选代表版本的原则：**每个 .NET TFM 取覆盖区间内的最低 CAD 版本**（API 面最小、运行时兼容最高），其余年份复用同一目录的程序集引用即可。

---

## 目录结构

```
ObjectARX-SDK/
├── 2006-R16.2/inc/    # AcMgd.dll, AcDbMgd.dll
├── 2013-R19.0/inc/    # 10 个托管 DLL
├── 2015-R20.0/inc/    # 10 个托管 DLL（主参考）
├── 2025-R25.0/inc/    # 13 个 DLL + 13 xml
├── 2027-R26.0/inc/    # 13 个 DLL + 13 xml（NuGet 补充）
├── tools/             # 工具脚本（gitignore，见下方）
├── sdk-index.json     # 机器可读版本索引（gitignore，可重建）
├── AutoCAD二开对照表_彩色.png / _黑白.png
├── LICENSE
└── README.md
```

每个版本目录内**只有 `inc/`**，且 `inc/` 内**只有托管 DLL（+xml）**。无 C++ 头、无 `.lib`、无 samples / docs。

---

## 在 .NET 项目里引用

以 `csproj` 为例（目标 net48，开发 2015–2024 插件）：

```xml
<ItemGroup>
  <Reference Include="AcMgd">
    <HintPath>..\ObjectARX-SDK\2015-R20.0\inc\AcMgd.dll</HintPath>
    <Private>false</Private>
  </Reference>
  <Reference Include="AcDbMgd">
    <HintPath>..\ObjectARX-SDK\2015-R20.0\inc\AcDbMgd.dll</HintPath>
    <Private>false</Private>
  </Reference>
  <Reference Include="AcCoreMgd">
    <HintPath>..\ObjectARX-SDK\2015-R20.0\inc\AcCoreMgd.dll</HintPath>
    <Private>false</Private>
  </Reference>
</ItemGroup>
```

要点：

- `<Private>false</Private>`：**不**把 SDK DLL 拷进输出目录。CAD 进程已加载同版本程序集，随包分发 / 合并会冲突。
- 目标框架按上表选：2006–2012 → `net35`（CLR2.0）；2013–2014 → `net45`；2015–2024 → `net48`；2025 → `net8.0`；2026–2027 → `net10.0`。
- 多目标（如 `net48` + `net8.0`）时，用 `TargetFramework` 条件选不同 `HintPath` 指向对应年份目录。

---

## 工具脚本（`tools/`，本仓库 gitignore，不入库）

三者均可离线重建索引 / 补充版本，不影响主仓库内容：

| 脚本 | 用途 |
|------|------|
| `fetch_acad_nuget.py` | 从官方 NuGet（`AutoCAD.NET` / `.Core` / `.Model`，Autodesk 署名 + Authenticode 签名，包版本 == R 号）按年份抓包解包到 `<year>-R<ver>/inc/`。例：`python tools/fetch_acad_nuget.py --year 2024` |
| `gen_sdk_index.py` | 扫描所有版本目录生成 `sdk-index.json`（版本 → 托管程序集映射，机器可读） |
| `slim_managed.py` | 按 13 个托管 stem 递归只保留 `dll+xml`，删掉其余文件与空目录（用于精简某个版本目录） |

---

## 还原被删的版本

精简删除了 2000–2005、2007–2012、2014、2016–2024、2026 等目录，需要时可还原：

1. **从 git 历史取回**（若该版本曾被提交过）：
   ```bash
   git checkout <commit> -- 2024-R24.3/
   ```
2. **从官方 NuGet 现抓**（推荐，2024 / 2026 等）：
   ```bash
   python tools/fetch_acad_nuget.py --year 2024
   ```
3. 重新生成索引：`python tools/gen_sdk_index.py`

---

## 完整版本对照表

原仓库的 AutoCAD 版本 ↔ Visual Studio / .NET 要求对照图仍保留，供查全版本开发环境：

![AutoCAD 二开对照表（彩色版）](AutoCAD二开对照表_彩色.png)

![AutoCAD 二开对照表（黑白版）](AutoCAD二开对照表_黑白.png)

---

## 许可证

- **仓库结构、README、`tools/` 脚本**：MIT License（见 `LICENSE`）。
- **Autodesk ObjectARX SDK 文件**（各版本 `inc/` 下的 DLL / XML）：版权归 Autodesk, Inc.，受其专有许可协议约束，仅用于学习 / 开发目的。

---

## 仓库信息

- 上游：`HexTTX/ObjectARX-SDK`
- 本 fork：`ldd1good/ObjectARX-SDK`（已精简，与上游内容不一致，预期）
- 最后更新：2026-10-13
