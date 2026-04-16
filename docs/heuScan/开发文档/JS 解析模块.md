# 一、项目概述
JS解析模块是通过解析前端JS/TS代码，提取其中的组件信息，外联URL信息，识别加密算法，提取固定密钥，提取泄露的注释以及敏感信息的模块。
# 二、项目功能拆分
## 2.1 组件信息提取
    组件信息提取的核心目标，不是简单判断前端使用了什么框架，而是从 JS/TS 代码中还原页面由哪些组件、模块和交互单元构成，以及这些结构在业务上承担什么角色。很多前端页面的真实业务边界并不完全体现在 HTML 结构中，而是体现在脚本层的组件注册、状态组织、事件绑定和页面组装逻辑中，因此该部分应承担“补全页面结构语义”的职责。

    该能力至少应覆盖三层分析价值：
    1. 页面组成信息：识别页面中存在的组件、模块、弹窗、表单、列表、详情区、管理区等结构性单元；
    2. 功能区块语义：判断这些组件更偏向登录、搜索、上传、审批、配置、管理、支付、报表等哪类业务功能；
    3. 关联线索补充：为组件与接口、路由、状态字段、权限控制、页面跳转之间建立静态语义上的关联基础。

    这一部分的边界应保持清晰：只做静态语义层面的组件信息抽取、归纳和角色判断，不展开具体 AST 落地方案、框架适配策略、组件命名规则或工程实现细节。

## 2.2 外联url信息识别
    外联url信息识别的目标，是从 JS/TS 中提取脚本主动引用或间接拼接出的外部通信目标，包括接口地址、第三方资源地址、跳转目标和潜在跨域调用线索。前端代码往往会沉淀大量未直接暴露在页面上的请求地址和依赖目标，因此该部分不仅用于补全资源关系，也用于发现隐藏接口、外部依赖面和可进一步关注的通信对象。

    从分析视角看，外联目标至少应按用途做大致归类：
    1. 接口类：业务 API、认证接口、配置接口、上传下载接口等；
    2. 静态资源类：脚本、图片、字体、配置文件、CDN 资源等；
    3. 第三方服务类：地图、短信、支付、对象存储、客服、身份认证等外部依赖；
    4. 埋点与监控类：日志上报、行为采集、错误监控、性能监控等地址；
    5. 跳转类：登录跳转、业务跳转、单点登录、外部链接跳转等目标。

    该部分的价值，在于帮助上游与下游模块理解前端系统对外部资源和服务的依赖面，发现隐藏业务接口、潜在边界扩展线索和可能涉及敏感通信的目标；但这里只描述识别与归类本身，不展开 URL 清洗、请求还原、归属判定或具体规则设计。

## 2.3 加密算法识别与固定密钥提取
    加密算法识别与固定密钥提取，关注的是 JS/TS 中与加密、编码、签名、摘要计算、令牌构造和参数混淆相关的逻辑片段。前端代码经常包含请求签名、口令处理、设备标识生成、加密传输包装等安全相关逻辑，这些内容既影响接口分析，也可能暴露较高价值的安全线索，因此该部分应承担“识别安全处理逻辑和潜在硬编码材料”的职责。

    该能力的输出方向主要有两个：
    1. 识别存在何种安全处理行为：例如前端是否存在加密、签名、摘要、编码、混淆、令牌拼装或认证材料加工等逻辑；
    2. 识别是否存在固定凭据或可复用安全材料：例如硬编码密钥、盐值、IV、Token、密文模板、默认口令片段或固定认证参数。

    这一部分的主要意义，是为后续安全分析、接口理解和风险判断提供线索，而不是直接做破解、还原、模拟调用或漏洞验证设计。因此其边界应停留在“识别出什么安全逻辑、暴露了什么固定材料、为何值得关注”这一层。

## 2.4 注释等泄露信息提取
    注释等泄露信息提取，关注的不仅是显式注释本身，还包括调试残留、未清理说明、临时提示语、内部命名、环境标识、路径备注、接口说明、报错提示和研发阶段留下的语义文本。很多前端代码中的高价值线索，并不体现在正式业务逻辑里，而是隐藏在这些“非主流程文本”中，因此该部分承担的是“补充业务语义和暴露研发痕迹”的职责。

    这部分信息的分析价值主要体现在：
    1. 补充业务语义：帮助理解某个接口、组件、页面或参数原本 intended 的用途；
    2. 暴露研发痕迹：识别测试环境、预发布环境、内部服务代号、历史遗留模块和未上线功能线索；
    3. 补充资产线索：从文本中发现内部域名、路径、接口命名、系统角色或环境区分信息。

    该能力的边界应限定为信息性泄露线索的提取与归类，不在这里扩展为敏感信息分级、风险评分、告警策略或处置流程设计。

## 2.5 标准化的输入与输出
    JS解析模块的输入与输出，应强调逻辑边界统一，而不是在这一部分直接展开字段级设计。该模块本身不负责主动抓取目标、发起验证或扩大分析范围，而是承接上游已经获取的脚本内容与上下文，对其中的前端语义进行结构化提炼，再输出给图谱、价值评估、报告输出等后续模块使用。

    在输入侧，本模块应主要面向以下几类来源：
    1. JS/TS 文件内容；
    2. 页面中的内联脚本片段；
    3. 上游传入的目标上下文，如页面地址、资源来源、目标归属信息；
    4. 与脚本关联的页面、资源或已有基础发现结果。

    在输出侧，本模块应主要沉淀以下几类结果：
    1. 组件与页面模块线索；
    2. 外联目标与依赖线索；
    3. 加密与签名逻辑线索；
    4. 固定密钥或可复用安全材料风险；
    5. 注释、说明文本及其他泄露线索；
    6. 面向下游消费的结构化摘要。

    这一部分需要明确的上下游关系是：JS解析模块只负责把脚本中的语义价值、依赖关系和安全线索提炼出来，不承担主动采集、漏洞验证或独立裁决风险等级的职责，而是为后续关联分析和结果表达提供基础素材。

# 三、提示词设计
    本模块建议以“受约束的静态语义分析 Agent”模式运行，而不是把任意脚本文本直接抛给大模型自由总结。提示词设计的目标，是让模型明确知道自己的职责边界、输入范围、输出结构、证据要求和禁止事项，尽量减少臆造结论、越界分析和无依据风险升级。

## 3.1 提示词设计目标

    提示词应同时解决以下四个问题：
    1. 角色清晰：模型只负责对 JS/TS 文本做静态分析、结构化提取和风险线索归类；
    2. 输入清晰：模型只能使用已提供的脚本内容、来源信息和上游上下文，不能自行假设不存在的页面或接口；
    3. 输出清晰：模型必须按组件、外联 URL、加密行为、固定材料、泄露信息五类结果输出，并给出证据；
    4. 边界清晰：模型不得主动联网、不得做漏洞验证、不得补造业务背景、不得把弱线索写成确定事实。

## 3.2 系统提示词约束

    推荐将系统提示词固定为“模块级常量”，不要在运行时频繁改写。系统提示词至少应覆盖以下约束：
    1. 分析对象是 JS/TS 代码及其元信息，不是完整网站；
    2. 仅做静态语义分析，不做运行时执行推断、真实网络请求模拟或破解设计；
    3. 所有发现必须附带证据片段或来源位置；
    4. 若证据不足，只能输出低置信或 `inferred=true` 的推断结果；
    5. 若输入超长或混淆严重，允许降级输出摘要和告警，而不是强行给出完整结果；
    6. 若发现内容超出授权范围，只能标记 warning，不得自行扩展分析目标。

## 3.3 推荐提示词模板

### 3.3.1 系统提示词模板

```text
你是 JS 解析模块（JS Semantic Analysis Agent），负责对给定的 JS/TS 代码做静态语义分析。

你的职责仅包括：
1. 提取组件与页面模块线索；
2. 提取外联 URL、接口地址和第三方依赖目标；
3. 识别加密、签名、编码、令牌构造等安全处理行为；
4. 识别可能硬编码的密钥、盐值、IV、Token、默认凭据等固定材料；
5. 提取注释、调试残留、环境标识、内部命名、接口说明等泄露线索。

你必须遵守以下约束：
1. 只能使用输入中提供的脚本文本、来源信息、页面上下文和上游引用；
2. 不得主动联网，不得假设未提供的接口响应、页面 DOM 或运行结果；
3. 不做漏洞验证、利用建议、破解设计或越权推断；
4. 所有发现都必须尽量给出证据片段、源位置或来源引用；
5. 证据不足时只能输出低置信结果，或标记 inferred=true；
6. 输出必须严格使用指定结构，不得返回自由散文。
```

### 3.3.2 任务提示词模板

```text
请根据输入的 JsAnalysisCommand 对脚本内容进行分析。

分析范围：
- analysis_scope: {{analysis_scope}}
- 允许的推断级别: {{allow_inference}}
- 最大单脚本字符数: {{max_code_chars}}
- 每类结果最大返回数: {{max_findings_per_type}}
- 混淆处理级别: {{deobfuscation_level}}
- 风险标注模式: {{risk_annotation_mode}}

输出要求：
1. 返回 component_findings、external_urls、crypto_findings、hardcoded_secrets、leakage_findings、summary、warnings；
2. 每条结果尽量给出 evidence、source_span、confidence、reason；
3. 对无法确认但值得保留的线索，标记 inferred=true；
4. 对超范围、超预算、混淆过重、证据不足等情况写入 warnings；
5. 不要输出未要求的解释性正文。
```

### 3.3.3 失败回退提示

```text
如果脚本内容超长、严重混淆或缺少必要上下文，优先输出：
1. 可以确认的高置信结果；
2. 无法继续细化的原因；
3. 建议下游复核或补充上下文的 warning；
4. 不要为了凑齐字段而臆造结果。
```

## 3.4 关键参数解释

### 3.4.1 提示词运行参数

| 参数名 | 类型 | 默认值 | 说明 | 取值建议 | 何时不应调高 |
| --- | --- | --- | --- | --- | --- |
| `analysis_scope` | enum | `full` | 控制当前任务重点分析范围，可取 `full`、`components_only`、`url_only`、`crypto_only`、`leakage_only` | 默认使用 `full`，仅在总控明确裁剪能力时收窄 | 当需要完整结果时不应使用单能力模式 |
| `max_code_chars` | int | `120000` | 单次送入模型的最大字符数，超过后应分块或降级 | 普通业务脚本建议 `60000-120000`，大型 bundle 可拆分后分别分析 | 混淆 bundle 直接调很高会造成上下文拥塞和结果漂移 |
| `max_findings_per_type` | int | `50` | 每类结果允许返回的最大发现数，防止大文件导致结果爆炸 | 常规场景 `20-50`，审计场景可提高到 `100` | 当下游只做摘要消费时不应拉太高 |
| `allow_inference` | enum | `limited` | 是否允许模型做有限推断，可取 `none`、`limited`、`moderate` | 默认 `limited`，只允许保守语义推断 | 证据弱、混淆重或高风险输出场景不应调到 `moderate` |
| `require_evidence_span` | bool | `true` | 是否要求每条结果尽量给出源码片段或位置范围 | 建议固定为 `true` | 只有输入源位置丢失时才允许降为 `false` |
| `deobfuscation_level` | enum | `basic` | 允许的去混淆强度，可取 `none`、`basic`、`enhanced` | 常规使用 `basic`，仅做字符串拼接和常见包装还原 | 不应在资源紧张或缺少边界控制时设为 `enhanced` |
| `risk_annotation_mode` | enum | `conservative` | 风险标注风格，可取 `off`、`conservative`、`security_focused` | 默认 `conservative`，仅对明显安全线索标风险 | 不应在普通功能梳理场景开启 `security_focused` |

### 3.4.2 参数使用原则

    这些参数的作用不是让模型“变聪明”，而是让模型的输出边界更稳定。工程上应遵循以下原则：
    1. `analysis_scope` 优先由总控控制，不建议在模块内部自由切换；
    2. `max_code_chars` 与 `max_findings_per_type` 必须联动控制，避免长脚本直接产出过量噪声结果；
    3. `allow_inference` 和 `risk_annotation_mode` 都属于高影响参数，默认应保守；
    4. `deobfuscation_level` 的提升，应以时间预算和误报成本可接受为前提；
    5. 一旦 `require_evidence_span=false`，下游应自动下调结果可信度。

## 3.5 输出格式约束与禁止事项

    为避免不同批次调用输出风格不一致，提示词中应显式写死以下约束：
    1. 返回 JSON 风格结构，不返回自然语言总结长文；
    2. 每类结果都应尽量包含 `reason`、`confidence`、`evidence`；
    3. 无证据时不得填写确定性字段；
    4. 同一线索若来自多个证据，可聚合，但必须保留 `upstream_refs`；
    5. 推断性结论必须加 `inferred=true`；
    6. 不得在结果中编造接口响应、用户身份、运行结果或漏洞可利用性。

# 四、输入示例
    JS解析模块的输入建议统一抽象为 `JsAnalysisCommand`，而不是直接把某个爬虫或浏览器抓到的原始对象透传进来。这样做的目的，是让上游采集模块、总控agent和下游消费模块都围绕同一套逻辑契约协作，避免输入格式随调用来源变化而漂移。

## 4.1 输入对象定义

### 4.1.1 `JsAnalysisCommand`

```json
{
  "task_id": "task_20260414_0001",
  "target": "https://app.example.com/dashboard",
  "page_url": "https://app.example.com/dashboard",
  "source_type": "page_script",
  "upstream_refs": [
    "event://obs/1201",
    "evidence://html/8801"
  ],
  "scripts": [],
  "analysis_scope": "full",
  "limits": {
    "max_scripts_per_task": 20,
    "max_single_script_chars": 120000,
    "max_total_code_chars": 300000
  },
  "parse_options": {
    "max_code_chars": 120000,
    "max_findings_per_type": 50,
    "allow_inference": "limited",
    "require_evidence_span": true,
    "deobfuscation_level": "basic",
    "risk_annotation_mode": "conservative"
  }
}
```

### 4.1.2 `ScriptPayload`

```json
{
  "script_id": "script_ext_001",
  "script_type": "external",
  "language": "js",
  "source_url": "https://app.example.com/static/app.3d9a1.js",
  "referer": "https://app.example.com/dashboard",
  "fetch_time": "2026-04-14T10:00:00+08:00",
  "content_digest": "sha256:demo_hash",
  "content": "const apiBase='/api'; function loadUser(){ return fetch(apiBase + '/user/profile') }"
}
```

### 4.1.3 `ParseOptions`

```json
{
  "max_code_chars": 120000,
  "max_findings_per_type": 50,
  "allow_inference": "limited",
  "require_evidence_span": true,
  "deobfuscation_level": "basic",
  "risk_annotation_mode": "conservative"
}
```

## 4.2 字段解释

### 4.2.1 `JsAnalysisCommand` 字段说明

| 字段名 | 类型 | 必填 | 默认值 | 说明 | 校验与约束 | 对下游的影响 |
| --- | --- | --- | --- | --- | --- | --- |
| `task_id` | string | 是 | - | 当前任务唯一标识 | 非空，建议由总控统一生成 | 用于关联所有发现和证据 |
| `target` | string | 是 | - | 当前分析目标，可为 URL 或资源标识 | 非空；应与任务范围一致 | 下游据此挂接到资产图谱 |
| `page_url` | string | 否 | `null` | 当前脚本所属页面 URL | 若 `source_type=page_script` 建议必填 | 相对路径解析和页面归属依赖该字段 |
| `source_type` | enum | 是 | - | 输入来源类型，建议 `page_script`、`file_script`、`bundle_script` | 仅允许白名单枚举 | 决定结果的默认置信度和归属策略 |
| `upstream_refs` | string[] | 否 | `[]` | 上游事件或证据引用 | 引用格式应可追踪 | 用于结果回放和审计 |
| `scripts` | array | 是 | - | 待分析脚本集合 | 至少 1 个 `ScriptPayload` | 模块核心输入载荷 |
| `analysis_scope` | enum | 否 | `full` | 本次分析范围 | 仅允许预定义枚举 | 影响结果种类和性能预算 |
| `limits` | object | 否 | 见默认值 | 任务级大小限制 | 超限时应降级而非失败 | 控制模块稳定性和结果规模 |
| `parse_options` | object | 否 | 见默认值 | 分析行为参数 | 由总控或运行配置传入 | 影响输出细度、风险风格和误报率 |

### 4.2.2 `ScriptPayload` 字段说明

| 字段名 | 类型 | 必填 | 默认值 | 说明 | 校验与约束 | 对下游的影响 |
| --- | --- | --- | --- | --- | --- | --- |
| `script_id` | string | 是 | - | 脚本唯一标识 | 非空；同任务内唯一 | 用于结果回指具体脚本 |
| `script_type` | enum | 是 | - | 脚本类型，建议 `external`、`inline`、`bundle`、`chunk` | 仅允许白名单枚举 | 影响相对路径解析和可信度 |
| `language` | enum | 否 | `js` | 脚本语言类型 | 建议 `js`、`ts`、`jsx`、`tsx` | 影响组件语义识别方式 |
| `source_url` | string | 否 | `null` | 外链脚本来源 URL | `external` 类型建议必填 | 用于第三方归属和资源追踪 |
| `referer` | string | 否 | `null` | 来源页面地址 | 允许为空 | 帮助恢复页面上下文 |
| `fetch_time` | string | 否 | `null` | 抓取时间 | 建议 ISO8601 | 便于审计与重放 |
| `content_digest` | string | 否 | `null` | 内容摘要 | 建议 `sha256:*` | 用于去重和缓存命中 |
| `content` | string | 是 | - | 脚本文本内容 | 不允许为空字符串 | 模块实际分析对象 |

### 4.2.3 `limits` 与 `ParseOptions` 字段说明

| 字段名 | 类型 | 默认值 | 说明 | 推荐取值 | 约束 |
| --- | --- | --- | --- | --- | --- |
| `limits.max_scripts_per_task` | int | `20` | 单任务允许分析的脚本数量上限 | `10-30` | 超限应拆分任务 |
| `limits.max_single_script_chars` | int | `120000` | 单脚本字符上限 | `60000-120000` | 超限应切块或降级 |
| `limits.max_total_code_chars` | int | `300000` | 任务内总代码字符数上限 | `150000-500000` | 超限应分批处理 |
| `parse_options.max_code_chars` | int | `120000` | 单次送模代码长度上限 | 与 `max_single_script_chars` 保持一致 | 不得大于任务级上限 |
| `parse_options.max_findings_per_type` | int | `50` | 每类结果最大条数 | `20-50` | 超限后按置信度截断 |
| `parse_options.allow_inference` | enum | `limited` | 推断级别 | `none` 或 `limited` | 高风险场景不宜过高 |
| `parse_options.require_evidence_span` | bool | `true` | 是否强制输出证据范围 | 建议固定 `true` | 关闭后应同步降置信 |
| `parse_options.deobfuscation_level` | enum | `basic` | 混淆处理级别 | `none`、`basic`、`enhanced` | 受时间预算限制 |
| `parse_options.risk_annotation_mode` | enum | `conservative` | 风险标注风格 | `off` 或 `conservative` | 避免默认过度安全化 |

## 4.3 输入边界说明

    输入契约除了字段本身，还应固定以下边界：
    1. 单文件大小建议不超过 `2 MB`，超出后优先做切块分析或仅提取高价值区段；
    2. 字符编码统一要求 UTF-8，非 UTF-8 内容应由上游先转码或标记 warning；
    3. 多脚本页面默认按脚本粒度拆分为多个 `ScriptPayload`，不要把整页几十个脚本强拼成一个大文本；
    4. 非 JS/TS 文本不作为本模块主输入，若误传 HTML、JSON、CSS，应返回 `warnings` 而不是强行分析；
    5. 页面内联脚本与外链脚本应保留独立 `script_id`，避免结果归属丢失；
    6. 如输入为压缩 bundle，应保留 `script_type=bundle` 标记，便于下游解释结果可信度。

## 4.4 输入示例

### 4.4.1 外链 JS 文件分析

```json
{
  "task_id": "task_20260414_0001",
  "target": "https://app.example.com/dashboard",
  "page_url": "https://app.example.com/dashboard",
  "source_type": "file_script",
  "upstream_refs": [
    "event://obs/1201",
    "evidence://html/8801"
  ],
  "scripts": [
    {
      "script_id": "script_ext_001",
      "script_type": "external",
      "language": "js",
      "source_url": "https://app.example.com/static/app.3d9a1.js",
      "referer": "https://app.example.com/dashboard",
      "fetch_time": "2026-04-14T10:00:00+08:00",
      "content_digest": "sha256:demo_hash_001",
      "content": "const apiBase='/api'; export function getUser(){ return fetch(apiBase + '/user/profile') }"
    }
  ],
  "analysis_scope": "full",
  "limits": {
    "max_scripts_per_task": 20,
    "max_single_script_chars": 120000,
    "max_total_code_chars": 300000
  },
  "parse_options": {
    "max_code_chars": 120000,
    "max_findings_per_type": 50,
    "allow_inference": "limited",
    "require_evidence_span": true,
    "deobfuscation_level": "basic",
    "risk_annotation_mode": "conservative"
  }
}
```

### 4.4.2 页面内联脚本分析

```json
{
  "task_id": "task_20260414_0002",
  "target": "https://oa.example.com/login",
  "page_url": "https://oa.example.com/login",
  "source_type": "page_script",
  "upstream_refs": [
    "event://obs/1305",
    "evidence://html/8813"
  ],
  "scripts": [
    {
      "script_id": "script_inline_003",
      "script_type": "inline",
      "language": "js",
      "source_url": null,
      "referer": "https://oa.example.com/login",
      "fetch_time": "2026-04-14T10:05:00+08:00",
      "content_digest": "sha256:demo_hash_003",
      "content": "const loginApi='/auth/login'; const redirect='/home'; document.querySelector('#loginBtn').onclick=()=>submitLogin();"
    }
  ],
  "analysis_scope": "full",
  "limits": {
    "max_scripts_per_task": 10,
    "max_single_script_chars": 60000,
    "max_total_code_chars": 100000
  },
  "parse_options": {
    "max_code_chars": 60000,
    "max_findings_per_type": 30,
    "allow_inference": "limited",
    "require_evidence_span": true,
    "deobfuscation_level": "none",
    "risk_annotation_mode": "conservative"
  }
}
```

### 4.4.3 混淆或压缩脚本分析

```json
{
  "task_id": "task_20260414_0003",
  "target": "https://m.example.com/",
  "page_url": "https://m.example.com/",
  "source_type": "bundle_script",
  "upstream_refs": [
    "event://obs/1408",
    "evidence://js/9102"
  ],
  "scripts": [
    {
      "script_id": "script_bundle_010",
      "script_type": "bundle",
      "language": "js",
      "source_url": "https://m.example.com/static/chunk-vendors.min.js",
      "referer": "https://m.example.com/",
      "fetch_time": "2026-04-14T10:12:00+08:00",
      "content_digest": "sha256:demo_hash_010",
      "content": "var _0x3ab1=['/api/order','AES','token','https://log.example.net'];(function(_0x1){...})();"
    }
  ],
  "analysis_scope": "full",
  "limits": {
    "max_scripts_per_task": 5,
    "max_single_script_chars": 120000,
    "max_total_code_chars": 120000
  },
  "parse_options": {
    "max_code_chars": 120000,
    "max_findings_per_type": 20,
    "allow_inference": "none",
    "require_evidence_span": true,
    "deobfuscation_level": "basic",
    "risk_annotation_mode": "security_focused"
  }
}
```

# 五、输出示例
    JS解析模块的输出建议统一抽象为 `JsAnalysisResult`。输出的目的不是生成报告正文，而是向图谱聚合、价值评估、报告输出等模块提供结构化发现。因此输出必须稳定、可追溯、可截断、可复核，而不能依赖调用方再去二次猜测字段含义。

## 5.1 输出对象定义

### 5.1.1 `JsAnalysisResult`

```json
{
  "task_id": "task_20260414_0001",
  "target": "https://app.example.com/dashboard",
  "component_findings": [],
  "external_urls": [],
  "crypto_findings": [],
  "hardcoded_secrets": [],
  "leakage_findings": [],
  "summary": {
    "scripts_analyzed": 1,
    "high_confidence_findings": 3,
    "inferred_findings": 1,
    "risk_overview": "发现认证接口、第三方日志上报地址和固定 token 构造线索"
  },
  "warnings": []
}
```

## 5.2 公共字段解释

| 字段名 | 类型 | 说明 | 约束 |
| --- | --- | --- | --- |
| `confidence` | enum | 结果可信度，建议 `low`、`medium`、`high`、`confirmed` | 证据弱时不得高于 `medium` |
| `evidence` | string | 用于支撑结论的关键片段摘要 | 应尽量保持简洁且可读 |
| `source_span` | object | 结果对应的源码位置，如 `start_line/end_line/start_col/end_col` | 无位置信息时应为空而非伪造 |
| `normalized_value` | string | 归一化后的值，如标准 URL、统一组件名、归并后的密钥类别 | 只在可归一化时填写 |
| `risk_level` | enum | 风险等级，建议 `info`、`low`、`medium`、`high` | 仅对明显安全相关结果使用 |
| `reason` | string | 该结果为何成立的简要解释 | 应与 evidence 对应 |
| `upstream_refs` | string[] | 上游事件、样本、证据引用 | 至少保留脚本或页面来源 |
| `inferred` | bool | 是否包含推断成分 | 只要有关键推断就必须为 `true` |

## 5.3 各类结果对象说明

### 5.3.1 `ComponentFinding`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| `finding_id` | string | 组件发现唯一标识 |
| `script_id` | string | 来源脚本 ID |
| `component_name` | string | 组件或模块名称 |
| `component_type` | string | 组件类型，如 `form`、`modal`、`list`、`admin_panel` |
| `business_role` | string | 业务角色，如登录、管理、支付、报表 |
| `related_routes` | string[] | 关联路由或页面路径 |
| `related_apis` | string[] | 关联接口 |
| `state_keys` | string[] | 关联状态字段 |
| `confidence` | enum | 组件识别置信度 |
| `evidence` | string | 关键源码片段摘要 |
| `source_span` | object | 源码范围 |
| `upstream_refs` | string[] | 上游引用 |

### 5.3.2 `ExternalUrlFinding`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| `finding_id` | string | 外联地址发现唯一标识 |
| `script_id` | string | 来源脚本 ID |
| `raw_value` | string | 原始提取值 |
| `normalized_value` | string | 标准化后的 URL 或路径 |
| `url_type` | string | `api`、`static`、`third_party_service`、`beacon`、`redirect` |
| `is_third_party` | bool | 是否第三方目标 |
| `related_component` | string | 关联组件或模块 |
| `related_api_purpose` | string | 地址用途说明 |
| `confidence` | enum | 识别置信度 |
| `evidence` | string | 关键片段摘要 |
| `source_span` | object | 源码范围 |
| `upstream_refs` | string[] | 上游引用 |

### 5.3.3 `CryptoBehaviorFinding`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| `finding_id` | string | 加密行为发现唯一标识 |
| `script_id` | string | 来源脚本 ID |
| `behavior_type` | string | 行为类型，如 `hashing`、`signing`、`aes_encrypt`、`token_building` |
| `algorithm_hint` | string | 可能的算法或实现提示 |
| `related_params` | string[] | 涉及的参数、字段或请求头 |
| `related_api` | string | 关联接口 |
| `risk_level` | enum | 风险等级 |
| `confidence` | enum | 置信度 |
| `reason` | string | 识别依据 |
| `evidence` | string | 关键片段 |
| `source_span` | object | 源码范围 |
| `upstream_refs` | string[] | 上游引用 |

### 5.3.4 `HardcodedSecretFinding`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| `finding_id` | string | 固定材料发现唯一标识 |
| `script_id` | string | 来源脚本 ID |
| `material_type` | string | 材料类型，如 `key`、`salt`、`iv`、`token_template`、`default_credential` |
| `raw_value` | string | 原始命中值，必要时可截断显示 |
| `normalized_value` | string | 脱敏或归一化值 |
| `is_possible_credential` | bool | 是否疑似凭据 |
| `related_usage` | string | 材料用途，如签名、加密、鉴权 |
| `risk_level` | enum | 风险等级 |
| `confidence` | enum | 置信度 |
| `evidence` | string | 关键证据片段 |
| `source_span` | object | 源码范围 |
| `upstream_refs` | string[] | 上游引用 |

### 5.3.5 `LeakageFinding`

| 字段名 | 类型 | 说明 |
| --- | --- | --- |
| `finding_id` | string | 泄露线索发现唯一标识 |
| `script_id` | string | 来源脚本 ID |
| `leak_type` | string | 泄露类型，如 `comment`、`debug_text`、`env_marker`、`internal_name` |
| `raw_text` | string | 原始文本 |
| `normalized_value` | string | 归类后的标准表达 |
| `sensitivity` | string | 敏感度，如 `low`、`medium`、`high` |
| `related_asset_hint` | string | 关联资产线索 |
| `review_recommended` | bool | 是否建议复核 |
| `confidence` | enum | 置信度 |
| `evidence` | string | 关键片段摘要 |
| `source_span` | object | 源码范围 |
| `upstream_refs` | string[] | 上游引用 |

## 5.4 输出约束

    为保证结构化结果可以直接进入下游模块，输出应满足以下约束：
    1. 证据必须可回溯，至少能回到 `script_id` 或 `upstream_refs`；
    2. 推断性结论必须标 `inferred=true`，不得伪装成确定结论；
    3. 没有证据的字段禁止硬填；
    4. 同类结果可以做聚合，但聚合后不能丢失原始证据引用；
    5. 风险等级只用于安全相关结果，普通组件识别不要滥打高风险；
    6. 对于超量结果，应优先保留高置信、高价值、高可解释性发现。

## 5.5 输出示例

### 5.5.1 普通业务脚本标准输出

```json
{
  "task_id": "task_20260414_0001",
  "target": "https://app.example.com/dashboard",
  "component_findings": [
    {
      "finding_id": "cmp_001",
      "script_id": "script_ext_001",
      "component_name": "UserProfilePanel",
      "component_type": "detail_panel",
      "business_role": "user_center",
      "related_routes": [
        "/dashboard",
        "/profile"
      ],
      "related_apis": [
        "/api/user/profile"
      ],
      "state_keys": [
        "userInfo",
        "profileLoading"
      ],
      "confidence": "high",
      "evidence": "function loadUser(){ return fetch(apiBase + '/user/profile') }",
      "source_span": {
        "start_line": 1,
        "end_line": 1,
        "start_col": 1,
        "end_col": 72
      },
      "upstream_refs": [
        "event://obs/1201",
        "evidence://html/8801"
      ],
      "inferred": false
    }
  ],
  "external_urls": [
    {
      "finding_id": "url_001",
      "script_id": "script_ext_001",
      "raw_value": "/api/user/profile",
      "normalized_value": "https://app.example.com/api/user/profile",
      "url_type": "api",
      "is_third_party": false,
      "related_component": "UserProfilePanel",
      "related_api_purpose": "获取用户资料",
      "confidence": "high",
      "evidence": "fetch(apiBase + '/user/profile')",
      "source_span": {
        "start_line": 1,
        "end_line": 1,
        "start_col": 43,
        "end_col": 71
      },
      "upstream_refs": [
        "event://obs/1201",
        "evidence://html/8801"
      ],
      "inferred": false
    }
  ],
  "crypto_findings": [],
  "hardcoded_secrets": [],
  "leakage_findings": [],
  "summary": {
    "scripts_analyzed": 1,
    "high_confidence_findings": 2,
    "inferred_findings": 0,
    "risk_overview": "识别出用户资料组件及其关联接口，未发现明显安全线索"
  },
  "warnings": []
}
```

### 5.5.2 含加密逻辑与敏感泄露的高价值输出

```json
{
  "task_id": "task_20260414_0003",
  "target": "https://m.example.com/",
  "component_findings": [
    {
      "finding_id": "cmp_010",
      "script_id": "script_bundle_010",
      "component_name": "OrderSubmitModule",
      "component_type": "action_module",
      "business_role": "order_submit",
      "related_routes": [
        "/order/submit"
      ],
      "related_apis": [
        "/api/order/create"
      ],
      "state_keys": [
        "orderToken",
        "deviceId"
      ],
      "confidence": "medium",
      "evidence": "命中订单提交路径、token 构造字段与 create 接口拼接逻辑",
      "source_span": null,
      "upstream_refs": [
        "event://obs/1408",
        "evidence://js/9102"
      ],
      "inferred": true
    }
  ],
  "external_urls": [
    {
      "finding_id": "url_010",
      "script_id": "script_bundle_010",
      "raw_value": "https://log.example.net",
      "normalized_value": "https://log.example.net/",
      "url_type": "beacon",
      "is_third_party": true,
      "related_component": "OrderSubmitModule",
      "related_api_purpose": "日志上报",
      "confidence": "high",
      "evidence": "字符串常量中出现固定日志上报地址",
      "source_span": null,
      "upstream_refs": [
        "event://obs/1408",
        "evidence://js/9102"
      ],
      "inferred": false
    }
  ],
  "crypto_findings": [
    {
      "finding_id": "cry_010",
      "script_id": "script_bundle_010",
      "behavior_type": "aes_encrypt",
      "algorithm_hint": "AES",
      "related_params": [
        "token",
        "deviceId",
        "timestamp"
      ],
      "related_api": "/api/order/create",
      "risk_level": "medium",
      "confidence": "medium",
      "reason": "命中 AES 关键字、加密前参数拼装和请求发送链路",
      "evidence": "字符串常量中包含 AES，且加密结果参与请求体构造",
      "source_span": null,
      "upstream_refs": [
        "event://obs/1408",
        "evidence://js/9102"
      ],
      "inferred": true
    }
  ],
  "hardcoded_secrets": [
    {
      "finding_id": "sec_010",
      "script_id": "script_bundle_010",
      "material_type": "token_template",
      "raw_value": "token",
      "normalized_value": "token::<template>",
      "is_possible_credential": true,
      "related_usage": "订单提交鉴权",
      "risk_level": "high",
      "confidence": "medium",
      "evidence": "固定 token 字段参与请求签名或加密逻辑",
      "source_span": null,
      "upstream_refs": [
        "event://obs/1408",
        "evidence://js/9102"
      ],
      "inferred": true
    }
  ],
  "leakage_findings": [
    {
      "finding_id": "leak_010",
      "script_id": "script_bundle_010",
      "leak_type": "env_marker",
      "raw_text": "pre-release",
      "normalized_value": "environment::pre-release",
      "sensitivity": "medium",
      "related_asset_hint": "预发布环境标识",
      "review_recommended": true,
      "confidence": "medium",
      "evidence": "命中环境说明字符串 pre-release",
      "source_span": null,
      "upstream_refs": [
        "event://obs/1408",
        "evidence://js/9102"
      ],
      "inferred": false
    }
  ],
  "summary": {
    "scripts_analyzed": 1,
    "high_confidence_findings": 1,
    "inferred_findings": 3,
    "risk_overview": "发现订单提交相关模块、第三方日志上报地址、疑似 AES 加密行为和固定 token 模板"
  },
  "warnings": [
    "脚本为压缩 bundle，部分结论带有推断成分",
    "源码位置范围缺失，建议保留原始样本供后续复核"
  ]
}
```

# 六、组件信息提取
    组件信息提取在工程上应被设计为“从脚本中恢复页面功能单元”的子能力，而不是单纯的框架识别过程。它的核心任务，是把脚本中零散的组件名、状态字段、事件函数、路由片段和接口调用线索组织成可被下游消费的 `ComponentFinding`。

## 6.1 输入前提与依赖

    组件提取至少依赖以下输入前提：
    1. 脚本文本可被正常读取；
    2. 存在基本的函数、对象、字符串或模块结构，不是完全不可读的二进制内容；
    3. 最好具备页面 URL、脚本来源和上游页面引用，便于做语义补足；
    4. 若语言为 TS/JSX/TSX，应保留语言标记，以便识别组件定义和页面模块语义。

    该子能力不依赖运行时 DOM，也不要求真实渲染结果，因此即使页面无法加载，只要脚本内容可得，仍可开展静态组件提取。

## 6.2 提取流程

    建议将组件信息提取拆成六步：
    1. 预处理：对脚本做切块、去除无关噪声、保留核心结构片段；
    2. 候选定位：识别函数名、模块名、导出对象、路由段、状态字段、事件处理器等候选组件线索；
    3. 类型归类：把候选线索归为表单、列表、详情、管理面板、上传模块、报表模块等结构性类型；
    4. 业务角色判断：结合命名、接口路径、状态字段和页面路径，推断其业务角色；
    5. 关联构建：建立组件与接口、路由、状态字段之间的静态关联；
    6. 去重归并：对同名、近义、上下游重复组件线索做归并，输出稳定结果。

## 6.3 组件分类策略

    组件分类不应只依赖名字，而应综合以下信号：
    1. 结构信号：是否出现表单提交、列表渲染、详情加载、弹窗控制等典型逻辑；
    2. 语义信号：命名中是否出现 `login`、`admin`、`config`、`report`、`upload` 等业务关键词；
    3. 路由信号：是否绑定 `/login`、`/admin`、`/user/profile` 等路径；
    4. 接口信号：是否调用特定接口，如认证、配置、审批、导出、订单等；
    5. 状态信号：是否维护 `token`、`userInfo`、`permissions`、`tableData`、`formData` 等状态字段。

    当多个信号同时命中时，可以输出更高置信度的 `business_role`；当只命中弱命名信号时，最多输出低到中置信结果。

## 6.4 `ComponentFinding` 结果对象设计

| 字段名 | 必填 | 说明 | 约束 |
| --- | --- | --- | --- |
| `finding_id` | 是 | 组件结果唯一标识 | 同一任务内唯一 |
| `script_id` | 是 | 来源脚本 | 必须能回指输入脚本 |
| `component_name` | 是 | 组件、模块或逻辑单元名称 | 无明确名称时可用稳定摘要名 |
| `component_type` | 是 | 结构性类型 | 建议使用预定义类别 |
| `business_role` | 否 | 业务角色判断 | 无法确认时允许为空 |
| `related_routes` | 否 | 相关路由或页面路径 | 允许多值 |
| `related_apis` | 否 | 关联接口地址 | 应优先引用标准化结果 |
| `state_keys` | 否 | 状态字段 | 仅保留高价值字段 |
| `confidence` | 是 | 识别置信度 | 证据不足时不得过高 |
| `evidence` | 是 | 关键依据摘要 | 应能解释结论来源 |
| `source_span` | 否 | 源码位置范围 | 缺失时为空 |
| `upstream_refs` | 否 | 来源引用 | 应至少包含脚本来源 |
| `inferred` | 是 | 是否带推断 | 默认 `false` |

## 6.5 关键参数与解释

| 参数名 | 默认值 | 说明 | 使用建议 |
| --- | --- | --- | --- |
| `component_detection_mode` | `balanced` | 组件识别模式，可取 `strict`、`balanced`、`broad` | 默认 `balanced`，仅在追求召回时用 `broad` |
| `component_name_confidence_threshold` | `0.65` | 组件名保留阈值，低于该阈值的名称不直接输出 | 命名混乱的 bundle 中可适度降低 |
| `state_binding_scan` | `true` | 是否扫描状态字段与组件绑定关系 | 组件语义分析建议开启 |
| `route_association_scan` | `true` | 是否建立组件与路由的静态关联 | 页面级脚本建议开启 |
| `api_binding_scan` | `true` | 是否建立组件与接口的静态关联 | 用于提升业务角色判断质量 |
| `max_component_nodes` | `100` | 单脚本最大组件候选数 | 超过后应按置信度截断 |

    这些参数的核心作用，是平衡“召回率”和“结果噪声”。例如：
    1. `component_detection_mode=broad` 更容易找出弱候选，但也更容易把普通工具函数误判为组件；
    2. `component_name_confidence_threshold` 过低会让命名不稳定的结果大量进入下游；
    3. `state_binding_scan`、`route_association_scan`、`api_binding_scan` 同时开启时，结果更完整，但也更消耗预算。

## 6.6 失败与降级

    组件提取应允许以下降级路径：
    1. 只保留高置信组件，不强行补齐完整组件树；
    2. 无法识别组件名称时，退化为“页面模块线索”而不是输出错误名称；
    3. 路由和接口无法稳定关联时，可只输出 `component_type` 和 `business_role`；
    4. 对严重混淆脚本，应返回 `warnings` 并提示“组件关系未完全恢复”。

    明确边界如下：
    1. 不做运行时渲染还原；
    2. 不承诺真实组件树百分之百重建；
    3. 不在文档层细化到框架专属 AST 节点规则；
    4. 不因名称包含 `admin` 就直接判定为高价值管理组件。

## 6.7 测试点与验收要点

    组件提取部分至少应满足以下验收：
    1. 能从常规业务脚本中识别出组件、关联接口和页面路径；
    2. 对内联脚本、模块脚本、压缩 bundle 均有稳定退化路径；
    3. 弱命名信号不会直接生成高置信业务角色；
    4. 结果能回指 `script_id` 和证据片段。

# 七、外联URL信息提取
    外联URL信息提取在工程上要解决的，不只是“把字符串里的 URL 抠出来”，而是对脚本中出现的接口、资源、第三方服务和跳转目标做可消费的归类、标准化和关联组织，使其既能服务图谱构建，也能支持价值评估和安全分析。

## 7.1 URL 来源分类

    外联目标的来源通常分为以下几类：
    1. 直接字面量：脚本中显式出现的完整 URL、相对路径、接口路径；
    2. 模板拼接：通过变量、模板字符串、路径片段拼接出的目标；
    3. 配置项引用：从 `baseUrl`、`apiHost`、`cdnHost` 等配置变量衍生的目标；
    4. SDK 封装：埋点、监控、客服、地图、短信等第三方 SDK 的初始化参数；
    5. 跳转逻辑：`location.href`、路由跳转、单点登录跳转等目标。

## 7.2 提取与拼接恢复流程

    建议将外联提取流程拆成五步：
    1. 直接命中：先提取显式 URL 和路径常量；
    2. 上下文恢复：结合 `page_url`、`source_url`、`referer` 恢复相对路径；
    3. 拼接归并：对模板字符串、前缀变量和路径片段做有限恢复；
    4. 用途分类：判断其属于接口、静态资源、第三方服务、埋点还是跳转；
    5. 关联挂接：将地址与组件、接口用途、页面模块建立关联。

    这里的“恢复”只应是保守恢复：能确认的就标准化，无法确认的就保留原样并降低置信度，不应强行把不完整字符串拼成确定 URL。

## 7.3 标准化规则层级

    外联地址标准化建议分三层：
    1. 原始层：保留脚本中原始值，便于回溯；
    2. 归一层：做相对路径补全、域名小写、默认路径补全等基础标准化；
    3. 分类层：补充 `url_type`、`is_third_party`、`related_api_purpose` 等业务语义标签。

    在标准化过程中应遵循两个原则：
    1. 不改变原始证据；
    2. 不对无法确认的主机名、协议或路径做强假设。

## 7.4 风险标签与业务标签

    外联 URL 除了基础类型，还应允许加上业务与风险标签。例如：
    1. `auth_api`、`payment_api`、`upload_api` 等业务标签；
    2. `third_party_beacon`、`external_storage`、`cross_origin_auth` 等依赖标签；
    3. 对明显敏感的目标，可以附带 `risk_level=low/medium/high`，但默认应保守。

## 7.5 `ExternalUrlFinding` 结果对象设计

| 字段名 | 必填 | 说明 | 约束 |
| --- | --- | --- | --- |
| `finding_id` | 是 | 唯一标识 | 同任务内唯一 |
| `script_id` | 是 | 来源脚本 | 必须可追踪 |
| `raw_value` | 是 | 原始命中值 | 不得被覆盖 |
| `normalized_value` | 否 | 标准化后的地址 | 无法确认时可为空 |
| `url_type` | 是 | 地址类型 | 建议枚举化 |
| `is_third_party` | 是 | 是否第三方 | 无法确认时按保守策略 |
| `related_component` | 否 | 关联组件 | 可为空 |
| `related_api_purpose` | 否 | 地址用途说明 | 仅在可解释时填写 |
| `confidence` | 是 | 置信度 | 模板恢复结果通常不宜过高 |
| `evidence` | 是 | 关键片段摘要 | 需支撑归类结论 |
| `source_span` | 否 | 源码范围 | 缺失时为空 |
| `upstream_refs` | 否 | 来源引用 | 用于审计 |
| `inferred` | 是 | 是否带推断 | 默认 `false` |

## 7.6 关键参数与解释

| 参数名 | 默认值 | 说明 | 使用建议 |
| --- | --- | --- | --- |
| `url_extraction_mode` | `balanced` | URL 提取模式，可取 `strict`、`balanced`、`aggressive` | 默认 `balanced` |
| `allow_relative_url_resolution` | `true` | 是否允许相对路径补全 | 页面级输入建议开启 |
| `base_url_required` | `false` | 是否要求必须有基准 URL 才做标准化 | 仅在高准确率场景开启 |
| `third_party_classification_mode` | `conservative` | 第三方归属判断风格 | 默认保守，避免误判同集团域名 |
| `track_beacon_detection` | `true` | 是否识别埋点、监控、日志上报目标 | 安全分析与资产图谱建议开启 |
| `max_urls_per_script` | `200` | 单脚本最大保留 URL 数 | 超限后按价值与置信度截断 |

    参数使用上应注意：
    1. `url_extraction_mode=aggressive` 会提高召回，但也更容易把普通字符串误判成 URL；
    2. `allow_relative_url_resolution=true` 时必须优先依赖 `page_url` 或 `source_url`，不能凭空补主机；
    3. `third_party_classification_mode` 不宜过激，否则容易把 CDN、SSO、集团共享域误归成外部第三方；
    4. `max_urls_per_script` 应受下游消费能力约束，不建议无限放大。

## 7.7 特殊场景处理约束

    文档层面应提前固定以下特殊场景的处理原则：
    1. 相对路径：优先基于 `page_url` 或 `source_url` 补全，缺失时保留原始路径；
    2. 模板字符串：只恢复可确定部分，变量位保留为占位表达；
    3. 配置拼接：若命中 `apiHost + '/user'` 这类结构，可输出“标准化候选值”并标 `inferred=true`；
    4. 埋点地址：应识别为 `beacon` 或 `third_party_service`，不要误标成业务 API；
    5. 跳转地址：应与接口请求分开建模，避免图谱关系混淆。

## 7.8 测试点与验收要点

    外联 URL 提取至少应满足以下验收：
    1. 能区分 API、静态资源、第三方服务、埋点和跳转目标；
    2. 对相对路径、模板拼接、配置前缀具备保守恢复能力；
    3. 不会把任意长字符串或日志文本误提为大量 URL；
    4. 输出结果包含原始值、标准化值、分类和证据。

# 八、加密算法识别与固定密钥提取
    这一部分在工程上应分成两条并行子能力：一条识别“安全处理行为”，一条识别“固定安全材料”。前者回答“脚本在做什么”，后者回答“脚本里写死了什么”。两者既有关联，又不能混为同一类结果。

## 8.1 识别目标范围

    当前阶段建议把识别目标控制在以下范围：
    1. 加密、解密、摘要、签名、编码、令牌构造等安全相关行为；
    2. 关键材料，如密钥、盐值、IV、固定 token、默认凭据、密文模板；
    3. 请求参数、请求头、设备标识、会话字段等与安全处理直接相关的上下文。

    不在本阶段直接覆盖：
    1. 破解算法；
    2. 逆向还原完整调用协议；
    3. 生成可利用攻击链；
    4. 直接确认漏洞存在。

## 8.2 常见信号来源

    识别安全逻辑时，常见信号包括：
    1. 关键字与库名，如 `AES`、`RSA`、`MD5`、`SHA256`、`CryptoJS`；
    2. 典型行为，如参数拼接后摘要、头部签名构造、请求体加密；
    3. 常量材料，如固定 key、iv、salt、secret、token 模板；
    4. 上下文信号，如登录、支付、订单、鉴权、签名参数名。

    这些信号只有在相互印证时，才应输出高置信风险结论。单独命中一个关键词时，最多作为弱线索保留。

## 8.3 结果分类

    建议将结果拆成两类：
    1. `CryptoBehaviorFinding`：描述存在什么安全处理行为，重点放在“行为与用途”；
    2. `HardcodedSecretFinding`：描述存在什么固定材料，重点放在“材料与风险”。

    这种拆分的价值在于：
    1. 便于下游把“接口加密逻辑”与“密钥泄露”区分处理；
    2. 便于风险分层，避免把普通编码行为直接当作高危材料泄露；
    3. 便于后续复核时分别追踪行为证据和材料证据。

## 8.4 风险判断边界

    风险判断建议遵循以下边界：
    1. 仅命中 Base64、URL 编码等简单编码行为时，不得直接视为加密风险；
    2. 仅出现 `token`、`secret` 这类命名，不足以直接判为高危固定凭据；
    3. 同时命中固定材料、使用场景和请求链路时，才适合给出中高风险；
    4. 无法确认材料是否真实可用时，应标记 `is_possible_credential=true` 而不是直接确认。

## 8.5 结果对象设计

### 8.5.1 `CryptoBehaviorFinding`

| 字段名 | 必填 | 说明 | 约束 |
| --- | --- | --- | --- |
| `finding_id` | 是 | 唯一标识 | 同任务内唯一 |
| `script_id` | 是 | 来源脚本 | 必须可追溯 |
| `behavior_type` | 是 | 行为类型 | 使用预定义分类 |
| `algorithm_hint` | 否 | 算法提示 | 无法确认时可为空 |
| `related_params` | 否 | 涉及参数 | 保留核心参数即可 |
| `related_api` | 否 | 关联接口 | 可为空 |
| `risk_level` | 否 | 风险等级 | 默认为保守级别 |
| `confidence` | 是 | 置信度 | 依据多信号综合判断 |
| `reason` | 是 | 识别依据 | 需与 evidence 对应 |
| `evidence` | 是 | 关键片段 | 应可读 |
| `source_span` | 否 | 源码范围 | 缺失时为空 |
| `upstream_refs` | 否 | 来源引用 | 用于审计 |
| `inferred` | 是 | 是否带推断 | 默认 `false` |

### 8.5.2 `HardcodedSecretFinding`

| 字段名 | 必填 | 说明 | 约束 |
| --- | --- | --- | --- |
| `finding_id` | 是 | 唯一标识 | 同任务内唯一 |
| `script_id` | 是 | 来源脚本 | 必须可追踪 |
| `material_type` | 是 | 材料类型 | 建议枚举化 |
| `raw_value` | 是 | 原始值 | 允许脱敏展示 |
| `normalized_value` | 否 | 归一化值 | 可用于脱敏或归并 |
| `is_possible_credential` | 是 | 是否疑似凭据 | 默认 `false` |
| `related_usage` | 否 | 材料用途 | 无法判断可为空 |
| `risk_level` | 否 | 风险等级 | 仅在证据足够时提高 |
| `confidence` | 是 | 置信度 | 受证据和上下文影响 |
| `evidence` | 是 | 关键片段 | 用于支撑结果 |
| `source_span` | 否 | 源码范围 | 可为空 |
| `upstream_refs` | 否 | 来源引用 | 用于审计 |
| `inferred` | 是 | 是否带推断 | 默认 `false` |

## 8.6 关键参数与解释

| 参数名 | 默认值 | 说明 | 使用建议 |
| --- | --- | --- | --- |
| `crypto_detection_depth` | `balanced` | 安全行为识别深度，可取 `basic`、`balanced`、`deep` | 默认 `balanced` |
| `constant_secret_scan` | `true` | 是否扫描固定材料 | 安全场景建议开启 |
| `signature_pattern_scan` | `true` | 是否识别签名、摘要、请求鉴权模式 | 分析接口鉴权时建议开启 |
| `decode_chain_limit` | `2` | 允许尝试还原的简单解码层数 | 默认保守，避免过度推演 |
| `secret_length_threshold` | `8` | 固定材料最小长度阈值 | 过低会引入大量普通字符串噪声 |
| `hardcoded_material_confidence_threshold` | `0.7` | 固定材料进入结果的置信度阈值 | 高噪声 bundle 中可适当提高 |

    参数选择应遵循以下原则：
    1. `crypto_detection_depth=deep` 只适合预算充足、且确实需要深入接口安全逻辑的场景；
    2. `decode_chain_limit` 不宜过高，否则容易把一般编码还原过程误当作安全突破；
    3. `secret_length_threshold` 是重要的降噪开关，过低会把普通字段值大量吸入结果；
    4. `hardcoded_material_confidence_threshold` 过低会使“疑似值”淹没真正高价值材料。

## 8.7 降级与误判控制

    为控制误报，应固定以下策略：
    1. 单一关键字命中只作为弱线索，不直接输出高风险；
    2. 算法库名称与请求链路未形成关联时，只输出行为提示，不输出材料结论；
    3. 固定字符串若未出现在鉴权、签名、加密、登录、支付等上下文中，应降低风险等级；
    4. 无法确认材料是否真实可用时，优先标记“疑似可复用安全材料”。

    边界要求如下：
    1. 不设计破解流程；
    2. 不给出利用建议；
    3. 不把简单编码等同于加密风险；
    4. 不把调试 token 示例直接认定为真实生产凭据。

## 8.8 测试点与验收要点

    加密与固定材料识别至少应满足以下验收：
    1. 能区分行为发现和材料发现两类结果；
    2. 对简单编码、普通常量和高风险固定材料有基本区分能力；
    3. 对订单、登录、鉴权等上下文中的安全逻辑能给出解释性输出；
    4. 高风险结果必须带证据和风险原因。

# 九、注释等泄露信息提取
    注释等泄露信息提取在工程上要解决的问题，是从脚本中的非主流程文本里恢复“研发阶段留下的高信息量线索”。这类线索经常不能直接定性为漏洞，但对理解系统边界、环境差异、接口用途和内部资产结构非常有价值。

## 9.1 泄露信息类型划分

    建议将泄露信息分为以下几类：
    1. `comment`：显式注释、块注释、行注释；
    2. `debug_text`：调试日志、临时打印、异常提示语；
    3. `env_marker`：测试、预发、灰度、内部环境标识；
    4. `internal_name`：内部系统代号、模块简称、服务名；
    5. `api_note`：接口说明、参数说明、 TODO/FIXME 类备注；
    6. `path_hint`：路径注释、隐藏页面提示、静态资源目录说明。

## 9.2 文本线索来源

    文本线索的来源不应只盯住注释本身，还应覆盖：
    1. 字符串常量；
    2. 报错提示与日志语句；
    3. 调试分支中的说明文本；
    4. 变量命名和对象字段命名中的内部标识；
    5. 未启用代码块中的历史说明。

## 9.3 提取与归类策略

    泄露信息提取建议遵循以下流程：
    1. 先提取显式注释与明显调试文本；
    2. 再识别环境标识、内部命名和路径说明；
    3. 对命中内容做归类，而不是直接输出原始杂乱文本；
    4. 根据文本内容和上下文做敏感度分层；
    5. 对可能关联内部资产、隐藏接口、预发布环境的结果增加复核建议。

    这里的关键是“归类优先于堆积”。如果只是把所有注释原样导出，结果会过于嘈杂，无法被图谱、评估和报告模块直接利用。

## 9.4 敏感度分层

    泄露信息的敏感度建议采用三层：
    1. `low`：一般功能说明、普通注释、非敏感调试文本；
    2. `medium`：内部命名、环境差异、隐藏路径提示、接口用途说明；
    3. `high`：暴露内部域名、环境凭据线索、未公开路径、敏感接口用途或高价值系统代号。

    需要强调的是，敏感度不等于漏洞严重度。这里的分层只表达“该文本线索对后续分析有多大价值或风险提示”，不直接替代漏洞分级。

## 9.5 `LeakageFinding` 结果对象设计

| 字段名 | 必填 | 说明 | 约束 |
| --- | --- | --- | --- |
| `finding_id` | 是 | 唯一标识 | 同任务内唯一 |
| `script_id` | 是 | 来源脚本 | 必须可追踪 |
| `leak_type` | 是 | 泄露类型 | 建议使用预定义分类 |
| `raw_text` | 是 | 原始文本 | 保留关键上下文 |
| `normalized_value` | 否 | 归类后的标准表达 | 无法归类可为空 |
| `sensitivity` | 是 | 敏感度 | 只表达线索价值，不等于漏洞等级 |
| `related_asset_hint` | 否 | 关联资产或环境提示 | 用于图谱和复核 |
| `review_recommended` | 是 | 是否建议复核 | 默认 `false` |
| `confidence` | 是 | 置信度 | 取决于文本清晰度与上下文 |
| `evidence` | 是 | 关键片段摘要 | 应支撑归类 |
| `source_span` | 否 | 源码范围 | 无位置信息可为空 |
| `upstream_refs` | 否 | 来源引用 | 用于审计 |
| `inferred` | 是 | 是否带推断 | 默认 `false` |

## 9.6 关键参数与解释

| 参数名 | 默认值 | 说明 | 使用建议 |
| --- | --- | --- | --- |
| `comment_scan_enabled` | `true` | 是否扫描显式注释 | 建议默认开启 |
| `debug_string_scan_enabled` | `true` | 是否扫描调试文本、日志提示 | 排查研发残留时建议开启 |
| `env_marker_scan` | `true` | 是否识别测试、预发、灰度等环境标识 | 资产边界分析建议开启 |
| `internal_name_scan` | `true` | 是否识别内部系统名、模块代号、服务简称 | 图谱与价值评估建议开启 |
| `sensitive_keyword_level` | `balanced` | 敏感关键词匹配强度，可取 `strict`、`balanced`、`broad` | 默认 `balanced` |
| `max_leakage_findings` | `100` | 单任务最大保留泄露线索数 | 超限后按敏感度和置信度截断 |

    参数使用原则如下：
    1. `sensitive_keyword_level=broad` 会明显提升召回，但也会引入大量普通文本噪声；
    2. `debug_string_scan_enabled` 在大型 bundle 中可能带来大量低价值日志文本，需要配合 `max_leakage_findings`；
    3. `env_marker_scan` 和 `internal_name_scan` 对图谱和价值评估帮助较大，建议默认开启；
    4. 即使所有扫描都开启，结果仍应先归类再输出，不宜直接把原始文本堆满。

## 9.7 误报控制与边界

    泄露信息提取最容易出现的问题，是把普通注释、正常报错文案或框架内置文本误当成高价值线索。为此建议固定以下控制规则：
    1. 只有同时出现上下文支持时，才把普通路径说明升级为高价值路径线索；
    2. 通用框架报错文案不应直接输出为业务泄露；
    3. 不因出现 `test`、`dev`、`debug` 单词就直接认定为有效环境线索；
    4. 内部命名若无法关联业务或资产，应保留为中低敏感度；
    5. 不把文本线索直接升级为“已确认敏感信息”或“已确认漏洞”。

## 9.8 测试点与验收要点

    泄露信息提取至少应满足以下验收：
    1. 能区分注释、调试文本、环境标识、内部命名等不同类型；
    2. 能对文本线索给出归类、敏感度和复核建议；
    3. 不会把大量普通报错或框架文本误判为高价值泄露；
    4. 结果能够回指脚本来源和证据片段。
