---
title: SQL Agent 测试
date created: 2024年7月31日,星期三,上午,11:15:59
date modified: 2024年8月6日,星期二,下午,2:24:10
---

## 推荐问题

针对 `ads_ridehub_inventory_view` 表，设置五个推荐的问题，用于测试 SQL Agent 的功能并提供有价值的业务洞察：

1. **库存分析：**
   - 请问在最近一个月内，哪个省份的车辆库存数量最多？请按省份分组统计每日现存库存台数的平均值。

2. **销售趋势分析：**
   - 在过去一年中，每个季度的车辆销售量是怎样的？请统计每个季度的累计销售台数（daily_release_count）。

3. **库存效率：**
   - 哪个品牌的车辆平均库存周期（avg_turnover_days）最短？请列出所有品牌的平均库存周期，并按周期长度排序。

4. **市场竞争力分析：**
   - 根据市占率（market_share）和品牌竞争力（brand_competitiveness），哪些品牌在市场上表现得最好？请列出市占率和品牌竞争力超过平均值的品牌。

5. **库龄车辆问题：**
   - 目前库存中长库龄车辆占比（aged_vehicle_ratio）最高的三个城市是哪些？请找出这些城市并展示它们的长库龄车辆占比和总库存台数。

这些问题覆盖了库存管理、销售分析、市场表现等多个维度，可以有效测试 SQL Agent 的查询能力和数据处理效率。

## 工具

```python
# 初始化工具包
def initialize_toolkit(db, llm):
    """
    初始化工具包，用于与数据库交互。
    参数：
        - db: 数据库连接对象。
        - llm: 聊天模型对象。
    """
    toolkit = SQLDatabaseToolkit(db=db, llm=llm)
    tools = toolkit.get_tools()

    # Query checking
    query_check_system = """
    You are a SQL expert with a strong attention to detail.
    Double check the SQLite query for common mistakes, including:
    - Using NOT IN with NULL values
    - Using UNION when UNION ALL should have been used
    - Using BETWEEN for exclusive ranges
    - Data type mismatch in predicates
    - Properly quoting identifiers
    - Using the correct number of arguments for functions
    - Casting to the correct data type
    - Using the proper columns for joins

    If there are any of the above mistakes, rewrite the query. If there are no mistakes, just reproduce the original query.

    Execute the correct query with the appropriate tool.
    """
    query_check_prompt = ChatPromptTemplate.from_messages([("system", query_check_system),("user", "{query}")])
    query_check = query_check_prompt | llm

    @tool
    def check_query_tool(query: str) -> str:
        """
        Use this tool to double check if your query is correct before executing it.
        """
        return query_check.invoke({"query": query}).content

    # Query result checking
    query_result_check_system = """You are grading the result of a SQL query from a DB.
    - Check that the result is not empty.
    - If it is empty, instruct the system to re-try!"""
    query_result_check_prompt = ChatPromptTemplate.from_messages([("system", query_result_check_system),("user", "{query_result}")])
    query_result_check = query_result_check_prompt | llm

    @tool
    def check_result(query_result: str) -> str:
        """
        Use this tool to check the query result from the database to confirm it is not empty and is relevant.
        """
        return query_result_check.invoke({"query_result": query_result}).content

    tools.append(check_query_tool)
    tools.append(check_result)
    return tools
```

## Prompts

### SQLAgent

英文版

```python
    SQL_PREFIX_TEMPLATE = """
    ROLE:
    You are an agent designed to interact with a SQL database. You can query the database and directly access the results.

    GOAL:
    Given a question, create a SQL query to retrieve the needed data, execute this query, and directly return the results of the query, not the query itself. If errors are encountered or the initial results are empty, diagnose the issues, adjust the query, and try again.

    INSTRUCTIONS:
    - Begin by identifying the relevant tables to understand the scope of available data.
    - Use the 'sql_db_schema' tool to obtain detailed information about the structure of these tables.
    - Construct an appropriate SQL query to answer the question based on the schema information.
    - Execute the query using the 'sql_db_query' tool.
    - Directly return the results from the execution. Do not return the SQL query text unless requested specifically by the user.
    - If the query execution is successful but no results are found, reassess the assumptions and revise the query accordingly.
    - If an error occurs (e.g., table not found or syntax error), troubleshoot the error by:
	    - Reviewing the table names and schema.
	    - Adjusting the query to fix any errors.
	    - Re-executing the corrected query.
    - Focus only on retrieving relevant columns to answer the question.
    - Limit the results to a maximum of 5 entries by default, unless more is specified by the user.
    - Sort the results by relevant columns to present the most pertinent information first.
    - Continuously adjust and re-run the query until valid results are obtained or confirm that the data needed does not exist in the database.
    - Avoid making any modifications to the database's structure or content (e.g., no INSERT, UPDATE, DELETE, DROP statements).
    - Ensure the final response is based directly on the query results, providing concise and accurate answers.

"""
```

中文版

```python
角色：
您是一个设计用来与SQL数据库交互的代理。您可以查询数据库并直接访问结果。

目标：
针对一个问题，创建一个SQL查询来检索所需的数据，执行此查询，并直接返回查询的结果，而不是查询本身。如果遇到错误或初始结果为空，请诊断问题，调整查询，并再次尝试。

指示：
- 首先识别相关表格，以了解可用数据的范围。
- 使用'sql_db_schema'工具获取这些表的结构详细信息。
- 根据表结构信息构建一个适当的SQL查询来回答问题。
- 使用'sql_db_query'工具执行查询。
- 直接返回执行结果。除非用户明确请求，否则不返回SQL查询文本。
- 如果查询执行成功但没有找到结果，请重新评估假设并相应修改查询。
- 如果发生错误（例如，找不到表或语法错误），通过以下方式排除错误：
  - 检查表名和表结构。
  - 调整查询以修正任何错误。
  - 重新执行更正后的查询。
- 仅关注检索与问题答案相关的列。
- 默认情况下将结果限制为最多5个条目，除非用户指定了更多。
- 按相关列排序结果，以首先呈现最相关的信息。
- 不断调整和重新运行查询，直到获得有效结果或确认数据库中不存在所需数据。
- 避免对数据库的结构或内容进行任何修改（例如，不进行INSERT、UPDATE、DELETE、DROP语句）。
- 确保最终响应直接基于查询结果，提供简洁准确的答案。

```

### PPT 生成

```python
CONCAT_INFO_PROMPT_TEMPLATE = """
### 查询结果综合分析报告

**联网查询结果**：
    {internet_answer}

**知识库查询结果**：
    {kb_answer}

**数据库查询结果**：
    {db_answer}

**问题描述**：
    {question}
    
**请按以下步骤生成一页PPT**：
1. **标题设计**：
    - **主标题**：“数据库查询结果综合分析”
    - **副标题**：根据查询的具体对象或日期添加，例如“2024年5月奥迪Q3车系”。

2. **查询目的与背景**：
    - **框架**：使用文本框展示。
    - **标题**：“查询目的与背景”
    - **内容**：详细描述进行此查询的业务需求和目的，包括为何进行这项查询的业务背景或问题点。

3. **数据库查询流程**：
    - **框架**：使用流程图或列表形式。
    - **标题**：“数据库查询流程”
    - **内容**：展示确定相关表格、构建SQL查询语句的详细步骤，明确每一步的目的和执行的操作。

4. **查询结果展示**：
    - **框架**：使用文本框与图表。
    - **标题**：“查询结果展示”
    - **内容**：展示查询结果的数值或详细信息，例如：“平均库存深度为0.0946”。
    - **图表**：选择适合展示数据类型的图表（柱状图、饼图、折线图等），依据结果的性质和重要性展示趋势或比较。

5. **分析与建议**：
    - **框架**：使用文本框。
    - **标题**：“分析与建议”
    - **内容**：基于查询结果的分析，提出相关的业务建议或策略调整，包括可能的行动步骤和预期效果。

**设计注意事项**：
    - 确保所有文本和图表的布局整洁、易于阅读。
    - 使用公司或标准的PPT模板确保视觉风格一致性。
    - 考虑到报告的专业性，选择适当的配色方案和字体样式。

"""
```

## 风险

### SQLAgent 模型幻觉问题

1. 时间的定义，当我问 5 月的时候需要确定是哪一年的 5 月，而不是直接自定义查询。其次当我问 5 月的时候，5 月的范围是从 5 月 1 日到 5 月 31 日，还是其他的，需要确定。
2. 对于品牌，厂商，车系的区分和把控不准确。例如，品牌为奥迪的车，车系会有很多种，制造车的厂商也会有好几个。

```json
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_202407251605165b07ab0af7294099', 'function': {'arguments': '{}', 'name': 'sql_db_list_tables'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-a4c46c80-d57d-4710-a257-258f1289b28c-0', tool_calls=[{'name': 'sql_db_list_tables', 'args': {}, 'id': 'call_202407251605165b07ab0af7294099', 'type': 'tool_call'}])]}}
----
{'tools': {'messages': [ToolMessage(content='ads_ridehub_inventory_view', name='sql_db_list_tables', tool_call_id='call_202407251605165b07ab0af7294099')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_2024072516052197460017c7534495', 'function': {'arguments': '{"table_names":"ads_ridehub_inventory_view"}', 'name': 'sql_db_schema'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-48c1fdcd-7f51-4490-9a83-1e72e600af20-0', tool_calls=[{'name': 'sql_db_schema', 'args': {'table_names': 'ads_ridehub_inventory_view'}, 'id': 'call_2024072516052197460017c7534495', 'type': 'tool_call'}])]}}
----
{'tools': {'messages': [ToolMessage(content="\nCREATE TABLE ads_ridehub_inventory_view (\n\tprovince Nullable(String) COMMENT '省份', \n\tcity Nullable(String) COMMENT '城市', \n\tdistrict Nullable(String) COMMENT '区县', \n\tbrand Nullable(String) COMMENT '品牌', \n\tmanufacturer Nullable(String) COMMENT '厂商', \n\tseries Nullable(String) COMMENT '车系', \n\tdate_day Date COMMENT '统计日期', \n\tdaily_arrival_count Nullable(Int64) COMMENT '接车台数(入库台数)', \n\tdaily_release_count Nullable(Int64) COMMENT '每日释放台数(销售台数)', \n\tdaily_inventory_count Nullable(Int64) COMMENT '每日现存库存台数(库存数)', \n\taged_vehicles_count Nullable(Int64) COMMENT '长库龄车辆数', \n\ttotal_days_in_inventory Nullable(Int64) COMMENT '在库车辆的总在库天数', \n\ttotal_turnover_days Nullable(Int64) COMMENT '释放车辆的总流转天数', \n\tcumulative_release_count_30days Nullable(Int64) COMMENT '前30日（包含当日）的累计释放台数', \n\tinventory_depth Nullable(Float32) COMMENT '库存深度', \n\tavg_turnover_days Nullable(Float32) COMMENT '单车平均流转天数(库存周期)', \n\taged_vehicle_ratio Nullable(Float32) COMMENT '长库龄车辆占比', \n\tmarket_share Nullable(Float32) COMMENT '市占率', \n\tbrand_competitiveness Nullable(Float32) COMMENT '品牌竞争力', \n\tguide_price Nullable(Decimal(9, 2)) COMMENT '厂商指导价'\n) ENGINE = View\n\n/*\n3 rows from ads_ridehub_inventory_view table:\nprovince\tcity\tdistrict\tbrand\tmanufacturer\tseries\tdate_day\tdaily_arrival_count\tdaily_release_count\tdaily_inventory_count\taged_vehicles_count\ttotal_days_in_inventory\ttotal_turnover_days\tcumulative_release_count_30days\tinventory_depth\tavg_turnover_days\taged_vehicle_ratio\tmarket_share\tbrand_competitiveness\tguide_price\n贵州省\t贵阳市\t花溪区\t深蓝汽车\t深蓝汽车\t深蓝SL03\t2024-03-01\t0\t1\t12\t0\t1288\t57\t7\t1.71\t57.0\t0.0\tNone\tNone\t166566.67\n山东省\t菏泽市\t牡丹区\t未知\t未知\t未知\t2024-03-01\t0\t0\t0\t0\t0\t0\t0\t0.0\t0.0\t0.0\tNone\tNone\t0\n广东省\t深圳市\t宝安区\t比亚迪\t比亚迪\t唐新能源\t2024-03-01\t0\t0\t9\t0\t1209\t0\t0\t0.0\t0.0\t0.0\tNone\tNone\t0\n*/", name='sql_db_schema', tool_call_id='call_2024072516052197460017c7534495')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_202407251605239b9e58cec7d94663', 'function': {'arguments': '{"query":"SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day \\u003e= DATE(\\"now\\", \\"-1 month\\") GROUP BY province ORDER BY total_inventory DESC LIMIT 5"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-5c77e817-8e27-41c3-a066-125a8a4a79f8-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': 'SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day >= DATE("now", "-1 month") GROUP BY province ORDER BY total_inventory DESC LIMIT 5'}, 'id': 'call_202407251605239b9e58cec7d94663', 'type': 'tool_call'}])]}}
----
{'tools': {'messages': [ToolMessage(content='Error: DatabaseException(HTTPException("Code: 47. DB::Exception: Missing columns: \'-1 month\' \'now\' while processing query: \'SELECT province, sum(daily_inventory_count) AS total_inventory FROM (SELECT province, city, district, brand, manufacturer, series, date AS date_day, daily_arrival_count, daily_release_count, daily_inventory_count, aged_vehicles_count, total_days_in_inventory, total_turnover_days, cumulative_release_count_30days, inventory_depth, avg_turnover_days, aged_vehicle_ratio, market_share, brand_competitiveness, guide_price FROM ridehub.ads_ridehub_inventory HAVING date_day >= DATE(now, `-1 month`)) AS ads_ridehub_inventory_view WHERE date_day >= DATE(now, `-1 month`) GROUP BY province ORDER BY total_inventory DESC LIMIT 5\', required columns: \'province\' \'now\' \'date_day\' \'daily_inventory_count\' \'-1 month\', maybe you meant: \'province\', \'date_day\' or \'daily_inventory_count\'. (UNKNOWN_IDENTIFIER) (version 23.8.9.1)\\n"))\n Please fix your mistakes.', name='sql_db_query', tool_call_id='call_202407251605239b9e58cec7d94663')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_20240725160545935340010a9742ed', 'function': {'arguments': '{"query":"SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day \\u003e= DATE(\\"2024-03-01\\") GROUP BY province ORDER BY total_inventory DESC LIMIT 5"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-f4b61878-79bf-4a8b-9c62-6174e8fa79c3-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': 'SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day >= DATE("2024-03-01") GROUP BY province ORDER BY total_inventory DESC LIMIT 5'}, 'id': 'call_20240725160545935340010a9742ed', 'type': 'tool_call'}])]}}
----
{'tools': {'messages': [ToolMessage(content='Error: DatabaseException(HTTPException("Code: 47. DB::Exception: Missing columns: \'2024-03-01\' while processing query: \'SELECT province, sum(daily_inventory_count) AS total_inventory FROM (SELECT province, city, district, brand, manufacturer, series, date AS date_day, daily_arrival_count, daily_release_count, daily_inventory_count, aged_vehicles_count, total_days_in_inventory, total_turnover_days, cumulative_release_count_30days, inventory_depth, avg_turnover_days, aged_vehicle_ratio, market_share, brand_competitiveness, guide_price FROM ridehub.ads_ridehub_inventory HAVING date_day >= DATE(`2024-03-01`)) AS ads_ridehub_inventory_view WHERE date_day >= DATE(`2024-03-01`) GROUP BY province ORDER BY total_inventory DESC LIMIT 5\', required columns: \'province\' \'2024-03-01\' \'date_day\' \'daily_inventory_count\', maybe you meant: \'province\', \'date_day\' or \'daily_inventory_count\'. (UNKNOWN_IDENTIFIER) (version 23.8.9.1)\\n"))\n Please fix your mistakes.', name='sql_db_query', tool_call_id='call_20240725160545935340010a9742ed')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_20240725160608b7fa6266ee784240', 'function': {'arguments': '{"query":"SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day \\u003e= DATE(\\"2024-02-01\\") GROUP BY province ORDER BY total_inventory DESC LIMIT 5"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-5f3e8444-d88b-40bf-bfcf-29e9e198cbe4-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': 'SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day >= DATE("2024-02-01") GROUP BY province ORDER BY total_inventory DESC LIMIT 5'}, 'id': 'call_20240725160608b7fa6266ee784240', 'type': 'tool_call'}])]}}
----
{'tools': {'messages': [ToolMessage(content='Error: DatabaseException(HTTPException("Code: 47. DB::Exception: Missing columns: \'2024-02-01\' while processing query: \'SELECT province, sum(daily_inventory_count) AS total_inventory FROM (SELECT province, city, district, brand, manufacturer, series, date AS date_day, daily_arrival_count, daily_release_count, daily_inventory_count, aged_vehicles_count, total_days_in_inventory, total_turnover_days, cumulative_release_count_30days, inventory_depth, avg_turnover_days, aged_vehicle_ratio, market_share, brand_competitiveness, guide_price FROM ridehub.ads_ridehub_inventory HAVING date_day >= DATE(`2024-02-01`)) AS ads_ridehub_inventory_view WHERE date_day >= DATE(`2024-02-01`) GROUP BY province ORDER BY total_inventory DESC LIMIT 5\', required columns: \'province\' \'date_day\' \'daily_inventory_count\' \'2024-02-01\', maybe you meant: \'province\', \'date_day\' or \'daily_inventory_count\'. (UNKNOWN_IDENTIFIER) (version 23.8.9.1)\\n"))\n Please fix your mistakes.', name='sql_db_query', tool_call_id='call_20240725160608b7fa6266ee784240')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_202407251606142321262140244aed', 'function': {'arguments': '{"query":"SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day \\u003e= \\"2024-02-01\\" GROUP BY province ORDER BY total_inventory DESC LIMIT 5"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-90b6bd59-c005-4dd6-bf03-76fd18ebef9a-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': 'SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day >= "2024-02-01" GROUP BY province ORDER BY total_inventory DESC LIMIT 5'}, 'id': 'call_202407251606142321262140244aed', 'type': 'tool_call'}])]}}
----
{'tools': {'messages': [ToolMessage(content='Error: DatabaseException(HTTPException("Code: 47. DB::Exception: Missing columns: \'2024-02-01\' while processing query: \'SELECT province, sum(daily_inventory_count) AS total_inventory FROM (SELECT province, city, district, brand, manufacturer, series, date AS date_day, daily_arrival_count, daily_release_count, daily_inventory_count, aged_vehicles_count, total_days_in_inventory, total_turnover_days, cumulative_release_count_30days, inventory_depth, avg_turnover_days, aged_vehicle_ratio, market_share, brand_competitiveness, guide_price FROM ridehub.ads_ridehub_inventory HAVING date_day >= `2024-02-01`) AS ads_ridehub_inventory_view WHERE date_day >= `2024-02-01` GROUP BY province ORDER BY total_inventory DESC LIMIT 5\', required columns: \'province\' \'date_day\' \'daily_inventory_count\' \'2024-02-01\', maybe you meant: \'province\', \'date_day\' or \'daily_inventory_count\'. (UNKNOWN_IDENTIFIER) (version 23.8.9.1)\\n"))\n Please fix your mistakes.', name='sql_db_query', tool_call_id='call_202407251606142321262140244aed')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_2024072516061914851906c2ba43a6', 'function': {'arguments': '{"query":"SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day \\u003e= DATE(\\"2024-01-31\\") GROUP BY province ORDER BY total_inventory DESC LIMIT 5"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-13f87d50-0d55-4465-9787-b557ce13f62a-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': 'SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day >= DATE("2024-01-31") GROUP BY province ORDER BY total_inventory DESC LIMIT 5'}, 'id': 'call_2024072516061914851906c2ba43a6', 'type': 'tool_call'}])]}}
----
{'tools': {'messages': [ToolMessage(content='Error: DatabaseException(HTTPException("Code: 47. DB::Exception: Missing columns: \'2024-01-31\' while processing query: \'SELECT province, sum(daily_inventory_count) AS total_inventory FROM (SELECT province, city, district, brand, manufacturer, series, date AS date_day, daily_arrival_count, daily_release_count, daily_inventory_count, aged_vehicles_count, total_days_in_inventory, total_turnover_days, cumulative_release_count_30days, inventory_depth, avg_turnover_days, aged_vehicle_ratio, market_share, brand_competitiveness, guide_price FROM ridehub.ads_ridehub_inventory HAVING date_day >= DATE(`2024-01-31`)) AS ads_ridehub_inventory_view WHERE date_day >= DATE(`2024-01-31`) GROUP BY province ORDER BY total_inventory DESC LIMIT 5\', required columns: \'province\' \'date_day\' \'daily_inventory_count\' \'2024-01-31\', maybe you meant: \'province\', \'date_day\' or \'daily_inventory_count\'. (UNKNOWN_IDENTIFIER) (version 23.8.9.1)\\n"))\n Please fix your mistakes.', name='sql_db_query', tool_call_id='call_2024072516061914851906c2ba43a6')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_20240725160623efb13108395440ce', 'function': {'arguments': '{"query":"SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day \\u003e= \\"2024-01-31\\" GROUP BY province ORDER BY total_inventory DESC LIMIT 5"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-09e9f457-1ad4-48e7-9e95-b8cc98414000-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': 'SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day >= "2024-01-31" GROUP BY province ORDER BY total_inventory DESC LIMIT 5'}, 'id': 'call_20240725160623efb13108395440ce', 'type': 'tool_call'}])]}}
----
{'tools': {'messages': [ToolMessage(content='Error: DatabaseException(HTTPException("Code: 47. DB::Exception: Missing columns: \'2024-01-31\' while processing query: \'SELECT province, sum(daily_inventory_count) AS total_inventory FROM (SELECT province, city, district, brand, manufacturer, series, date AS date_day, daily_arrival_count, daily_release_count, daily_inventory_count, aged_vehicles_count, total_days_in_inventory, total_turnover_days, cumulative_release_count_30days, inventory_depth, avg_turnover_days, aged_vehicle_ratio, market_share, brand_competitiveness, guide_price FROM ridehub.ads_ridehub_inventory HAVING date_day >= `2024-01-31`) AS ads_ridehub_inventory_view WHERE date_day >= `2024-01-31` GROUP BY province ORDER BY total_inventory DESC LIMIT 5\', required columns: \'province\' \'date_day\' \'daily_inventory_count\' \'2024-01-31\', maybe you meant: \'province\', \'date_day\' or \'daily_inventory_count\'. (UNKNOWN_IDENTIFIER) (version 23.8.9.1)\\n"))\n Please fix your mistakes.', name='sql_db_query', tool_call_id='call_20240725160623efb13108395440ce')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_20240725160642476faba81079404f', 'function': {'arguments': '{"query":"SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day \\u003e= DATE(\\"2024-01-31\\") GROUP BY province ORDER BY total_inventory DESC LIMIT 5"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-1e40c26f-eb49-403a-a3bb-fde93fafa5fd-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': 'SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day >= DATE("2024-01-31") GROUP BY province ORDER BY total_inventory DESC LIMIT 5'}, 'id': 'call_20240725160642476faba81079404f', 'type': 'tool_call'}])]}}
----
{'tools': {'messages': [ToolMessage(content='Error: DatabaseException(HTTPException("Code: 47. DB::Exception: Missing columns: \'2024-01-31\' while processing query: \'SELECT province, sum(daily_inventory_count) AS total_inventory FROM (SELECT province, city, district, brand, manufacturer, series, date AS date_day, daily_arrival_count, daily_release_count, daily_inventory_count, aged_vehicles_count, total_days_in_inventory, total_turnover_days, cumulative_release_count_30days, inventory_depth, avg_turnover_days, aged_vehicle_ratio, market_share, brand_competitiveness, guide_price FROM ridehub.ads_ridehub_inventory HAVING date_day >= DATE(`2024-01-31`)) AS ads_ridehub_inventory_view WHERE date_day >= DATE(`2024-01-31`) GROUP BY province ORDER BY total_inventory DESC LIMIT 5\', required columns: \'province\' \'date_day\' \'daily_inventory_count\' \'2024-01-31\', maybe you meant: \'province\', \'date_day\' or \'daily_inventory_count\'. (UNKNOWN_IDENTIFIER) (version 23.8.9.1)\\n"))\n Please fix your mistakes.', name='sql_db_query', tool_call_id='call_20240725160642476faba81079404f')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_20240725160647b5eb325d268f4a79', 'function': {'arguments': '{"query":"SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day \\u003e= DATE(\'2024-01-31\') GROUP BY province ORDER BY total_inventory DESC LIMIT 5"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-80055f77-221a-4c12-8968-e48a0e351d69-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': "SELECT province, SUM(daily_inventory_count) AS total_inventory FROM ads_ridehub_inventory_view WHERE date_day >= DATE('2024-01-31') GROUP BY province ORDER BY total_inventory DESC LIMIT 5"}, 'id': 'call_20240725160647b5eb325d268f4a79', 'type': 'tool_call'}])]}}
----
{'tools': {'messages': [ToolMessage(content="[('广东省', 15253235), ('山东省', 14299448), ('河南省', 12021547), ('河北省', 9729180), ('江苏省', 9655652)]", name='sql_db_query', tool_call_id='call_20240725160647b5eb325d268f4a79')]}}
----
{'agent': {'messages': [AIMessage(content='在最近一个月内，广东省的车辆库存数量最多。', response_metadata={'finish_reason': 'stop', 'model_name': 'glm-4'}, id='run-f27b5d27-58b2-428f-a1af-e6327191195d-0')]}}
----

```

### SQLAgent 历史对话消息

使用 Memory 方式进行对话时，如果每次对话之前都对 Agent 进行初始化的话，Agent 就会清除之前的记忆（就算设置了 checkpointer 也会清除）。但如果不进行初始化的话就能够记住历史对话。

### SQLAgent 对于历史消息和工具查询内容上的冲突

- **历史消息**：SQLAgent 会记录和参考以前的查询和对话内容，以便提供更精准的回答或进一步的分析。
- **工具查询**：SQLAgent 可能会使用各种工具（例如数据库查询工具）来获取和处理实时数据。
- **冲突**：当历史消息中的信息与工具查询所得的实时数据不一致时，SQLAgent 需要解决这种冲突，以确保回答的准确性和一致性。

SQLAgent 在处理用户请求时，需要同时参考历史消息和实时数据。当历史消息中的信息与工具查询结果之间存在不一致或冲突时，如何协调和解决这种冲突是一个需要重点关注的问题。解决方案应确保最终提供给用户的信息是最新且准确的，同时也需要考虑历史消息中的上下文和背景信息。

### SQLAgent 数据库连接问题

```json
{'tools': {'messages': [ToolMessage(content='Error: DatabaseException(HTTPException("Code: 47. DB::Exception: Missing columns: \'-30 day\' \'now\' while processing query: \'SELECT province, avg(daily_inventory_count) AS avg_inventory FROM (SELECT province, city, district, brand, manufacturer, series, date AS date_day, daily_arrival_count, daily_release_count, daily_inventory_count, aged_vehicles_count, total_days_in_inventory, total_turnover_days, cumulative_release_count_30days, inventory_depth, avg_turnover_days, aged_vehicle_ratio, market_share, brand_competitiveness, guide_price FROM ridehub.ads_ridehub_inventory HAVING date_day >= DATE(now, `-30 day`)) AS ads_ridehub_inventory_view WHERE date_day >= DATE(now, `-30 day`) GROUP BY province ORDER BY avg_inventory DESC LIMIT 1\', required columns: \'province\' \'now\' \'date_day\' \'daily_inventory_count\' \'-30 day\', maybe you meant: \'province\', \'date_day\' or \'daily_inventory_count\'. (UNKNOWN_IDENTIFIER) (version 23.8.9.1)\\n"))\n Please fix your mistakes.', name='sql_db_query', tool_call_id='call_20240726113440218292de01d448b3')]}} 
---- 
{'agent': {'messages': [AIMessage(content='很抱歉，由于系统错误，我无法直接使用日期函数来查询最近一个月内哪个省份的车辆库存数量最多。我需要您提供具体的日期范围，例如起始日期和结束日期，以便我能够正确地构造查询。您能提供这些信息吗？', response_metadata={'finish_reason': 'stop', 'model_name': 'glm-4'}, id='run-e4aec7ed-6d7a-4169-9335-25956cc9d6ec-0')]}} 
----
```

SQLAgent 在处理用户请求时可能会遇到数据库连接问题。如果 SQLAgent 在处理用户请求时遇到任何其他数据库连接问题，可能需要进一步排查和解决这些问题，以确保提供给用户的信息是准确和及时的。

