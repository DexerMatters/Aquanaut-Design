# 对到19db5d8的审查

### 当前存在的问题

### P0 / P1 — 应尽快处理

| #    | 发现                                                         | 来源                  | 现状核实                                         |
| ---- | ------------------------------------------------------------ | --------------------- | ------------------------------------------------ |
| A1   | **CI 全红且 fork 未启用 Actions**；根因 `gradle.properties:7` 硬编码 `/usr/lib/jvm/java-21-graalvm` | 主报告 R2+R11         | 仍在（10-01 复核）                               |
| A2   | **美术许可 ND/SA 自相矛盾**：`gradle.properties:33` 写 CC BY-NC-**ND**，README 与 LICENSE-ART 正文写 CC BY-NC-**SA**，LICENSE-ART 内链接还指向 ND 法律文本 | 主报告 R1（残留部分） | 修复一半后仍剩此矛盾（10-01 复核）；一行取齐即可 |
| A3   | **调查板演示进度写进真实存档**：`InvestigationSavedData` 无参构造器硬编码星星与已发现链接（注释自认开发脚手架） | 分支审评 P1-1         | 已随 `32dd50f` 进 main，未修                     |
| A4   | **调查进度无游戏内来源**：`InvestigationBoardApi` 三个写方法零调用方，玩家看到的永远是 A3 的假数据 | 分支审评 P1-2         | 已进 main，未修                                  |
| A5   | **额外氧气存静态 Map 不持久化 + 登录自动补满**（`AirSupplyHelper.EXTRA_AIR`、`PlayerAirEvents`） | 主报告 R6             | 未变                                             |
| A6   | **世界生成 Mixin 强侵入**：`RandomStateMixin` 全局 continents×3；`NoiseBasedChunkGeneratorMixin` 整体替换 doFill（本批 blend 重构优化了其内部性能，注入面未变） | 主报告 R3/R4          | 未变（10-01 diff 复核）                          |
| A7   | **mods.toml 描述仍为 "Example mod description."**，authors/logo/issueTracker 空 | 主报告 R7             | 仍在（10-01 复核）                               |
| A8   | **管道源/汇抽象零实现**：`AirConnector` 无任何 `implements`，泵/汇玩法未落地（计划已搁置 4 个月） | 主报告 R10            | 未变                                             |
| A9   | **12 个 main-harness 测试不入 gradle test / CI**（尽管 @Test 总数已 302，这批仍靠手动） | 主报告 R5             | 未变                                             |
| A10  | **架构长期债**：24/28 包单一大 SCC（`BiomeRegistry↔MiddleLevelOceanRegion` 真实跨包环）；三个上帝类（FishMovementController WMC 235 / BaseFishEntity 157 方法 / NotebookScreen WMC 149） | 主报告 R8/R9          | 未变；新代码未加剧                               |

### P2 — 计划内改进

| #    | 发现                                                         | 来源                         |
| ---- | ------------------------------------------------------------ | ---------------------------- |
| B1   | **照片 PNG 内嵌物品组件随物品全量同步**（≤384KB/张，箱子堆放理论 10MB 级包量）——建议改 ID 引用集中存储 | 第二轮（新）                 |
| B2   | **Faithful 贴图运行时下载**（`GenerateAquanautNotebookTexture.py:76`）：构建不可复现 + 响应不限长 + 第三方派生素材与本项目美术许可兼容性存疑——建议内置或自绘 | 第二轮（新）                 |
| B3   | 调查板 payload 用 JSON 字符串传输（与旧 `AquariumInventorySyncPayload` 同型坏味），建议结构化 StreamCodec | 分支审评（新）               |
| B4   | `InvestigationCatalog` prerequisites 只查存在性不查环，未来做前置解锁即踩坑 | 分支审评（新）               |
| B5   | 测试 shim `net.minecraft.core.Direction` 遮蔽真实类且 ordinal 语义不符（0=NORTH vs 真实 0=DOWN） | 分支审评（新）               |
| B6   | `InvestigationProgress.deserialize` 吞掉全部异常静默清零进度 | 分支审评（新）               |
| B7   | `SavedData.Factory` 误用 `DataFixTypes.LEVEL`                | 分支审评（新）               |
| B8   | `network → client` 反向依赖（payload lambda 惰性引用 Client 类；无崩溃风险但属分层违规；新增 payload 复制了同模式） | 主报告 R13                   |
| B9   | 仓库卫生：根目录 `net/` 1,014 行垫片 + `logs/latest.log` 仍入库 | 主报告 R12（10-01 复核仍在） |
| B10  | `scripts/` 无 requirements.txt（Pillow 面随新脚本扩大）      | 主报告 R14                   |
| B11  | 旧 14 生物 `models.zip` 源工程仍不入库（新资产 `blockbench-scripts/*/src/` 已入库，部分缓解） | 主报告 R15                   |
| B12  | Mixin refmap 手写陈旧 + remap 标志混用                       | 主报告 R17                   |
| B13  | `EntityMixin.java:23-28` 注释与代码不符（引述位置已重构，待逐字复核） | 主报告 R16                   |

### P3 / 观察项

- 调查板：重复同步无条件替换已打开 Screen；BE 懒初始化竞态；`/setblock` 绕过多格放置契约（降级安全）；提交信息拼写 "Intialize"。
- 相机：10 tick 授权窗口在 >500ms 延迟下误拒（可放宽 20）；`PhotoTextureCache.FAILED` 满 64 全清可改 LRU。
- blend：`softMax` 全零权重返回 NaN 依赖调用方兜底；`guardedFloor` 可省一次分配。

### 一些建议

**新增代码质量是全项目最高档**：相机管线（服务端授权令牌 + PNG 逐块 CRC 校验 + 有界缓存）、blend 地形重构（纯函数化 + halo 网格跨 chunk 位级一致）、水世界预设（纯数据包）。这些模块的工程水准应作为后续开发范式。

### 待确认问题

① 生产环境手写 refmap 是否无副作用；② CI 实际报错确认；③ "Useful Hats 互操作"全历史零痕迹；④ 6 个 notebook 垫片测试失败是否仍在；⑤ 是否存在 Modrinth/CurseForge 发布页（仓库内无链接、无 Release）；⑥ 美术许可最终意图 ND 还是 SA（维护者裁决后按 A2 一行取齐）；⑦ `logs/latest.log` 的 14 条 GeckoLib 动画错误是否已修；⑧ AirConnector 落地时间表。