You are a senior DevOps / Site-Reliability engineer. The current directory is an empty `ops/` folder I just created in my user home. The machine already has **Pi** installed and I have **passwordless sudo** enabled. Do NOT ask me any questions — make reasonable decisions and scaffold everything yourself, then fill in the machine-specific facts by inspecting the system.

Build a complete **Pi 运维知识库** here. This must be **Pi 专属**（不是通用 agent）——它依赖 Pi 的项目级资源：`AGENTS.md` 自动加载、`.pi/extensions/*.ts` 扩展、`.pi/skills/*/SKILL.md` 技能自动发现。不要把任何机制改成其他 agent 的写法（不要写 `.cursorrules`、`CLAUDE.md`、`settings.json` 之类）。

# 一、目标架构

这套知识库解决「AI 运维」的四个痛点：

1. **记不住**：每次会话开工不知道这台机器有哪些服务、之前改过什么、还有什么没收尾 → 用 `AGENTS.md`（静态注入）+ 扩展（动态注入）解决。
2. **反复踩坑**：同一个坑踩两次 → 用 `pitfalls/` 四段式坑档案解决。
3. **改完不留痕**：改了什么没记录 → 用 `logs/changelog.md` 只追加 + ops-record 归档纪律解决。
4. **规范漂移**：文档和现实不一致 → 用 `inventory/` 实测快照 + `specs/` 活文档解决。

工作机制是三层：

| 层 | 载体 | 触发 | 作用 |
| --- | --- | --- | --- |
| 静态注入 | `AGENTS.md` | Pi 启动即加载 | 系统概况 + 检索路由 + 维护纪律 |
| 动态注入 | `.pi/extensions/ops-context.ts` | 每次会话 agent 首次启动 | 注入「最近变更摘要 + 未解决坑提醒」 |
| 流程技能 | `.pi/skills/ops-*/SKILL.md` | 按描述自动触发 | 排障 SOP / 归档纪律 |

# 二、要创建的完整文件清单（严格按此结构，缺一不可）

```
ops/
├── AGENTS.md                          # 常驻上下文（写完按下方模板，占位符用实测值替换）
├── README.md                          # 仓库说明 + 目录树 + 自动化机制表
├── .gitignore                         # 忽略 .pi/tasks/、.wrangler/、*.lock
├── specs/
│   ├── conventions.md                 # 写作约定（下方模板）
│   ├── deployment.md                  # 服务部署 house standard（下方模板）
│   ├── maintenance.md                 # 监控/备份/更新/日志 + 对账审计（下方模板）
│   └── decisions.md                   # 决策账本（仅建头部，后续追加）
├── runbooks/                          # 空目录（服务手册按需一事一档）
│   └── .gitkeep
├── pitfalls/
│   └── _index.md                      # 坑索引表头（下方模板）
├── logs/
│   └── changelog.md                   # 变更日志（下方模板，只追加）
├── inventory/                         # 系统资产快照（这三份用实测填，见第三部分）
│   ├── machine.md
│   ├── services.md
│   ├── hosts.md
│   └── network.md
├── reports/                           # 审计报告只读存档
│   └── .gitkeep
├── scripts/                           # 可复用运维脚本
│   └── .gitkeep
├── templates/
│   └── pitfall.md                     # 坑文档四段式模板（下方模板）
└── .pi/
    ├── extensions/
    │   └── ops-context.ts             # 动态上下文注入扩展（下方代码，原样）
    └── skills/
        ├── ops-record/
        │   └── SKILL.md               # 归档纪律技能（下方模板，原样）
        └── ops-troubleshoot/
            └── SKILL.md               # 排障 SOP 技能（下方模板，原样）
```

# 三、第三部分：先用命令实测，再填 inventory 与 runbooks

写完上面所有模板/代码后，**不要停在骨架**。立即实测这台机器的现状并落盘，否则知识库是空的、没价值：

1. 采集事实：
   - `hostname` / `lscpu` / `free -h` / `df -h /` / `ip addr` （填 machine.md）
   - `docker ps -a` / `ss -tlnp` / `systemctl list-units --type=service --state=running`（填 services.md：Docker 服务表 + 非 Docker 服务表）
   - `ip addr` + 组网命令（如 `tailscale status`，若装了）填 network.md（局域网 / 组网 / Docker 网桥三层）
   - SSH 别名、其它主机台账填 hosts.md（若暂时没有就写「待补充」）
2. 为每个正在运行的重要服务建 `runbooks/<服务>.md`：端口 / compose 目录 / 数据目录 / 常用操作 / 排障，头部写「最后验证：YYYY-MM-DD（实测）」。
3. 如果你在搭建过程中发现自己踩了坑，立即用 `templates/pitfall.md` 建一篇 pitfall 并登记进 `pitfalls/_index.md`。
4. 全部完成后：`git init` → `git add -A` → `git commit -m "init: 运维知识库脚手架 + 实测快照"`。

# 四、各文件的模板内容

> 下面所有 `<>` 占位符（如 `<主机名>`）在你自己实测后替换成真值；模板里的「代理/组网/共享数据库」等词同样替换成这台机器真实存在的组件，不存在的整段删掉，不要无中生有。

## 4.1 AGENTS.md

```markdown
# <项目名> —— AI 工作规则

你在 <主机名> 服务器的运维仓库中工作。本文件是你的常驻上下文：任何针对这台机器的系统操作都必须遵循以下规则。

## 系统概况（速览）

- 主机：**<主机名>** · <系统>（rolling）· <CPU> · <内存> · <磁盘>
- 网络：局域网 `<LAN_IP>`（<网卡>）· **组网虚拟网 `<组网IP>`（自托管 headscale）**
- 代理：<代理内核>（mixed-port `127.0.0.1:7890`，rule 模式；如有容器专用口一并写明）
- 服务形态：Docker Compose 为主（compose 在 `/srv/compose/<service>/`，数据在 `/data/services/<service>/`）
- sudo：已开免密，AI 可直接 `sudo -n`；危险操作（删数据、下线共享服务、动代理/组网）仍须先说明影响并确认
- 完整事实快照：inventory/machine.md · services.md · network.md

## 检索路由（先查资料再动手）

| 场景 | 先看什么 |
| --- | --- |
| 开始任何维护/部署任务 | `inventory/services.md` 确认端口与依赖现状 |
| 操作某个具体服务 | 对应 `runbooks/<服务>.md` |
| 排查故障/异常 | `pitfalls/_index.md` → 命中坑文档 → 实测 |
| 需要「之前改过什么」 | `logs/changelog.md` |
| 部署新服务 / 下线服务 | `specs/deployment.md` |
| 监控 / 备份 / 更新 | `specs/maintenance.md` |
| 不确定本仓库怎么写 | `specs/conventions.md` |

## 维护纪律

1. 动手前：对照 inventory 确认端口/依赖不冲突；不确定的事实以实测为准（`ss -tlnp`、`systemctl cat`、`docker ps`），不凭记忆或旧文档。
2. 完成后必须归档（ops-record 流程）：`logs/changelog.md` 追加一行；踩新坑建 pitfalls 文档并更新 `_index.md`；拓扑变化同步 inventory；规范过时修订 specs。
3. 诚实记录：changelog 只追加不改历史；坑文档写全「现象→原因→解法→预防」。
4. 危险操作先确认：删数据、下线共享服务、动代理/组网（会断网）之前说明影响并获确认。
```

## 4.2 README.md

```markdown
# <项目名> —— <主机名> 系统运维知识库

由 AI（Pi 会话中的助手机器人）负责日常维护，配合三层自动化机制工作。

## 目录结构

（见仓库根目录实际树，逐项注释每个目录用途）

## 自动化机制（AI 的「条件反射」）

| 层 | 位置 | 作用 |
| --- | --- | --- |
| 静态注入 | `AGENTS.md` | Pi 启动即加载：知道去哪查资料、遵守什么纪律 |
| 动态注入 | `.pi/extensions/ops-context.ts` | 会话启动注入「最近变更摘要 + 未解决坑提醒」；`/ops-brief` 手动刷新 |
| 流程技能 | `.pi/skills/ops-troubleshoot/` | 排障 SOP：先查坑索引 → 定位服务 → 实测 → 归档 |
| | `.pi/skills/ops-record/` | 收尾纪律：变更后强制归档 |

> `.pi/` 下属项目级资源，首次在本目录启动 Pi 时会请求信任该项目，信任一次即可。
```

## 4.3 .gitignore

```gitignore
.pi/tasks/
.wrangler/
*.lock
```

## 4.4 .pi/extensions/ops-context.ts（原样写入，不要改代码逻辑）

```typescript
/**
 * ops-context —— <主机名> 运维知识库动态上下文注入
 *
 * 会话内首次 agent 启动时，自动读取：
 *   1. logs/changelog.md 最近变更（尾部若干行）
 *   2. pitfalls/_index.md 中状态为「未解决」的条目
 * 以一条对用户不可见的消息注入 LLM 上下文，让 AI 一开工就知道
 * "之前干了什么、还有什么没收尾"。
 *
 * /ops-brief 命令可手动查看摘要并强制下轮重新注入。
 */
import type { ExtensionAPI } from "@earendil-works/pi-coding-agent";
import { readFile } from "node:fs/promises";
import { join } from "node:path";

const CHANGELOG_TAIL_LINES = 12;
const MAX_BRIEF_CHARS = 2400;

async function readText(path: string): Promise<string | null> {
	try {
		return await readFile(path, "utf8");
	} catch {
		return null;
	}
}

function changelogTail(raw: string): string | null {
	const lines = raw
		.split("\n")
		.map((l) => l.trim())
		.filter((l) => l.startsWith("- "));
	if (lines.length === 0) return null;
	return lines.slice(-CHANGELOG_TAIL_LINES).join("\n");
}

interface OpenPitfall {
	title: string;
	file: string;
}

function openPitfalls(raw: string): OpenPitfall[] {
	const results: OpenPitfall[] = [];
	for (const line of raw.split("\n")) {
		if (!line.trim().startsWith("|")) continue;
		const cells = line.split("|").map((c) => c.trim());
		// | 日期 | 标题 | 标签 | 状态 | 文档 |
		if (cells.length < 6) continue;
		if (cells[1] === "日期") continue; // header row
		const status = cells[4] ?? "";
		if (!status.includes("未解决")) continue;
		results.push({ title: cells[2] ?? "(无标题)", file: cells[5] ?? "" });
	}
	return results;
}

export default function opsContextExtension(pi: ExtensionAPI) {
	let injected = false;

	const buildBrief = async (cwd: string): Promise<string> => {
		const parts: string[] = [];

		const changelog = await readText(join(cwd, "logs", "changelog.md"));
		if (changelog) {
			const tail = changelogTail(changelog);
			if (tail) parts.push(`【最近运维变更】\n${tail}`);
		}

		const index = await readText(join(cwd, "pitfalls", "_index.md"));
		if (index) {
			const open = openPitfalls(index);
			if (open.length > 0) {
				const list = open.map((p) => `- ${p.title} ${p.file}`).join("\n");
				parts.push(
					`【未解决的坑 ×${open.length}】处理相关任务前先读对应文档：\n${list}`,
				);
			}
		}

		if (parts.length === 0) return "";
		let brief =
			"[ops-context] <主机名> 运维知识库自动简报。维护纪律见 AGENTS.md；任务收尾必须走 ops-record 归档流程。\n\n";
		brief += parts.join("\n\n");
		if (brief.length > MAX_BRIEF_CHARS) {
			brief = `${brief.slice(0, MAX_BRIEF_CHARS)}\n…(已截断，完整内容请读源文件)`;
		}
		return brief;
	};

	pi.on("before_agent_start", async (event, ctx) => {
		if (injected) return undefined;
		const brief = await buildBrief(ctx.cwd);
		if (!brief) return undefined;
		injected = true;
		return {
			message: {
				customType: "ops-context",
				content: brief,
				display: false,
			},
		};
	});

	pi.on("session_start", () => {
		injected = false;
	});

	pi.registerCommand("ops-brief", {
		description: "显示 ops 知识库简报并刷新下次注入",
		handler: async (_args, ctx) => {
			injected = false; // 下轮 agent 启动重新注入最新内容
			const brief = await buildBrief(ctx.cwd);
			ctx.ui.notify(
				brief || "ops 简报为空（changelog/pitfalls 为空或缺失）",
				"info",
			);
		},
	});
}
```

## 4.5 .pi/skills/ops-record/SKILL.md

```markdown
---
name: ops-record
description: 运维动作收尾归档纪律。完成任何系统变更、故障修复、服务部署/下线、配置修改之后必须使用；也用于把新踩的坑沉淀为 pitfall 文档。保证知识库与系统状态始终一致。
---

# 运维归档流程（做完事必须走，缺一不可）

## 1. 记 changelog（必做）

向 `logs/changelog.md` 末尾追加一行：

```text
- YYYY-MM-DD [类别] 一句话结果 (@操作者)
```

类别：`deploy` / `fix` / `maint` / `pitfall` / `cron`。只追加，不改历史。

## 2. 踩了新坑 → 建 pitfall 文档

1. 复制 `templates/pitfall.md` 为 `pitfalls/YYYY-MM-DD-slug.md`（slug 英文短横线）
2. 写全四段：现象（报错原文）/ 原因 / 解法（可复制的命令）/ 预防
3. 在 `pitfalls/_index.md` 表格加一行；问题已解决则标「已解决」

## 3. 拓扑变化 → 同步 inventory

新增/删除/改端口/改依赖任何服务，当天更新 `inventory/services.md`（必要时 `network.md`），并刷新文件头快照日期。

## 4. 规范过时 → 修订 specs

发现 specs/ 与现实不符，直接修改对应文件。

## 5. git 提交 + 收尾自检

```bash
cd ~/ops && git add -A && git commit -m "<类别>: <一句话>"
```

自检清单：

- [ ] changelog 有新条目？
- [ ] 新坑有文档且进了索引？
- [ ] inventory 与现实一致？
- [ ] git 已提交？

最后向用户报告归档了哪些文件。
```

## 4.6 .pi/skills/ops-troubleshoot/SKILL.md

```markdown
---
name: ops-troubleshoot
description: 系统排障 SOP。当出现服务不可达、容器异常退出、端口不通、报错堆栈、磁盘/内存告警、网络断连等需要排查定位的场景时使用。指导按固定顺序检索知识库并实测验证。
---

# 排障 SOP

严格按以下顺序进行，不要跳步。

## 0. 先查坑（30 秒，命中率最高）

读 `pitfalls/_index.md`，按症状匹配标签（docker / network / proxy / systemd / ssh…）。命中就先读对应坑文档——历史问题大多已经踩过并写了解法。

## 1. 定位对象

查 `inventory/services.md` 找到涉事服务的：宿主端口、compose 目录、数据目录、依赖关系（如某服务依赖共享数据库）。非 Docker 服务看同文件「非 Docker 服务」表。

## 2. 实测现状（以命令输出为准，不信记忆）

```bash
# 容器类
docker ps -a --filter name=<name>          # 状态？(healthy)？
docker logs --tail 100 <name>              # 最近日志
docker inspect --format '{{.State.Health.Status}}' <name>

# systemd 类
systemctl status <unit>
journalctl -u <unit> -n 100 --no-pager

# 端口与资源
ss -tlnp | grep :<port>
df -h / && free -h && uptime
```

## 3. 网络类问题：分层排查

1. 容器内 → 2. 宿主绑定（127.0.0.1）→ 3. 组网层（Tailscale IP）→ 4. 局域网（LAN IP）

- 记住地址体系（见 inventory/network.md），确认用户从哪一层访问
- ⚠️ 若代理内核开了 TUN/fake-IP，容器内连宿主服务要用网关 IP，不用 host.docker.internal

## 4. 回溯变更

翻 `logs/changelog.md` 最近的条目：「之前改了什么」往往就是根因。

## 5. 解决后必须归档（走 ops-record skill）

新坑 → pitfalls 四段式文档 + 更新 _index.md；一切处置 → changelog 追加；拓扑变化 → 同步 inventory。

## 红线

- 需要重启的共享服务（如共享数据库）先说明影响再动手
- 代理 / 组网出问题会导致断外网/断组网，操作前告知用户
```

## 4.7 specs/conventions.md

```markdown
# 本仓库写作约定

## 语言

中文为主，技术术语/命令保留英文。

## pitfalls（坑文档）

- 文件名：`YYYY-MM-DD-slug.md`（slug 用英文短横线）
- 内容用 `templates/pitfall.md` 四段式：现象 → 原因 → 解法 → 预防
- 新建后**必须**同步在 `_index.md` 表格加一行；解决后把状态改「已解决」，**不删除文档**
- 标签从既有标签中选/新增小写短词

## logs/changelog.md

- 只追加，永不修改或删除历史条目
- 格式：`- YYYY-MM-DD [类别] 一句话结果 (@操作者)`
- 类别：`deploy` `fix` `maint` `pitfall` `cron`
- 一天多条按时间顺序排列

## inventory/

- 一律以实测为准（ss/systemctl/docker ps），更新时刷新文件头「快照日期」
- 服务拓扑任何变化当天同步

## specs/

- 活文档，直接修订保持最新；重大方向变化在 changelog 记一条 `maint`
- `specs/decisions.md` 决策账本：关键决策以 `## D-NNN 标题` 追加，append-only

## git

- 启用 git；每次归档完成后 commit（消息格式 `<类别>: <一句话>`）
```

## 4.8 specs/deployment.md

```markdown
# 服务部署规范（house standard）

## 目录与文件约定

- Compose 定义：`/srv/compose/<service>/compose.yml`（父目录 root 属主，创建命令交用户跑 sudo）
- 密钥：`.env` 放同目录，`env_file: .env` 加载；**绝不内联密钥**
- 持久化数据：named volume 或 `/data/services/<service>/`；无状态服务不需要卷和 .env
- compose.yml 头部写注释：说明遵循的约定 + 端口映射理由

## compose.yml 必备要素

```yaml
services:
  <service>:
    image: ...
    restart: unless-stopped
    ports:
      - "127.0.0.1:<port>:<container_port>"      # 双绑：本机
      - "<组网IP>:<port>:<container_port>"        # 双绑：组网层
    env_file: .env            # 有密钥时
    healthcheck: ...          # 必须有，healthy 是部署成功的验收标准
```

## 部署工作流

1. 调研项目：README → compose/Dockerfile，关键问题：有无现成镜像？后端还是纯静态？持久化？默认端口？
2. 预检：`docker manifest inspect <image>`（架构？）→ `ss -tln`（端口空闲？）→ `docker pull <image>`（registry 可达？）
3. 建目录（sudo 交用户）→ 写 compose.yml → `docker compose up -d`
4. 验证：`docker ps --filter name=<service>` 显示 `(healthy)` 即为成功。⚠️ 不要用外部 curl 探活作为验证手段

## 升级流程

```bash
cd /srv/compose/<service>
docker compose pull && docker compose up -d
docker image prune -f
```

## 下线/清空流程

1. `docker compose down --remove-orphans`
2. 清源码 clone；`docker rmi <image>` 回收空间
3. compose 目录/数据目录属 root → 交用户 `sudo rm -rf`
4. 验证：`docker ps -a --filter name=<service>` 为空、网络消失
5. **归档**：更新 inventory/services.md + 记 changelog（走 ops-record）
```

## 4.9 specs/maintenance.md

```markdown
# 系统维护规范

## 监控

- 后台定时巡检（watchdog 之类）：基线对比容器/端口/资源，异常投递（Telegram 等）。**修改监控范围后要重新 rebaseline**，否则新增/删除服务会误报。
- 告警阈值参考在此列出（CPU/内存/磁盘/温度/Swap 等），按本机 16 线程/32G 实际定。

## 备份

| 对象 | 方式 | 位置 |
| --- | --- | --- |
| （列表：关键服务数据/数据库/dotfiles/散落备份） | 手动/定时 | 统一落 `/data/backups/<服务或批次>/` |

原则：重要数据先备份再动手；删数据目录前必须确认有备份；临时备份不进 home。

## 更新

- 系统包：`sudo pacman -Syu`（交用户跑）；关注 `.pacnew` 合并
- 容器镜像：逐服务 `docker compose pull && up -d`；升级前看上游 changelog 有无破坏性变更；升级后确认 healthy 并记 changelog
- 配置不确定时查官方文档，不要猜环境变量格式

## 文档对账审计（防活文档漂移）

- **触发**：重大变更群后（组网迁移、大清理、批量上下线）或至少每月一次，由 AI 巡检
- **做法**：以 `inventory/network.md`（组网 IP/网段）与 `inventory/services.md`（服务绑定）为权威事实，grep 全仓活文档（specs/',skills/,runbooks/,README,AGENTS.md）里的「已下线技术名/旧 IP/旧网段」残留
- **判读**：历史类文件（pitfalls/changelog/reports、inventory 中带 ⛔/已下线标记处）的旧引用属正当存档，不动；活文档里还在「教 AI 怎么做」的旧引用必须收敛
- **产出**：漂移点直接修 + changelog 记 `maint`；批量漂移 → `reports/` 追加审计报告

## 日志位置速查

| 来源 | 命令 |
| --- | --- |
| systemd 服务 | `journalctl -u <unit> -n 100 --no-pager` |
| 容器 | `docker logs --tail 100 <name>` |
| 系统 | `journalctl -p err -b` |
```

## 4.10 specs/decisions.md

```markdown
# 决策账本

> 关键决策以 `## D-NNN 标题` 追加（背景/决定/后果三行），append-only 不修改历史。

## D-001 建立决策账本

- **背景**：需要把不可逆的关键决定（架构取舍、命名约定、回退方案）沉淀下来，避免日后「当初为什么这么选」失考。
- **决定**：本文件作为 append-only 决策账本，重大方向变化同步在 changelog 记一条 `maint`。
- **后果**：后续决策均可回溯，来源可考。
```

## 4.11 templates/pitfall.md

```markdown
---
slug: <YYYY-MM-DD-slug>
tags: []
---

# <一句话标题>

> 日期：<YYYY-MM-DD> · 状态：未解决/已解决 · 标签：`<tag1>` `<tag2>`

## 现象

（报错原文 / 症状，尽量贴原始输出）

## 原因

（根因分析，几句话讲清机制）

## 解法

（可复制的命令 / 步骤）

## 预防

（下次怎么避免 / 关联文档 / 关联决策 D-xxx）
```

## 4.12 logs/changelog.md

```markdown
# 变更日志

> 只追加，不修改历史。格式：`- YYYY-MM-DD [类别] 一句话结果 (@操作者)`，见 specs/conventions.md。

- <YYYY-MM-DD> [maint] 创建运维知识库脚手架：specs/pitfalls/logs/inventory/templates + AGENTS.md + ops-context 扩展 + 2 个 skills (@操作者)
```

## 4.13 pitfalls/_index.md

```markdown
# 坑索引

> 排障第一步先查这张表。按标签检索，状态「未解决」的问题会在每次会话启动时被注入提醒。

| 日期 | 标题 | 标签 | 状态 | 文档 |
| --- | --- | --- | --- | --- |
```

## 4.14 inventory 三份（模板 + 实测填）

`machine.md`：

```markdown
# 机器档案

> 快照：YYYY-MM-DD（实测）。硬件/系统/关键路径，以 `hostname`/`lscpu`/`free -h`/`df -h` 为准。

## 硬件与系统

| 项 | 值 |
| --- | --- |
| 主机名 | <主机名> |
| 系统 | <发行版> |
| CPU / 内存 / 磁盘 | <实测值> |

## 关键路径 / 权限 / 组网

（sudo 免密、代理、组网、防火墙等，逐项实测记录）
```

`services.md`（头 + 两张表）：

```markdown
# 服务清单

> 快照：YYYY-MM-DD（docker ps / ss -tlnp 实测）。端口绑定约定见 specs/deployment.md。

## Docker 服务

| 服务 | 镜像 | 宿主端口 | compose 目录 | 说明 |
| --- | --- | --- | --- | --- |

## 非 Docker 服务

| 服务 | 端口 | 管理方式 | 说明 |
| --- | --- | --- | --- |

> 已停用 / 按需的服务单独列表，标下线日期与终档位置。
```

`network.md`：

```markdown
# 网络拓扑

> 快照：YYYY-MM-DD。网络类改动风险高，动手前向用户确认。

## 网络视图

| 层 | 地址 | 用途 |
| --- | --- | --- |
| 局域网 | <网卡> `<LAN_IP>/24` | 家庭内网 |
| 组网虚拟网 | `<组网IP>`（headscale 自托管） | 跨网组网 + 服务暴露 |
| Docker 网桥 | `172.x` 地址池 | 每 compose 项目一个网桥 |

## 节点清单 / 出站代理链 / 服务暴露约定

（实测填写）
```

`hosts.md`：

```markdown
# 主机台账

> 全部设备/服务器 + SSH 别名 + 密钥映射，在线状态以组网命令实测为准。

| 节点 | 系统 | 地址 | 在线 | 备注 |
| --- | --- | --- | --- | --- |
```

# 五、最后强制自检（逐项确认，缺一补上）

搭建完成后，逐项自查并向用户汇报：

1. `AGENTS.md`、`README.md`、`.gitignore`、4 份 specs、2 份 .gitkeep 目录骨架 —— 已建？
2. `.pi/extensions/ops-context.ts` —— 已按模板原样写入？
3. `.pi/skills/ops-record/SKILL.md` 与 `ops-troubleshoot/SKILL.md` —— 已按模板写入，frontmatter 的 `name`/`description` 正确？
4. `templates/pitfall.md`、`logs/changelog.md`、`pitfalls/_index.md` —— 已建？
5. `inventory/` 四份 —— 已用本机实测数据填，非空模板？
6. 每个在跑的重要服务 —— 已有对应 `runbooks/<服务>.md`？
7. `git init` + 首次 commit —— 已做？

汇报格式：列出创建的文件清单 + 实测采集到的关键事实（主机名/端口数/容器数/组网网段），并提醒用户「首次在本目录启动 Pi 时信任该项目」「以后 Pi 会话会自动加载 AGENTS.md + 注入 ops-context 简报，可用 /ops-brief 查看」。