# Pricing

*Cost structure for Gemini API usage (January 2025)*

<!-- METADATA
Source: https://ai.google.dev/pricing
Verified: 2025-01-14
Key Info: Free tier available, significant price differences by model
Note: Prices subject to change, check official pricing page
-->

## Quick Reference
- **Free tier**: Available for all models
- **Input cheaper than output**: ~3-10x difference  
- **Cached content**: 75% discount
- **Context tiers**: ≤200K vs >200K tokens

## Pricing by Model (Per 1M Tokens)

### Gemini 2.5 Pro
| Type | ≤200K | >200K |
|------|-------|-------|
| Input | $1.25 | $2.50 |
| Output | $10.00 | $15.00 |
| Cached | $0.31 | $0.625 |

### Gemini 2.5 Flash
| Type | Text/Image/Video | Audio |
|------|-----------------|-------|
| Input | $0.30 | $1.00 |
| Output | $2.50 | $2.50 |
| Cached | $0.075 | $0.25 |

### Gemini 2.5 Flash-Lite (Preview)
| Type | Text/Image/Video | Audio |
|------|-----------------|-------|
| Input | $0.10 | $0.50 |
| Output | $0.40 | $0.40 |
| Cached | $0.025 | $0.125 |

### Gemini 1.5 Pro
| Type | ≤128K | >128K |
|------|-------|-------|
| Input | $1.25 | $2.50 |
| Output | $5.00 | $10.00 |
| Cached | $0.31 | $0.625 |

### Gemini 1.5 Flash
| Type | ≤128K | >128K |
|------|-------|-------|
| Input | $0.075 | $0.15 |
| Output | $0.30 | $0.60 |
| Cached | $0.019 | $0.038 |

## Free Tier Limits

| Model | RPM | TPM |
|-------|-----|-----|
| 2.5 Pro | 5 | 250K |
| 2.5 Flash | 10 | 250K |
| 2.0 Flash | 15 | 1M |
| 2.0 Flash-Lite | 30 | 1M |
| 1.5 Pro | 2 | 32K |
| 1.5 Flash | 15 | 1M |

## Token Counting

### Text
- ~4 characters = 1 token
- Whitespace and punctuation count

### Media
| Type | Tokens |
|------|--------|
| Image (any size) | 258 |
| Audio (per second) | 32 |
| Video (per second) | 258 |

### Examples
```python
# 10-word prompt ≈ 15 tokens
# 1 image + 50 words ≈ 333 tokens
# 30-second video ≈ 7,740 tokens
```

## Cost Optimization

### 1. Use Caching
```python
# 75% discount on cached content
from google import genai
client = genai.Client()
# Context caching available for large documents
```

### 2. Choose Right Model
- Budget: Flash-Lite (cheapest)
- General use: 2.5 Flash (best value)
- Complex: 2.5 Pro (when needed)

### 3. Optimize Prompts
- Shorter system prompts
- Reuse common instructions via caching
- Batch similar requests

### 4. Streaming Responses
- Stop generation early if needed
- Reduces output tokens

## Billing Details

### Charges
- Billed monthly
- Per 1,000 tokens (shown as per 1M)
- Rounded up to nearest 1K

### Free Trial
- $300 credit for new accounts
- 90-day expiration
- All models included

### Payment Methods
- Credit/debit cards
- Google Cloud billing account
- Invoicing (approved accounts)

## Cost Calculator

```python
def calculate_cost(model, input_tokens, output_tokens, cached=False):
    prices = {
        'gemini-2.5-flash': {
            'input': 0.30, 'output': 2.50,
            'cached': 0.075
        },
        'gemini-2.5-pro': {
            'input': 1.25, 'output': 10.00,
            'cached': 0.31
        }
    }
    
    p = prices[model]
    input_cost = (input_tokens / 1_000_000) * 
                 (p['cached'] if cached else p['input'])
    output_cost = (output_tokens / 1_000_000) * p['output']
    
    return input_cost + output_cost
```

## Budget Controls

1. Set spending limits in Console
2. Configure alerts at thresholds
3. Use quota limits as safeguard
4. Monitor dashboard daily

## See Also
- [Rate Limits](rate-limits.md)
- [Models](../MODELS.md)