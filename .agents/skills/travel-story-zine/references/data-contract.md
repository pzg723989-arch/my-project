# 数据与状态约定 v1

所有 ID 在记录创建后保持稳定，不使用数组下标作 ID。JSON 使用 UTF-8；缺失信息用 null 或空数组表达，不使用虚构占位照片。所有路径必须指向用户授权的实际素材，禁止在输出中嵌入凭据、带签名访问令牌或不必要的精确 GPS。

## 每日记录

必需字段：

| 字段 | 约定 |
| --- | --- |
| schema_version | 固定为 "1.0" |
| day_id / trip_id | 稳定字符串 |
| revision / base_revision | 当前与修改所依据版本；初始 1 / null |
| date / timezone | YYYY-MM-DD；时区未知用 null |
| locations | 用户确认的地点，按行程顺序 |
| itinerary / memory | 用户原文；memory 可为 null |
| facts | {id, text, source, photo_id}；source 为 user 或 visible_photo，photo_id 可 null |
| photos | 照片记录数组，见下 |
| selected_photos / hero_photo_id | 入选 ID 数组及主图 ID；没有照片时主图 null |
| copy | {mode, title, blocks}；mode 为 record、light_narrative、minimal |
| spreads | 稳定跨页 ID 数组所对应的页面对象 |
| status | draft 或 confirmed |
| pending_questions / assumptions | 字符串数组 |
| checks | {name, status, detail} 数组；status 为 passed、failed、not_run |
| persistence | pending 或 saved；只有实际写入成功才用 saved |

照片记录：
- id、source_ref（实际可访问的本地路径或资产 ID）。
- captured_at、timezone、width_px、height_px：无实测证据填 null。
- ordering_basis：exif、user、upload；不能把 upload 当拍摄时间。
- category：human、nature、pending；tags 为辅助标签数组。
- recommendation：selected、alternative、not_recommended；reason 为选择依据。
- duplicate_group_id 可 null；must_keep、hidden 为布尔值。
- observations：{id, text} 数组，只记实际可见内容。
- analysis_status：reviewed 或 unavailable；quality_notes 为字符串数组。

文案 title 为 {text, source_ids}，blocks 每项为 {id, kind, text, source_ids}，kind 为 opening、story、caption。引用 facts.id 或 photo.observations.id，不允许悬空引用。纯日期和地点标签不算正文；开篇和 story 共计默认 2—4 块。

跨页对象：id、template、background、body_block_ids、pages。pages 必须含 left 和 right 两页，每页为 {side, placements}。placement 为 {photo_id, box, fit, crop, subject_safe_checked}：
- box 使用相对于单页的归一化 [x,y,w,h]，0≤x,y，w,h>0，x+w≤1，y+h≤1。
- fit 为 contain 或 cover；contain 的 crop 为 null，完整保留画面。
- cover 的 crop 是原图归一化 [x,y,w,h]；未知主体时不自动采用 cover。
- subject_safe_checked 仅在实际检查后为 true，否则 false，并将裁切检查记为 not_run。
- body_block_ids 指向 copy.blocks；标题和地点标签可由渲染器的固定标题区域呈现。正文须由模板在图片之外预留区域，不自动压到照片上；溢出即检查失败。

## 旅行 manifest

包含 schema_version、trip_id、revision、title、date_range、cover、days、spread_order、export_settings。

- days：{day_id, date, confirmed_revision, record_ref} 数组。record_ref 指向实际保存记录；未保存时为 null，并报告待保存。
- spread_order：稳定 spread ID 的有序数组，是已确认翻阅顺序的权威来源。
- cover：{photo_id, title}，主图未知时 photo_id=null。
- export_settings：{page_width_mm, page_height_mm, bleed_mm, target_dpi, profile}；未知用 null。profile 为 digital 或 print。
- date_range 为 {start, end}，仅用真实日期。

追加的前置条件：当天 confirmed、所有照片与文字引用有效、必留无遗漏、已保存版本检查无冲突。同一 day_id 与 confirmed_revision 的重复追加无效果。修改 confirmed_revision 只在新草稿确认后执行。对已确认内容的任何改变均可追溯到用户指令，不自动接受 AI 重新生成。

manifest 的目录按日期展示与物理 spread_order 分离；插入旧日期的正式页面须用户确认。页码是派生值，封面和目录的页数变化不能改变照片、裁切及已确认文字。
