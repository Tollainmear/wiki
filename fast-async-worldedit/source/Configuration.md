FAWE 开箱即用，非常高效，但是在限制和队列操作方面的配置设置对于对你的 FAWE 插件进行微调也很有用。请参考下列的内容以及默认配置文件中的注释行。  

如果你将配置更改后发现编辑操作有问题的话，请在进行其他微调操作之前将那一项改成默认设置。

要想重置 config.yml 文件的话，请删除它然后重启服务器或使用 **`/fawe`** 命令重新载入 FAWE 插件。

## 异步操作

WorldEdit 自己会在放置方块之前尝试将每个编辑操作完全储存在内存之中。

WorldEdit 也会在内存中储存撤销（历史）方块，在大型编辑时这会增加内存的需求，甚至会导致服务器内存溢出崩溃，或导致编辑失败。

FAWE 提供了大量的速度改善，它将每个编辑操作都分割成了异步的线程。

并且只要可以放置方块之时，方块就会开始放置，而不是等待操作完成。

这让全局编辑的整体速度更快。

同时，因为方块在准备完毕之时就放置完成，它们不会占用内存空间。

FAWE 也能够配置将撤销（历史）方块储存在硬盘之上，这可以与其他性能的改善搭配使用，最终使 FAWE 近乎可以在最小的影响的情况下完成无限大小的编辑。

## 限制设置

这些设置能够让你控制一些用户组一次编辑可以放置的 WorldEdit 方块的数目。“default”（默认）小节就是为非管理组用户和没有权限跳过限制的用户所准备的。

查看 **fawe.bypass** 和 **fawe.admin** 权限来获取更多信息。 

你也能通过使用权限节点来定义某些特定用户组的限制，请查看下方的默认配置文件中的注释行。  

## 队列设置

这小节中的设置能够让你微调异步队列操作，影响内存的使用情况与编辑速度。   

### 本地队列

FAWE 会将每个编辑命令放在分割的异步线程之中，而不是服务器主线程，队列中的方块由每个玩家的本地队列来处理。

在队列中，方块按照区域来排序，这能够让 FAWE 使用更高效的优化算法，最终将方块放置（分配）进世界之中。

举个例子，WorldEdit会使用插件本体来执行光照和物理效果，然后发包，每个方块都会改变。

但是另一方面，FAWE将它队列中的方块按照区块分开检测，使用更高效的优化算法，将方块放置进世界中。 

在编辑开始时，本地队列进入未完成阶段。

当需要处理的队列方块为简单的 //set 或者是 //copy 这种类型的命令时，方块已经直接准备好放置了。

当使用其他命令（例如 //smooth 和 //deform）来编辑方块时，变化的方块会根据世界上的其他方块的位置决定，并且相关联的方块可能要改变好几次。（代表性地，所有本地更改都会在64个值得方块变化的区块之中完成） 

当编辑完全完成之后，本地队列变化成完成状态。

### 全局队列

FAWE 使用全局队列来真正将所有激活的本地队列的编辑方块放置。

全局队列的处理顺序是按照编辑的优先级的，所以提交时间较长的编辑会首先放置其中的方块。全局队列中的方块放置在世界中是在服务器的主线程上工作的。

FAWE 总是会放置在本地队列中完成的方块。同时，在放置未完成的本地队列中的方块之前，完成的本地队列的列表必须是空的。

全局队列和本地队列连接起来以后编辑正在开发中。（在代码内部这是作为两个队列而执行的，——一个是完成的本地队列，另一个是未完成的）

### 效率改善

FAWE 也尝试通过“在没有完成的本地队列时，放置未完成的本地队列中的方块”的方式来改善插件的效率。

* **target–size**  

FAWE 使用 **target–size** 来控制放置未完成队列中的方块数目，同时确保拥有放置的方块稳定不会再次改变的可能。

一些例如 //smooth 和 //deform 的命令，如果它们在命令仍然在处理中分配方块的话结果可能不会按照你想象的那样平滑或变形（虽然这些通常不会修改周围64个区块内的东西）。

这些方块的改变会依照世界上其他方块的位置，并且一些方块可能要多次改变。

在这种情况下，在64个区块进入队列之后任何单个方块都会被不会被重新修改了。

但是对于简单的操作来说不会有这种问题，例如 //copy 和 //set，因为这些方块不会进行多次的改变。

如果没有完成的本地队列等候放置方块，且target–size已设置更多在处理中的编辑区块进入队列的话，全局队列会直接开始处理仍然在处理中的可放置的方块。

这项设置能够确保像 //smooth 和 //deform 这样的命令稳定，同时它也能确保全局队列保持放置可用的方块，而不是闲置。

默认的 **target–size** 的值是64.

如果不设置 **target–size** 的话，使用 /fastmode 运行巨大的编辑会导致未完成的队列的填充速度比 **max–wait–ms** 设置的清空速度更快。

减少 **target–size** 可能会削弱像 //smooth 和 //deform 这样的命令的表现情况，因为花费在重新改变方块上的时间没了，方块已经放置了。

同时，也因为这些命令会依赖于其他方块的位置，如果 **target–size** 设置不够大的话，来自未完成队列中的方块会首先放置，但会晚些再改变，方块放置的最终结果和先处理完成再改变的最终结果会有所不同。

这会导致命令的表现不正确。

所以请将 **target–size** 选项的值设定的足够大来避免这个问题。

增加 **target–size** 的值可能不是很有帮助，因为大多数相关联的变更都是在64个区块的处理时间内完成的。

同时如果将它设置的很大的话，就会减少 FAWE 能够阻止大型编辑使服务器内存溢出的能力。 

* **max–wait–ms**  

FAWE 对于放置已经完成处理的编辑的方块是非常高效的。

但是，等待完成既耗费时间，也占用内存，因此，本设置就是用来尝试将全局队列中的方块分配，而不是让它在编辑还没有完成，但有能放置的方块时闲置的。

全局队列会在没有完成的本地队列，且全局队列空闲时间比 **max–wait–ms** 所设置的时间长（单位为毫秒）时开始放置仍然在处理中的方块。

**max–wait–ms** 设置也能解决区块读取连接超时的问题。区块通常的读取速度都是小于一秒的，所以默认的 **max–wait–ms** 的值是 1000，用于避免编辑冻结或是区块读取错误。

如果区块没有及时加载的话请增加这个数值。

## 默认配置文件  

默认的配置文件展示在了下面。

请注意服务器生成的配置文件是不带注释的。

```YML
# 前六项是不能配置的
issues: "https://github.com/boy0001/FastAsyncWorldedit/issues"
wiki: "https://github.com/boy0001/FastAsyncWorldedit/wiki/"
date: "27 Aug 2017 14:00:00 GMT"
build: "https://ci.athion.net/job/FastAsyncWorldEdit/<build>"
commit: "https://github.com/boy0001/FastAsyncWorldedit/commit/<hash>"
platform: "<platform>"
# 可用选项：de
# 请创建pull request来贡献翻译： https://github.com/boy0001/FastAsyncWorldedit/new/master/core/src/main/resources
language: ''
# 允许插件更新
update: true
# 将使用的统计发送到 mcstats.org
metrics: true
# FAWE 在没有足够的可用内存时会跳过处理区块
prevent-crashes: false
# 设置为 true 来启用 WorldEdit 对每个区域的限制。（例如： PlotSquared 或 WorldGuard）
# 要想在区域中启用 WorldEdit 的话，用户需要许可：
# fawe.<plugin>  权限。查看权限页面来获取支持的区域插件。
region-restrictions: true
# FAWE 在内存占用超过这个百分比时会取消非管理员做出的编辑
#  - 使用 `/wea` 或 `//fast` 或 `fawe.bypass` 跳过检测
#  - 写入 100 或 -1 禁用本项。
max-memory-percent: 95

clipboard:
  # 替代储存到内存中，将剪切板储存到硬盘上
  #  - 速度些许变慢
  #  - 每方块占用 2 字节
  use-disk: true
  # 压缩剪切板减少文件大小：
  #  - TODO: 目前还没有实现在硬盘上的压缩随机访问
  #  - 0 = 不进行压缩
  #  - 1 = 快速压缩
  #  - 2-17 = 较慢速度压缩
  compression-level: 1
  # 在删除之前储存在硬盘上的时间，单位为天
  delete-after-days: 1

lighting:
  # 是否应该在重新计算光照完成后再发包
  delay-packet-sending: true
  async: true
  # 重新计算光照的模式：
  #  - 0 = 无（不重新计算光照）
  #  - 1 = 优化（重新计算光源和改变的方块的光照）
  #  - 2 = 全局（缓慢重新计算每个方块的光照）
  mode: 1

# 一般的游戏刻限制（对 WorldEdit 来说不必要，但是可以防止滥用）
tick-limiter:
  # 是否启用限制
  enabled: true
  # 间隔，单位游戏刻
  interval: 20
  # 在每个间隔间最大的掉落方块数目（每区块）
  falling: 64
  # 每个间隔中最大执行的物理效果数目（每区块）
  physics: 8192
  # 每间隔最大生成的物品数目（每区块）
  items: 256

web:
  # 剪切板的网页接口
  #  - 所有的剪切板都匿名私有储存
  #  - 下载可以被用户删除
  #  - 支持剪切板的上传，下载和保存
  url: "http://empcraft.com/fawe/"
  # 资源的网页接口
  #  - 所有剪切板都是组织好后公开的
  #  - 资源能够被搜索，选择和下载
  assets: "http://empcraft.com/assetpack/"

extent:
  # 当这些插件拖慢了 WorldEdit 的操作速度时不要提示控制台
  #  - 如果你需要改变本选项时你会在控制台收到提示信息
  allowed-plugins: []
  # 当第三方延展插件安装后应该显示调试信息吗？
  debug: true

# 实验中的选项，你可以冒险试试
experimental:
  # [不安全] 直接修改区域文件（已过期——请使用Anvil命令）
  #  - 使用不当会导致世界错误！
  anvil-queue-mode: false
  # [安全] 动态增加渲染的区块的数目
  #  - 需要 Paper 服务端： ci.destroystokyo.com/job/PaperSpigot/
  #  - 将你的服务器视距上设置为 1 (spigot.yml, server.properties)
  #  - 取决于服务器的 tps 和玩家的移动
  #  - 请给我们提供反馈信息
  dynamic-chunk-rendering: false

# 这跟 FAWE 怎么放置区块相关联
queue:
  # 这个数值应该和你拥有的处理器的核数相同
  #  - 将此项设置为1，如果你需要可信的 `/timings` 的话
  parallel-threads: 8
  progress:
    # 显示一位用户的编辑的进度的恒定title
    #  - false = 禁用
    #  - title = 显示进度title
    #  - chat = 在聊天栏中显示进度
    display: "false"
    # 编辑进度多久显示一次
    interval: 1
    # 发送进度的延迟，单位毫秒（快些的话编辑不会刷屏）
    delay: 5000
  # 在编辑比这数目还多的区块时：
  #  - FAWE 会在所有运算完成之前直接开始放置方块
  #  - 大些的值会些微减少CPU的运算时间
  #  - 小写的值可以减少内存使用
  #  - 太小的值会导致一些操作坏掉（例如 deform）
  target-size: 64
  # 强制 FAWE 开始放置区块，不管一项编辑是否完成处理
  #  - 大些的值会些微减少CPU的运算时间
  #  - 小写的值可以减少内存使用
  #  - 太小的值会导致一些操作坏掉（例如 deform）
  max-wait-ms: 1000
  # 增加或减少队列强度（毫秒） [-50,50]:
  #     0 = 平衡性能与稳定性
  #     -10 = 为区块放置少分配 10ms
  # 值太大的话会造成卡顿（你可能感觉没关系）
  # 值太小的话编辑进度会很慢
  extra-time-ms: 0
  # 在提速操作之前读取多少数量的区块
  #  - 值太小的话会导致FAWE等待主线程的请求
  #  - 值太大的话会占用更多内存也不会感觉变快
  preload-chunks: 32
  # 抛弃已经闲置了一段时间（ms）的编辑
  #  - 例如：一款插件创建了 EditSession 但没有使用它做任何事
  #  - 这只会对不正确使用 WorldEdit 的旧版 API 的插件起作用
  discard-after-ms: 60000

history:
  # 历史记录是否应储存在硬盘上：
  #  - 空闲大量的内存
  #  - 服务器重启可以存留
  #  - 可以无限撤销
  #  - 不会影响编辑的性能，依照 `combine-stages`
  use-disk: true
  # 使用数据库来储存硬盘储存的概要：
  #  - 启用检测和回档
  #  - 不会影响性能
  use-database: true
  # 在分配时记录到历史中：
  #  - 速度更快，因为它避免了重复的方块检查
  #  - 压缩可能会糟糕一些，因为分配顺序不同
  combine-stages: true
  # 大些的压缩等级减少历史记录的大小，但以占用 CPU 使用率为代价
  # 0 = 不压缩字节数组（最快）
  # 1 = 1 级快速压缩（默认）
  # 2 = 2 x 快速
  # 3 = 3 x 快速
  # 4 = 1 x 中速, 1 x 快速
  # 5 = 1 x 中速, 2 x 快速
  # 6 = 1 x 中速, 3 x 快速
  # 7 = 1 x 慢速, 1 x 中速, 1 x 快速
  # 8 = 1 x 慢速, 1 x 中速, 2 x 快速
  # 9 = 1 x 慢速, 1 x 中速, 3 x 快速 （最佳压缩文件）
  # 注意：如果你使用硬盘的话，请最好使用压缩，因为较小的文件可以储存的更快
  compression-level: 3
  # 压缩的缓冲大小：
  #  - 较大 = 更好的比率，但是占用更多内存
  #  - 必须在以下范围中 [64, 33554432]
  buffer-size: 531441
  # 编辑时最大等待一个区块加载的时间，单位是毫秒。
  #  (50ms = 1 游戏刻, 0 = 最快).
  #  默认值 100 应该对大多数情况来说都是安全的。
  # 
  # 需要读取区块的操作（例如：复制）在没有及时读取区块时
  # 会使用上个区块作为过滤器，这会出现大量复制出的方块。
  # 每个读取区块的操作在读取区块时通常都需要25-50ms，在服务器卡顿时甚至更多。
  # 所以100ms的等待时间如果区块在10ms内加载完毕了的话也不需要等待100ms。
  # 
  # 本值也可以作为万一区块无法读取时（不管是什么原因）操作超时的值
  # 如果操作超时了的话，操作就会使用上个区块作为过滤器，
  # 然后显示出一条错误信息，在这种情况下，你需要要么将选区调整的更小，
  # 要么将本项的值设置的大一些。
  # 输入 0 这个值的速度会很快因为它即不会阻止读取区块也不会等待。
  chunk-wait-ms: 1000
  # 在几天之后删除硬盘上的历史记录
  delete-after-days: 7
  # 在玩家登出时是否删除内存中的历史记录（不影响硬盘存储）
  delete-on-logout: true
  # 对于一些使用 WorldEdit 的插件来说，历史记录是否默认启用：
  #  - 禁用本项速度会更快
  #  - 使用 FAWE API 的插件不会受影响
  enable-for-console: true
  # 是否储存反撤销的相关信息：
  #  - 历史记录文件要大大约 20%
  #  - 允许使用 /redo 命令
  store-redo: true
  # 仅记录所有比 4096x256x256 小的编辑：
  #  - 减少历史记录文件的大小大概10%
  small-edits: false

# 一些路径的文件夹名
paths:
  # 将任何Minecraft或Mod的Jar文件放在这里来使用方块的材质
  textures: "textures"
  heightmap: "heightmap"
  history: "history"
  # 群组服务器可以使用相同的剪切板
  clipboard: "clipboard"
  # 是否分离每个玩家schematic文件的路径
  per-player-schematics: true
# "default" 限制组会影响没有特殊限制权限的用户。
# 要想给某人不同的限制，请复制默认限制组
# 然后重新给他命个名（例如：newbie）。然后给予用户限制的
# 权限节点，使用该限制名（例如： fawe.limit.newbie  ）
limits:
  default:
    # 能够同时运行的操作（例如：命令）
    max-actions: 1
    # 每次最大改变的方块数量（例如：使用 `//set stone` ）。
    max-changes: 50000000
    # 每次最大检测的方块数量（例如：使用 `//count stone` 不会改变方块的命令）
    max-checks: 50000000
    # 一次更改失败的最大次数（例如：玩家没有访问该区域的权限）
    max-fails: 50000000
    # 最大允许的笔刷递归次数（例如： `//brush smooth` ）
    max-iterations: 1000
    # 一次最大能够允许的实体数目（例如：牛）
    max-entities: 1337
    # 包括 Banner, Beacon, BrewingStand, Chest, CommandBlock, 
    # CreatureSpawner, Dispenser, Dropper, EndGateway, Furnace, Hopper, Jukebox, 
    # NoteBlock, Sign, Skull, Structure 的最大方块状态
    max-blockstates: 1337
    # 玩家历史文件的最大尺寸，单位是MB：
    #  - 超过这个尺寸的历史文件，不管是在硬盘上还是内存中都会被删除
    max-history-mb: -1
    # //calc 能够执行的每次操作的最大时间，单位毫秒 
    max-expression-ms: 50
    # 动画化方块放置：
    #  - 为方块放置增加延迟（ms/方块）
    #  - 使用虚伪的延迟会导致使用更多CPU与内存
    speed-reduction: 0
    # 放置区块，而不是单个方块：
    #  - 禁用本项会大幅度降低性能
    #  - 只有在与动画化方块放置冲突时才禁用本项
    fast-placement: true
    # WorldEdit 应该使用玩家的物品栏放置物品吗？
    # 0 = 不使用物品栏（创造模式）
    # 1 = 移除和放置都使用物品栏（免费建筑区）
    # 2 = 仅放置使用物品栏（生存模式）
    inventory-mode: 0
    # 大型的编辑是否需要确认（需要区块数目大于16384）
    confirm-large: true
```