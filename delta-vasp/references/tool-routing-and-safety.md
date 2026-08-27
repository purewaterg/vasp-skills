# VASP 公共工具路由与安全门

## 按动作读取

所有任务先读目标体系 `04_docs/文档索引.md`、当前参数/结构决策和目标批次事实；只有需要核对具体脚本的
CLI、退出码、稳定 I/O 或实现身份时才读 `docs/规范/脚本接口清单.md` 的对应条目或直接运行 `--help`。再按动作增量读取：

| 动作 | 再读 |
|---|---|
| 新建/归档批次 | `docs/工作流/计算批次创建与归档流程.md` |
| 准备/输入预检 | `docs/工作流/VASP计算规范.md` |
| 启动/恢复/队列状态 | `docs/工作流/VASP队列与调度.md` |
| 单任务监控/验收 | `docs/工作流/VASP脚本与监控.md` |
| 生命周期全貌或动作跨两阶段以上 | `docs/工作流/工作流入口.md`，再读所涉专项文档 |

不要默认通读上述全部文件，也不要递归展开索引或预读其他 Skill。历史审计、旧 campaign 和故障文档只在现行证据冲突、恢复身份不清或出现对应故障时读取。文档冲突时停止并核对当前入口和日期版本，不用 Skill 示例覆盖项目现行接口。

## 权威工具

| 阶段 | 入口 | 默认边界 |
|---|---|---|
| 注册批次 | `.delta/scripts/create_run.py` | 写 `说明.md`/manifest；不启动 VASP |
| 初始化 campaign | `.delta/scripts/vasp_queue.py --init-campaign` | 只写正式批次 `01_source/`；人工填写后才能用 |
| 输入验收 | `.delta/scripts/vasp_accept.py --input-only <dir>` | 只读；不批准科学参数 |
| 静态启动器检查 | `.delta/scripts/run_vasp.sh --check-only <ranks>` | 只检查输入、环境、路径、资源和覆盖风险；不启动 MPI 或 VASP |
| MPI runtime 检查 | `.delta/scripts/run_vasp.sh --mpi-runtime-check <ranks>` | 明确启动无副作用 MPI bootstrap/socket 检查，不启动 VASP；仅实际生产环境或用户明确要求 |
| 队列预检 | `.delta/scripts/vasp_queue.py ... --check-only` | 不创建生产 state，不启动任务 |
| 正式启动 | `.delta/scripts/vasp_queue.py ... --detach` | 需要明确启动授权和正式 campaign |
| 恢复核对 | `.delta/scripts/vasp_queue.py ... --reconcile-only` | 不自动重跑 |
| 单任务/批量验收 | `vasp_accept.py`、`vasp_report.py --check-only` | 只读复核 |
| 发布技术报告 | `vasp_report.py` | analysis/docs 分流；不产生人工科学结论 |
| 索引与链接 | `update_indexes.py --check`、`check_markdown_links.py --check` | 不读取大型 VASP 输出 |

命令的完整参数和退出码始终以 `docs/规范/脚本接口清单.md` 及 `--help` 为准。

## 安全门

- **启动门**：用户明确要求真实启动；campaign 绑定已注册批次并经人工确认；所有只读预检通过；正式队列独立托管。
- **参数门**：结构、赝势和科学参数由体系正式文档/冻结批次输入给出；预检通过不等于参数获批。
- **覆盖门**：全新任务目录；任何 VASP 产物、集中式 runtime、历史 `.runtime` 或不同内容文件都拒绝覆盖。
- **恢复门**：先 reconcile；同科学参数才递增 attempt，参数/目的改变则新建 batch。
- **完成门**：公共验收满足对应静态/弛豫条件；随后仍需独立的科学审阅。
- **清理门**：现役生命周期工具只读，不含删除许可。停止、失败、诊断和已启动目录默认保留。

固定 ranks、CPU 租约和槽位释放后即时补位由公共队列唯一管理。任何历史 dynamic ranks 代码或字段只作追溯，不得启用。
