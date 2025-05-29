```
import asyncio

from fastapi import FastAPI, HTTPException

from typing import Any, Dict, Optional

from contextlib import AsyncExitStack

import os

import traceback # Added for more detailed error logging

  

# Added imports for OpenAI

from openai import OpenAI

  

# 启用真实的 MCP 客户端库导入

from mcp import ClientSession, StdioServerParameters

from mcp.client.stdio import stdio_client

  

# --- OpenAI API 密钥 (硬编码 - 存在严重安全风险！) ---

openai_api_key = "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"

openai_client = OpenAI(api_key=openai_api_key)

  

app = FastAPI(title="MCP Client API - Twitter Agent Only", version="0.4.1")

  

# 配置MCP服务器脚本的路径 (仅Twitter)

MCP_SERVER_SCRIPTS = {

"twitter": "/home/ubuntu/twmcp/twitter_mcp_server.py",

}

  

# 用于与 MCP 服务器通信的函数 (internal)

async def call_mcp_tool_via_protocol(platform: str, tool_name: str, params: Dict[str, Any]) -> Dict[str, Any]:

server_script_path = MCP_SERVER_SCRIPTS.get(platform)

if not server_script_path:

raise HTTPException(status_code=404, detail=f"MCP server script for platform '{platform}' not configured.")

  

if not os.path.exists(server_script_path):

raise HTTPException(status_code=500, detail=f"MCP server script not found at path: {server_script_path}")

  

# GPT processing for individual calls is no longer initiated from here directly,

# as the only entry point is /gpt_agent which handles its own GPT processing flow.

# params.pop("__gpt_process__", None) # This was for the removed generic endpoint

  

print(f"[MCP_REQUEST_LOG] Platform: {platform}, Tool: {tool_name}, Params: {params}")

  

command = "python"

server_params = StdioServerParameters(

command=command,

args=[server_script_path],

)

  

mcp_tool_result_content = None

  

try:

async with AsyncExitStack() as exit_stack:

stdio_transport_manager = stdio_client(server_params)

stdio_transport = await exit_stack.enter_async_context(stdio_transport_manager)

stdio_reader, stdio_writer = stdio_transport

  

async with ClientSession(stdio_reader, stdio_writer) as session:

await session.initialize()

tool_result = await session.call_tool(tool_name, params)

if hasattr(tool_result, 'content'):

mcp_tool_result_content = tool_result.content

else:

mcp_tool_result_content = {"raw_result": str(tool_result)}

# GPT processing for individual calls is handled by the /gpt_agent logic flow

return mcp_tool_result_content # Return raw result, /gpt_agent will process all results together

  

except FileNotFoundError:

raise HTTPException(status_code=500, detail=f"Command '{command}' not found. Ensure it's in PATH.")

except ConnectionRefusedError:

raise HTTPException(status_code=503, detail=f"Failed to connect to MCP server for {platform} at {server_script_path}.")

except RuntimeError as e:

print(f"MCP Runtime error for {platform} ({tool_name}): {str(e)}")

traceback.print_exc()

raise HTTPException(status_code=500, detail=f"MCP client runtime error: {str(e)}")

except Exception as e:

print(f"Unexpected error calling MCP service {platform} ({tool_name}): {type(e).__name__} - {str(e)}")

traceback.print_exc()

raise HTTPException(status_code=500, detail=f"An unexpected error occurred: {str(e)}")

  
  

# The generic /{platform}/{tool_name} endpoint is removed as per user request.

# All interactions are now expected to go through /gpt_agent.

  

@app.post("/gpt_agent", summary="让GPT自动决定调用哪些Twitter MCP工具并处理结果，支持迭代深化查询")

async def gpt_agent_endpoint(payload: Dict[str, Any]) -> Dict[str, Any]:

"""

payload: {"task": "请获取 twitter 用户 elonmusk 的信息并用中文总结他的最新5条推文。"}

该端点会尝试分阶段处理：

1. 初步解析用户任务，进行第一轮工具调用（可能主要是获取ID或基础信息）。

2. 评估第一轮结果，如果需要更多信息以满足用户原始意图（且原始意图暗示了多步骤），则规划并执行第二轮衍生工具调用。

3. 整合所有结果，生成最终答复。

"""

import json

original_task = payload.get("task")

if not original_task:

raise HTTPException(status_code=400, detail="Missing 'task' in payload.")

  

# --- 日志和返回结构初始化 ---

agent_log = {

"original_task": original_task,

"stages": []

}

  

# --- 服务端能力描述 (Twitter Only) ---

# (这个字符串会在多个提示中重复使用，确保其准确性和完整性)

TWITTER_TOOLS_CAPABILITIES = """**Twitter (`twitter`):

* `get_user_info(screenname: str, rest_id: str)`: 获取用户信息 (包括用户ID如rest_id, screenname, 描述, 粉丝数, 关注数等)。参数 `{\"screenname\": \"USER_SCREENNAME\", \"rest_id\": \"USER_REST_ID\"}` (rest_id 可在不知道时留空或尝试仅用screenname)

* `get_user_timeline(screenname: str)`: 获取用户最新的推文时间线。参数 `{\"screenname\": \"USER_SCREENNAME\"}`

* `get_user_followers(screenname: str, blue_verified: int = 0)`: 获取用户粉丝列表。参数 `{\"screenname\": \"USER_SCREENNAME\"}`

* `get_tweet_info(tweet_id: str)`: 获取单条推文详情。参数 `{\"tweet_id\": \"TWEET_ID\"}`

* (其他工具如 search_tweets, get_user_following 等也可按需使用)

"""

  

# --- STAGE 1: 初始任务解析和首轮调用 ---

stage1_log = {"name": "Stage 1: Initial Parse & Call", "status": "pending"}

agent_log["stages"].append(stage1_log)

  

initial_parse_prompt_messages = [

{"role": "system", "content": f"""你是一个API的初步解析代理。用户的任务是关于Twitter的。你的目标是：

1. 理解用户的最终意图。

2. 生成第一批尝试性的API调用（`calls`列表），主要是为了获取完成最终意图所必需的核心实体ID或基础信息。例如，如果用户想看某用户名的帖子，第一步可能是先调用 `get_user_info` 来确认用户存在并获取其规范的 `screenname` 或 `rest_id`。

3. 生成一个初步的后续处理指令（`process_instructions`），说明在获取这些初步信息后，接下来可能需要做什么来满足用户的完整请求。

  

可用工具及参数如下：

{TWITTER_TOOLS_CAPABILITIES}

  

重要：`calls`列表中的每个条目必须包含 `platform`: 'twitter'。

输出必须是严格的JSON格式：

{{\"calls\": [{{\"platform\": \"twitter\", \"tool_name\": \"...\", \"params\": {{...}}}}, ...], \"process_instructions\": \"...\"}}

例如，如果任务是"获取elonmusk的最新5条推文"，一个好的初步计划可能是：

{{\"calls\": [{{\"platform\": \"twitter\", \"tool_name\": \"get_user_info\", \"params\": {{"screenname\": \"elonmusk\"}}}}], \"process_instructions\": \"获取用户信息后，如果用户存在，下一步使用其screenname调用get_user_timeline获取推文。\"}}

"""},

{"role": "user", "content": f"用户原始任务：{original_task}"}

]

stage1_log["initial_parse_prompt"] = initial_parse_prompt_messages

  

initial_calls = []

initial_process_instructions = ""

initial_results = {}

  

try:

gpt_response = openai_client.chat.completions.create(

model="gpt-4o-mini", messages=initial_parse_prompt_messages, max_tokens=1000, temperature=0.1

)

parsed_content = gpt_response.choices[0].message.content

stage1_log["initial_parse_gpt_response"] = parsed_content

# Simplified JSON parsing (assuming well-formed or needs robust handling as before)

try:

initial_plan = json.loads(parsed_content.strip().replace("```json", "").replace("```", "").strip())

except Exception as e_json:

initial_plan = json.loads(parsed_content.strip()) # Try again without ``` replacement if first one failed

initial_calls = initial_plan.get("calls", [])

initial_process_instructions = initial_plan.get("process_instructions", "")

stage1_log["parsed_initial_calls"] = initial_calls

stage1_log["parsed_initial_process_instructions"] = initial_process_instructions

  

if not initial_calls or not isinstance(initial_calls, list):

raise ValueError("GPT未能生成有效的初始 calls 列表")

if not initial_process_instructions:

# Fallback if no specific instructions for next step were given

initial_process_instructions = f"根据以下初步结果，决定下一步骤以完成用户任务: {original_task}"

  

for i, call_info in enumerate(initial_calls):

platform = call_info.get("platform", "").lower()

tool_name = call_info.get("tool_name")

params = call_info.get("params", {})

if platform != "twitter" or not tool_name:

initial_results[f"call_{i}_skipped"] = "Invalid platform or missing tool_name for initial call."

continue

if tool_name == "get_user_info" and "rest_id" not in params:

params["rest_id"] = ""

try:

result = await call_mcp_tool_via_protocol("twitter", tool_name, params)

initial_results[f"twitter_{tool_name}_{i}"] = result

except Exception as e:

initial_results[f"twitter_{tool_name}_{i}_error"] = str(e)

stage1_log["initial_call_results"] = initial_results

stage1_log["status"] = "completed"

except Exception as e:

stage1_log["status"] = "failed"

stage1_log["error"] = f"Stage 1 (Initial Parse) failed: {type(e).__name__} - {str(e)}"

agent_log["final_gpt_processed_result"] = stage1_log["error"]

traceback.print_exc()

return agent_log # Early exit if stage 1 fails critically

# --- STAGE 2: 评估初步结果，规划衍生调用 ---

stage2_log = {"name": "Stage 2: Derivative Planning", "status": "pending"}

agent_log["stages"].append(stage2_log)

  

derivative_calls = []

final_process_instructions = initial_process_instructions # Start with initial, might be refined

derivative_results = {}

  

# Only proceed to stage 2 if stage 1 was successful and produced some results or a plan

if stage1_log["status"] == "completed":

plan_derivative_prompt_messages = [

{"role": "system", "content": f"""你是一个Twitter任务深化专家。你已获得以下信息：

1. 用户原始任务：`{original_task}`

2. 初步执行计划的指示：`{initial_process_instructions}`

3. 初步API调用结果：`{json.dumps(initial_results, ensure_ascii=False, default=str)}`

你的任务是：

A. 评估初步结果和指示，判断是否需要进一步的API调用来完全满足用户的原始任务。考虑用户是否可能需要从初步结果中的ID进行更深入的查询（例如，获取了用户信息，但用户实际想看推文）。

B. 如果需要，生成一个`derivative_calls`列表（JSON格式，同上一阶段的`calls`）。这些调用应该使用从初步结果中提取的信息作为参数（例如，用获取到的`screenname`或`rest_id`去调用`get_user_timeline`）。

C. 生成一个`final_process_instructions`字符串，这个指令将用于指导最终的GPT如何整合 *所有* 结果（初步的和衍生的）来回答用户的原始任务。

可用工具及参数（仅限Twitter）：

{TWITTER_TOOLS_CAPABILITIES}

如果不需要衍生调用，则`derivative_calls`应为空列表 `[]`。

输出必须是严格的JSON格式：

{{\"derivative_calls\": [{{\"platform\": \"twitter\", \"tool_name\": \"...\", \"params\": {{...}}}}, ...], \"final_process_instructions\": \"...\"}}

例如，如果初步获取了用户信息，用户的原始任务是看推文，且初步指示也是如此，那么衍生调用可能是获取时间线，最终指示是总结推文。

如果初步结果已满足任务，则：

{{\"derivative_calls\": [], \"final_process_instructions\": \"初步结果已满足任务，请基于这些结果总结并回应用户。\"}}

"""},

{"role": "user", "content": "请根据上述信息，规划下一步。"}

]

stage2_log["plan_derivative_prompt"] = plan_derivative_prompt_messages

  

try:

gpt_response = openai_client.chat.completions.create(

model="gpt-4o-mini", messages=plan_derivative_prompt_messages, max_tokens=1500, temperature=0.1

)

parsed_content = gpt_response.choices[0].message.content

stage2_log["plan_derivative_gpt_response"] = parsed_content

try:

derivative_plan = json.loads(parsed_content.strip().replace("```json", "").replace("```", "").strip())

except Exception as e_json_deriv:

derivative_plan = json.loads(parsed_content.strip()) # Try again without ``` replacement

  

derivative_calls = derivative_plan.get("derivative_calls", [])

final_process_instructions = derivative_plan.get("final_process_instructions", initial_process_instructions) # Fallback to initial if not provided

stage2_log["parsed_derivative_calls"] = derivative_calls

stage2_log["parsed_final_process_instructions"] = final_process_instructions

  

if derivative_calls and isinstance(derivative_calls, list):

for i, call_info in enumerate(derivative_calls):

platform = call_info.get("platform", "").lower()

tool_name = call_info.get("tool_name")

params = call_info.get("params", {})

if platform != "twitter" or not tool_name:

derivative_results[f"deriv_call_{i}_skipped"] = "Invalid platform or tool_name for derivative call."

continue

# Parameter population logic (e.g., from initial_results) would be complex here.

# For now, assume GPT correctly formulates params including those from initial_results.

# A more robust solution would involve explicit mapping or placeholder replacement.

if tool_name == "get_user_info" and "rest_id" not in params:

params["rest_id"] = ""

try:

result = await call_mcp_tool_via_protocol("twitter", tool_name, params)

derivative_results[f"twitter_{tool_name}_deriv_{i}"] = result

except Exception as e:

derivative_results[f"twitter_{tool_name}_deriv_{i}_error"] = str(e)

stage2_log["derivative_call_results"] = derivative_results

stage2_log["status"] = "completed"

except Exception as e:

stage2_log["status"] = "failed"

stage2_log["error"] = f"Stage 2 (Derivative Planning) failed: {type(e).__name__} - {str(e)}"

traceback.print_exc()

# Continue to final processing with whatever information was gathered so far.

# The final processing step should be able to handle incomplete data.

  

# --- STAGE 3: 最终结果整合 ---

stage3_log = {"name": "Stage 3: Final Processing", "status": "pending"}

agent_log["stages"].append(stage3_log)

  

all_collated_results = {**initial_results, **derivative_results} # Merge results

  

final_processing_prompt_messages = [

{"role": "system", "content": """你是一个数据处理和分析专家。你的任务是根据提供的指令，整合所有API调用结果，并生成一个清晰、结构化的中文回复来直接回答用户的原始请求。原始请求是：`{original_task}`

处理指令如下：`{final_process_instructions}`

所有API调用结果（初步和衍生的）汇总如下：

`{json.dumps(all_collated_results, ensure_ascii=False, default=str)}`

请严格按照处理指令来总结数据。如果指令要求从结果中提取特定信息（例如用户A的粉丝数），请确保你的回复中包含这些具体信息，而不仅仅是重复API的原始输出。

如果某些预期的信息由于API调用失败或数据不足而无法获取，请在回复中明确指出，并如果可能，给出简要原因或建议（例如，用户可能不存在，API临时问题等）。

你的输出应该是直接面向用户的最终答案。"""},

{"role": "user", "content": "请根据以上信息生成最终答复。"}

]

stage3_log["final_processing_prompt"] = final_processing_prompt_messages

  

try:

gpt_response = openai_client.chat.completions.create(

model="gpt-4o-mini", messages=final_processing_prompt_messages, max_tokens=2000, temperature=0.7

)

final_gpt_processed_result = gpt_response.choices[0].message.content

stage3_log["final_gpt_response"] = final_gpt_processed_result

agent_log["final_gpt_processed_result"] = final_gpt_processed_result

stage3_log["status"] = "completed"

except Exception as e:

stage3_log["status"] = "failed"

stage3_log["error"] = f"Stage 3 (Final Processing) failed: {type(e).__name__} - {str(e)}"

agent_log["final_gpt_processed_result"] = stage3_log["error"]

traceback.print_exc()

  

return agent_log

  

if __name__ == "__main__":

import uvicorn

print("Starting MCP Client API server (Twitter Agent Only) on http://localhost:8080")

print("Access OpenAPI docs at http://localhost:8080/docs")

print("\nExample POST request to gpt_agent for Twitter tasks:")

print("""curl -X POST -H "Content-Type: application/json" -d '{"task": "获取用户elonmusk在Twitter的粉丝数"}' http://localhost:8080/gpt_agent""" )

uvicorn.run(app, host="0.0.0.0", port=8080)
```