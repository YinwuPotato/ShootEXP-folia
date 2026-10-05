# ShootEXP — 经验射击

**最新版本：v1.3.3** | [下载 Release](https://github.com/YinwuPotato/ShootEXP-folia/releases/tag/v1.3.3)

Player interaction based experience shooting system.

玩家通过蹲起交互攻击目标，累积施法次数后射出经验物品；其他玩家拾取后右键食用获得经验值。

> ⚡ Folia 兼容调度。

---

## Features | 功能

| 功能 | 说明 |
|------|------|
| 🎯 **经验射击** | 蹲下瞄准附近的玩家/生物持续攻击，累积施法次数后射出「粘稠的经验」物品 |
| 🍖 **右键食用** | 拾取后右键食用经验物品获得经验值，支持原版 / SkillAPI 经验类型 |
| 🔗 **耦合系统** | 攻击者与目标配对（Couple），自动超时清理、玩家退出时停止任务 |
| 🎚️ **开关控制** | 玩家可独立切换消息接收、被攻击权限（命令 / GUI） |
| 🖥️ **设置 GUI** | `/shootexp gui` 图形化查看状态、重载配置、切换开关 |
| ⚙️ **可配置** | 攻击次数 / 射出量支持 exp4j 公式，声音、恢复、实体类型等完全自定义 |
| 🧵 **Folia** | 完全兼容区域线程调度 |

---

## Commands | 命令

| 命令 | 说明 |
|------|------|
| `/shootexp` | 显示帮助信息 |
| `/shootexp help` | 显示帮助信息 |
| `/shootexp status [玩家]` | 查看玩家状态（射出次数 / 经验存量） |
| `/shootexp item <所有者> <赠予者> <数量>` | 获取经验物品 |
| `/shootexp restore <all\|times\|stock> <玩家> [数量]` | 恢复玩家状态 |
| `/shootexp set <玩家> <射出次数> <经验存量>` | 设置玩家状态 |
| `/shootexp reload` | 重载配置 |
| `/shootexp toggle <messages\|attack>` | 切换消息接收 / 被攻击权限 |
| `/shootexp gui` | 打开设置 GUI |

权限节点（完整名）：

| 权限 | 默认 | 覆盖命令 |
|---|---|---|
| `shootexp.status` | 所有人 | `/shootexp status` |
| `shootexp.toggle` | 所有人 | `/shootexp toggle` |
| `shootexp.gui` | 所有人 | `/shootexp gui` |
| `shootexp.item` | OP | `/shootexp item` |
| `shootexp.restore` | OP | `/shootexp restore` |
| `shootexp.set` | OP | `/shootexp set` |
| `shootexp.reload` | OP | `/shootexp reload` |

> ⚠️ `shootexp.toggle` 与 `shootexp.gui` 目前只在 `plugin.yml` 里声明，代码中并未做
> `hasPermission` 检查 —— 即所有玩家都能使用这两个命令。若需要严格限制，需在代码里补检查。

---

## Configuration | 配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `lang` | `zh_CN` | 语言文件（zh_CN / en_US） |
| `private-message` | `false` | 私密消息，只发送给当事人而非全服广播 |
| `max-stock` | `1000` | 最大经验存量 |
| `required-attack-times` | `1.618^SHOOT + 10` | 施法所需的攻击次数（exp4j 公式，可用变量 SHOOT / STOCK / MAXSTOCK） |
| `shoot-amount` | `STOCK / 2` | 每次施法射出的经验量（exp4j 公式） |
| `entity-type` | `Player`, `Creature` | 可施法目标的实体类型 |
| `exp-type` | `VANILLA` | 经验类型：`VANILLA` / `SKILLAPI`（MMOCORE 预留） |
| `attack.distance` | `2.0` | 有效攻击距离（方块） |
| `attack.timeout` | `100` | 攻击超时（tick） |
| `restore.shoot.period` | `6000` | 施法次数恢复间隔（tick） |
| `restore.shoot.amount` | `1` | 每次恢复的施法次数 |
| `restore.stock.period` | `6000` | 经验存量恢复间隔（tick） |
| `restore.stock.amount` | `200` | 每次恢复的经验存量 |
| `custom-model-data.enable` | `false` | 是否启用经验物品自定义模型数据 |
| `custom-model-data.value` | `0` | 自定义模型数据值 |
| `sound.attack` / `shoot` / `shoot-no-exp` / `eat` | ... | 攻击、射出、无经验射出、食用时的音效 |

---

## Build | 构建

```bash
git clone https://github.com/YinwuPotato/ShootEXP-folia.git
cd ShootEXP-folia
mvn clean package
```

产出：`target/ShootEXP-1.3.3.jar`

> 本插件**不依赖** `YinwuPluginLib`（它 `extends JavaPlugin`，是独立的 Folia 分支）。
> 父 POM（`net.yinwu:YinwuPlugins:1.0.1`）已随仓库提供在 `parent/pom.xml`，无需额外操作。

---

## Dependencies | 依赖

- **Java 21**
- **Paper API 1.21+**（provided）
- **Folia**（兼容区域线程调度）
- **exp4j 0.4.8**（compile 范围，已被 shade 打进 jar）—— 用于解析 `required-attack-times` / `shoot-amount` 的数学公式
- ⚠️ **Brewery**（**必须安装**）—— 用于判断右键的是不是酒桶/酿药锅
- ⚠️ **SkillAPI**（**必须安装**）—— `exp-type: SKILLAPI` 时用于发放 SkillAPI 经验

### ⚠️ 为什么 Brewery / SkillAPI 不能缺席

`plugin.yml` 里它们声明为 `softdepend`，但**实际不能缺席**：
`EatListener` 在**方法体内**直接引用了 `com.dre.brewery.api.BreweryApi` 与 `com.sucy.skill.SkillAPI`
（`EatListener.java:42` 与 `:54`），两者都是 `provided` 范围、不会打进 jar。
JVM 校验方法字节码时会解析这些类型引用，因此**缺任一插件都会让右键食用抛出 `NoClassDefFoundError`**，
经验物品无法食用 —— 即使 `exp-type` 没设成 `SKILLAPI`、即使从不靠近酒桶。

> 如果你希望它们真正可选，需要把这两处调用拆到独立类里并在入口用反射调用（当前代码未这样做）。

## License | 许可证

MIT —— 见 [LICENSE](LICENSE)。原始插件作者为 Fengshuai(R_Josef)，本仓库是 Folia 兼容分支（维护：Qumingjam）。

---

## Links | 链接

- 仓库：[github.com/YinwuPotato/ShootEXP-folia](https://github.com/YinwuPotato/ShootEXP-folia)
- 插件作者：Fengshuai(R_Josef)
- 仓库维护：Qumingjam
