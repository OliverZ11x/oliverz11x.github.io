---
date created: 2025/5/26 13:41
date modified: 2025/5/26 13:43
---

推特接口

https://rapidapi.com/alexanderxbx/api/twitter-api45/

用户信息

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/screenname.php?screenname=elonmusk&rest_id=44196397' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

用户时间线

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/timeline.php?screenname=elonmusk' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

以后

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/following.php?screenname=elonmusk' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

追随者

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/followers.php?screenname=elonmusk&blue_verified=0' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

推文信息

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/tweet.php?id=1671370010743263233' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

隶属关系

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/affilates.php?screenname=x' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

用户媒体

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/usermedia.php?screenname=elonmusk' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

转推

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/retweets.php?id=1700199139470942473' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

趋势

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/trends.php?country=UnitedStates' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

搜索

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/search.php?query=cybertruck&search_type=Top' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

检查转推

curl --request GET \ --url https://twitter-api45.p.rapidapi.com/checkretweet.php \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

用户回复

curl --request GET \ --url https://twitter-api45.p.rapidapi.com/replies.php \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

推文线程

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/tweet_thread.php?id=1738106896777699464' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

最新回复

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/latest_replies.php?id=1738106896777699464' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

检查关注

curl --request GET \ --url https://twitter-api45.p.rapidapi.com/checkfollow.php \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

列表时间线

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/listtimeline.php?list_id=1343798673386434560' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

社区帖子 搜索 最新

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/search_communities_latest.php?query=superman' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

社区帖子搜索顶部

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/search_communities_top.php?query=superman' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

社区搜索

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/search_communities.php?query=superman' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

社区帖子

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/community_timeline.php?community_id=1783990533192651232' \ --header 'Content-Type: application/json' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

列出关注则

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/list_followers.php?list_id=1177128103228989440' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

列出成员

curl --request GET \ --url 'https://twitter-api45.p.rapidapi.com/list_members.php?list_id=1177128103228989440' \ --header 'x-rapidapi-host: twitter-api45.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

---

[GitHub - Seym0n/tiktok-mcp: Model Context Protocol (MCP) with TikTok integration](https://github.com/Seym0n/tiktok-mcp)

[TikTok MCP | TikNeuron](https://tikneuron.com/tools/tiktok-mcp)

###### **==社交媒体-tiktok==**

### tiktok_available_subtitles

**描述：**
查找可用的字幕，即 TikTok 视频的内容。这用于查找 TikTok 视频是否有任何可用的内容（字幕）。返回视频的可用字幕，该字幕可以采用不同的语言和不同的格式，如自动语音识别、机器翻译或创作者字幕以及不同的语言。

**输入参数：**

- `tiktok_url`（必填）：TikTok 视频 URL，例如 [https://www.tiktok.com/@username/video/1234567890](https://www.tiktok.com/@username/video/1234567890) 或 [https://vm.tiktok.com/1234567890](https://vm.tiktok.com/1234567890)

### [](https://mcp.so/server/tiktok-mcp/Seym0n?tab=content#tiktok_get_subtitle)tiktok_get_subtitle

**描述：**
获取 TikTok 视频 URL 的副标题（内容）。这用于获取 TikTok 视频的字幕、内容或上下文。如果未提供语言代码，工具将返回自动语音识别的副标题。

**输入参数：**

- `tiktok_url`（必填）：TikTok 视频 URL，例如 [https://www.tiktok.com/@username/video/1234567890](https://www.tiktok.com/@username/video/1234567890) 或 [https://vm.tiktok.com/1234567890](https://vm.tiktok.com/1234567890)
- `language_code`（可选）：字幕的语言代码，例如，en 表示英语，es 表示西班牙语，fr 表示法语等。

### [](https://mcp.so/server/tiktok-mcp/Seym0n?tab=content#tiktok_get_post_details)tiktok_get_post_details

**描述：**
获取 TikTok 帖子的详细信息。返回视频的详细信息，例如：

- 描述
- 创建者用户名
- 主题标签
- 点赞、分享、评论、观看次数和书签数
- 创建日期
- 视频时长

**输入参数：**

- `tiktok_url`（必填）：TikTok 视频 URL，例如 [https://www.tiktok.com/@username/video/1234567890](https://www.tiktok.com/@username/video/1234567890) 或 [https://vm.tiktok.com/1234567890](https://vm.tiktok.com/1234567890)

---

https://rapidapi.com/rockapis-rockapis-default/api/linkedin-api8

社交媒体-Linkedin

获取配置文件数据

curl --request GET \ --url 'https://linkedin-data-api.p.rapidapi.com/?username=adamselipsky' \ --header 'x-rapidapi-host: linkedin-data-api.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

按url获取配置文件数据

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-profile-data-by-url?url=https%3A%2F%2Fwww.linkedin.com%2Fin%2Fadamselipsky%2F' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

搜索人物

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-profile-data-by-url?url=https%3A%2F%2Fwww.linkedin.com%2Fin%2Fadamselipsky%2F' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

按url搜索人员

curl --request POST \ --url https://linkedin-api8.p.rapidapi.com/search-people-by-url \ --header 'Content-Type: application/json' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com' \ --data '{"url":"https://www.linkedin.com/search/results/people/?currentCompany=%5B%221035%22%5D&geoUrn=%5B%22103644278%22%5D&keywords=max&origin=FACETED_SEARCH&sid=%3AB5"}'

获取配置文件最近的活动时间

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-profile-recent-activity-time?username=adamselipsky' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取个人资料的帖子

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-profile-posts?username=adamselipsky' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取个人资料帖子和评论

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-profile-post-and-comments?urn=7181285160586211328' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取个人资料帖子评论

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-profile-posts-comments?urn=7169084130104737792&sort=mostRelevant&page=1' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取个人资料的评论

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-profile-comments?username=williamhgates' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取个人资料链接和关注者数量

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/connection-count?username=adamselipsky' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取个人资料数据以及链接和关注者数

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/data-connection-count?username=adamselipsky' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取给定的推荐

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-given-recommendations?username=ryanroslansky&start=0' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取收到的推荐

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-received-recommendations?username=ryanroslansky&start=0' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取个人资料反应

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-profile-likes?username=adamselipsky&start=0' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取个人资料数据、连接和关注

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/profile-data-connection-count-posts?username=adamselipsky' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

个人资料数据和推荐

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/all-profile-data?username=ryanroslansky' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取类似配置文件

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/similar-profiles?url=https%3A%2F%2Fwww.linkedin.com%2Fin%2Fwilliamhgates%2F' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取保护技能的个人资料职位

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/profiles/position-skills?username=tedgaubert' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取公司详细信息

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-company-details?username=google' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

按ID获取公司详细信息

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-company-details-by-id?id=1441' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

搜索公司

curl --request POST \ --url https://linkedin-api8.p.rapidapi.com/companies/search \ --header 'Content-Type: application/json' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com' \ --data '{"keyword":"","locations":[103644278],"companySizes":["D","E","F","G"],"hasJobs":true,"industries":[96,4],"page":1}'

获取公司职位

curl --request POST \ --url https://linkedin-api8.p.rapidapi.com/company-jobs \ --header 'Content-Type: application/json' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com' \ --data '{"companyIds":[5383240,2382910],"page":1,"sort":"mostRecent"}'

按域获取公司

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-company-by-domain?domain=apple.com' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取公司员工人数

curl --request POST \ --url https://linkedin-api8.p.rapidapi.com/get-company-employees-count \ --header 'Content-Type: application/json' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com' \ --data '{"companyId":"1441","locations":[]}'

获取公司职位计数

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-company-jobs-count?companyId=1441' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com‘

获取公司的帖子

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-company-posts?username=google&start=0' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取公司的帖子评论

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-company-post-comments?urn=7179144327430844416&sort=mostRelevant&page=1' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

查找电子邮件地址

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/linkedin-to-email?url=https%3A%2F%2Fwww.linkedin.com%2Fin%2Ftaylorotwell%2F' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取作业详细信息

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-job-details?id=4090994054' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取个人资料的已发布的职位

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/profiles/posted-jobs?username=jipeng-han' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

搜索帖子

curl --request POST \ --url https://linkedin-api8.p.rapidapi.com/search-posts \ --header 'Content-Type: application/json' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com' \ --data '{"keyword":"microsoft","sortBy":"date_posted","datePosted":"","page":1,"contentType":"","fromMember":["ACoAAAEkwwAB9KEc2TrQgOLEQ-vzRyZeCDyc6DQ","ACoAAANuWM8BtmA18VYdgqPtIWt6GhBCTDXToV4","ACoAAA8BYqEBCGLg_vT_ca6mMEqkpp9nVffJ3hc"],"fromCompany":[1441,1035],"mentionsMember":["ACoAAAEkwwAB9KEc2TrQgOLEQ-vzRyZeCDyc6DQ","ACoAAA8BYqEBCGLg_vT_ca6mMEqkpp9nVffJ3hc"],"mentionsOrganization":[1441,1035],"authorIndustry":[96,4],"authorCompany":[1035],"authorTitle":""}'

获取帖子

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-post?url=https%3A%2F%2Fwww.linkedin.com%2Ffeed%2Fupdate%2Furn%3Ali%3Aactivity%3A7219434359085252608%2F' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取帖子的转发

curl --request POST \ --url https://linkedin-api8.p.rapidapi.com/posts/reposts \ --header 'Content-Type: application/json' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com' \ --data '{"urn":"7245786832909557760","page":1,"paginationToken":""}'

获取帖子的回应

curl --request POST \ --url https://linkedin-api8.p.rapidapi.com/get-post-reactions \ --header 'Content-Type: application/json' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com' \ --data '{"url":"https://www.linkedin.com/posts/google_welcome-to-the-gemini-era-activity-7196224250288955393-hd08/?utm_source=share&utm_medium=member_desktop","page":1}'

获取用户文章

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-user-articles?url=https%3A%2F%2Fwww.linkedin.com%2Fin%2Fwilliamhgates%2F&username=williamhgates&page=1' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取文章

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-article?url=https%3A%2F%2Fwww.linkedin.com%2Fpulse%2Fhidden-costs-unreliable-electricity-bill-gates%2F' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取文章评论

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-article-comments?url=https%3A%2F%2Fwww.linkedin.com%2Fpulse%2F2024-corporate-climate-pivot-bill-gates-u89mc%2F%3FtrackingId%3DV85mkekwT9KruOXln2gzIg%253D%253D&page=1&sort=REVERSE_CHRONOLOGICAL' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

获取文章回应

curl --request GET \ --url 'https://linkedin-api8.p.rapidapi.com/get-article-reactions?url=https%3A%2F%2Fwww.linkedin.com%2Fpulse%2F2024-corporate-climate-pivot-bill-gates-u89mc%2F%3FtrackingId%3DV85mkekwT9KruOXln2gzIg%253D%253D&page=1' \ --header 'x-rapidapi-host: linkedin-api8.p.rapidapi.com'

---

社交媒体-tiktok

https://rapidapi.com/Lundehund/api/tiktok-api23/

获取用户信息

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/info?uniqueId=taylorswift' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com' \ --header 'x-rapidapi-key: 8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50'

获取用户信息（包括用户区域）

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/info-with-region?uniqueId=tiktok' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

按ID获取用户信息

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/info-by-id?userId=107955' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取用户关注者

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/followers?secUid=MS4wLjABAAAAqB08cUbXaDWqbD6MCga2RbGTuhfO2EsHayBYx08NDrN7IE3jQuRDNNN6YwyfH6_6&count=30&minCursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取用户关注

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/followings?secUid=MS4wLjABAAAAY3pcRUgWNZAUWlErRzIyrWoc1cMUIdws4KMQQAS5aKN9AD1lcmx5IvCXMUJrP2dB&count=30&minCursor=0&maxCursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取用户帖子

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/posts?secUid=MS4wLjABAAAAqB08cUbXaDWqbD6MCga2RbGTuhfO2EsHayBYx08NDrN7IE3jQuRDNNN6YwyfH6_6&count=35&cursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取用户热门文章

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/popular-posts?secUid=MS4wLjABAAAAqB08cUbXaDWqbD6MCga2RbGTuhfO2EsHayBYx08NDrN7IE3jQuRDNNN6YwyfH6_6&count=35&cursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取用户最早的帖子

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/oldest-posts?secUid=MS4wLjABAAAAqB08cUbXaDWqbD6MCga2RbGTuhfO2EsHayBYx08NDrN7IE3jQuRDNNN6YwyfH6_6&count=30&cursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取用户最喜欢帖子

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/liked-posts?secUid=MS4wLjABAAAA-hnFaH9aGUYLRspPmUXT3nZOha3-CEyChdtqwlyFaG1M_kAi4MD0AaZkbuIsPIzc&count=30&cursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取用户播放列表

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/playlist?secUid=MS4wLjABAAAAWFGzG2-_92Qv9a1I2XZIto24-CxXu2BRC3-HbwH13Y2SIsqt_j1XrNrvYOSHN6q4&count=20&cursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取用户重新发布

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/user/repost?secUid=MS4wLjABAAAA-hnFaH9aGUYLRspPmUXT3nZOha3-CEyChdtqwlyFaG1M_kAi4MD0AaZkbuIsPIzc&count=30&cursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

搜索常规（顶部）

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/search/general?keyword=cat&cursor=0&search_id=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

搜索视频

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/search/video?keyword=cat&cursor=0&search_id=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

搜索账户

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/search/account?keyword=catt&cursor=0&search_id=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

搜索Live

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/search/account?keyword=catt&cursor=0&search_id=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取文章详细信息

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/post/detail?videoId=7306132438047116586' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取帖子的评论

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/post/comments?videoId=6574657885953933314&count=50&cursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取帖子的回复评论

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/post/comment/replies?videoId=7230348754455481601&commentId=7230359281404740357&count=6&cursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取相关文章

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/post/related?videoId=7311302323228167429&count=16&cursor=0' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

获取热点文章

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/post/trending?count=16' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'

下载视频

curl --request GET \ --url 'https://tiktok-api23.p.rapidapi.com/api/download/video?url=https%3A%2F%2Fwww.tiktok.com%2F%40taylorswift%2Fvideo%2F7288965373704064286' \ --header 'x-rapidapi-host: tiktok-api23.p.rapidapi.com'
