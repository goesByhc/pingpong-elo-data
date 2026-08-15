2026 乒超联赛（中国乒乓球俱乐部超级联赛）常规赛 原始采集数据

采集时间：2026-08-15（常规赛 7-23 ~ 8-1 结束后不久）
状态：部分覆盖（约 36 盘 / 9 场团体赛有逐盘数据，另有若干场仅团队比分）

来源：
- sina.cn/news/detail/（微博"草莓牛奶特别甜"等账号逐场发布帖，含完整逐盘+局分）
- k.sina.com.cn 文章（体育先锋眼等作者的分析文，含部分逐盘数据）
- 360kuai 文章 meta Description（前 2 盘局分）
- 网易/腾讯新闻

文件说明：
- articles_fetched.json     首批 20 个链接抓取结果
- sina_posts.json           种子页提取的 2026 乒超赛果帖（15 条）
- parsed_matches.json       帖解析出的场次（含重复，需按 队+日期 去重）
- extracted_strict.json     严格过滤后的场次（球队名单+胜方8分）
- ksina_articles_review.json k.sina 文章比分摘要
- *_sina_post*.html         原始帖页面存档

编译后的标准 CSV：data/sources/cttsl_2026.csv（36 盘，已验证）
