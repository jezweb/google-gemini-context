# Google Gemini API Context

Compact, verified documentation for Google Gemini API. Each file contains essential information in under 500 words for efficient AI assistant context.

## 🚀 Quick Navigation

### Core References
- [**MODELS.md**](MODELS.md) - Available models and capabilities
- [**INDEX.md**](INDEX.md) - Complete keyword index for searching

### Common Concepts
- [Authentication](common/authentication.md) - API keys, OAuth setup
- [Rate Limits](common/rate-limits.md) - Quotas and usage limits
- [Pricing](common/pricing.md) - Cost per model/operation
- [Safety](common/safety.md) - Content filters and policies
- [Regions](common/regions.md) - Geographic availability
- [Errors](common/errors.md) - Error codes reference

### JavaScript/Node.js
- [Quickstart](javascript/quickstart.md) - Setup and first request
- [Client Setup](javascript/client-setup.md) - SDK configuration
- [Text Generation](javascript/text-generation.md) - Basic completions
- [Chat](javascript/chat.md) - Multi-turn conversations
- [Vision](javascript/vision.md) - Image understanding/generation
- [Audio](javascript/audio.md) - Speech and music
- [Video](javascript/video.md) - Video processing
- [Embeddings](javascript/embeddings.md) - Text embeddings
- [Function Calling](javascript/function-calling.md) - Tool use
- [Structured Output](javascript/structured-output.md) - JSON mode
- [Streaming](javascript/streaming.md) - Real-time responses
- [Caching](javascript/caching.md) - Context caching
- [Batch](javascript/batch.md) - Batch processing
- [Live API](javascript/live-api.md) - WebSocket connections
- [File Handling](javascript/file-handling.md) - Uploads
- [Error Handling](javascript/error-handling.md) - JS exceptions

### Python
- [Quickstart](python/quickstart.md) - Setup and first request
- [Client Setup](python/client-setup.md) - SDK configuration
- [Text Generation](python/text-generation.md) - Basic completions
- [Chat](python/chat.md) - Multi-turn conversations
- [Vision](python/vision.md) - Image understanding
- [Function Calling](python/function-calling.md) - Tool use
- [Structured Output](python/structured-output.md) - JSON mode

### REST API
- [Quickstart](rest-api/quickstart.md) - cURL basics
- [Endpoints](rest-api/endpoints.md) - Base URLs
- [Authentication](rest-api/authentication.md) - Headers setup
- [Text Generation](rest-api/text-generation.md) - Completion requests
- [Chat](rest-api/chat.md) - Message format
- [Vision](rest-api/vision.md) - Base64 images
- [Audio](rest-api/audio.md) - Audio formats
- [Multipart](rest-api/multipart.md) - File uploads
- [Function Calling](rest-api/function-calling.md) - Tool definitions
- [Structured Output](rest-api/structured-output.md) - Response schemas
- [Streaming](rest-api/streaming.md) - Server-sent events
- [Batch](rest-api/batch.md) - Batch endpoint
- [Models Endpoint](rest-api/models-endpoint.md) - List models
- [Error Responses](rest-api/error-responses.md) - HTTP codes

### Examples
- **JavaScript**: [Chat App](examples/javascript/chat-app.md) | [Image Analyzer](examples/javascript/image-analyzer.md) | [RAG System](examples/javascript/rag-system.md)
- **Python**: [Chat CLI](examples/python/chat-cli.md) | [Document QA](examples/python/document-qa.md) | [Agent](examples/python/agent.md)
- **REST**: [cURL Scripts](examples/rest/curl-scripts.md) | [Postman](examples/rest/postman-collection.md)

### Migration Guides
- [From OpenAI](migration/from-openai.md) - OpenAI → Gemini
- [From Anthropic](migration/from-anthropic.md) - Claude → Gemini
- [From PaLM](migration/from-palm.md) - PaLM → Gemini

## 📋 Usage Tips

1. **For AI Assistants**: Include relevant files as context when working with Gemini API
2. **Navigation**: Use INDEX.md to find topics by keyword
3. **Examples**: Check language-specific examples for common patterns
4. **Updates**: Files include version info and last update date

## 🔄 Version

Documentation current as of: **January 2025**  
Gemini API Version: **v1beta**  
Latest Models: Gemini 2.5 Flash/Pro, Gemini 1.5 Pro/Flash

## 🔗 Official Resources

- [Official Docs](https://ai.google.dev/gemini-api/docs)
- [AI Studio](https://aistudio.google.com)
- [API Console](https://console.cloud.google.com)

## 📊 File Verification Summary

### What's Been Verified (2025-01-14)
Each verified file contains metadata with:
- Source URL from official documentation
- Verification date
- Key changes and notes

### Critical Updates Applied
1. **SDK Changes**: All files updated to use new SDK packages
   - JS: `@google/generative-ai` → `@google/genai`
   - Python: `google-generativeai` → `google-genai`

2. **API Pattern Changes**:
   - No more session objects (`start_chat()`)
   - Manual history management required
   - Tools passed in each request

3. **Model Updates**:
   - Default to `gemini-2.5-flash`
   - Added new 2.5 series models
   - Removed experimental suffixes

### Files Created During Verification
- `python/vision.md` - Image processing with new SDK
- `python/function-calling.md` - Tool use patterns
- `python/structured-output.md` - JSON mode configuration

### Important Notes
- **CLAUDE.md** - Essential reference showing common mistakes
- **Metadata blocks** - Check HTML comments for verification status
- **Token efficiency** - All files optimized for AI context windows