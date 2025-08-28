# 🔌 API Integration Guide

This guide explains how to integrate real APIs with the LLM Agent POC, replacing the demo simulations with production-ready implementations.

## 🔑 API Keys Setup

### 1. OpenAI API Integration

**Get API Key:**
1. Visit https://platform.openai.com/api-keys
2. Create account and generate API key
3. Add billing method (required for usage)

**Implementation:**
```javascript
// Replace simulateLLMCall function in index.html
async function callLLM() {
    const provider = document.getElementById('provider').value;
    const model = document.getElementById('model').value;
    const apiKey = document.getElementById('apiKey').value;
    
    if (provider === 'openai') {
        const response = await fetch('https://api.openai.com/v1/chat/completions', {
            method: 'POST',
            headers: {
                'Authorization': `Bearer ${apiKey}`,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                model: model,
                messages: conversation,
                tools: TOOLS,
                tool_choice: "auto",
                max_tokens: 1000
            })
        });
        
        if (!response.ok) {
            throw new Error(`OpenAI API error: ${response.statusText}`);
        }
        
        return await response.json();
    }
    // Add other providers (Anthropic, Google) similarly
}
```

### 2. Google Custom Search API

**Setup:**
1. Go to https://developers.google.com/custom-search/v1/overview
2. Enable Custom Search API in Google Cloud Console
3. Create a Custom Search Engine at https://cse.google.com/cse/
4. Get API key and Search Engine ID

**Implementation:**
```javascript
// Replace googleSearch function
async function googleSearch(query) {
    const GOOGLE_API_KEY = 'your-google-api-key';
    const SEARCH_ENGINE_ID = 'your-search-engine-id';
    
    try {
        const response = await fetch(
            `https://www.googleapis.com/customsearch/v1?key=${GOOGLE_API_KEY}&cx=${SEARCH_ENGINE_ID}&q=${encodeURIComponent(query)}&num=5`
        );
        
        if (!response.ok) {
            throw new Error(`Google Search API error: ${response.statusText}`);
        }
        
        const data = await response.json();
        
        const results = {
            query: query,
            total_results: data.searchInformation?.totalResults || 0,
            results: data.items?.map(item => ({
                title: item.title,
                snippet: item.snippet,
                url: item.link,
                display_url: item.displayLink
            })) || []
        };
        
        return JSON.stringify(results, null, 2);
    } catch (error) {
        return `Search error: ${error.message}`;
    }
}
```

### 3. AI Pipe Proxy Integration

**Setup:**
1. Visit AI Pipe documentation for API access
2. Register and get API key
3. Choose available workflows

**Implementation:**
```javascript
// Replace aiPipe function
async function aiPipe(workflow, data) {
    const AI_PIPE_API_KEY = 'your-aipipe-key';
    const AI_PIPE_BASE_URL = 'https://api.aipipe.com/v1';
    
    try {
        const response = await fetch(`${AI_PIPE_BASE_URL}/workflows/${workflow}/execute`, {
            method: 'POST',
            headers: {
                'Authorization': `Bearer ${AI_PIPE_API_KEY}`,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                input: data,
                parameters: {}
            })
        });
        
        if (!response.ok) {
            throw new Error(`AI Pipe API error: ${response.statusText}`);
        }
        
        const result = await response.json();
        
        return JSON.stringify({
            workflow: workflow,
            input: data,
            result: result.output,
            status: result.status,
            execution_time: result.execution_time
        }, null, 2);
    } catch (error) {
        return `AI Pipe error: ${error.message}`;
    }
}
```

## 🔒 Security Best Practices

### 1. Environment Variables
```javascript
// Create config.js (add to .gitignore)
const CONFIG = {
    OPENAI_API_KEY: 'your-openai-key',
    GOOGLE_API_KEY: 'your-google-key',
    GOOGLE_SEARCH_ENGINE_ID: 'your-search-id',
    AI_PIPE_API_KEY: 'your-aipipe-key'
};

// In index.html, load config
<script src="config.js"></script>
```

### 2. Rate Limiting
```javascript
class RateLimiter {
    constructor(maxRequests, timeWindow) {
        this.maxRequests = maxRequests;
        this.timeWindow = timeWindow;
        this.requests = [];
    }
    
    async checkLimit() {
        const now = Date.now();
        this.requests = this.requests.filter(time => now - time < this.timeWindow);
        
        if (this.requests.length >= this.maxRequests) {
            throw new Error('Rate limit exceeded. Please wait.');
        }
        
        this.requests.push(now);
    }
}

// Usage
const openaiLimiter = new RateLimiter(60, 60000); // 60 requests per minute
```

### 3. Error Recovery
```javascript
async function retryApiCall(apiFunction, maxRetries = 3, delay = 1000) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await apiFunction();
        } catch (error) {
            if (i === maxRetries - 1) throw error;
            await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
        }
    }
}
```

## 🚀 Advanced Integrations

### 1. Anthropic Claude API
```javascript
async function callClaude() {
    const response = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
            'x-api-key': ANTHROPIC_API_KEY,
            'Content-Type': 'application/json',
            'anthropic-version': '2023-06-01'
        },
        body: JSON.stringify({
            model: 'claude-3-sonnet-20240229',
            messages: conversation,
            tools: TOOLS,
            max_tokens: 1000
        })
    });
    
    return await response.json();
}
```

### 2. Google Gemini API
```javascript
async function callGemini() {
    const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=${GOOGLE_API_KEY}`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            contents: conversation.map(msg => ({
                role: msg.role === 'assistant' ? 'model' : 'user',
                parts: [{ text: msg.content }]
            })),
            tools: TOOLS
        })
    });
    
    return await response.json();
}
```

## 🔧 CORS Solutions

### Option 1: CORS Proxy
```javascript
const CORS_PROXY = 'https://cors-anywhere.herokuapp.com/';
const apiUrl = `${CORS_PROXY}https://api.openai.com/v1/chat/completions`;
```

### Option 2: Browser Extension
Install a CORS browser extension for development (not recommended for production).

### Option 3: Backend Proxy
Create a simple Express.js proxy server:
```javascript
// proxy-server.js
const express = require('express');
const cors = require('cors');
const app = express();

app.use(cors());
app.use(express.json());

app.post('/api/openai', async (req, res) => {
    const response = await fetch('https://api.openai.com/v1/chat/completions', {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(req.body)
    });
    
    const data = await response.json();
    res.json(data);
});

app.listen(3001);
```

## 📊 Usage Monitoring

### 1. Token Counting
```javascript
function estimateTokens(text) {
    // Rough estimation: 1 token ≈ 4 characters
    return Math.ceil(text.length / 4);
}

function calculateCost(tokens, model) {
    const pricing = {
        'gpt-4': { input: 0.03, output: 0.06 }, // per 1K tokens
        'gpt-3.5-turbo': { input: 0.001, output: 0.002 }
    };
    
    return (tokens / 1000) * pricing[model].input;
}
```

### 2. Usage Analytics
```javascript
class UsageTracker {
    constructor() {
        this.usage = JSON.parse(localStorage.getItem('apiUsage') || '{}');
    }
    
    track(provider, model, tokens, cost) {
        const today = new Date().toISOString().split('T')[0];
        if (!this.usage[today]) this.usage[today] = {};
        if (!this.usage[today][provider]) this.usage[today][provider] = {};
        if (!this.usage[today][provider][model]) {
            this.usage[today][provider][model] = { tokens: 0, cost: 0, requests: 0 };
        }
        
        this.usage[today][provider][model].tokens += tokens;
        this.usage[today][provider][model].cost += cost;
        this.usage[today][provider][model].requests += 1;
        
        localStorage.setItem('apiUsage', JSON.stringify(this.usage));
    }
    
    getToday() {
        const today = new Date().toISOString().split('T')[0];
        return this.usage[today] || {};
    }
}
```

## 🧪 Testing APIs

### 1. API Health Check
```javascript
async function testApiConnections() {
    const tests = [
        { name: 'OpenAI', test: () => callLLM() },
        { name: 'Google Search', test: () => googleSearch('test') },
        { name: 'AI Pipe', test: () => aiPipe('echo', 'test') }
    ];
    
    for (const { name, test } of tests) {
        try {
            await test();
            console.log(`✅ ${name} API working`);
        } catch (error) {
            console.error(`❌ ${name} API failed:`, error.message);
        }
    }
}
```

### 2. Mock Mode Toggle
```javascript
const IS_DEMO_MODE = !document.getElementById('apiKey').value;

async function callLLM() {
    if (IS_DEMO_MODE) {
        return await simulateLLMCall();
    } else {
        return await realLLMCall();
    }
}
```

## 📝 Integration Checklist

- [ ] API keys configured securely
- [ ] Rate limiting implemented  
- [ ] Error handling and retry logic
- [ ] CORS issues resolved
- [ ] Token counting and cost monitoring
- [ ] Fallback to demo mode
- [ ] API health checks
- [ ] Security best practices followed
- [ ] Production environment variables
- [ ] Monitoring and analytics setup

## 🆘 Troubleshooting

**Issue**: 401 Unauthorized
**Solution**: Check API key format and permissions

**Issue**: 429 Rate Limit
**Solution**: Implement rate limiting and backoff

**Issue**: CORS blocked
**Solution**: Use proxy server or CORS headers

**Issue**: High costs
**Solution**: Implement token limits and monitoring

---

**Next Steps**: Follow DEPLOYMENT.md for production deployment guidelines.
