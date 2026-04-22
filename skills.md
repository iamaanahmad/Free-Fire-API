# AI Agent Skills for Free Fire API

> **Comprehensive Guide for AI Agents Integrating with the Free Fire API**

This document outlines the skills, capabilities, and best practices for AI agents working with the Free Fire API. Whether you're building Discord bots, analytics dashboards, or custom integrations, this guide will help you maximize the API's potential.

---

## 🤖 Overview

The Free Fire API is optimized for AI-assisted development. AI agents can help developers by:
- Understanding endpoint purposes and parameters
- Generating code in multiple languages (Python, JavaScript, PHP, cURL, etc.)
- Handling authentication automatically
- Implementing error handling best practices
- Creating comprehensive integrations with proper data validation

---

## 🎯 Core AI Agent Skills

### 1. **API Authentication & Security**

**Skill:** Implement secure authentication patterns

AI agents should:
- Generate code using header authentication (`x-api-key`) as the recommended method
- Handle API keys securely (never hardcode in production code)
- Implement environment variable usage for credentials
- Validate API responses for authentication errors (401)

**Example Implementation Pattern:**
```python
import os
import requests

# Secure API key handling
API_KEY = os.environ.get('FREE_FIRE_API_KEY')

headers = {
    'x-api-key': API_KEY
}

# Always validate authentication
response = requests.get(endpoint_url, headers=headers)
if response.status_code == 401:
    raise Exception("Invalid API key - check your credentials")
```

---

### 2. **Parameter Validation**

**Skill:** Validate input parameters before making API calls

AI agents should validate:
- **UID Format:** 9-10 digit numeric string
- **Region Values:** Must be one of: `sg`, `ind`, `br`
- **Language Codes:** Valid options: `en`, `ar`, `es`, `id`, `pt`, `ru`, `vi`
- **Map Codes:** Alphanumeric, 8-12 characters for Craftland

**Example Validation:**
```python
import re

def validate_uid(uid):
    """Validate Free Fire player UID"""
    if not re.match(r'^\d{9,10}$', str(uid)):
        raise ValueError("UID must be a 9-10 digit number")
    return str(uid)

def validate_region(region):
    """Validate game region"""
    valid_regions = ['sg', 'ind', 'br']
    if region not in valid_regions:
        raise ValueError(f"Region must be one of: {valid_regions}")
    return region
```

---

### 3. **Error Handling & Retry Logic**

**Skill:** Implement comprehensive error handling

AI agents should handle:
- **400 Bad Request:** Invalid parameters
- **401 Unauthorized:** Invalid API key
- **404 Not Found:** Player or resource doesn't exist
- **429 Rate Limit:** Too many requests
- **500 Server Error:** API service issues

**Example Error Handling:**
```python
import time
from requests.exceptions import RequestException

def call_api_with_retry(url, headers, params, max_retries=3):
    """Call API with automatic retry logic"""
    for attempt in range(max_retries):
        try:
            response = requests.get(url, headers=headers, params=params)
            
            if response.status_code == 200:
                return response.json()
            elif response.status_code == 404:
                return {"error": "Player not found"}
            elif response.status_code == 429:
                # Rate limit - wait and retry
                wait_time = 60 * (attempt + 1)
                print(f"Rate limited. Waiting {wait_time}s...")
                time.sleep(wait_time)
                continue
            elif response.status_code == 401:
                raise Exception("Invalid API key")
            else:
                response.raise_for_status()
                
        except RequestException as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)  # Exponential backoff
    
    return {"error": "Max retries exceeded"}
```

---

### 4. **Response Caching**

**Skill:** Implement caching to reduce API calls and avoid rate limits

AI agents should:
- Cache player info responses (low change frequency)
- Use shorter TTL for dynamic data (stats, leaderboards)
- Implement cache invalidation strategies
- Respect rate limits (100 requests/hour on free tier)

**Example Caching Implementation:**
```python
from functools import lru_cache
import time

# Simple in-memory cache with TTL
cache = {}

def cached_api_call(cache_key, api_function, ttl_seconds=300):
    """Cache API responses with TTL"""
    current_time = time.time()
    
    if cache_key in cache:
        cached_data, timestamp = cache[cache_key]
        if current_time - timestamp < ttl_seconds:
            return cached_data
    
    # Make API call
    data = api_function()
    cache[cache_key] = (data, current_time)
    return data
```

---

### 5. **Multi-Language Code Generation**

**Skill:** Generate code examples in multiple programming languages

AI agents should be able to generate equivalent implementations in:
- **Python** (with requests library)
- **JavaScript/Node.js** (with fetch or axios)
- **PHP** (with cURL)
- **cURL** (for command-line testing)
- **Java, Go, Ruby** (for enterprise applications)

---

### 6. **Data Transformation & Formatting**

**Skill:** Process and format API responses for different use cases

AI agents should:
- Parse JSON responses correctly
- Extract relevant data fields
- Format data for display (Discord embeds, web dashboards, etc.)
- Calculate derived metrics (win rates, K/D ratios, etc.)

**Example Data Transformation:**
```python
def format_player_stats(stats_data):
    """Transform raw stats into readable format"""
    player_stats = stats_data['data']
    
    formatted = {
        'player_id': player_stats['uid'],
        'solo': {
            'win_rate': f"{(player_stats['solo']['wins'] / player_stats['solo']['gamesPlayed'] * 100):.1f}%",
            'kd_ratio': f"{player_stats['solo']['kd']:.2f}",
            'total_kills': player_stats['solo']['kills']
        },
        # ... format other modes
    }
    
    return formatted
```

---

## 🚀 Common Use Cases

### Use Case 1: Discord Bot - Player Lookup

**Skill Required:** Combine API calls with Discord bot framework

```python
import discord
from discord.ext import commands

bot = commands.Bot(command_prefix='!')

@bot.command(name='player')
async def player_info(ctx, region: str, uid: str):
    """Get player information"""
    try:
        # Validate inputs
        region = validate_region(region)
        uid = validate_uid(uid)
        
        # Call API
        player_data = get_player_info(region, uid)
        
        # Create Discord embed
        embed = discord.Embed(title=f"Player: {player_data['name']}")
        embed.add_field(name="Level", value=player_data['level'])
        embed.add_field(name="Rank", value=player_data['rank'])
        
        await ctx.send(embed=embed)
        
    except Exception as e:
        await ctx.send(f"Error: {str(e)}")
```

---

### Use Case 2: Analytics Dashboard

**Skill Required:** Aggregate data from multiple endpoints

```python
def create_player_dashboard(region, uid):
    """Create comprehensive player dashboard"""
    
    # Fetch all player data
    info = get_player_info(region, uid)
    stats = get_player_stats(region, uid)
    wishlist = get_player_wishlist(region, uid)
    ban_status = check_ban_status(uid)
    
    # Aggregate dashboard data
    dashboard = {
        'profile': info,
        'statistics': stats,
        'wishlist_items': len(wishlist),
        'account_status': 'Active' if not ban_status['isBanned'] else 'Banned',
        'total_games': sum([
            stats['solo']['gamesPlayed'],
            stats['duo']['gamesPlayed'],
            stats['squad']['gamesPlayed']
        ])
    }
    
    return dashboard
```

---

## 🛡️ Best Practices for AI Agents

1. **Always Validate Inputs:** Never trust user input - validate all parameters
2. **Handle Errors Gracefully:** Provide helpful error messages to users
3. **Implement Rate Limiting:** Track API usage to stay within limits
4. **Cache Strategically:** Reduce API calls for static data
5. **Log API Interactions:** Track errors and performance for debugging
6. **Use Async Operations:** For bots and high-traffic applications
7. **Test Edge Cases:** Handle missing data, banned accounts, invalid regions
8. **Document Your Code:** Generate clear comments and documentation

---

## 📚 Additional Resources

- **API Documentation:** https://docs.freefirecommunity.com
- **Get API Key:** https://developers.freefirecommunity.com
- **Support:** developers@freefirecommunity.com
- **GitHub:** https://github.com/ashqking/Free-Fire-API
