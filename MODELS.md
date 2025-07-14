# Gemini Models Reference

*Current models, capabilities, and selection guide (January 2025)*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/models
Verified: 2025-01-14
Key Models: gemini-2.5-flash, gemini-2.5-pro, gemini-2.5-flash-lite
Note: Preview/experimental models subject to change
-->

## Stable Models

### Gemini 2.5 Pro
- **Model ID**: `gemini-2.5-pro`
- **Input tokens**: 1,048,576
- **Output tokens**: 65,536
- **Features**: Multimodal, thinking, function calling, code execution
- **Strengths**: Complex reasoning, advanced coding, multimodal understanding
- **Cost**: Premium tier

### Gemini 2.5 Flash (Recommended)
- **Model ID**: `gemini-2.5-flash`
- **Input tokens**: 1,048,576
- **Output tokens**: 65,536
- **Features**: Multimodal, thinking, function calling, code execution
- **Strengths**: Large scale processing, low-latency, high volume tasks
- **Cost**: Mid tier

### Gemini 1.5 Pro
- **Model ID**: `gemini-1.5-pro`
- **Input tokens**: 2,097,152
- **Output tokens**: 8,192
- **Features**: Multimodal, function calling, code execution
- **Strengths**: Wide-range reasoning tasks, longest context
- **Cost**: Premium tier

## Preview Models

### Gemini 2.5 Flash-Lite
- **Model ID**: `gemini-2.5-flash-lite-preview-06-17`
- **Input tokens**: 1,000,000
- **Output tokens**: 64,000
- **Strengths**: Cost efficiency, low latency
- **Note**: Experimental, may change

### Gemini 2.0 Flash
- **Model ID**: `gemini-2.0-flash`
- **Context**: 1M tokens
- **Strengths**: Fast inference
- **Features**: Multimodal

### Text Embedding
- **Model ID**: `text-embedding-004`
- **Dimensions**: 768
- **Purpose**: Semantic search, RAG
- **Max input**: 2048 tokens

## Model Selection Guide

| Use Case | Recommended Model |
|----------|------------------|
| General chat/assistance | gemini-2.5-flash |
| Complex analysis | gemini-2.5-pro |
| High volume/budget | gemini-2.5-flash-lite-preview-06-17 |
| Advanced reasoning | gemini-2.5-pro (with thinking) |
| Long documents | gemini-1.5-pro |
| Real-time/streaming | gemini-2.5-flash |
| Embeddings | text-embedding-004 |

## Key Capabilities

### All Multimodal Models Support:
- Text generation
- Image understanding
- Audio transcription (most formats)
- Video understanding (up to 1 hour)
- Function calling
- JSON mode
- System instructions

### Exclusive to Pro Models:
- Code execution
- Extended context (>1M)
- Advanced reasoning

### Exclusive to 2.0 Models:
- Native multimodal generation
- Native tool use (no JSON)
- Spatial understanding
- Audio generation

## Token Limits

| Model | Input Tokens | Output Tokens |
|-------|-------------|---------------|
| 2.5 Pro | 1,048,576 | 65,536 |
| 2.5 Flash | 1,048,576 | 65,536 |
| 2.5 Flash-Lite | 1,000,000 | 64,000 |
| 1.5 Pro | 2,097,152 | 8,192 |

## Rate Limits (Free Tier)

| Model | RPM | TPM |
|-------|-----|-----|
| 2.5 Pro | 5 | 250,000 |
| 2.5 Flash | 10 | 250,000 |
| 2.0 Flash | 15 | 1,000,000 |
| 2.0 Flash-Lite | 30 | 1,000,000 |
| 1.5 Pro | 2 | 32,000 |

## Version Notes
- Models ending in `-exp` are experimental
- Version numbers (e.g., `-002`) indicate iterations
- Latest models auto-update unless version specified
- Deprecated: gemini-pro, gemini-pro-vision (use 1.5+)