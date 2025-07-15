# REST API URL Context

*Fetch and analyze web content directly in prompts*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/url-context
Verified: 2025-01-14
Models: Gemini 2.5 Pro/Flash/Flash-Lite, Gemini 2.0 Flash
Note: Experimental feature, 99.6% token reduction
-->

## Quick Reference
- **Endpoint**: /models/{model}:generateContent
- **Tool**: urlContext in tools array
- **Benefit**: 99.6% token reduction
- **Limit**: Up to 20 URLs per request
- **Feature**: Direct web analysis

## Basic URL Context

```bash
# Analyze a single URL
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Summarize the main points from https://example.com/article"
      }]
    }],
    "tools": [{
      "urlContext": {}
    }]
  }'
```

## Multiple URLs

```bash
# Compare content from multiple URLs
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "Compare the pricing plans from:\n- https://service1.com/pricing\n- https://service2.com/pricing\n- https://service3.com/pricing\n\nCreate a comparison table with features and costs."
      }]
    }],
    "tools": [{
      "urlContext": {}
    }]
  }'
```

## Combined with Google Search

```bash
# Use both URL context and Google Search
curl -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent?key=$API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [{
        "text": "First search for recent AI news, then analyze the top 3 results in detail."
      }]
    }],
    "tools": [
      {
        "urlContext": {}
      },
      {
        "googleSearch": {}
      }
    ]
  }'
```

## Data Extraction Script

```bash
#!/bin/bash
# extract_data_from_urls.sh

API_KEY="your-api-key"
MODEL="gemini-2.5-flash"

extract_product_data() {
  local urls=("$@")
  
  # Build URL list
  local url_list=""
  for url in "${urls[@]}"; do
    url_list="${url_list}- ${url}\n"
  done
  
  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/$MODEL:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "Extract the following information from each product page:\n- Product name\n- Price\n- Key features (list)\n- Availability status\n\nURLs to analyze:\n'"$url_list"'\n\nReturn as JSON array."
        }]
      }],
      "tools": [{
        "urlContext": {}
      }],
      "generationConfig": {
        "responseMimeType": "application/json"
      }
    }' | jq '.'
}

# Usage
product_urls=(
  "https://store.com/product1"
  "https://store.com/product2"
  "https://store.com/product3"
)

extract_product_data "${product_urls[@]}"
```

## News Aggregation

```bash
# Aggregate news from multiple sources
aggregate_news() {
  local topic="$1"
  shift
  local urls=("$@")
  
  # Build prompt with URLs
  local prompt="Analyze these news articles about $topic:\n"
  for url in "${urls[@]}"; do
    prompt="${prompt}- ${url}\n"
  done
  
  prompt="${prompt}\nProvide:\n1. Common themes across all articles\n2. Unique perspectives from each source\n3. Overall summary of the situation\n4. Key facts and figures mentioned"
  
  curl -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "'"$prompt"'"
        }]
      }],
      "tools": [{
        "urlContext": {}
      }]
    }'
}

# Example
news_urls=(
  "https://techcrunch.com/article1"
  "https://theverge.com/article2"
  "https://wired.com/article3"
)

aggregate_news "AI developments" "${news_urls[@]}"
```

## Research Assistant

```bash
#!/bin/bash
# research_assistant.sh

analyze_topic() {
  local topic="$1"
  local urls="$2"  # JSON array of URLs
  
  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "Research '"$topic"' using these sources:\n'"$(echo "$urls" | jq -r '.[] | "- " + .')"'\n\nProvide:\n- Executive summary\n- Key findings from each source\n- Common agreements\n- Contradictions or debates\n- Gaps in information\n- Recommendations for further research"
        }]
      }],
      "tools": [{
        "urlContext": {}
      }]
    }'
}

fact_check() {
  local claim="$1"
  local sources="$2"  # JSON array of URLs
  
  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "Fact-check this claim: \"'"$claim"'\"\n\nUsing these sources:\n'"$(echo "$sources" | jq -r '.[] | "- " + .')"'\n\nDetermine:\n1. Is the claim supported?\n2. What evidence exists?\n3. Are there contradictions?\n4. Overall verdict with confidence level"
        }]
      }],
      "tools": [{
        "urlContext": {}
      }]
    }'
}

# Usage
climate_urls='[
  "https://climate.nasa.gov/evidence/",
  "https://www.ipcc.ch/report/ar6/wg1/",
  "https://www.noaa.gov/climate"
]'

analyze_topic "climate change impacts" "$climate_urls"
```

## Documentation Parser

```bash
# Parse technical documentation
parse_docs() {
  local query="$1"
  shift
  local doc_urls=("$@")
  
  # Build URL list
  local url_list=""
  for url in "${doc_urls[@]}"; do
    url_list="${url_list}- ${url}\n"
  done
  
  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "Using these documentation pages:\n'"$url_list"'\n\nAnswer this question: '"$query"'\n\nInclude:\n- Direct answer\n- Code examples if relevant\n- Best practices mentioned\n- Related topics to explore"
        }]
      }],
      "tools": [{
        "urlContext": {}
      }]
    }'
}

# Example
api_docs=(
  "https://api.example.com/docs/authentication"
  "https://api.example.com/docs/endpoints"
  "https://api.example.com/docs/errors"
)

parse_docs "How do I handle rate limiting?" "${api_docs[@]}"
```

## Competitive Analysis

```bash
# Analyze competitor websites
competitive_analysis() {
  cat > /tmp/request.json <<EOF
{
  "contents": [{
    "parts": [{
      "text": "Analyze these company websites:\n$(printf '- %s\n' "$@")\n\nCompare:\n1. Product offerings\n2. Pricing strategies\n3. Target audience\n4. Unique value propositions\n5. Website user experience\n\nCreate a competitive analysis matrix."
    }]
  }],
  "tools": [{
    "urlContext": {}
  }],
  "generationConfig": {
    "responseMimeType": "application/json"
  }
}
EOF

  curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d @/tmp/request.json | jq '.'
  
  rm /tmp/request.json
}

# Usage
competitive_analysis \
  "https://company1.com" \
  "https://company2.com" \
  "https://company3.com"
```

## Batch Processing

```bash
#!/bin/bash
# batch_url_analysis.sh

batch_analyze() {
  local analysis_type="$1"
  local max_urls=20
  
  # Read URL groups from stdin (JSON format)
  # Expected: {"group_name": ["url1", "url2", ...], ...}
  local input=$(cat)
  
  echo "$input" | jq -r 'to_entries | .[] | .key as $group | .value | {group: $group, urls: .}' | while read -r line; do
    local group=$(echo "$line" | jq -r '.group')
    local urls=$(echo "$line" | jq -r '.urls | .[:20] | map("- " + .) | join("\n")')
    local count=$(echo "$line" | jq -r '.urls | length')
    
    echo "Processing group: $group (${count} URLs)"
    
    response=$(curl -s -X POST \
      "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "contents": [{
          "parts": [{
            "text": "Analyze these '"$group"' pages for '"$analysis_type"':\n'"$urls"'"
          }]
        }],
        "tools": [{
          "urlContext": {}
        }]
      }')
    
    echo "{\"group\": \"$group\", \"analysis\": $response}" | jq '.'
    sleep 2  # Rate limiting
  done
}

# Usage
echo '{
  "tech_blogs": [
    "https://techblog1.com/latest",
    "https://techblog2.com/trending"
  ],
  "news_sites": [
    "https://news1.com/tech",
    "https://news2.com/innovation"
  ]
}' | batch_analyze "latest AI trends"
```

## Error Handling

```bash
# Safe URL analysis with validation
safe_url_analysis() {
  local prompt="$1"
  shift
  local urls=("$@")
  
  # Validate URLs
  local valid_urls=()
  for url in "${urls[@]}"; do
    if [[ $url =~ ^https?:// ]]; then
      valid_urls+=("$url")
    else
      echo "Skipping invalid URL: $url" >&2
    fi
  done
  
  # Check if we have valid URLs
  if [ ${#valid_urls[@]} -eq 0 ]; then
    echo "No valid URLs provided" >&2
    return 1
  fi
  
  # Limit to 20 URLs
  if [ ${#valid_urls[@]} -gt 20 ]; then
    echo "Limiting to first 20 URLs (provided: ${#valid_urls[@]})" >&2
    valid_urls=("${valid_urls[@]:0:20}")
  fi
  
  # Build URL list
  local url_list=""
  for url in "${valid_urls[@]}"; do
    url_list="${url_list}- ${url}\n"
  done
  
  # Make request with error handling
  response=$(curl -s -w "\n%{http_code}" -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=$API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "contents": [{
        "parts": [{
          "text": "'"$prompt"'\n\nURLs:\n'"$url_list"'"
        }]
      }],
      "tools": [{
        "urlContext": {}
      }]
    }')
  
  http_code=$(echo "$response" | tail -n1)
  body=$(echo "$response" | sed '$d')
  
  if [ "$http_code" -eq 200 ]; then
    echo "$body" | jq -r '.candidates[0].content.parts[0].text'
  else
    echo "Error: HTTP $http_code" >&2
    echo "$body" | jq '.error' >&2
    return 1
  fi
}
```

## URL Validation Function

```bash
# Validate and clean URLs
validate_urls() {
  local urls=("$@")
  local valid_urls=()
  local invalid_urls=()
  
  for url in "${urls[@]}"; do
    # Basic URL validation
    if [[ $url =~ ^https?://[a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(\.[a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*(/.*)?$ ]]; then
      valid_urls+=("$url")
    else
      invalid_urls+=("$url")
    fi
  done
  
  # Report results
  echo "Valid URLs: ${#valid_urls[@]}"
  echo "Invalid URLs: ${#invalid_urls[@]}"
  
  if [ ${#invalid_urls[@]} -gt 0 ]; then
    echo "Invalid URLs found:"
    printf '  - %s\n' "${invalid_urls[@]}"
  fi
  
  # Return valid URLs
  printf '%s\n' "${valid_urls[@]}"
}
```

## Progress Tracking

```bash
# Analyze URLs with progress reporting
analyze_with_progress() {
  local analysis_prompt="$1"
  local total_urls="$2"
  local chunk_size=20
  local processed=0
  
  while IFS= read -r url; do
    urls_chunk+=("$url")
    
    if [ ${#urls_chunk[@]} -eq $chunk_size ] || [ $((processed + ${#urls_chunk[@]})) -eq $total_urls ]; then
      # Process chunk
      echo "Progress: $((processed + ${#urls_chunk[@]}))/$total_urls"
      
      safe_url_analysis "$analysis_prompt" "${urls_chunk[@]}"
      
      processed=$((processed + ${#urls_chunk[@]}))
      urls_chunk=()
      
      # Rate limiting
      [ $processed -lt $total_urls ] && sleep 2
    fi
  done
}

# Usage
echo "https://example1.com
https://example2.com
https://example3.com" | analyze_with_progress "Analyze these websites" 3
```

## Limitations
- Max 20 URLs per request
- Best with standard web pages
- Experimental feature (may change)
- Daily quotas apply
- Token reduction varies by content

## See Also
- [Grounding](grounding.md) - Google Search integration
- [Document Understanding](document-understanding.md)
- [Text Generation](text-generation.md)