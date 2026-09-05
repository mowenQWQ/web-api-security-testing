---
name: web-api-security-testing
description: "Reusable round-based workflow for Web API security testing on authorized targets — pre-flight checks (token validity, JS change detection, findings ledger), in-flight techniques (JS-first analysis, idempotency verification, backgrounded quota tests), fixed post-round closure (report, archive, ledger update). Use when the user starts a testing round, asks to retest known vulnerabilities, or says "start today's testing". 关键词：API安全测试、测试轮次、漏洞复查、测试报告。Keywords: API security testing, test rounds, vulnerability retest, JS chunk analysis, idempotency verification"
version: "1.0.0"
---

# Web API 安全测试轮次执行流程

> 前置：授权确认 + TOOLS.md 中的站点专属信息（token/密码/端点清单/漏洞账本）

## 一、测试前（3件事）

1. **Token验证**：`GET /user/checkLoginStatus`（或等价端点）确认认证有效
2. **JS变更检测**：提取首页主JS文件名，与上次记录对比；变更则重新分析chunk
3. **读排期**：TOOLS.md 漏洞账本中到期的复查项（7天→1月→半年→1年）+ 上次报告"后续方向"

## 二、测试中（按手册选手段）

- **手段来源**：`reports/security/安全测试手段手册.md`（已验证手段）+ `reports/security/未使用手段清单.md`（待用手段池）
- **JS先读后测**：参数定义、渲染方式、端点全集都在JS chunk里，比盲测高效10倍
- **开源项目可拉源码**：GitHub raw 拉服务端源码对照，逻辑分支（如 if(!uid) 的额度豁免）一眼看穿，比黑盒快10倍
- **行为学验证额度类修复**：无法直击代码时用精确计数封口——带uid发2条+无uid发28条=30条封口，第31条429；若uid不占额度，第29条无uid请求本应入库
- **网络捕获验证幂等**：Playwright `page.on('response')` 抓请求/响应body，直接证明 `already:true` 静默跳过，比看最终列表靠谱
- **长测试后台化**：打满额度类测试要跑几分钟，Bash session会超时——脚本写日志用 nohup 后台跑 + sleep轮询日志
- **指纹/时区模拟**：Playwright context 级 `timezone_id`/`user_agent` 参数可模拟跨时区/跨设备；同浏览器不同context指纹相同（canvas/UA等一致→同seed→同结果），要变卦必须改UA
- **铁律**：
  - 危险操作（删改他人数据）前先GET完整内容存档
  - 副作用最小化：最小金额、成对关注、测试帖即测即删
  - 同款漏洞模式立即sweep全站端点
  - 新模块先问用户"这功能上线了吗"（前端没入口≠后端关闭）
  - sweep必须读body业务码，不能只看HTTP状态码
  - 白名单校验会干扰漏洞验证：编造的卦名/等级被400挡住，先用正则从页面JS提取合法值域再测

## 三、测试后（5件事，顺序固定）

1. **报告存档**：`reports/security/{站点}/YYYY-MM-DD_HHMM_roundNN_findings.md`（含修改操作说明章节）
2. **立即发送报告**（send_file），不问"要不要"
3. **更新手段手册+清单**：新手段/新知识补进对应章节
4. **更新 TOOLS.md 漏洞账本**：修复状态/新漏洞/复查排期
5. **日记补记**（凌晨归前一天规则）

## 踩坑记录

- **报告不主动发会被批评**：测完只汇报不发包，用户两次追问"报告呢"后确立铁律
- **业务定性需要用户输入**：某内容模块"前端未开放"的事实让 LOW 升 MEDIUM——技术测不出来业务上下文
- **活动API字段名**：某活动接口字段是 total_amount/total_count 不是 amount/count，猜错字段报默认值错误（可借此枚举字段名）
- **HTTP 200陷阱**：某业务站把业务层 403/404 包在 HTTP 200 里返回，管理端 sweep 的 4 个"200"全是 403
- **频率窗口计数**：400 响应也计入频率限制（发帖端点实测），等待要留足余量
- **意外即测试**：网关504导致脚本重复提交相同数据，反成幂等机制的天然验证——事故数据先别清，看能不能当证据
- **异常数据先对账再定性**：入库数与预期不符时，先拉服务端全量记录对时间戳/批次，可能发现了别的机制在工作（如幂等拦截）
- **「可信凭证」免费铸造=免检通道**：uid/票据类凭证若可无成本获取（如免费签发端点），围绕它的配额校验形同虚设——配额检查要对所有路径统一生效
