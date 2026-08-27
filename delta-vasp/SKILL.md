---
name: delta-vasp
description: 在候选和科学参数已获人工批准后，通过 DELTA 公共入口准备、预检、明确启动、监控、对账、技术验收、技术报告及归档 VASP 批次。用于已登记生产批次、获批的新计算、运行恢复和验收；输出止于可追溯技术状态。不要选择候选、猜测参数、把 completed/pass 当成科学完成，或未经单独人工门自动承接设计与启动、验收与科学审阅。
---

# DELTA VASP 生产计算操作

按公共生命周期操作已获科学批准的 VASP 批次。技术工具不猜科学参数，也不把任务完成升级为科学问题完成。

## 输入与操作级别

先确认体系、科学目的、结构与参数依据、批次/attempt、任务阶段、期望验收、允许的写入范围，以及用户是否明确授权启动真实 VASP。未明确授权启动时，只做到准备、只读检查或预检。

从体系 `04_docs/文档索引.md` 定位当前参数与结构表示决策；按本次操作级别从[公共工具路由与安全门](references/tool-routing-and-safety.md)选择必要文档，不默认通读全部工作流。

声子批次还必须遵守[公共谐性声子工作流](../../../docs/工作流/声子工作流.md)的 NAC 前置门；本 Skill 不在此重复科学规则。

## Codex 与受限执行环境边界

完整受限环境和 `--check-only` 语义见[公共脚本与监控](../../../docs/工作流/VASP脚本与监控.md)。本 Skill 只保留执行边界：

- Codex 默认不启动 MPI/Hydra/VASP；普通维护不做“顺手” runtime 验证。
- `--check-only` 不是 MPI launchability 证据；`--mpi-runtime-check` 只在实际生产环境或用户明确要求时使用。
- 出现 Hydra/socket 权限错误时停止，不换 launcher、核数或科学参数重试；日常只做静态检查、dry-run 和已有测试。

## 最小上下文

默认只读根 `AGENTS.md`、目标体系文档索引/参数决策、批次 `manifest.yaml`、`说明.md`、正式 `campaign.json` 和本次动作对应的公共接口。每个任务只加载本 Skill，不预读其他 Skill，不递归展开索引。监控时增量读取 queue/runtime/`进度.md`；验收时只读摘要和必要日志末尾。历史、旧 campaign、故障文档和大型输出只在身份冲突、来源缺口或明确故障时追溯。

## Born/NAC 与声子后处理路由

完整 NAC、Phonopy、标准 band 和固定产物规则唯一见[公共谐性声子工作流](../../../docs/工作流/声子工作流.md)。本 Skill 只保留门：

- 声子批次元数据必须声明 NAC applicability；若为 `required`，正式后处理前必须有与当前结构/体积、组成、POTCAR/Hamiltonian
  和原子顺序匹配且已验收的 Born/ε∞ 证据。
- NAC 缺失或不匹配时可记录 `NAC_PRECONDITION_MISSING`，不得把 non-NAC 诊断发布为最终 stability/thermodynamics 结论。
- Born response 只能在独立批准的批次中准备或验收；发现 NAC 缺失不得自动启动该批次。
- harmonic phonon 技术完成前必须通过公共标准 band/provenance 门；具体命令和产物清单按上述工作流读取。

## 操作顺序

1. 按[工具路由与安全门](references/tool-routing-and-safety.md)确认批次事实、参数依据和当前动作；批次身份规则不在 Skill 重复定义。
2. 用体系准备器的 `--check-only`/`--dry-run` 核对冻结来源和覆盖风险，并只向全新 `02_work/` 生成输入。
3. 若当前批次要复用旧批次已验收资产，先复制到当前批次对应的 `02_work/` 或产品目录，再执行后处理或生成依赖任务；保留旧源目录，
   不移动、覆盖或仅以软链接替代当前副本。为每项复用登记来源 batch/task/workdir、目标路径、验收证据、协议/结构兼容性和源/目标哈希。
   体积依赖的声子力任务按当前体积保留在 `v{volume}/03_phonon/PHONON-DISP-*`；历史最终 FC2/BORN 等体积级资产若位移协议不同，
   只复用已验证最终产品，并显式记录协议差异，不机械拼接历史位移目录。
4. 依次执行 `vasp_accept.py --input-only`、代表最大 ranks 的 runner `--check-only` 和同一 campaign 的队列 `--check-only`。
5. 只有用户明确授权且 campaign 经人工确认，才用公共队列 `--detach`/`--supervised` 启动；完成一次启动稳定性检查后即可会话交接。
6. 稳定后默认不陪跑到完成。恢复先用同一 config/campaign 执行 `--status` 或 `--reconcile-only`；没有完整成功证据时暂停，
   不自动重算、改参或用 `CONTCAR` 覆盖 `POSCAR`。
7. 用 `vasp_accept.py` 和 `vasp_report.py --check-only` 完成技术验收；声子任务另过正式 workflow 的标准 band/provenance 门。
8. 如实更新 manifest/说明和输出链接，运行索引/链接检查；停止、失败和诊断证据保留。

## 经验接口

普通生产任务过程中不修改经验、Skill、规范或 AGENTS。公共工作流和当前体系文档始终优先；仅在异常排查时查 `docs/经验记录/README.md`，并针对本次 campaign、attempt、环境和输入复核。任务结束时可标记“经验候选”及证据；后续由独立的 `$delta-experience` 任务治理。经验不能绕过预检、启动授权或验收。

## 停止与交接

输入必须包括研究者已批准的候选/结构、科学参数依据、正式批次和当前操作授权；缺一则停在准备清单或只读预检。`delta-design` 的候选包不是生产授权，用户对未来“准备并启动”的概括也不能预先确认尚未生成的 campaign。输出是运行/技术状态、验收与技术报告，到此停止。需要解释结果或提出阶段 gate 时，只把验收证据、正式标签和报告路径交给 `$delta-review`；不得自动调用它或启动其建议的下一阶段。

## 禁止事项

- 不自动启动、续算、重试或切换参数；不在旧输出目录原地重跑。
- 不从 Skill、硬编码默认值或其他体系复制 ENCUT、K 点、U、赝势、磁性、并行参数。
- 不恢复已封存的 dynamic ranks，不建立体系私有第二套队列/监控/验收。
- 不删除 runtime/输出来绕过启动器保护，不修改已有 VASP 输入输出。
- 不完整读取大型 `OUTCAR`/`vasprun.xml`，不读取 `WAVECAR`/`CHGCAR`。
- 不把 `completed` 或公共验收通过写成科学 Go。

## 验收与输出

操作记录至少给出批次与 attempt、参数依据、运行模式、执行/未执行的命令、输入检查、启动授权、任务状态、失败证据、逐阶段验收、资源分配、报告位置和待人工科学审阅项。没有启动真实 VASP 时明确写明。
