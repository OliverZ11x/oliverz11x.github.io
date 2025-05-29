---
date created: 2025/5/26 13:41
date modified: 2025/5/26 13:43
---
[[内容理解]]使用微软的内容理解服务用于解析视频音。

**推特**

```python
import httpx

from typing import Any, Dict

import asyncio

  

# Attempt to import FastMCP from the expected location

try:

from mcp.server.fastmcp.server import FastMCP

except ImportError:

# Fallback or raise an error if MCP framework structure is different

# For now, we'll define a dummy FastMCP if not found,

# but the user needs to ensure their MCP environment is set up.

print("Warning: mcp.server.fastmcp.server.FastMCP not found. Using a dummy placeholder.")

print("Please ensure your MCP framework is installed and the import path is correct.")

class FastMCP:

def __init__(self, host="0.0.0.0", port=8000):

self.host = host

self.port = port

print(f"Dummy FastMCP initialized on {self.host}:{self.port}")

  

def tool(self):

def decorator(func):

return func

return decorator

def run(self):

print(f"Dummy FastMCP run method called. In a real scenario, this would start the server.")

# In a real server, you might have an asyncio loop here.

# For the dummy server, we'll just simulate running the test main.

# if __name__ == "twitter_mcp_server": # A bit of a hack to ensure it runs in test

async def run_main_test():

await main()

print("Dummy FastMCP is 'running' the test main function.")

asyncio.run(run_main_test())

  
  

# Instantiate the server object with one of the expected names (e.g., mcp, server, app)

mcp = FastMCP(host="0.0.0.0", port=8000)

  
  

TWITTER_API_KEY = "8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50"

TWITTER_API_HOST = "twitter-api45.p.rapidapi.com"

TWITTER_API_BASE_URL = "https://twitter-api45.p.rapidapi.com"

USER_AGENT = "MCPTwitterClient/1.0"

  

async def make_twitter_request(endpoint: str, params: Dict[str, Any] = None) -> Dict[str, Any] | str:

"""Sends a request to the Twitter API and handles errors."""

headers = {

"x-rapidapi-key": TWITTER_API_KEY,

"x-rapidapi-host": TWITTER_API_HOST,

"User-Agent": USER_AGENT

}

url = f"{TWITTER_API_BASE_URL}/{endpoint}"

async with httpx.AsyncClient() as client:

try:

response = await client.get(url, headers=headers, params=params, timeout=30.0)

response.raise_for_status()

return response.json()

except httpx.HTTPStatusError as e:

return f"HTTP error occurred: {e.response.status_code} - {e.response.text}"

except Exception as e:

return f"An error occurred: {str(e)}"

  

@mcp.tool()

async def get_user_info(screenname: str, rest_id: str) -> Dict[str, Any] | str:

"""

Fetches user information from Twitter.

  

Args:

screenname: The Twitter screen name (e.g., elonmusk).

rest_id: The rest_id of the user (e.g., 44196397).

"""

params = {"screenname": screenname, "rest_id": rest_id}

return await make_twitter_request("screenname.php", params)

  

@mcp.tool()

async def get_user_timeline(screenname: str) -> Dict[str, Any] | str:

"""

Fetches the timeline for a given user.

  

Args:

screenname: The Twitter screen name.

"""

params = {"screenname": screenname}

return await make_twitter_request("timeline.php", params)

  

@mcp.tool()

async def get_user_following(screenname: str) -> Dict[str, Any] | str:

"""

Fetches the list of users a given user is following.

  

Args:

screenname: The Twitter screen name.

"""

params = {"screenname": screenname}

return await make_twitter_request("following.php", params)

  

@mcp.tool()

async def get_user_followers(screenname: str, blue_verified: int = 0) -> Dict[str, Any] | str:

"""

Fetches the list of followers for a given user.

  

Args:

screenname: The Twitter screen name.

blue_verified: Filter by blue verified status (0 or 1). Defaults to 0.

"""

params = {"screenname": screenname, "blue_verified": blue_verified}

return await make_twitter_request("followers.php", params)

  

@mcp.tool()

async def get_tweet_info(tweet_id: str) -> Dict[str, Any] | str:

"""

Fetches information for a specific tweet.

  

Args:

tweet_id: The ID of the tweet.

"""

params = {"id": tweet_id}

return await make_twitter_request("tweet.php", params)

  

@mcp.tool()

async def get_affiliates(screenname: str) -> Dict[str, Any] | str:

"""

Fetches affiliates for a given user.

  

Args:

screenname: The Twitter screen name (e.g., x).

"""

params = {"screenname": screenname}

return await make_twitter_request("affilates.php", params) # Note: API endpoint might have a typo "affilates.php"

  

@mcp.tool()

async def get_user_media(screenname: str) -> Dict[str, Any] | str:

"""

Fetches media posted by a user.

  

Args:

screenname: The Twitter screen name.

"""

params = {"screenname": screenname}

return await make_twitter_request("usermedia.php", params)

  

@mcp.tool()

async def get_retweets(tweet_id: str) -> Dict[str, Any] | str:

"""

Fetches retweets for a specific tweet.

  

Args:

tweet_id: The ID of the tweet.

"""

params = {"id": tweet_id}

return await make_twitter_request("retweets.php", params)

  

@mcp.tool()

async def get_trends(country: str) -> Dict[str, Any] | str:

"""

Fetches trending topics for a country.

  

Args:

country: The country for which to fetch trends (e.g., UnitedStates).

"""

params = {"country": country}

return await make_twitter_request("trends.php", params)

  

@mcp.tool()

async def search_tweets(query: str, search_type: str = "Top") -> Dict[str, Any] | str:

"""

Searches tweets based on a query.

  

Args:

query: The search query (e.g., cybertruck).

search_type: Type of search (e.g., Top, Latest). Defaults to "Top".

"""

params = {"query": query, "search_type": search_type}

return await make_twitter_request("search.php", params)

  

@mcp.tool()

async def check_retweet(tweet_id: str, user_id: str) -> Dict[str, Any] | str: # Assuming user_id is needed

"""

Checks if a user has retweeted a specific tweet.

The API documentation for this endpoint is missing parameters in your example.

Assuming it needs tweet_id and user_id.

  

Args:

tweet_id: The ID of the tweet.

user_id: The ID of the user to check.

"""

# The provided curl example for checkretweet.php did not show parameters.

# Assuming it takes 'tweet_id' and 'user_id' or similar. Adjust as needed.

params = {"id": tweet_id, "user_id": user_id} # Placeholder params

# You might need to find the correct parameters for this endpoint from the API documentation.

return await make_twitter_request("checkretweet.php", params)

  

@mcp.tool()

async def get_user_replies(screenname: str) -> Dict[str, Any] | str: # Assuming screenname is needed

"""

Fetches replies for a user.

The API documentation for this endpoint is missing parameters in your example.

Assuming it needs screenname.

  

Args:

screenname: The Twitter screen name.

"""

# The provided curl example for replies.php did not show parameters.

# Assuming it takes 'screenname' or 'user_id'. Adjust as needed.

params = {"screenname": screenname} # Placeholder params

return await make_twitter_request("replies.php", params)

  

@mcp.tool()

async def get_tweet_thread(tweet_id: str) -> Dict[str, Any] | str:

"""

Fetches a tweet thread.

  

Args:

tweet_id: The ID of the starting tweet in the thread.

"""

params = {"id": tweet_id}

return await make_twitter_request("tweet_thread.php", params)

  

@mcp.tool()

async def get_latest_replies(tweet_id: str) -> Dict[str, Any] | str:

"""

Fetches the latest replies to a tweet.

  

Args:

tweet_id: The ID of the tweet.

"""

params = {"id": tweet_id}

return await make_twitter_request("latest_replies.php", params)

  

@mcp.tool()

async def check_follow(user_a_screenname: str, user_b_screenname: str) -> Dict[str, Any] | str: # Assuming parameters

"""

Checks if one user follows another.

The API documentation for this endpoint is missing parameters in your example.

Assuming it needs two user identifiers.

  

Args:

user_a_screenname: Screen name of the first user.

user_b_screenname: Screen name of the second user.

"""

# The provided curl example for checkfollow.php did not show parameters.

# Assuming it takes two screen names or user IDs. Adjust as needed.

params = {"screenname_a": user_a_screenname, "screenname_b": user_b_screenname} # Placeholder params

return await make_twitter_request("checkfollow.php", params)

  

@mcp.tool()

async def get_list_timeline(list_id: str) -> Dict[str, Any] | str:

"""

Fetches the timeline for a Twitter list.

  

Args:

list_id: The ID of the Twitter list.

"""

params = {"list_id": list_id}

return await make_twitter_request("listtimeline.php", params)

  

@mcp.tool()

async def search_communities_latest(query: str) -> Dict[str, Any] | str:

"""

Searches community posts (latest) by query.

  

Args:

query: The search query.

"""

params = {"query": query}

return await make_twitter_request("search_communities_latest.php", params)

  

@mcp.tool()

async def search_communities_top(query: str) -> Dict[str, Any] | str:

"""

Searches community posts (top) by query.

  

Args:

query: The search query.

"""

params = {"query": query}

return await make_twitter_request("search_communities_top.php", params)

  

@mcp.tool()

async def search_communities(query: str) -> Dict[str, Any] | str:

"""

Searches communities by query.

  

Args:

query: The search query.

"""

params = {"query": query}

return await make_twitter_request("search_communities.php", params)

  

@mcp.tool()

async def get_community_timeline(community_id: str) -> Dict[str, Any] | str:

"""

Fetches the timeline for a community.

  

Args:

community_id: The ID of the community.

"""

params = {"community_id": community_id}

return await make_twitter_request("community_timeline.php", params)

  

@mcp.tool()

async def get_list_followers(list_id: str) -> Dict[str, Any] | str:

"""

Fetches followers of a Twitter list.

  

Args:

list_id: The ID of the Twitter list.

"""

params = {"list_id": list_id}

return await make_twitter_request("list_followers.php", params)

  

@mcp.tool()

async def get_list_members(list_id: str) -> Dict[str, Any] | str:

"""

Fetches members of a Twitter list.

  

Args:

list_id: The ID of the Twitter list.

"""

params = {"list_id": list_id}

return await make_twitter_request("list_members.php", params)

  

# Example of how you might run this (you'll need to adapt this to your MCP framework)

async def main():

# Example usage of one of the tools

# Note: If tools are registered to the mcp instance, you might call them via mcp.get_user_info(...)

# or the decorator might make them directly callable as is.

print("Attempting to fetch user info for elonmusk...")

user_info = await get_user_info("elonmusk", "44196397")

print("User Info:", user_info)

  

# Example: Get timeline for a user

# print("\nAttempting to fetch timeline for elonmusk...")

# timeline = await get_user_timeline("elonmusk")

# print("User Timeline:", timeline)

  
  

if __name__ == "__main__":

# The 'mcp dev' command or similar should handle starting the server.

# This block can be used for direct testing of tools if needed.

print("Running twitter_mcp_server.py directly for testing purposes.")

print(f"MCP server object ('mcp') is: {type(mcp)}")

if hasattr(mcp, 'run') and isinstance(mcp, FastMCP_imported_or_dummy): # Check if it's the real or dummy FastMCP which has run()

print("To run the actual server, use your MCP framework's command (e.g., 'mcp dev twitter_mcp_server.py')")

print("Executing main() for tool testing...")

asyncio.run(main())

elif hasattr(mcp, 'run'): # If it's the dummy fallback

print("Running the dummy FastMCP's run() method for testing.")

mcp.run()

else:

print("FastMCP object does not have a run method or is not the expected type. Executing main() for testing.")

asyncio.run(main())

  

print("\nScript execution finished.")

print("If you ran this with 'python twitter_mcp_server.py', it was for testing.")

print("For the actual server, use your MCP framework's CLI, e.g., 'mcp dev twitter_mcp_server.py'.")

  

# Helper to access the imported FastMCP or the dummy for the conditional in __main__

FastMCP_imported_or_dummy = None

try:

from mcp.server.fastmcp.server import FastMCP as FastMCP_real

FastMCP_imported_or_dummy = FastMCP_real

except ImportError:

class FastMCP_dummy_check: pass

FastMCP_imported_or_dummy = FastMCP_dummy_check

if isinstance(mcp, FastMCP): # checks if mcp is the dummy class defined above

FastMCP_imported_or_dummy = mcp.__class__
```

---

```python
import httpx

from typing import Any, Dict

import asyncio

  

# Attempt to import FastMCP from the expected location

try:

from mcp.server.fastmcp.server import FastMCP

except ImportError:

print("Warning: mcp.server.fastmcp.server.FastMCP not found. Using a dummy placeholder.")

print("Please ensure your MCP framework is installed and the import path is correct.")

class FastMCP:

def __init__(self, host="0.0.0.0", port=8000):

self.host = host

self.port = port

print(f"Dummy FastMCP initialized on {self.host}:{self.port}")

  

def tool(self):

def decorator(func):

return func

return decorator

def run(self):

print(f"Dummy FastMCP run method called. In a real scenario, this would start the server.")

async def run_main_test():

await main()

print("Dummy FastMCP is 'running' the test main function.")

asyncio.run(run_main_test())

  

# Instantiate the server object

mcp = FastMCP(host="0.0.0.0", port=8000)

  

# LinkedIn API configuration

LINKEDIN_API_KEY = "8a6f2294eemshf9fe6251f184d77p119f82jsna3cb26339d50"

LINKEDIN_API_HOST = "linkedin-api8.p.rapidapi.com"

LINKEDIN_API_BASE_URL = "https://linkedin-api8.p.rapidapi.com"

LINKEDIN_DATA_API_HOST = "linkedin-data-api.p.rapidapi.com"

LINKEDIN_DATA_API_BASE_URL = "https://linkedin-data-api.p.rapidapi.com"

USER_AGENT = "MCPLinkedinClient/1.0"

  

async def make_linkedin_request(endpoint: str, params: Dict[str, Any] = None, method: str = "GET", host: str = LINKEDIN_API_HOST, base_url: str = LINKEDIN_API_BASE_URL, data: Any = None) -> Dict[str, Any] | str:

headers = {

"x-rapidapi-key": LINKEDIN_API_KEY,

"x-rapidapi-host": host,

"User-Agent": USER_AGENT

}

url = f"{base_url}/{endpoint}"

async with httpx.AsyncClient() as client:

try:

if method == "GET":

response = await client.get(url, headers=headers, params=params, timeout=30.0)

elif method == "POST":

response = await client.post(url, headers=headers, json=data, timeout=30.0)

else:

return f"Unsupported HTTP method: {method}"

response.raise_for_status()

return response.json()

except httpx.HTTPStatusError as e:

return f"HTTP error occurred: {e.response.status_code} - {e.response.text}"

except Exception as e:

return f"An error occurred: {str(e)}"

  

@mcp.tool()

async def get_profile_by_username(username: str) -> Dict[str, Any] | str:

"""

获取配置文件数据 (Get profile data by username)

"""

params = {"username": username}

return await make_linkedin_request("", params=params, host=LINKEDIN_DATA_API_HOST, base_url=LINKEDIN_DATA_API_BASE_URL)

  

@mcp.tool()

async def get_profile_by_url(url: str) -> Dict[str, Any] | str:

"""

按url获取配置文件数据 (Get profile data by LinkedIn profile URL)

"""

params = {"url": url}

return await make_linkedin_request("get-profile-data-by-url", params=params)

  

@mcp.tool()

async def search_people_by_url(url: str) -> Dict[str, Any] | str:

"""

按url搜索人员 (Search people by LinkedIn search URL)

"""

data = {"url": url}

return await make_linkedin_request("search-people-by-url", method="POST", data=data)

  

@mcp.tool()

async def get_profile_recent_activity_time(username: str) -> Dict[str, Any] | str:

"""

获取配置文件最近的活动时间 (Get profile's recent activity time)

"""

params = {"username": username}

return await make_linkedin_request("get-profile-recent-activity-time", params=params)

  

@mcp.tool()

async def get_profile_posts(username: str) -> Dict[str, Any] | str:

"""

获取个人资料的帖子 (Get profile's posts)

"""

params = {"username": username}

return await make_linkedin_request("get-profile-posts", params=params)

  

@mcp.tool()

async def get_company_details(username: str) -> Dict[str, Any] | str:

"""

获取公司详细信息 (Get company details by username)

"""

params = {"username": username}

return await make_linkedin_request("get-company-details", params=params)

  

@mcp.tool()

async def get_company_by_domain(domain: str) -> Dict[str, Any] | str:

"""

按域获取公司 (Get company by domain)

"""

params = {"domain": domain}

return await make_linkedin_request("get-company-by-domain", params=params)

  

@mcp.tool()

async def get_post_by_url(url: str) -> Dict[str, Any] | str:

"""

获取帖子 (Get post by URL)

"""

params = {"url": url}

return await make_linkedin_request("get-post", params=params)

  

@mcp.tool()

async def get_user_articles(url: str, username: str, page: int = 1) -> Dict[str, Any] | str:

"""

获取用户文章 (Get user articles)

"""

params = {"url": url, "username": username, "page": page}

return await make_linkedin_request("get-user-articles", params=params)

  

@mcp.tool()

async def get_profile_post_and_comments(urn: str) -> Dict[str, Any] | str:

"""

获取个人资料帖子和评论

"""

params = {"urn": urn}

return await make_linkedin_request("get-profile-post-and-comments", params=params)

  

@mcp.tool()

async def get_profile_posts_comments(urn: str, sort: str = "mostRelevant", page: int = 1) -> Dict[str, Any] | str:

"""

获取个人资料帖子评论

"""

params = {"urn": urn, "sort": sort, "page": page}

return await make_linkedin_request("get-profile-posts-comments", params=params)

  

@mcp.tool()

async def get_profile_comments(username: str) -> Dict[str, Any] | str:

"""

获取个人资料的评论

"""

params = {"username": username}

return await make_linkedin_request("get-profile-comments", params=params)

  

@mcp.tool()

async def get_connection_count(username: str) -> Dict[str, Any] | str:

"""

获取个人资料链接和关注者数量

"""

params = {"username": username}

return await make_linkedin_request("connection-count", params=params)

  

@mcp.tool()

async def get_data_connection_count(username: str) -> Dict[str, Any] | str:

"""

获取个人资料数据以及链接和关注者数

"""

params = {"username": username}

return await make_linkedin_request("data-connection-count", params=params)

  

@mcp.tool()

async def get_given_recommendations(username: str, start: int = 0) -> Dict[str, Any] | str:

"""

获取给定的推荐

"""

params = {"username": username, "start": start}

return await make_linkedin_request("get-given-recommendations", params=params)

  

@mcp.tool()

async def get_received_recommendations(username: str, start: int = 0) -> Dict[str, Any] | str:

"""

获取收到的推荐

"""

params = {"username": username, "start": start}

return await make_linkedin_request("get-received-recommendations", params=params)

  

@mcp.tool()

async def get_profile_likes(username: str, start: int = 0) -> Dict[str, Any] | str:

"""

获取个人资料反应

"""

params = {"username": username, "start": start}

return await make_linkedin_request("get-profile-likes", params=params)

  

@mcp.tool()

async def profile_data_connection_count_posts(username: str) -> Dict[str, Any] | str:

"""

获取个人资料数据、连接和关注

"""

params = {"username": username}

return await make_linkedin_request("profile-data-connection-count-posts", params=params)

  

@mcp.tool()

async def all_profile_data(username: str) -> Dict[str, Any] | str:

"""

个人资料数据和推荐

"""

params = {"username": username}

return await make_linkedin_request("all-profile-data", params=params)

  

@mcp.tool()

async def similar_profiles(url: str) -> Dict[str, Any] | str:

"""

获取类似配置文件

"""

params = {"url": url}

return await make_linkedin_request("similar-profiles", params=params)

  

@mcp.tool()

async def profiles_position_skills(username: str) -> Dict[str, Any] | str:

"""

获取保护技能的个人资料职位

"""

params = {"username": username}

return await make_linkedin_request("profiles/position-skills", params=params)

  

@mcp.tool()

async def get_company_details_by_id(id: str) -> Dict[str, Any] | str:

"""

按ID获取公司详细信息

"""

params = {"id": id}

return await make_linkedin_request("get-company-details-by-id", params=params)

  

@mcp.tool()

async def search_companies(keyword: str, locations: list, companySizes: list, hasJobs: bool, industries: list, page: int) -> Dict[str, Any] | str:

"""

搜索公司

"""

data = {

"keyword": keyword,

"locations": locations,

"companySizes": companySizes,

"hasJobs": hasJobs,

"industries": industries,

"page": page

}

return await make_linkedin_request("companies/search", method="POST", data=data)

  

@mcp.tool()

async def company_jobs(companyIds: list, page: int = 1, sort: str = "mostRecent") -> Dict[str, Any] | str:

"""

获取公司职位

"""

data = {"companyIds": companyIds, "page": page, "sort": sort}

return await make_linkedin_request("company-jobs", method="POST", data=data)

  

@mcp.tool()

async def get_company_employees_count(companyId: str, locations: list = []) -> Dict[str, Any] | str:

"""

获取公司员工人数

"""

data = {"companyId": companyId, "locations": locations}

return await make_linkedin_request("get-company-employees-count", method="POST", data=data)

  

@mcp.tool()

async def get_company_jobs_count(companyId: str) -> Dict[str, Any] | str:

"""

获取公司职位计数

"""

params = {"companyId": companyId}

return await make_linkedin_request("get-company-jobs-count", params=params)

  

@mcp.tool()

async def get_company_posts(username: str, start: int = 0) -> Dict[str, Any] | str:

"""

获取公司的帖子

"""

params = {"username": username, "start": start}

return await make_linkedin_request("get-company-posts", params=params)

  

@mcp.tool()

async def get_company_post_comments(urn: str, sort: str = "mostRelevant", page: int = 1) -> Dict[str, Any] | str:

"""

获取公司的帖子评论

"""

params = {"urn": urn, "sort": sort, "page": page}

return await make_linkedin_request("get-company-post-comments", params=params)

  

@mcp.tool()

async def linkedin_to_email(url: str) -> Dict[str, Any] | str:

"""

查找电子邮件地址

"""

params = {"url": url}

return await make_linkedin_request("linkedin-to-email", params=params)

  

@mcp.tool()

async def get_job_details(id: str) -> Dict[str, Any] | str:

"""

获取作业详细信息

"""

params = {"id": id}

return await make_linkedin_request("get-job-details", params=params)

  

@mcp.tool()

async def profiles_posted_jobs(username: str) -> Dict[str, Any] | str:

"""

获取个人资料的已发布的职位

"""

params = {"username": username}

return await make_linkedin_request("profiles/posted-jobs", params=params)

  

@mcp.tool()

async def search_posts(keyword: str, sortBy: str = "date_posted", datePosted: str = "", page: int = 1, contentType: str = "", fromMember: list = None, fromCompany: list = None, mentionsMember: list = None, mentionsOrganization: list = None, authorIndustry: list = None, authorCompany: list = None, authorTitle: str = "") -> Dict[str, Any] | str:

"""

搜索帖子

"""

data = {

"keyword": keyword,

"sortBy": sortBy,

"datePosted": datePosted,

"page": page,

"contentType": contentType,

"fromMember": fromMember or [],

"fromCompany": fromCompany or [],

"mentionsMember": mentionsMember or [],

"mentionsOrganization": mentionsOrganization or [],

"authorIndustry": authorIndustry or [],

"authorCompany": authorCompany or [],

"authorTitle": authorTitle

}

return await make_linkedin_request("search-posts", method="POST", data=data)

  

@mcp.tool()

async def get_post_reposts(urn: str, page: int = 1, paginationToken: str = "") -> Dict[str, Any] | str:

"""

获取帖子的转发

"""

data = {"urn": urn, "page": page, "paginationToken": paginationToken}

return await make_linkedin_request("posts/reposts", method="POST", data=data)

  

@mcp.tool()

async def get_post_reactions(url: str, page: int = 1) -> Dict[str, Any] | str:

"""

获取帖子的回应

"""

data = {"url": url, "page": page}

return await make_linkedin_request("get-post-reactions", method="POST", data=data)

  

@mcp.tool()

async def get_article(url: str) -> Dict[str, Any] | str:

"""

获取文章

"""

params = {"url": url}

return await make_linkedin_request("get-article", params=params)

  

@mcp.tool()

async def get_article_comments(url: str, page: int = 1, sort: str = "REVERSE_CHRONOLOGICAL") -> Dict[str, Any] | str:

"""

获取文章评论

"""

params = {"url": url, "page": page, "sort": sort}

return await make_linkedin_request("get-article-comments", params=params)

  

@mcp.tool()

async def get_article_reactions(url: str, page: int = 1) -> Dict[str, Any] | str:

"""

获取文章回应

"""

params = {"url": url, "page": page}

return await make_linkedin_request("get-article-reactions", params=params)

  

# Example main for testing

async def main():

print("Fetching LinkedIn profile for adamselipsky...")

profile = await get_profile_by_username("adamselipsky")

print("Profile:", profile)

print("Fetching LinkedIn company details for google...")

company = await get_company_details("google")

print("Company:", company)

  

if __name__ == "__main__":

print("Running linkedin_mcp_server.py directly for testing purposes.")

print(f"MCP server object ('mcp') is: {type(mcp)}")

if hasattr(mcp, 'run'):

print("Running the FastMCP's run() method for testing.")

mcp.run()

else:

print("FastMCP object does not have a run method. Executing main() for testing.")

asyncio.run(main())

print("\nScript execution finished.")

print("If you ran this with 'python linkedin_mcp_server.py', it was for testing.")

print("For the actual server, use your MCP framework's CLI, e.g., 'mcp dev linkedin_mcp_server.py'.")
```

---

