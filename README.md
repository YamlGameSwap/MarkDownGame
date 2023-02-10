# Markdown 文字冒险

这是一个**文字冒险游戏**引擎，可以通过输入**动作**或**指令**来推进剧情、获取帮助。

游戏剧本完全使用`yaml`格式编写，你也可以定制自己的游戏剧情，尽情`fork`，创作更好玩的文字冒险游戏。

## 使用方法

本包不包含剧本文件，示例剧本请从这里下载：<https://github.com/YicongCao/MarkdownGame/tree/master/scripts>

```bash
npm install -g mdgame
mdgame -s <剧本文件路径>
# 下载剧本后即可游玩
mdgame -s scripts/harrypotter.yaml  # 玩哈利波特与魔法石(第一章)
mdgame -s scripts/holmes.yaml       # 玩福尔摩斯(斑点带子案)
```

## 代码结构

```bash
├── game_offline.js   # 离线游玩入口
├── package.json      # 工程文件
├── play.js           # 引擎
├── save_offline.js   # 离线游玩存档
├── script_loader.js  # 剧本加载器
└── scripts           # 剧本存档目录
    ├── 996adv.yaml
    ├── harrypotter.yaml
    └── holmes.yaml
```

## 游戏机制

游戏机制基于舞台、章节、选择、行为和动态条件。

舞台是由章节组成的，每节都有剧情描述和几种选择，引擎会把用户输入的指令进行关键词匹配，来命中一种选择。这种选择会有对应的行为，行为包括：

- 章节跳转
- 变量增减
- 流程控制

除了显示声明章节跳转，还可以编写动态条件。动态条件的表达式通常跟变量有关，比如 `fubao > 5`，每当触发变量增减的行为后，引擎就会检查动态条件，如果`fubao(福报)`值满足条件，就会触发该动态条件所对应的行为，该行为依然可以是章节跳转、变量增减、流程控制等。

文字冒险游戏的精髓，就在于让用户对千差万别的输入文字给出的不同效果感到惊奇，通过作出开放式的选择，来推进剧情。就像 Sheldon 所说：

> 文字冒险游戏使用了世界上最强大的 GPU：想象力。

## 剧本格式

游戏剧本是 `script.yaml` ，其 loader 是 `script_loader.js`，每个回合的处理引擎是 `play.js`，最终呈现出来的交互入口是 `game_offline.js`，很明显，与之对应的还应该有一个 `game_online.js`（提供 bot 形式的游玩界面），但目前还没提交上来。

目前你能在剧本中配置如下内容：

- 舞台：该字段名为`stages`，由一系列**章节**`chapters`构成
- 章节：由**故事** `story`、**选项**`choices`组成，每个选项都包含**关键词**`keywords`、**描述**`description`、**行为**`action`、**参数**`param`
- 变量：该字段名为 `variables`，是一个列表，可以在此处声明变量，然后：
  - 根据玩家的输入，编辑剧情内的分支选项`choice`，设置行为`action`，来改变变量的值
  - 根据变量的值，编辑动态条件`dynamics`，设置条件`conditions`，来触发一定的行为`action`
- 动态条件：该字段名为 `dynamics`，提供与变量相关的动态功能
- 默认区：该字段名为 `defaults`，当玩家输入没有匹配任何章节内的分支选项时，流程会走到默认区，默认区一般提供如下功能：
  - 帮助信息：帮助信息没必要在每个章节中配置成选项，可以放到默认区来配置
  - 特殊指令：全局流程性的指令，可以放到这里，比如开始游戏、重置进度等
  - 默认回复：当玩家随意输入内容时，也需要有一个友好回复，就在此处配置

### 剧情示例

以哈利波特与魔法石为例，剧情配置如下，变量区声明了6个变量：

- rounds，表示回合数，这是个特殊变量，即便不做配置，引擎也会每个回合自动给这个变量+1
- health，健康值，剧情选项会操作该变量，当健康值归零时，可以触发动态条件（Game Over）
- qsnake，这个变量用来记录某个情节中，哈利与巨蟒对话的次数，当此数值达到 4 时，可以触发动态条件（章节跳转）

在分支选项中，由如下字段需要解释：

- keywords，该选项要匹配的关键字
- description，如果该选项触发了分支跳转，则显示下一章节的剧情；如果没触发，则显示这个字段配置的内容
- action，动作类型：
  - none，什么也不做，直接给用户显示 description 字段的内容
  - goto，章节跳转，跳转后给用户显示新章节的剧情，param 填章节序号
  - incr，变量自增，param 填变量名
  - decr，变量自减，param 填变量名
  - calc，复杂变量运算，param 填变量修改后的表达式（如`(qsnake>=4)?0:qsnake`）
  - reset，重置剧情到 “1.1”，一般不会使用该动作

```yaml
title: 哈利波特与魔法石
msgtype: markdown
# 变量: 可以记录某些变量的值、动态触发某些章节
variables: [rounds, credit, study, love, health, qsnake]
# 舞台: 由不定数量的章节组成游戏本体
stages:
  "1.1":
    # 章节名称，会显示在消息标题的位置
    chapter: 序章 大难不死的男孩 
    # 本节剧情
    story: |-
      > @sender，欢迎回来魔法世界。自从德思礼夫妇一觉醒来在大门口台阶上发现他们的外甥，已经快十年过去了，女贞路却几乎没有变化。湛蓝的天空上悬着几片云朵，太阳依旧升到整洁的花园上，☀️阳光洒满他们的起居室，只有壁炉台上的照片显示出流失了多少时光。照片上的大头娃娃骑着一辆🚴自行车、乘坐🎠旋转木马、和母亲拥吻，他们的儿子达力显然已经不再是个小婴儿了。这栋房子里，🙈没有任何迹象表明这儿还住着另一个男孩。
      
      请@bot，输入“继续”阅读下一节剧情: 
      - **继续**
    # 输入选项: 关键词匹配，用户输入若含有该词，视为选择了此选项
    choices:
      # 关键词
      - keywords: [继续, 下一步, continue, next]
      # 描述: 若选项行为没有触发章节跳转，则显示描述消息
        description: ""
      # 行为: goto、none、incr、decr、calc，分别是章节跳转、无、变量自增、变量自减、变量运算
        action: [goto, calc]
        param: ["1.2", "health+7"]
```

### 动态条件示例

以**健康值减到0时，自动GameOver章节: 12.1**为例，需要配置动态条件如下：

```yaml
# 动态条件: 输入选项结算后，会检查动态条件，若条件成立，则执行操作
dynamics:
  - conditions:
      chapter: "1.*"
      expression: health <= 0
    action: goto
    param: "12.1"
```

### 默认回复示例

以帮助文档、重置章节、和 fallback 回复为例，需要配置默认区如下：

```yaml
# 默认区: 当用户输入没有命中章节内所覆盖的选项时，走到这里
defaults:
  # 条件: 满足章节条件或关键词条件，则触发该默认行为
  - conditions:
      chapter: "*"
      keywords: [help, man, 帮助, 怎么玩, 你是谁]
    action: none
    description: |-
      > @sender，欢迎来到`@title`，这是一个`文字冒险游戏`，你通过输入`动作`或`指令`来推进剧情、获取帮助。
      > 几乎每个90后，都曾梦想过作为一名🔯魔法师，进入霍格沃茨的校园；都曾经想象自己置身哈利波特的剧情中，或是作出🎲改变魔法世界的决策、或是体会⛳魁地奇的欢乐、或是去充满危险的🐸禁林里冒险。
      > 现在，让我们以bot的形式，梦回童年的魔法世界，自己就是哈利，试试看你的决策，会书写出怎样的故事。
      > 每个人都有`独立的进度和存档`，建议拉到小群中调戏和游玩呢😄
      
      游戏基本操作如下: 
      - **继续游戏**: 直接@我即可
      - **开始游戏**: start
      - **帮助**: help、man
      - **提示**: hint
      - **重置**: reset
  - conditions:
      chapter: "*"
      keywords: [hint, 提示]
    action: none
    description: "@sender，本小节没有提示😂"
  - conditions:
      chapter: "*"
      keywords: [reset, start, 重置, 回到开始, 重新开始, 开始游戏]
    action: reset
    description: "重置章节"
  - conditions:
      chapter: "*"
      keywords: []
    action: none
    description: "> 先生实在抱歉，可是你说话好像一个麻瓜🌚🌝"
```

