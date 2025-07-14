# Claude AI Assistant Guide for Google Gemini API

*Critical corrections and common mistakes to avoid when working with Gemini API*

## ⚠️ CRITICAL: Package Names Have Changed

### JavaScript/Node.js
```javascript
// ❌ WRONG (Claude often suggests this)
npm install @google/generative-ai
import { GoogleGenerativeAI } from "@google/generative-ai";

// ✅ CORRECT (as of 2025)
npm install @google/genai
import { GoogleGenAI } from "@google/genai";
```

### Python
```python
# ❌ WRONG (Claude often suggests this)
pip install google-generativeai
import google.generativeai as genai

# ✅ CORRECT (as of 2025)
pip install google-genai
from google import genai
```

## 🔴 Model Names - Use Latest

```javascript
// ❌ WRONG - Claude often uses these outdated models
model: "gemini-pro"
model: "gemini-pro-vision"
model: "gemini-2.0-flash-exp"
model: "gemini-1.5-pro-002"

// ✅ CORRECT - Current recommended models
model: "gemini-2.5-flash"     // Best for most tasks
model: "gemini-2.5-pro"       // For complex reasoning
model: "gemini-2.5-flash-lite-preview-06-17"  // Budget option
```

## 📦 SDK Initialization

### JavaScript
```javascript
// ❌ WRONG - Old pattern
const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
const model = genAI.getGenerativeModel({ model: "gemini-pro" });

// ✅ CORRECT - New pattern
const ai = new GoogleGenAI({});  // Auto-reads GEMINI_API_KEY env var
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Your prompt"
});
```

### Python
```python
# ❌ WRONG - Old pattern
genai.configure(api_key=os.environ["GEMINI_API_KEY"])
model = genai.GenerativeModel('gemini-pro')
response = model.generate_content("prompt")

# ✅ CORRECT - New pattern
client = genai.Client()  # Auto-reads GEMINI_API_KEY env var
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Your prompt"
)
```

## 🌐 REST API Headers

```bash
# ❌ WRONG - Claude often capitalizes this
-H "X-Goog-Api-Key: $GEMINI_API_KEY"

# ✅ CORRECT - Lowercase is recommended
-H "x-goog-api-key: $GEMINI_API_KEY"
```

## 💰 Pricing Tiers

```python
# ❌ WRONG - Claude may use old pricing
# Gemini 1.5 Flash: $0.075/1M input tokens

# ✅ CORRECT - Current pricing (2025)
# Gemini 2.5 Flash: $0.30/1M input (text), $1.00/1M (audio)
# Gemini 2.5 Pro: $1.25/1M input (≤200K), $2.50/1M (>200K)
```

## 🚦 Rate Limits

```python
# ❌ WRONG - Claude may mention daily limits
# "Free tier: 1,500 requests per day"

# ✅ CORRECT - No daily limits, only per-minute
# Free tier examples:
# - Gemini 2.5 Pro: 5 RPM, 250K TPM
# - Gemini 2.5 Flash: 10 RPM, 250K TPM
```

## 🛡️ Safety Settings

```python
# ❌ WRONG - Missing 5th category
categories = [
    "HARM_CATEGORY_HARASSMENT",
    "HARM_CATEGORY_HATE_SPEECH", 
    "HARM_CATEGORY_SEXUALLY_EXPLICIT",
    "HARM_CATEGORY_DANGEROUS_CONTENT"  # Wrong name
]

# ✅ CORRECT - 5 categories, correct names
categories = [
    "HARM_CATEGORY_HARASSMENT",
    "HARM_CATEGORY_HATE_SPEECH",
    "HARM_CATEGORY_SEXUALLY_EXPLICIT", 
    "HARM_CATEGORY_DANGEROUS",          # No "_CONTENT"
    "HARM_CATEGORY_CIVIC_INTEGRITY"     # Don't forget this one
]
```

## 📝 Common Method Mistakes

### JavaScript Chat
```javascript
// ❌ WRONG - Old SDK pattern
const chat = model.startChat();
const result = await chat.sendMessage("Hello");

// ✅ CORRECT - New SDK pattern
const chat = await ai.chats.create({ model: "gemini-2.5-flash" });
const result = await chat.send("Hello");
```

### Python Streaming
```python
# ❌ WRONG - Old parameter
response = model.generate_content("prompt", stream=True)

# ✅ CORRECT - New config parameter
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="prompt",
    config={"stream": True}
)
```

## 🔍 Token Limits

```yaml
# ❌ WRONG - Claude may use old limits
Gemini 1.5 Pro: 1M context window

# ✅ CORRECT - Current limits
Gemini 2.5 Pro: 1,048,576 input, 65,536 output
Gemini 2.5 Flash: 1,048,576 input, 65,536 output
Gemini 1.5 Pro: 2,097,152 input, 8,192 output (still largest input)
```

## 🎯 Quick Validation Checklist

When Claude generates Gemini code, check:

1. ✅ Package name: `@google/genai` (JS) or `google-genai` (Python)
2. ✅ Import: `GoogleGenAI` (JS) or `from google import genai` (Python)
3. ✅ Model: Use `gemini-2.5-flash` or `gemini-2.5-pro`
4. ✅ Headers: Use lowercase `x-goog-api-key`
5. ✅ No daily rate limits mentioned
6. ✅ 5 harm categories including CIVIC_INTEGRITY
7. ✅ Correct initialization pattern for SDK

## 📚 Reference Links

Always verify with official docs:
- https://ai.google.dev/gemini-api/docs
- https://ai.google.dev/gemini-api/docs/quickstart

## 💡 Pro Tips

1. **Environment Variable**: Both SDKs auto-read `GEMINI_API_KEY` - no need to pass it
2. **Default Model**: When in doubt, use `gemini-2.5-flash`
3. **Streaming**: Config parameter changed in new SDKs
4. **Thinking Mode**: Use `thinking_config` in Python for reasoning tasks
5. **Context Caching**: Available but syntax differs from old SDK

Remember: Gemini API is rapidly evolving. When Claude's suggestions don't work, check if the SDK has been updated!