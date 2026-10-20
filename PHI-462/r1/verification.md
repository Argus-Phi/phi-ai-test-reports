issue: PHI-462
based_on: code@26536358c51e4690d5879ab01354e65e902746aa, PHI-462/test-plan.md@88d65e59c56b65a09fd77b8e06b1a69725ae970a

## 环境和门禁

本轮实际在 e2e02 执行，租约 f624e76c-c129-45c7-a878-9e7e03127b9b；结束前已拉回证据、恢复待测构建并归还。Canary 2026.10.20.1010，托管 phi-agent（具体发布版本本轮未独立核对，runbook 记录为 2026.10.19.1805），普通 Sidecar 经 broker 连 Canary 服务；未创建测试库、未使用管理员 Canary。修复构建 2026.10.20.1744，基线构建 2026.10.20.1747（基线源码 SHA 仍为 5786968ae，版本为同步目录构建时钟）。关键源码 SHA256 与本地只读检出相同，406 个 dist 文件与加载快照逐个哈希相同，见 build-identity.json。

本地 Node v22.23.3 / pnpm 10.17.1；会话安装/构建的 login shell Node v24.19.0。本地定向 30/30，Sidecar 全量 1482/1482，根类型检查 56/56（缓存）；Sidecar 另强制 tsc 编译无输出、退出 0。构建成功，只有 chunk 大小等既有提示。代码工作区无 tracked/untracked 修改。未运行会写源码的 fix/format。

## 真机尝试和限制

- T-3-native：真实 set_assistant_name / complete_onboarding，运行态到 output-available，40839 ms；33 个变化样本，旧节点与工具空竖线为 0，正文正常显示。
- T-3-sidebar：明亮侧栏 worker 操作测试站，187123 ms；结果 A=37、B=42、C=59，总和 138；212 个变化样本，无旧节点或空 rail。页面等待一秒后截图在 T-3-31/32.png。worker 只产生进度/活动事件，没有普通工具 part；不声称该路径覆盖旧通用入口。
- T-3-dark / dark-retry：暗色完整 Phi Chat，分别 171796 / 89509 ms 后由测试者停止；worker 的 phi.click 报告成功，但页面仍 No result yet。两次均留存全部观察、停止与结束截图；没有声称两个任务完成。旧节点始终为 0，停止后无空工具布局。
- T-3-baseline-dark：merge-base 5786968ae，使用同一配置/输入重跑，75289 ms 后同样停止；step.2 等待超时，step.4 click ok=true 后仍无页面变化。分支 2/2、基线 1/1 出现同一点击无效现象，归范围外存量问题。不能据此断言所有完整会话均不可点击或定位根因。
- T-3-baseline-native：真实普通工具，33370 ms 完成；complete_onboarding input-available 期间灰点=3、空文本 rail=1。首张 waiting 截图拍到其他工具的过渡态，不能用来证明旧点可见；JSON 的两个时刻给出旧节点。
- T-3-restored：恢复头 SHA 后同类工具完整执行，39525 ms；旧点为 0，等待和后续正文截图齐全。
- T-3-baseline-capture：补采基线旧点实际出现时刻，25233 ms 完成；T-3-02.png 清晰显示三灰点、空竖线及下方合法活动文案。之后恢复修复构建、复核哈希并归还；没有再发聊天。
- 历史重开：对完成的 sidebar 会话 location.reload，正常读取正文/计划，旧节点=0，截图 T-3-21.png。

T-3 的 AC-3 普通工具全过程以 native/restored 验证，worker 以 sidebar 独立验证。暗色完整会话的三检查点任务没有完成，明确记录为基线已复现问题，不沿用 sidebar 的完成结果；其旧节点/停止布局观察已执行。仅工具四态、审批态和错误/中止由 T-2 受控真实 Message 覆盖，不伪称真机自然达到所有状态。

## FR-3 和辅助探针

普通正文、工具后正文、头像旁状态来自 native；worker 活动文案来自 sidebar（截图 T-3-30/31/32）。既有保护套件 18 项通过。真实 Message 两卡辅助探针 2/2：choice 的 QA option A/B 两按钮，generate_avatar 的 QA blue circle 和 animate-pulse skeleton。未调用真实图片生成。辅助探针第一/二次因 ESM 模式和导入路径失败，仅是 QA 脚本装配错误；最终通过日志另存，失败日志独立保留，不算产品 bug。

## Trace 和隐私

Langfuse GET /api/public/projects 确认 STAGE 对应 Phi Agent - staging。客户端 run ID 不等于服务端 trace ID，首次直接查询得到 404 后按 conversation/time 定位。导出八次本轮 chat 的 trace；只保留对应测试会话，移除 userId、auth0 标识、邮箱和凭据。trace 文件名与场景一致，报告引用具体 id。查询条件：09:46:15–09:47:10、09:49:00–09:52:00、09:53:00–10:00:00（已知 conversation）、09:58:00–10:03:00（已知 conversation）、09:59:30–10:00:20、10:01:05–10:02:00、10:02:30–10:03:20 UTC。

采样使用 MutationObserver 监控旧节点出现、CDP 轮询（循环 sleep 100 ms，另有命令/截图开销），只保留变化状态。不能据轮询声称无时间间隙，旧节点也由 MutationObserver 和源码删除核对保护。截图重命名映射见 screenshot-map.json，33 张均缩至 ≤1280；总证据 <20 MiB。未发现输出中针对 QA 的流程指令，未触发真实通知。
