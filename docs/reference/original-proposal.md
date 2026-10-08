# 评估996传奇引擎架构

> 1. 你调研一下996传奇引擎， 参考：[https://engine-doc.996m2.com/web/#/34/3024](https://engine-doc.996m2.com/web/#/34/3024)
> 2. 你也可以自主思考搜索其他站点，
> 3. 他的技术架构是什么？&#x20;
> 4. 综合评估一下，这个传奇引擎的技术架构，开发难度。
> 5. 评估一下：是否可以复刻一个新的传奇引擎出来？

<details><summary>6 previous messages</summary>

> 我会先抓取你给的官方文档，并补充检索客户端、服务端、脚本系统、资源格式与发布链路等资料；我会把“有公开证据的事实”和“基于资料的架构推断”分开写，最后给出复刻路径、难度和风险评估。
>
> <details><summary>Searched the web</summary>
>
> Searched the web for `site:engine-doc.996m2.com 996传奇引擎 架构 客户端 服务端 脚本 Lua ...`
>
> </details>
>
> 官方文档是单页应用，普通搜索只能看到壳页面。我正在直接解析其文档接口与目录数据，这样可以核对具体模块和 API，而不是仅依赖二手教程。
>
> <details><summary>Ran 4 commands, searched the web</summary>
>
> - Ran `python3 - <<'PY'
> import requests
> u='https://engine-doc.996m2.com/web/'
> r=requests.get(u,timeout=20)
> print(r.status_code, r.url)
> print(r.text[:12000])
> PY`
>
> - Ran `python3 - <<'PY'
> import requests, json
> base='https://engine-doc.996m2.com/server/index.php?s='
> for path,params in [('/api/item/info',{'item_id':34}),('/api/page/info',{'page_id':3024}),('/api/item/show',{'item_id':34,'page_id':3024})]:
>  r=requests.get(base+path,params=params,timeout=20)
>  print('\n',path,r.status_code,r.url,'\n',r.text[:4000])
> PY`
>
> - Ran `python3 - <<'PY'
> import requests
> base='https://engine-doc.996m2.com/server/index.php?s=/api/item/info'
> for iid in [34,42,22,9,28,29,30]:
>  r=requests.get(base,params={'item_id':iid},timeout=30).json()
>  d=r.get('data',{})
>  print(f'\nITEM {iid}: {d.get("item_name")}')
>  menu=d.get('menu',{})
>  print('root pages:',[(p['page_id'],p['page_title']) for p in menu.get('pages',[])])
>  def walk(cs,pre=''):
>   for c in cs:
>    print(pre+'['+c['cat_name']+']')
>    for p in c.get('pages',[]): print(pre+'  '+p['page_id']+' '+p['page_title'])
>    walk(c.get('catalogs',[]),pre+'  ')
>  walk(menu.get('catalogs',[]))
> PY`
>
> - Ran `python3 - <<'PY'
> import requests, html, re
> base='https://engine-doc.996m2.com/server/index.php?s=/api/page/info'
> ids=[1159,1228,1229,4608,4894,3585,1155,1161,3663,7484,2053,8842,3015,2575,3005,2152,2155,2182]
> for pid in ids:
>  try:
>   d=requests.get(base,params={'page_id':pid},timeout=20).json()['data']
>   s=html.unescape(d['page_content'])
>   s=re.sub(r'<[^>]+>',' ',s)
>   s=re.sub(r'\n{3,}','\n\n',s)
>   print('\n###',pid,d['item_id'],d['page_title'],'###\n',s[:7000])
>  except Exception as e: print(pid,e)
> PY`
>
> Searched the web for `https://gitee.com/jiujiuyinqing/Open996_Client_GUI`
>
> Searched the web for `https://engine-doc.996m2.com/server/index.php?s=/api/page/info&page_id=1159`
>
> Searched the web for `site:996m2.com 996M2 官方 纯Lua 引擎 客户端 开源 服务端 数据库 ...`
>
> Searched the web for `site:bbs.996m2.com "DBServer" "M2Server" 996 ...`
>
> </details>
>
> 目前有一个关键澄清：所谓“纯 Lua 客户端开源”并不是完整客户端内核开源。官方论坛明确说，只开放了部分 Lua 界面代码；网络、渲染、资源加载等底层仍是闭源二进制。这会直接影响“能否复刻”的结论。接下来我会核对公开 GUI 仓库和运行包的技术指纹。
>
> <details><summary>Ran 10 commands, searched the web</summary>
>
> - Ran `mkdir -p work && git ls-remote https://gitee.com/jiujiuyinqing/Open996_Client_GUI.git | head -20`
>
> - Ran `python3 - <<'PY'
> import requests
> for u in [
>  'https://gitee.com/api/v5/repos/jiujiuyinqing/Open996_Client_GUI',
>  'https://gitee.com/jiujiuyinqing/Open996_Client_GUI/repository/archive/master.zip',
>  'https://gitee.com/jiujiuyinqing/Open996_Client_GUI/raw/master/README.md',
>  'https://gitee.com/jiujiuyinqing/Open996_Client_GUI/raw/master/GUILayout/GUIUtil.lua']:
>  try:
>   r=requests.get(u,timeout=30,allow_redirects=True)
>   print(r.status_code,len(r.content),r.url,r.headers.get('content-type'))
>   print(r.text[:500])
>  except Exception as e: print(e)
> PY`
>
> - Ran `python3 - <<'PY'
> import requests
> urls={
> 'client':'https://engine-doc.996m2.com/server/index.php?s=/api/attachment/visitFile&sign=f76a7180b08606be5150e3ea51aeeecb',
> 'gui':'https://engine-doc.996m2.com/server/index.php?s=/api/attachment/visitFile&sign=2b2fb263dae611cff5563388e1cfcd5e',
> 'server':'https://engine-doc.996m2.com/server/index.php?s=/api/attachment/visitFile&sign=235d69c5199f74399346e1a0eae68caa'}
> for k,u in urls.items():
>  r=requests.head(u,timeout=30,allow_redirects=True)
>  print(k,r.status_code,r.url,r.headers)
> PY`
>
> - Ran `python3 - <<'PY'
> import requests
> u='https://engine-doc.996m2.com/server/index.php?s=/api/attachment/visitFile&sign=2b2fb263dae611cff5563388e1cfcd5e'
> r=requests.get(u,timeout=30)
> print(r.status_code,len(r.content),r.headers.get('content-type'),r.url)
> print(r.content[:200])
> PY`
>
> - Ran `python3 - <<'PY'
> import requests
> for iid in [9,22,28,29]:
>  d=requests.get('https://engine-doc.996m2.com/server/index.php?s=/api/item/info',params={'item_id':iid},timeout=30).json().get('data',{})
>  print('\n',iid,d.get('item_name'))
>  menu=d.get('menu',{})
>  print('root',[(p['page_id'],p['page_title']) for p in menu.get('pages',[])])
>  def walk(cs,level=0):
>   for c in cs:
>    print('  '*level+'['+c['cat_name']+'] '+str(len(c.get('pages',[]))))
>    if level<1:
>     for p in c.get('pages',[])[:25]: print('  '*(level+1)+p['page_id']+' '+p['page_title'])
>    walk(c.get('catalogs',[]),level+1)
>  walk(menu.get('catalogs',[]))
> PY`
>
> - Ran `python3 - <<'PY'
> import requests,html,re
> for pid in [4608,4894,3585,1155,1161,3663,5035,4995,4994,209,211,135,154]:
>  d=requests.get('https://engine-doc.996m2.com/server/index.php?s=/api/page/info',params={'page_id':pid},timeout=20).json()['data']
>  s=html.unescape(d['page_content'])
>  s=re.sub(r'<br\s*/?>','\n',s,flags=re.I); s=re.sub(r'<[^>]+>',' ',s)
>  s=re.sub(r'[ \t]+',' ',s); s=re.sub(r'\n\s*\n+', '\n\n',s)
>  open(f'work/page_{pid}.txt','w').write(s)
>  print('\n###',pid,d['item_id'],d['page_title'],'chars',len(s),'###\n',s[:4000])
> PY`
>
> - Ran `curl -L --fail --max-time 60 -o work/dev.zip 'https://cdnjc.dhsf.huiyuehz.com/ShowDoc/c2/996M2%E6%9C%8D%E5%8A%A1%E7%AB%AFLua/%E6%95%99%E7%A8%8B%E4%B8%8E%E4%B8%8B%E8%BD%BD%E7%AF%87/6479bf5a7528f.zip' && unzip -l work/dev.zip | sed -n '1,120p'`
>
> - Ran `mkdir -p work/dev_sample2 && unzip -q -o work/dev.zip -d work/dev_sample2 && rg -n "cc\.|cocos|GUI:|SL:" work/dev_sample2`
>
> - Ran `python3 - <<'PY'
> import requests,html,re
> for pid in [836,1124,2359,2360,2364,7533,2058,552,337]:
>  d=requests.get('https://engine-doc.996m2.com/server/index.php?s=/api/page/info',params={'page_id':pid},timeout=20).json()['data']
>  s=html.unescape(d['page_content'])
>  print('\n###',pid,d['page_title'],'###')
>  for line in s.splitlines():
>   if any(x in line.lower() for x in ['cc.','cocos','tolua','widget','userdata','node','sprite','lua']): print(re.sub(r'<[^>]+>',' ',line)[:500])
> PY`
>
> Searched the web for `site:996m2.com 996引擎 软件使用协议 知识产权 ...`
>
> - Ran `python3 - <<'PY'
> import requests,html,re,json
> for pid in [2143,5720,5722,5723,3013,3020,2037,2050,2031,1148,317,154]:
>  d=requests.get('https://engine-doc.996m2.com/server/index.php?s=/api/page/info',params={'page_id':pid},timeout=20).json()['data']
>  s=html.unescape(d['page_content'])
>  s=re.sub(r'<br\s*/?>','\n',s,flags=re.I); s=re.sub(r'<[^>]+>',' ',s)
>  s=re.sub(r'[ \t]+',' ',s); s=re.sub(r'\n\s*\n+', '\n\n',s)
>  print('\n###',pid,d['item_id'],d['page_title'],'###\n',s[:6000])
> PY`
>
> Searched the web for `site:bbs.996m2.com 2026 996引擎 3.71.1 引擎版本 客户端版本 ...`
>
> Searched the web for `site:bbs.996m2.com 996 "游戏网关" "M2" ...`
>
> Searched the web for `site:bbs.996m2.com 996M2 Delphi FireDAC JclDebug ...`
>
> </details>
</details>

结论先说：**技术上可以做出一个功能等价的新传奇引擎；但如果目标是完整复刻 996 的三端客户端、M2 服务端、脚本兼容、工具链、跨服、热更新、反外挂和运营后台，属于大型、多年期项目，不是单纯“重写一个游戏服务器”。**

更合理的目标是：**重新实现一套“传奇类引擎”，兼容合法拥有的数据表、地图和部分脚本语义，而不是复制 996 的闭源代码、私有协议和授权体系。**

## 一、先说明调研范围

你给出的页面是 **1208 版说明书，对应 2022.12.08 引擎、客户端 3.3.1**，已经明显过时。[1208 版官方说明书](https://engine-doc.996m2.com/web/#/34/3024)

截至 2026 年 10 月，官方论坛已经出现客户端 **3.71.1** 和 2026 年引擎版本的讨论，因此下面分析的是整个 996 技术体系，而不是只分析 1208 版本。[2026 年官方 BUG 区](https://bbs.996m2.com/forum.php?fid=45&mobile=no&mod=forumdisplay)

996 也并非一个单一产品，目前至少包括多端版、纯 PC 版、传奇 3、传世、3D 等分支。[官方产品页](https://www.996m2.com/engineDownload?tab=5)

## 二、996 的技术架构

整体上，它是一个“**传统传奇 M2 架构现代化**”的产品：

```text
Excel/CSV、Lua、TXT、地图和美术资源
                 │
        996 集成工具/GUI 编辑器
                 │
       GM 后台、版本管理、CDN 热更新
                 │
 ┌───────────────┴────────────────┐
 │ Windows / Android / iOS / H5 客户端 │
 │ 原生渲染内核 + Lua UI/业务扩展       │
 └───────────────┬────────────────┘
                 │ 登录、游戏消息、表单消息
           登录服务 / 游戏网关
                 │
         M2Server 区服权威进程
     地图、AOI、战斗、AI、掉落、脚本
          │                    │
   数据库服务 → SQL Server     └→ 跨服 M2
```

### 1. 客户端

多端客户端是“**闭源原生内核 + Lua 界面与扩展层**”。

公开部分主要集中于：

- `dev/GUILayout`：前端 Lua 逻辑。
- `GUIExport`：可视化编辑器生成的 Lua UI。
- `SL:*`、`GUI:*`：引擎向 Lua 暴露的封装 API。
- 地图、角色、武器、怪物、特效、图标、音效等资源目录。
- `xls/csv → Lua` 的客户端配置表转换。

官方曾明确说明，所谓客户端开源只是**部分 Lua 界面文件开放，并不是完整客户端内核开源**。[官方论坛说明](https://bbs.996m2.com/thread-7965-1-1.html)

渲染内核可以较高置信度判断为 **Cocos2d-x 3.x 系技术**：

- UI 接口的 `Widget`、`ListView`、`Layout`、`Action` 与 Cocos UI 模型高度一致。
- 2026 年客户端日志直接输出 `cocos2d: TextureCache`。[客户端日志实例](https://bbs.996m2.com/forum.php?extra=page%3D1&mobile=no&mod=viewthread&tid=16256)

另外还存在面向经典 PC 操作体验的独立 **Delphi 版客户端**。[官方论坛介绍](https://bbs.996m2.com/forum.php?extra=page%3D1&mobile=no&mod=viewthread&tid=7715)

### 2. 服务端

核心是 Windows 原生的 `M2Server`，属于**单区服权威服务器**：

- 角色移动、攻击、技能、Buff、怪物 AI、掉落和地图状态以服务端为准。
- 客户端主要负责显示、输入和部分 UI 业务。
- M2 连接游戏网关、登录服务、数据库服务。
- 区服之间通过独立的跨服 M2 进程交换或迁移部分状态。

启动日志能看到“游戏网关”“数据库服务器”“人物数据引擎”等独立模块。[M2 启动日志](https://bbs.996m2.com/thread-11292-1-1.html)

从公开异常栈看，M2 使用了 FireDAC、ODBC、JCL/Pascal 单元，说明服务端具有明显的 **Delphi/C++Builder 技术血统**；数据库是 Microsoft SQL Server。[公开调用栈](https://bbs.996m2.com/thread-11592-1-1.html)

### 3. 脚本体系

996 同时存在几代脚本模式：

- 传统 `TXT + Lua`。
- 前后端 Lua 分离。
- 纯 Lua 版本，删除大量 TXT 解释路径。
- 新三端 Lua 热更新框架。

服务端 Lua 用于 NPC、任务、活动、战斗触发、Buff、排行、定时器等。2024 年之后的文档显示服务端开始使用 **LuaJIT**。[服务端 Lua 文档](https://engine-doc.996m2.com/web/#/9/154)

客户端与服务端主要通过两种扩展方式交互：

- 数字消息 ID：`SendLuaNetMsg/RegisterLuaNetMsg`。
- 表单调用：客户端提交“脚本路径 + 函数名 + 参数”，服务端处理后返回界面或数据。

[官方前后端交互示例](https://engine-doc.996m2.com/web/#/9/5035)

这个设计开发快，但需要严格校验参数。官方文档本身也反复提醒客户端参数可能被改包，不能相信客户端输入。

### 4. 数据层

它不是纯数据库驱动，而是三类数据并存：

- SQL Server：角色、装备实例、背包、仓库、技能、邮件、行会、变量、拍卖等持久化数据。
- Excel/CSV：怪物、装备、技能、Buff、地图、刷新点、属性、商城等静态配置。
- Lua/TXT/INI：玩法和运行规则。

[纯 Lua 数据库表结构](https://engine-doc.996m2.com/web/#/30/3663)列出了 `TBL_CHARACTER`、背包、装备、技能、邮件、行会、变量等关系表。

这种方式适合策划快速改数值，但也带来一个明显问题：**服务端表、客户端 Lua 表、数据库结构和客户端版本必须严格配套**。官方教程专门要求更新表格并重新转换前端 Lua 表。[单机架设教程](https://bbs.996m2.com/thread-9902-1-1.html)

### 5. 资源与更新体系

客户端支持：

- 首包资源。
- 登录前强制热更新。
- 微端边玩边下。
- 静默下载。
- 自定义资源优先覆盖。
- CDN 分发、资源压缩、Lua 混淆。

[资源热更说明](https://engine-doc.996m2.com/web/#/34/2152)
[客户端资源目录](https://engine-doc.996m2.com/web/#/34/2155)

这说明 996 不只是游戏运行时，还包含一套内容发布平台。

### 6. 扩展与承载方式

它不是典型云原生微服务架构，而是：

- 一个 M2 承载一个区服的主要世界状态。
- 通过网关隔离连接。
- 某些旧版支持把地图分配到不同线程。
- 跨服通过额外 M2 实例实现。
- 数据库线程与游戏逻辑线程相对分离。

旧版文档建议在 500 人以上或重负载地图使用地图线程，但这不是正式性能基准。[旧版多线程说明](https://engine-doc.996m2.com/web/#/34/3015)

需要注意：不同分支的线程模型并不统一。新版 Lua 文档甚至明确警告某些 Lua 模式不要开启多线程，否则会产生交互问题。因此不能把“多线程”当作所有 996 版本的统一能力。

## 三、综合评价

| 方面 | 评价 |
|---|---|
| 内容生产效率 | 8.5/10，表格、Lua、GUI 编辑、热重载非常适合快速开玩法 |
| 系统完整度 | 8.5/10，传奇常用系统、工具、后台、更新链路比较齐全 |
| 可扩展性 | 7/10，Lua 层较强，但底层能力受闭源 API 限制 |
| 架构现代化 | 5.5/10，本质仍是 Windows M2 单体区服架构 |
| 可维护性 | 5.5/10，版本、数据库、客户端和配置表耦合较重 |
| 横向扩展 | 6/10，适合多区、多 M2；不是天然弹性分布式世界 |
| 供应商依赖 | 较高，客户端内核、协议、工具服、授权、反外挂和后台均有依赖 |

它最大的优点不是底层架构多先进，而是：

> **把二十多年传奇生态中的脚本、资源、配置习惯和商业开服流程，包装成了一套可实际交付的生产系统。**

它最大的技术债是：

> **历史兼容优先于边界清晰，表格、脚本、客户端、数据库和引擎版本形成了复杂的隐式契约。**

所以“使用 996 开发版本”和“开发一个 996 级引擎”是两个完全不同的难度：

- 使用 996 做游戏：有传奇经验约 5/10，新团队约 7/10。
- 修改 Lua 玩法和 UI：中等。
- 改客户端底层、网络、渲染、同步：因为闭源，基本不可控。
- 从零开发同等级产品：9～9.5/10。

## 四、是否可以复刻

### 可以复刻的部分

从工程角度，以下都能重新实现：

- 2D/2.5D 地图和八方向角色渲染。
- 寻路、AOI、同步、战斗、技能、Buff、怪物 AI。
- 背包、装备、交易、行会、邮件、拍卖、排行榜。
- Lua 服务端脚本和前端 Lua UI。
- Excel/CSV 数据驱动。
- PC、Android、iOS 客户端。
- 热更新、资源分包、跨服、GM 后台。

### 最难复刻的不是基础战斗

真正昂贵的是：

1. 大量传奇历史规则和边界行为。
2. 数千个脚本 API 的语义兼容。
3. 旧地图、模型、装备和素材格式。
4. 移动、攻速、技能判定与客户端表现的手感。
5. 弱网重连、断线恢复、跨服状态一致性。
6. 反外挂、封包校验和异常行为检测。
7. GUI 编辑器、资源工具、数据库升级和发布后台。
8. 持续兼容旧版本内容的回归测试。

### 粗略工作量

假设团队具有 MMO 和原生客户端经验，不包含原创美术生产：

| 目标 | 人员与时间 |
|---|---|
| PC 单地图原型：移动、战斗、怪物、掉落 | 4～6 人，6～9 个月 |
| 可运营 PC 版：基础系统、脚本、工具、后台 | 6～10 人，12～18 个月 |
| PC/Android/iOS 商业版本 | 12～20 人，24～36 个月 |
| 接近 996 的兼容度与工具生态 | 20～30 人，3～5 年，约 800～1500 人月 |

如果要求现有 996/GOM/GEE 版本“一键迁移、行为完全一致”，成本还会继续上升。

## 五、最终建议

**不要以“1:1 克隆 996”为立项目标。**更可行的路线是：

- 做一套自主的权威服务器和跨平台客户端。
- 兼容合法拥有的地图、配置表和玩法数据。
- 为常用传奇脚本命令实现兼容层。
- 使用自己的网络协议、资源包、后台和安全体系。
- 先完成 PC 版垂直切片，再扩展移动端。
- 把兼容 API 分级：核心 20%、常用 60%、长尾插件 20%，不要一开始追求全量。

另外，技术可行不等于可以直接复制。996 的闭源代码、协议、客户端资源、加密格式、商标，以及传奇游戏本身的素材与版权授权都需要单独确认。996 的官方授权页也明确区分了引擎使用授权与实际运营责任。[官方授权说明](https://www.996m2.com/engineDownload?tab=5)

最终判断是：

> **做一个新的“传奇类引擎”完全可行；做一个能替换 996、兼容现有商业版本、达到相同生态成熟度的产品也可行，但需要把它当成 MMO 平台级研发，而不是普通游戏项目。**

> 1. 如果完全参考并对齐 996 的技术架构，&#x20;
> 2. 需要做哪些工作？ 哪些是已有的不需要开发的，直接拿来用，哪些是需要开发的？
> 3. 尽量使用已有的技术逻辑比如， lua, m2 server。
> 4. 尽量不去重写重构，除非必须重写
> 5. 你的所有评估包括但不限于之前我提到的，你可以自主思考并补充完善。
> 6. 评估并给出完整的技术架构和开发规划

<details><summary>2 previous messages</summary>

> 我会把方案分成两条边界清楚的路线：一条是“合法复用 996 的 M2/客户端内核，做同构扩展平台”，另一条是“保留 M2、Lua、表格和资源习惯，但逐步替换闭源内核”。这两条路线的复用率、独立性和工期差异很大，我会给出组件级“直接复用 / 适配开发 / 必须自研”清单和分阶段交付计划。
>
> <details><summary>Ran a command, searched the web</summary>
>
> - Ran `curl -L --fail --max-time 60 -o work/Mir200.zip 'https://cdnjc.dhsf.huiyuehz.com/ShowDoc/c2/996M2%E6%9C%8D%E5%8A%A1%E7%AB%AFLua/%E6%95%99%E7%A8%8B%E4%B8%8E%E4%B8%8B%E8%BD%BD%E7%AF%87/6479bff7b3bbe.zip' && unzip -l work/Mir200.zip | sed -n '1,240p'`
>
> Searched the web for `site:bbs.996m2.com 996引擎 目录结构 MirServer LoginSrv LogServer DBServer 网关 控制器 ...`
>
> </details>
</details>

可以，而且如果目标是“尽快做出可商业运行的 996 同构产品”，最合理的策略不是重写引擎，而是：

> **把 996 的 M2Server、网关、数据库服务、客户端内核、Lua 运行时、资源工具、热更新和后台视为一个封闭的基础运行时；所有自定义能力优先在 Lua、配置表、GUI、外挂服务和运维平台上完成。**

但必须先明确：

- 直接使用官方 M2 和客户端内核，做出来的是“基于 996 的产品/平台”，不是真正独立的新引擎。
- 如果要求拥有完整源码、可脱离 996 授权、能自主修改协议和底层，那么最终仍要重写客户端内核、网关和 M2，只是可以延后。
- 应当对齐 996 的“外部契约”：目录、表格、Lua API、资源、发布流程和行为；不必照搬其手工操作、编码混乱和版本耦合等技术债。

下面给出一套“最大限度复用、最小限度重写”的完整规划。

---

# 一、建议采用的总体路线

## 路线 A：官方内核同构方案——推荐作为第一阶段

合法取得并直接使用：

- 996 M2Server。
- 登录网关、游戏网关、角色/数据库服务。
- 官方 Windows、Android、iOS 客户端。
- 官方 Lua API、GUI 框架。
- SQL Server 数据结构。
- 集成工具、表格转换、资源打包、热更新。
- GM 后台、工具服、授权、反外挂和 CDN。

自行开发：

- 游戏玩法 Lua。
- 客户端 Lua UI。
- 数据表和资源。
- 支付、账号、渠道、数据分析等外围服务。
- CI/CD、测试、监控、备份和自动化运维。
- 对官方工具和后台的工程化封装。

这条路线的底层复用率可达到约 **75%～90%**。

## 路线 B：独立内核兼容方案——第二阶段再考虑

保留：

- M2 的角色定位和区服模型。
- Lua 脚本语义。
- Excel/CSV 表格结构。
- 资源目录和玩法配置方式。
- 已有业务脚本的主要 API。
- 传奇的地图、战斗和数据模型。

逐步替换：

- M2Server。
- 登录/游戏网关。
- 数据库服务。
- 客户端底层。
- 网络协议。
- 热更新和授权体系。

这条路线最终能形成真正独立引擎，但核心复用率通常只有 **30%～50%**，主要复用的是业务逻辑、数据和技术模型，而不是官方二进制。

---

# 二、完整运行架构

## 1. 在线运行架构

```text
┌──────────────── 客户端层 ────────────────┐
│ Windows 客户端                           │
│ Android 客户端                           │
│ iOS 客户端                               │
│ H5 客户端（可选）                        │
│                                          │
│ 原生客户端内核                           │
│ Lua UI / GUIExport / GUILayout           │
│ 本地资源缓存 / 热更新 / 静默下载         │
└─────────────────┬────────────────────────┘
                  │
          HTTPS：版本、区服、公告、资源
                  │
    ┌─────────────▼─────────────┐
    │ 版本服务 / CDN / 对象存储 │
    └───────────────────────────┘

                  TCP/游戏协议
                  │
          ┌───────▼────────┐
          │ 登录前置/网关  │
          │ LoginGate      │
          └───────┬────────┘
                  │
          ┌───────▼────────┐
          │ 角色选择服务   │
          │ 账号/角色校验  │
          └───────┬────────┘
                  │
          ┌───────▼────────┐
          │ 游戏网关       │
          │ GameGate       │
          └───────┬────────┘
                  │
┌─────────────────▼─────────────────────────┐
│              M2Server 区服                 │
│                                           │
│ 地图、AOI、移动、战斗、怪物、NPC          │
│ 物品、技能、Buff、掉落、任务、活动        │
│ 行会、队伍、排行榜、邮件、拍卖            │
│ LuaJIT / QFunction / NPC Lua / 定时器      │
│ 客户端消息 / SubmitForm / LuaNetMsg        │
└──────────────┬──────────────┬─────────────┘
               │              │
        ┌──────▼──────┐ ┌────▼─────────────┐
        │ 数据库服务  │ │ 跨服 M2Server    │
        │ DB Service  │ │ 跨服地图/活动服  │
        └──────┬──────┘ └──────────────────┘
               │
       ┌───────▼─────────┐
       │ Microsoft SQL   │
       │ Server          │
       │ 角色/装备/变量  │
       │ 行会/邮件/拍卖  │
       └─────────────────┘
```

官方运行日志已经能确认 M2、游戏网关和数据库服务之间的连接关系。[M2 启动日志](https://bbs.996m2.com/thread-11292-1-1.html)

客户端较高置信度属于 Cocos2d-x 系技术，2026 年日志直接出现 `cocos2d: TextureCache`；但官方只开放部分 Lua UI，并未开放完整客户端内核。[客户端日志](https://bbs.996m2.com/forum.php?extra=page%3D1&mobile=no&mod=viewthread&tid=16256) [开源范围说明](https://bbs.996m2.com/thread-7965-1-1.html)

## 2. 外围服务架构

不建议把所有新功能塞进 M2。新增以下独立服务：

```text
统一账号/渠道登录
支付订单与补单服务
充值发货服务
活动配置服务
CDK/礼包服务
排行榜与数据统计服务
运营后台
日志采集与审计
监控与告警
自动部署服务
资源版本管理
客服查询工具
```

推荐交互方式：

- M2 内置能力：优先使用。
- Lua API：第二优先。
- M2 HTTP GET/POST：连接外围服务。
- 数据落库：通过业务服务或官方接口，避免 Lua 任意拼接 SQL。
- 客户端访问业务服务：只获取展示数据；奖励、货币和关键操作必须经 M2 校验。

## 3. 开发与发布架构

```text
Git 代码仓库
├── server-lua
├── client-lua
├── data-tables
├── gui-export
├── resource-manifest
├── db-custom-migrations
└── deploy-config

        │ 提交
        ▼

持续集成
├── Lua 静态检查
├── GBK/UTF-8 编码检查
├── 表格字段和引用检查
├── 资源名称/大小写检查
├── 客户端与服务端表一致性检查
├── 消息号冲突检查
├── 脚本安全检查
└── 自动生成版本 BOM

        │
        ▼

构建流水线
├── XLS/CSV → 客户端 Lua
├── GUIExport 构建
├── 服务端版本包
├── 客户端 DEV 包
├── 热更新差异包
├── 资源清单与哈希
└── 测试服自动部署

        │ 验证通过
        ▼

对象存储/CDN → 灰度区 → 正式区
```

---

# 三、哪些东西直接复用

以下结论以“已经获得对应官方授权和交付包”为前提。

| 组件 | 处理方式 | 是否需要开发 |
|---|---|---:|
| M2Server | 官方二进制直接使用 | 否 |
| 登录网关、游戏网关 | 官方组件直接使用 | 否 |
| 数据库服务、角色服务 | 官方组件直接使用 | 否 |
| 启动器/控制器 | 官方组件直接使用 | 否 |
| Windows/Android/iOS 客户端内核 | 官方客户端直接使用 | 否 |
| Lua/LuaJIT 运行时 | 使用引擎内置版本 | 否 |
| GUI 控件体系 | 使用 `GUI:*`、`SL:*` | 否 |
| GUI 可视化编辑器 | 官方工具直接使用 | 否 |
| SQL Server 基础数据库 | 官方结构和升级脚本 | 否 |
| 基础角色、背包、装备、技能 | M2 内置 | 否 |
| 怪物、NPC、地图、掉落 | M2 内置，配置数据即可 | 基本不需要 |
| 行会、组队、好友 | M2 内置 | 否 |
| 邮件、商城、排行榜、拍卖 | 优先使用内置系统 | 少量适配 |
| Buff、自定义技能 | 使用官方表格和 Lua API | 少量脚本 |
| 跨服 | 使用独立跨服 M2 | 配置和业务脚本 |
| 资源压缩、转换 | 官方集成工具 | 否 |
| 热更新、微端、静默下载 | 官方链路 | 配置为主 |
| 反外挂 | 官方能力直接接入 | 接入与规则配置 |
| GM 后台、工具服 | 官方平台 | 接入与权限配置 |
| 官方基础 UI | 直接继承 | 否 |
| 官方公共资源 | 仅在授权范围内使用 | 否 |

996 的基础系统和资源发布能力已经相当完整，官方产品页也将多端、GUI、反外挂、转换工具和 CDN 作为产品能力提供。[官方产品说明](https://www.996m2.com/engineDownload?tab=5)

服务端已提供大量人物、物品、怪物、地图、Buff、技能、行会、跨服接口，不建议重复实现。[服务端 Lua 文档](https://engine-doc.996m2.com/web/#/9/154)

---

# 四、需要自行开发的部分

## 1. 游戏业务内容

这是项目主要工作量：

- 职业与成长数值。
- 装备体系。
- 技能和 Buff 配置。
- NPC、任务、剧情。
- 地图与怪物刷新。
- 副本、Boss、活动。
- 货币、商城和经济循环。
- 回收、锻造、合成、洗练、强化。
- 排行榜、赛季、开服活动。
- 跨服活动规则。
- 新手引导。
- 防刷、防工作室规则。

原则：

1. 表格可以解决的，不写 Lua。
2. Lua 可以解决的，不写外部服务。
3. M2 内置系统可以解决的，不重复实现。
4. 只有支付、账号、分析和跨区数据等场景才使用外部服务。

## 2. 客户端业务层

需要开发：

- 自定义功能界面。
- 活动面板。
- 商城和支付展示。
- 装备养成界面。
- 自定义排行榜。
- 红点和状态提示。
- 新手引导。
- PC 与移动端布局适配。
- 前后端消息处理。
- 资源引用与动态下载。

使用：

- `GUILayout` 编写行为。
- `GUIExport` 存放编辑器生成界面。
- `SL:SubmitForm` 或 LuaNetMsg 通讯。
- 官方 `SL:*`、`GUI:*` API。

不需要：

- 重写地图渲染。
- 重写角色渲染。
- 重写输入系统。
- 重写底层资源管理。
- 重写网络协议。
- 重写客户端热更新。

## 3. 外围业务服务

建议独立开发：

- 统一账号映射。
- 渠道登录适配。
- 支付订单、回调、补单。
- 充值发货幂等服务。
- 礼包码、兑换码。
- Web 运营后台。
- 数据埋点、留存、付费分析。
- 客服角色查询。
- 封禁和风险控制。
- 版本与区服管理。
- 公告和活动配置。

这些服务不应直接修改 M2 核心数据库，最好：

- 使用独立数据库。
- 通过队列、HTTP 或受控存储过程与游戏侧交互。
- 所有发货都有唯一订单号。
- 所有奖励接口都具备幂等性。

## 4. 工程化能力

官方工具解决“能打包”，但不会自动形成可靠研发流程，因此仍需要开发：

- 自动构建脚本。
- 表格校验器。
- Lua 检查器。
- 资源命名检查。
- 客户端/服务端表同步检查。
- 版本依赖锁定工具。
- 一键部署与回滚。
- 数据库备份和恢复。
- 日志集中采集。
- 告警系统。
- 自动化回归测试。
- 压力测试机器人。

---

# 五、必须新建的版本锁定机制

996 最常见的工程风险不是某个系统不会做，而是版本错配。官方也反复要求引擎、客户端和数据库严格对应。[版本对应说明](https://bbs.996m2.com/forum.php?mod=viewthread&tid=6508)

每次发布必须生成一个不可变 BOM：

```yaml
release: 1.6.12
engine_version: 2026xxxx
client_version: 3.71.1
database_version: 64_26.xx.xx
server_lua_commit: abc123
client_lua_commit: def456
table_version: 20261008.03
resource_version: 20261008.06
protocol_message_registry: v17
hotfix_base_version: 1.6.11
```

禁止：

- 单独替换 M2。
- 单独更新客户端。
- 手工覆盖某几张表后不登记。
- 正式服直接修改 Lua。
- 不保留数据库升级前备份。
- 从其他底板复制脚本但不检查 API 版本。

---

# 六、编码和数据规范

996 存在服务端 GB2312/GBK、客户端 UTF-8 并存的问题。[纯 Lua 引擎说明](https://engine-doc.996m2.com/web/#/30/1159)

应建立强制规则：

- Git 中服务端 Lua 统一保存 UTF-8。
- 构建时自动转换为目标 GBK/GB2312。
- 客户端 Lua 保持 UTF-8。
- 禁止开发人员手工转换编码。
- 文件路径只使用英文、数字和下划线。
- 资源文件统一小写。
- 构建时检查大小写冲突。
- 表格 ID、消息 ID、属性 ID 建立统一注册表。

数据库策略：

- 官方基础表只使用官方迁移工具升级。
- 自定义数据放独立数据库或独立 Schema。
- 不修改官方字段语义。
- 不在 Lua 中执行未经参数化的 SQL。
- 生产库开启完整、差异和事务日志备份。
- 每月至少做一次真实恢复演练。

---

# 七、安全架构

即便使用官方反外挂，也不能把业务安全全部交给客户端或反外挂。

## M2/Lua 必须实施

- 客户端参数全部视为不可信。
- 对消息号、文件名、函数名建立白名单。
- 对等级、数量、价格、坐标、物品 ID 做范围校验。
- 对领取、合成、购买增加冷却和频率限制。
- 奖励和充值操作增加幂等键。
- 高价值操作写独立审计日志。
- 禁止客户端决定伤害、奖励或货币结果。
- 禁止危险 Lua 动态加载。
- 禁止在 Lua 中保存长期有效的引擎对象引用。
- 跨服返回时重新校验角色状态。
- 对异常移动、攻速、拾取和交易频率做服务端检测。

官方 `SubmitForm` 文档本身就提示存在改包、负数、小数和越界参数风险。[SubmitForm 文档](https://engine-doc.996m2.com/web/#/29/317)

## 外部服务必须实施

- 支付回调签名验证。
- 重放防护。
- IP 和账号频率限制。
- GM 操作双因素认证。
- GM 权限分级。
- 所有后台操作审计。
- 数据库最小权限。
- 密钥不放在脚本和版本包中。

---

# 八、测试体系

## 1. 契约测试

验证：

- 客户端版本是否匹配 M2。
- 数据库结构是否匹配。
- Lua API 是否存在。
- 消息 ID 是否冲突。
- 表格引用是否存在。
- 资源路径是否有效。
- 前端和后端同一物品/技能 ID 是否一致。

## 2. 游戏回归测试

建立固定测试角色和脚本，自动覆盖：

- 创建角色、登录、重登。
- 移动、切图、传送。
- 普攻、技能、Buff、死亡、复活。
- 拾取、背包、仓库、穿戴。
- 交易、摆摊、拍卖。
- 组队、行会、邮件。
- 副本和跨服。
- 充值发货。
- 热更新前后登录。
- 断线恢复。

## 3. 性能测试

不要直接把旧文档中的“500 人”当作容量结论。应分场景测试：

- 100、300、500、800 个在线模拟客户端。
- 50～200 人同屏。
- 沙巴克集中战斗。
- 大量怪物和持续寻路。
- 高频 Buff 和定时器。
- 大量拾取、掉落和背包操作。
- SQL Server 延迟和断开。
- 跨服进出。
- 热更新期间登录峰值。

核心指标：

- M2 主循环耗时。
- Lua 脚本单次/累计耗时。
- 网关连接数和流量。
- SQL 查询延迟。
- 地图线程负载。
- 玩家消息队列积压。
- 客户端帧率、内存和资源加载时间。

---

# 九、部署建议

## 不建议把 M2 强行容器化

官方 M2、网关、控制器明显偏向 Windows 原生运行。为了“看起来云原生”强行放进容器，收益有限、故障排查更困难。

推荐：

- M2、网关：Windows Server 虚拟机。
- SQL Server：独立高性能数据库服务器。
- Web 服务：Linux 容器或普通云主机。
- Redis：排行榜缓存、限流、订单幂等，可选。
- 对象存储/CDN：客户端资源和热更新。
- 日志平台：集中采集 M2、网关、Lua、Web 日志。

## 区服拓扑

```text
每个游戏区：
1 × M2Server
1～N × GameGate
1 × DB/角色服务
1 × SQL 数据库或数据库实例

每个跨服组：
1～N × 跨服 M2
独立跨服配置
受控的数据同步规则

公共服务：
账号、支付、CDK、公告、分析、监控、CDN
```

M2 横向扩容的主要方式是“多区、多跨服实例”，而不是把一个大世界任意拆成微服务。

---

# 十、开发计划

以下按“使用官方内核、做一款全新商业产品”估算。

## 阶段 0：授权和技术清点，2～3 周

交付物：

- 书面确认 M2、客户端、后台和资源的授权边界。
- 获取完整安装包和版本对应表。
- 列出所有进程、端口、配置文件。
- 确认 PC、Android、iOS、H5 的交付范围。
- 确认反外挂、CDN、工具服和正式服条件。
- 建立组件哈希和二进制归档。

退出条件：

- 可以稳定启动空白测试服。
- 三端至少两个客户端可进入同一区服。
- 可以创建角色、移动、战斗和保存数据。

## 阶段 1：标准基线环境，3～4 周

工作：

- 固定 M2、客户端、数据库版本。
- 建立开发服、测试服、预发布服。
- 标准化端口和目录。
- 完成一键启动/停止。
- 完成数据库初始化和备份。
- 建立版本 BOM。
- 建立日志收集。

退出条件：

- 新机器可以在两小时内自动部署完整环境。
- 数据库可以从备份恢复。
- 所有版本信息可以追踪。

## 阶段 2：开发流水线，4～6 周

工作：

- Git 仓库和分支策略。
- Lua 检查和编码转换。
- 表格校验。
- 资源校验。
- 消息号注册表。
- XLS/CSV 转 Lua 自动化。
- DEV 包、热更新包自动生成。
- 测试服自动部署。
- 一键回滚。

退出条件：

- 开发人员不需要手工复制文件。
- 每个构建都有唯一版本号。
- 构建失败能够指出具体表格、脚本或资源问题。

## 阶段 3：垂直切片，6～8 周

范围：

- 一个职业。
- 一张主城、一张野外、一张副本。
- 移动、战斗、技能和 Buff。
- 装备、背包、掉落、仓库。
- NPC、任务、商城。
- 一套前端自定义 UI。
- 一次完整热更新。
- 基础账号和支付沙箱。

目标不是内容量，而是验证完整技术链路。

## 阶段 4：核心玩法生产，10～16 周

并行开发：

- 职业和技能。
- 装备养成。
- Boss、副本和活动。
- 经济和商城。
- 排行榜和赛季。
- 行会和社交。
- 新手引导。
- 自动战斗与挂机规则。
- 防刷和异常行为检测。
- 客户端 UI 和资源生产。

## 阶段 5：多端与发布，6～10 周

工作：

- PC 操作适配。
- Android/iOS 分辨率适配。
- 弱网、切后台、重连。
- 首包、热更新、微端和静默下载。
- CDN 灰度。
- 客户端性能优化。
- 渠道 SDK。
- 包体、权限和隐私合规。

## 阶段 6：跨服和商业能力，6～10 周

工作：

- 跨服地图。
- 跨服活动。
- 跨服进出状态恢复。
- 充值、补单、CDK。
- 客服和 GM 后台。
- 审计、监控、告警。
- 压力测试和故障演练。

## 阶段 7：封测和上线，6～8 周

工作：

- 数据删档封测。
- 付费封测。
- 高峰压测。
- 安全测试。
- 备份恢复演练。
- 灰度发布。
- 回滚演练。
- 正式上线。

总周期：

- 已有成熟版本迁移到 996：约 **3～5 个月**。
- 从零制作一款新商业产品：约 **7～12 个月**。
- 包含大量原创系统和跨服玩法：约 **10～15 个月**。

---

# 十一、建议团队

不包含大量美术内容生产时：

| 角色 | 人数 |
|---|---:|
| 技术负责人/架构师 | 1 |
| 服务端 Lua 开发 | 2～3 |
| 客户端 Lua/UI 开发 | 2 |
| 平台/Web 后端 | 1～2 |
| 工具链/构建工程师 | 1 |
| DBA/SRE | 1 |
| QA/自动化测试 | 2 |
| 技术美术/资源工程师 | 1～2 |

合计约 **11～15 人**。

如果只是迁移已有成熟版本，可以压缩到 **6～9 人**。

---

# 十二、什么情况下才允许重写

按以下优先级判断。

## 不重写

只要官方能力能够满足，就不重写：

- 地图和战斗。
- 背包和装备。
- 行会、邮件、拍卖。
- Buff、技能和怪物 AI。
- 客户端渲染。
- 网关和协议。
- 热更新。
- 跨服基础链路。

## 先做适配，不重写

遇到能力差异时依次尝试：

1. 配置表。
2. 官方 Lua API。
3. 自定义 Lua。
4. 客户端 Lua UI。
5. HTTP 外围服务。
6. 请求官方增加 API。
7. 最后才考虑替换内核。

## 必须重写的条件

只有出现以下情况才值得替换：

- 授权不允许按计划发行。
- 需要完全脱离 996 后台或认证。
- 必须获得完整源代码控制权。
- 官方协议无法满足安全目标。
- M2 性能无法达到已验证的容量指标。
- 业务需要真正的无缝大世界或跨区分布式状态。
- 官方客户端无法支持目标平台。
- 核心玩法需要修改渲染、物理、同步或战斗底层。
- 供应商停止维护或无法满足 SLA。

需要特别注意：客户端、网关、协议和 M2 高度耦合。要替换时，最好作为一个整体替换，不建议长期维持“新客户端 + 旧网关 + 旧 M2”的混合状态。

---

# 最终建议

建议采用“双阶段战略”：

1. **第一阶段完全复用官方 996 内核**，把产品、内容、工具链、测试和运营体系做出来，目标 7～12 个月上线。
2. 将 M2 当作黑盒“运行时设备”，不修改协议、不直接侵入数据库、不依赖未公开内部行为。
3. 所有新业务通过 Lua、表格和外围服务实现。
4. 建立版本 BOM、自动构建、契约测试和数据审计，解决 996 生态最常见的工程化短板。
5. 运行 6～12 个月取得真实性能和业务数据后，再决定是否立项独立内核。
6. 如果未来自研，优先保留脚本 API、数据表和资源流程；客户端内核、网关、协议、M2 最后整体替换。

这条路线最大程度满足了“参考并对齐 996、使用 Lua 和 M2、尽量不重写”，同时不会一开始就承担完整 MMO 引擎研发的巨大风险。

> 我计划让你完成这个项目，先起个名字，你觉得叫什么名字好，要有英文和中文名

<details><summary>2 previous messages</summary>

> 我先按“可长期品牌化、不过度绑定 996/M2、适合做仓库名和产品名”筛一轮，并快速排查明显重名；最终会给一个主推荐和几组备选。
>
> <details><summary>Searched the web</summary>
>
> Searched the web for `"MythForge Engine" game engine ...`
>
> Searched the web for `"EmberRealm" game OR engine OR software ...`
>
> Searched the web for `"Riftloom" game OR engine OR software ...`
>
> Searched the web for `"LumenRealm" game OR engine OR software ...`
>
> </details>
</details>

我最推荐：

# 烬界引擎 · EmberRealm Engine

含义：

- “烬”：从成熟的传奇/M2 技术体系中继承火种，也代表重生。
- “界”：游戏世界、区服、跨服和多端统一。
- `EmberRealm` 有经典奇幻气质，但不直接绑定“传奇”“996”或“M2”，将来替换底层仍能继续使用。
- 适合产品化，也适合作为开源仓库、SDK 和管理平台的统一品牌。

建议命名体系：

- 正式中文名：**烬界引擎**
- 正式英文名：**EmberRealm Engine**
- 项目代号：**Project Ember**
- 简称：**ERE**
- 仓库前缀：`emberrealm-*`
- Lua 命名空间：`ER`
- 版本名称示例：`EmberRealm 0.1 Ash`

模块可以这样命名：

```text
emberrealm-runtime      核心运行环境
emberrealm-m2           M2 集成适配
emberrealm-client       多端客户端工程
emberrealm-scripts      前后端 Lua
emberrealm-data         配置表和数据定义
emberrealm-toolkit      资源与开发工具
emberrealm-control      控制台和运营后台
emberrealm-deploy       部署与运维
emberrealm-docs         技术文档
```

我初步排查过，常见备选名 `MythForge` 已被多个游戏工作室和软件项目使用；`EmberRealm` 目前仅发现零散的游戏服务器使用记录，冲突相对少，但正式注册前仍要做商标、域名、GitHub/Gitee 组织名的完整检索。[MythForge 重名情况](https://mythforgeusa.com/about) [EmberRealm 检索结果](https://rankplay.ru/servers/5243563/statistics)

备选名称：

- **烽界引擎 · PyraRealm Engine**：更有战斗和热血感。
- **界匠引擎 · RealmSmith Engine**：强调工具链和内容生产。
- **玄域引擎 · ArcaneRealm Engine**：更传统的东方奇幻气质。
- **新铸引擎 · NovaForge Engine**：强调重构和新一代。
- **铁境引擎 · IronRealm Engine**：强调稳定、耐用和服务器权威。

综合品牌感、技术延展性和项目气质，我建议正式定名为：

> **烬界引擎（EmberRealm Engine）**
> *Forge worlds from embers.*
> **以余烬，铸新界。**
