# MO2 管理 / 修改上古卷轴5 整合包时 xEdit 的使用指南

> 依据：xEdit 源码（本仓库 `xEdit/xeInit.pas`、`Core/wbCommandLine.pas` 等）与 MO2 官方/社区文档（Nexus 论坛、STEP Wiki、xEdit 官方文档）整理。
> 适用：用 Mod Organizer 2（MO2）管理 Skyrim SE/AE 或 LE 整合包（含 Wabbajack 类受控列表），并用 xEdit（SSEEdit / TES5Edit）查看、清理、打补丁。

---

## 一、核心铁律：必须经由 MO2 启动，不能单独双击

xEdit 通过**读取** `AppData\Local\<游戏名>\Plugins.txt` 来决定加载顺序。源码 `xEdit/xeInit.pas:567/580`：

```
wbPluginsFileName := GetCSIDLShellFolder(CSIDL_LOCAL_APPDATA) + wbGameName2 + '\Plugins.txt'
```

- **从 MO2 的 executable 列表启动**时，MO2 的 USVFS 会把 `AppData`、`Data` 全部**虚拟化成当前 profile 的版本**，于是 xEdit 看到的就是和游戏一模一样的合并后加载顺序。
- **在 MO2 外直接双击** `SSEEdit.exe`：它读到的是真实系统的 `Plugins.txt` 和真实 `Data`，**完全看不到你装的 mod**，等于在错误环境里操作。
  - 论坛里“C 盘 AppData 莫名生成 `Skyrim Special Edition` 文件夹”的困惑，本质就是没走 MO2。该文件夹是 xEdit/游戏的正常设置缓存位，**通过 MO2 启动时被虚拟化成 profile 版，无需手动改路径，也不必担心它在 C 盘**。

---

## 二、xEdit 的放置位置要求

**有要求：必须放在 MO2 的“非虚拟化真实目录”里**，推荐：
- MO2 实例自带的 `tools\SSEEdit\`（Wabbajack 整合包通常已如此），或
- 你自己的 `D:\Modding\Tools\xEdit\`

**不要**放在 `mods\` 里或游戏 `Data\` 里。原因（源码佐证）：
- xEdit 的**设置文件**写在 exe 同目录（`xeInit.pas:586`：`wbProgramPath + wbAppName + wbToolName + '.ini'`）。放真实目录才能保证设置不被虚拟化、不丢失。
- MO2 需要 exe 的**真实路径**来注入 USVFS；放 mod 里会被 MO2 当成 mod 管理，且其 dll 会被误判为 Data 文件。
- xEdit 自动定位游戏靠注册表 / Steam / `-D:`，对 exe 自身位置没有硬性路径要求，只要“真实、非 mod”即可。

---

## 三、从 MO2 启动的步骤

1. MO2 顶部齿轮 → **Executables（修改可执行程序）**
2. `[+]` → **Add from file…** → 选中 `SSEEdit.exe`（真实路径）
3. 标题随便起（如 `SSEEdit`），参数按需填（见下），点 **OK**
4. 右上角运行下拉框选它 → **Run**

MO2 默认就会给它挂上 USVFS，无需额外设置。

---

## 四、要加什么参数

**普通修改记录：其实什么都不用加**，MO2 已把环境喂对了。常用可选参数：

| 参数 | 作用 | 何时加 |
|---|---|---|
| `-IKnowWhatImDoing` | 关闭编辑警告、解锁高级功能 | 建议常加 |
| `-SSE` / `-TES5` | 指定游戏模式（SE/AE 用 `-SSE`，LE 用 `-TES5`） | 若 exe 名不是 `SSEEdit.exe` 时要加 |
| `-AllowMasterFilesEdit` | 允许编辑 ESM（master） | 需配合上一条；清理官方 master 时 |
| `-quickautoclean` | 快速自动清理单个插件（重复3次+报告） | 建第二个条目“Quick Auto Clean”专用 |
| `-autoload` | 跳过插件选择框，按 plugins.txt 全载 + 载入 ModGroups | 想直接进 |
| `-o:<路径>\` | 新建文件输出目录（默认落到 MO2 的 Overwrite） | 想直接输出到某 mod 时 |
| `-D: -P: -G: -C: -B: -T: -S:` | 显式指定 Data / plugins.txt / 存档 / 缓存 / 备份 / 临时 / 脚本目录 | 高级，一般不用 |

注意：**所有路径型参数结尾必须带反斜杠**（源码用 `IncludeTrailingPathDelimiter` 处理，见 `xeInit.pas`）。

**关于 `-moprofile:`**：这是给**老版 MO1** 的 `hook.dll` 机制用的（`xeInit.pas:488` 的 `..\Mod Organizer\hook.dll` + `:1571` 的 `-moprofile`），由 `wbMOHookFile` 加载。 **MO2 不需要也不建议用**，MO2 靠 USVFS，别画蛇添足。

---

## 五、针对“整合包”的特别注意事项（坑）

1. **不要直接改整合包自带的 mod 源文件**。整合包（尤其 Wabbajack）是“受控/可复现”的，直接编辑底层 mod 会被更新覆盖、也破坏可复现性。正确做法：用 xEdit **做 patch / override（右键复制记录到新插件）**，把产物放到更上层的独立 mod。
2. **输出会落在 Overwrite**：xEdit 新建/另存插件默认写到虚拟 Data（`xeInit.pas:478`：`wbOutputPath := wbDataPath`），被 USVFS 重定向到 MO2 的 **Overwrite**。改完务必把 Overwrite 里的产出**整理成独立 mod**，否则切换 profile / 重装会很乱（Overwrite 是实例级、不随 profile 备份）。
3. **清理官方 ESM**（如 `Update.esm` 的 ITM/UDR）才需要 `-AllowMasterFilesEdit -IKnowWhatImDoing`，且务必用 Quick Auto Clean 流程，谨慎操作。
4. **游戏模式要对应**：MO2 实例是 SE/AE，就用 `SSEEdit.exe` 或加 `-SSE`；LE 才用 `TES5Edit.exe` / `-TES5`，否则会读错游戏数据。
5. 改完 load order（启用/禁用/排序 mod）后，**重新从 MO2 启动 xEdit** 才会反映新顺序；xEdit 有缓存，必要时可用 `-resetsettings` 或启动时按 `Shift+Ctrl+Alt` 清设置。

---

## 六、一句话总结

**放真实 Tools 目录、用 MO2 添加为可执行程序并从 MO2 启动、普通编辑加个 `-IKnowWhatImDoing` 足矣、产物从 Overwrite 收进独立 mod、别碰整合包底层文件。**
