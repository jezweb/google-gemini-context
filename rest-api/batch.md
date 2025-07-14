# Batch in REST API

*Process multiple requests efficiently with Batch API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/batch
API Version: v1beta
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: Bulk processing, cost savings, async execution
-->

## Quick Reference
- **Create**: `POST /v1beta/batches`
- **Status**: `GET /v1beta/batches/{batch_id}`
- **Results**: `GET /v1beta/batches/{batch_id}/results`
- **Cost savings**: ~50% discount for batch requests

## Create Batch Job

```bash
curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/batches" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "requests": [
      {
        "model": "models/gemini-2.5-flash",
        "contents": [{
          "parts": [{
            "text": "Summarize: The quick brown fox jumps over the lazy dog."
          }]
        }]
      },
      {
        "model": "models/gemini-2.5-flash",
        "contents": [{
          "parts": [{
            "text": "Translate to Spanish: Hello, how are you?"
          }]
        }]
      }
    ]
  }'
```

## Response Format

```json
{
  "name": "batches/batch_abc123",
  "state": "PROCESSING",
  "createTime": "2025-01-14T10:30:00Z",
  "totalRequests": 2,
  "completedRequests": 0
}
```

## Check Batch Status

```bash
BATCH_ID="batch_abc123"

curl -X GET \
  "https://generativelanguage.googleapis.com/v1beta/batches/$BATCH_ID" \
  -H "x-goog-api-key: $GEMINI_API_KEY"
```

## Get Batch Results

```bash
curl -X GET \
  "https://generativelanguage.googleapis.com/v1beta/batches/$BATCH_ID/results" \
  -H "x-goog-api-key: $GEMINI_API_KEY"
```

## Complete Batch Processing Script

```bash
#!/bin/bash
# batch_processor.sh

INPUT_FILE="$1"
TASK="$2"

# Create batch from input file
create_batch() {
    local requests="["
    local first=true
    
    while IFS= read -r line; do
        if [ "$first" = true ]; then
            first=false
        else
            requests+=","
        fi
        
        requests+='{
            "model": "models/gemini-2.5-flash",
            "contents": [{
                "parts": [{
                    "text": "'"$TASK"': '"$line"'"
                }]
            }]
        }'
    done < "$INPUT_FILE"
    
    requests+="]"
    
    curl -s -X POST \
        "https://generativelanguage.googleapis.com/v1beta/batches" \
        -H "x-goog-api-key: $GEMINI_API_KEY" \
        -H "Content-Type: application/json" \
        -d '{
            "requests": '"$requests"'
        }'
}

# Monitor batch progress
monitor_batch() {
    local batch_id="$1"
    
    while true; do
        RESPONSE=$(curl -s -X GET \
            "https://generativelanguage.googleapis.com/v1beta/batches/$batch_id" \
            -H "x-goog-api-key: $GEMINI_API_KEY")
        
        STATE=$(echo "$RESPONSE" | jq -r '.state')
        COMPLETED=$(echo "$RESPONSE" | jq -r '.completedRequests // 0')
        TOTAL=$(echo "$RESPONSE" | jq -r '.totalRequests // 0')
        
        echo "Batch $batch_id: $STATE ($COMPLETED/$TOTAL)"
        
        if [ "$STATE" = "COMPLETED" ]; then
            return 0
        elif [ "$STATE" = "FAILED" ]; then
            echo "Batch failed!"
            return 1
        fi
        
        sleep 30
    done
}

# Get and display results
get_results() {
    local batch_id="$1"
    
    curl -s -X GET \
        "https://generativelanguage.googleapis.com/v1beta/batches/$batch_id/results" \
        -H "x-goog-api-key: $GEMINI_API_KEY" | \
    jq -r '.results[] | .response.candidates[0].content.parts[0].text'
}

# Main execution
echo "Creating batch job..."
BATCH_RESPONSE=$(create_batch)
BATCH_ID=$(echo "$BATCH_RESPONSE" | jq -r '.name' | sed 's/batches\///')

if [ "$BATCH_ID" = "null" ]; then
    echo "Failed to create batch:"
    echo "$BATCH_RESPONSE" | jq '.error'
    exit 1
fi

echo "Batch created: $BATCH_ID"
echo "Monitoring progress..."

if monitor_batch "$BATCH_ID"; then
    echo "Getting results..."
    get_results "$BATCH_ID"
else
    echo "Batch processing failed"
    exit 1
fi
```

## Python Implementation

```python
import requests
import json
import time

class BatchAPI:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://generativelanguage.googleapis.com/v1beta"
        self.headers = {
            "x-goog-api-key": api_key,
            "Content-Type": "application/json"
        }
    
    def create_batch(self, requests):
        url = f"{self.base_url}/batches"
        
        data = {"requests": requests}
        
        response = requests.post(url, headers=self.headers, json=data)
        result = response.json()
        
        if "error" in result:
            raise Exception(f"Batch creation failed: {result['error']['message']}")
        
        return result
    
    def get_batch_status(self, batch_id):
        url = f"{self.base_url}/batches/{batch_id}"
        response = requests.get(url, headers=self.headers)
        return response.json()
    
    def get_batch_results(self, batch_id):
        url = f"{self.base_url}/batches/{batch_id}/results"
        response = requests.get(url, headers=self.headers)
        return response.json()
    
    def wait_for_completion(self, batch_id, poll_interval=30):
        while True:
            status = self.get_batch_status(batch_id)
            
            state = status.get("state")
            completed = status.get("completedRequests", 0)
            total = status.get("totalRequests", 0)
            
            print(f"Batch {batch_id}: {state} ({completed}/{total})")
            
            if state == "COMPLETED":
                return self.get_batch_results(batch_id)
            elif state == "FAILED":
                raise Exception(f"Batch failed: {status.get('error', 'Unknown error')}")
            
            time.sleep(poll_interval)
    
    def process_texts(self, texts, task_template):
        # Create batch requests
        requests = []
        for text in texts:
            request = {
                "model": "models/gemini-2.5-flash",
                "contents": [{
                    "parts": [{
                        "text": task_template.format(text=text)
                    }]
                }]
            }
            requests.append(request)
        
        # Submit batch
        batch = self.create_batch(requests)
        batch_id = batch["name"].replace("batches/", "")
        
        print(f"Created batch: {batch_id}")
        
        # Wait for results
        results = self.wait_for_completion(batch_id)
        
        # Extract text responses
        responses = []
        for result in results.get("results", []):
            text = result["response"]["candidates"][0]["content"]["parts"][0]["text"]
            responses.append(text)
        
        return responses

# Usage
batch_api = BatchAPI(os.environ["GEMINI_API_KEY"])

texts = [
    "The weather is sunny today.",
    "I love reading books in the evening.",
    "Technology is advancing rapidly."
]

task_template = "Analyze the sentiment of: {text}"

results = batch_api.process_texts(texts, task_template)
for i, result in enumerate(results):
    print(f"Text {i+1} sentiment: {result}")
```

## Large Dataset Processing

```bash
#!/bin/bash
# large_dataset_batch.sh

DATASET_FILE="$1"
BATCH_SIZE=100
TASK="Summarize this text"

# Split dataset into batches
split_dataset() {
    local file="$1"
    local size="$2"
    
    split -l "$size" "$file" batch_chunk_
}

# Process single batch chunk
process_chunk() {
    local chunk_file="$1"
    local chunk_num="$2"
    
    echo "Processing chunk $chunk_num..."
    
    # Create batch request
    local requests="["
    local first=true
    
    while IFS= read -r line; do
        if [ "$first" = true ]; then
            first=false
        else
            requests+=","
        fi
        
        requests+='{
            "model": "models/gemini-2.5-flash",
            "contents": [{
                "parts": [{
                    "text": "'"$TASK"': '"$line"'"
                }]
            }]
        }'
    done < "$chunk_file"
    
    requests+="]"
    
    # Submit batch
    BATCH_RESPONSE=$(curl -s -X POST \
        "https://generativelanguage.googleapis.com/v1beta/batches" \
        -H "x-goog-api-key: $GEMINI_API_KEY" \
        -H "Content-Type: application/json" \
        -d '{
            "requests": '"$requests"'
        }')
    
    BATCH_ID=$(echo "$BATCH_RESPONSE" | jq -r '.name' | sed 's/batches\///')
    echo "Chunk $chunk_num batch ID: $BATCH_ID"
    
    # Wait for completion
    while true; do
        STATUS=$(curl -s -X GET \
            "https://generativelanguage.googleapis.com/v1beta/batches/$BATCH_ID" \
            -H "x-goog-api-key: $GEMINI_API_KEY")
        
        STATE=$(echo "$STATUS" | jq -r '.state')
        
        if [ "$STATE" = "COMPLETED" ]; then
            break
        elif [ "$STATE" = "FAILED" ]; then
            echo "Chunk $chunk_num failed!"
            return 1
        fi
        
        sleep 30
    done
    
    # Get results
    curl -s -X GET \
        "https://generativelanguage.googleapis.com/v1beta/batches/$BATCH_ID/results" \
        -H "x-goog-api-key: $GEMINI_API_KEY" | \
    jq -r '.results[] | .response.candidates[0].content.parts[0].text' > "results_chunk_$chunk_num.txt"
    
    echo "Chunk $chunk_num completed"
}

# Main processing
echo "Splitting dataset into chunks of $BATCH_SIZE..."
split_dataset "$DATASET_FILE" "$BATCH_SIZE"

# Process each chunk
chunk_num=1
for chunk_file in batch_chunk_*; do
    process_chunk "$chunk_file" "$chunk_num"
    chunk_num=$((chunk_num + 1))
done

# Combine results
echo "Combining results..."
cat results_chunk_*.txt > final_results.txt

# Cleanup
rm batch_chunk_* results_chunk_*.txt

echo "Processing complete. Results in final_results.txt"
```

## Batch with Metadata

```json
{
  "requests": [
    {
      "model": "models/gemini-2.5-flash",
      "contents": [{
        "parts": [{
          "text": "Analyze sentiment: I love this product!"
        }]
      }],
      "metadata": {
        "customer_id": "12345",
        "review_id": "rev_001",
        "source": "website"
      }
    },
    {
      "model": "models/gemini-2.5-flash",
      "contents": [{
        "parts": [{
          "text": "Analyze sentiment: The service was terrible."
        }]
      }],
      "metadata": {
        "customer_id": "67890",
        "review_id": "rev_002",
        "source": "mobile_app"
      }
    }
  ]
}
```

## Cost Estimation Script

```bash
#!/bin/bash
# estimate_batch_cost.sh

INPUT_FILE="$1"

# Count total requests
TOTAL_REQUESTS=$(wc -l < "$INPUT_FILE")

# Estimate tokens (rough calculation)
TOTAL_CHARS=$(wc -c < "$INPUT_FILE")
ESTIMATED_TOKENS=$((TOTAL_CHARS / 4))  # Rough estimate: 4 chars per token

# Calculate costs
REGULAR_COST=$(echo "scale=6; $ESTIMATED_TOKENS * 0.000001" | bc)
BATCH_COST=$(echo "scale=6; $REGULAR_COST * 0.5" | bc)
SAVINGS=$(echo "scale=6; $REGULAR_COST - $BATCH_COST" | bc)

echo "Batch Cost Estimation"
echo "===================="
echo "Total requests: $TOTAL_REQUESTS"
echo "Estimated tokens: $ESTIMATED_TOKENS"
echo "Regular API cost: \$$REGULAR_COST"
echo "Batch API cost: \$$BATCH_COST"
echo "Estimated savings: \$$SAVINGS"
echo ""

# Recommend batch size
if [ "$TOTAL_REQUESTS" -gt 1000 ]; then
    RECOMMENDED_BATCHES=$(((TOTAL_REQUESTS + 999) / 1000))
    echo "Recommendation: Split into $RECOMMENDED_BATCHES batches of ~1000 requests each"
else
    echo "Recommendation: Process as single batch"
fi
```

## JavaScript Implementation

```javascript
class BatchAPI {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseUrl = "https://generativelanguage.googleapis.com/v1beta";
    }
    
    async createBatch(requests) {
        const response = await fetch(`${this.baseUrl}/batches`, {
            method: 'POST',
            headers: {
                'x-goog-api-key': this.apiKey,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ requests })
        });
        
        const result = await response.json();
        
        if (result.error) {
            throw new Error(`Batch creation failed: ${result.error.message}`);
        }
        
        return result;
    }
    
    async getBatchStatus(batchId) {
        const response = await fetch(`${this.baseUrl}/batches/${batchId}`, {
            headers: {
                'x-goog-api-key': this.apiKey
            }
        });
        
        return await response.json();
    }
    
    async getBatchResults(batchId) {
        const response = await fetch(`${this.baseUrl}/batches/${batchId}/results`, {
            headers: {
                'x-goog-api-key': this.apiKey
            }
        });
        
        return await response.json();
    }
    
    async waitForCompletion(batchId, pollInterval = 30000) {
        while (true) {
            const status = await this.getBatchStatus(batchId);
            
            console.log(`Batch ${batchId}: ${status.state} (${status.completedRequests || 0}/${status.totalRequests || 0})`);
            
            if (status.state === "COMPLETED") {
                return await this.getBatchResults(batchId);
            } else if (status.state === "FAILED") {
                throw new Error(`Batch failed: ${status.error || 'Unknown error'}`);
            }
            
            await new Promise(resolve => setTimeout(resolve, pollInterval));
        }
    }
    
    async processTexts(texts, taskTemplate) {
        // Create batch requests
        const requests = texts.map(text => ({
            model: "models/gemini-2.5-flash",
            contents: [{
                parts: [{
                    text: taskTemplate.replace("{text}", text)
                }]
            }]
        }));
        
        // Submit batch
        const batch = await this.createBatch(requests);
        const batchId = batch.name.replace("batches/", "");
        
        console.log(`Created batch: ${batchId}`);
        
        // Wait for results
        const results = await this.waitForCompletion(batchId);
        
        // Extract text responses
        return results.results.map(result => 
            result.response.candidates[0].content.parts[0].text
        );
    }
}

// Usage
const batchAPI = new BatchAPI(process.env.GEMINI_API_KEY);

const texts = [
    "The weather is sunny today.",
    "I love reading books in the evening.",
    "Technology is advancing rapidly."
];

const results = await batchAPI.processTexts(texts, "Analyze the sentiment of: {text}");
results.forEach((result, i) => {
    console.log(`Text ${i + 1} sentiment: ${result}`);
});
```

## Error Handling

```bash
# Check for batch creation errors
BATCH_RESPONSE=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/batches" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d "$REQUEST_DATA")

if echo "$BATCH_RESPONSE" | jq -e '.error' > /dev/null; then
    ERROR_MSG=$(echo "$BATCH_RESPONSE" | jq -r '.error.message')
    
    if [[ "$ERROR_MSG" == *"quota exceeded"* ]]; then
        echo "Batch quota exceeded - try smaller batches"
    elif [[ "$ERROR_MSG" == *"invalid request"* ]]; then
        echo "Check request format and model availability"
    else
        echo "Error: $ERROR_MSG"
    fi
    exit 1
fi

# Extract batch ID
BATCH_ID=$(echo "$BATCH_RESPONSE" | jq -r '.name' | sed 's/batches\///')
echo "Batch created: $BATCH_ID"
```

## Best Practices

1. **Batch Size**: Keep batches under 1000 requests for optimal processing
2. **Cost Savings**: Use for large-scale processing to get ~50% discount
3. **Monitoring**: Regularly check batch status and progress
4. **Error Handling**: Handle failed requests gracefully
5. **Organization**: Use metadata to track and organize results
6. **Optimization**: Group similar requests for better efficiency

## See Also
- [Authentication](authentication.md) - API key setup
- [Rate Limits](rate-limits.md) - Usage limits
- [Error Handling](error-handling.md) - Handle failures