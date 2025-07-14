# Document Understanding in REST API

*Analyze documents, PDFs, and structured content via REST API*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/document-processing
API Version: v1beta
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: PDF analysis, table extraction, document structure
-->

## Quick Reference
- **Endpoint**: `/v1beta/models/{model}:generateContent`
- **File types**: PDF, DOCX, TXT, images with text
- **Max size**: 20MB inline, 2GB via File API
- **Encoding**: Base64 in JSON

## Basic Document Analysis

```bash
# Encode PDF to base64
PDF_BASE64=$(base64 -i contract.pdf)

curl -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "inline_data": {
            "mime_type": "application/pdf",
            "data": "'"$PDF_BASE64"'"
          }
        },
        {
          "text": "Analyze this document and provide a summary of key points"
        }
      ]
    }]
  }'
```

## Large Document Processing (File API)

```bash
#!/bin/bash
# upload_and_analyze.sh

DOCUMENT_FILE="$1"
ANALYSIS_PROMPT="$2"

# Step 1: Upload document
echo "Uploading document..."
FILE_SIZE=$(stat -c%s "$DOCUMENT_FILE" 2>/dev/null || stat -f%z "$DOCUMENT_FILE")
MIME_TYPE="application/pdf"

INIT_RESPONSE=$(curl -s -X POST \
  "https://generativelanguage.googleapis.com/upload/v1beta/files" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "X-Goog-Upload-Command: start" \
  -H "X-Goog-Upload-Header-Content-Length: $FILE_SIZE" \
  -H "X-Goog-Upload-Header-Content-Type: $MIME_TYPE" \
  -H "Content-Type: application/json" \
  -d '{"file": {"display_name": "'"$(basename "$DOCUMENT_FILE")"'"}}' \
  -D -)

UPLOAD_URL=$(echo "$INIT_RESPONSE" | grep -i "x-goog-upload-url:" | cut -d' ' -f2 | tr -d '\r')

# Upload file data
echo "Uploading file data..."
UPLOAD_RESPONSE=$(curl -s -X POST "$UPLOAD_URL" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Length: $FILE_SIZE" \
  -H "X-Goog-Upload-Offset: 0" \
  -H "X-Goog-Upload-Command: upload, finalize" \
  --data-binary @"$DOCUMENT_FILE")

FILE_NAME=$(echo "$UPLOAD_RESPONSE" | jq -r '.file.name')
FILE_URI=$(echo "$UPLOAD_RESPONSE" | jq -r '.file.uri')

# Wait for processing
echo "Processing document..."
while true; do
  STATUS=$(curl -s \
    "https://generativelanguage.googleapis.com/v1beta/$FILE_NAME" \
    -H "x-goog-api-key: $GEMINI_API_KEY" | jq -r '.state')
  
  if [ "$STATUS" = "ACTIVE" ]; then
    echo "Document ready!"
    break
  elif [ "$STATUS" = "FAILED" ]; then
    echo "Document processing failed!"
    exit 1
  fi
  
  echo -n "."
  sleep 5
done

# Analyze document
echo "Analyzing document..."
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{
      "parts": [
        {
          "file_data": {
            "file_uri": "'"$FILE_URI"'",
            "mime_type": "'"$MIME_TYPE"'"
          }
        },
        {
          "text": "'"$ANALYSIS_PROMPT"'"
        }
      ]
    }]
  }' | jq -r '.candidates[0].content.parts[0].text'

# Clean up
curl -s -X DELETE \
  "https://generativelanguage.googleapis.com/v1beta/$FILE_NAME" \
  -H "x-goog-api-key: $GEMINI_API_KEY"
```

## Contract Analysis Script

```bash
#!/bin/bash
# contract_analyzer.sh

CONTRACT_FILE="$1"

analyze_contract_section() {
    local prompt="$1"
    local section_name="$2"
    
    echo "=== $section_name ==="
    
    PDF_BASE64=$(base64 -i "$CONTRACT_FILE")
    
    curl -s -X POST \
        "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
        -H "x-goog-api-key: $GEMINI_API_KEY" \
        -H "Content-Type: application/json" \
        -d '{
            "contents": [{
                "parts": [
                    {
                        "inline_data": {
                            "mime_type": "application/pdf",
                            "data": "'"$PDF_BASE64"'"
                        }
                    },
                    {
                        "text": "'"$prompt"'"
                    }
                ]
            }]
        }' | jq -r '.candidates[0].content.parts[0].text'
    
    echo ""
}

# Analyze different aspects of the contract
analyze_contract_section "Analyze this contract and provide: 1. Contract type and purpose 2. Parties involved 3. Key terms and conditions 4. Important dates and deadlines 5. Financial terms 6. Termination clauses 7. Risk factors or unusual terms" "CONTRACT SUMMARY"

analyze_contract_section "Extract all dates mentioned in this contract and their significance" "KEY DATES"

analyze_contract_section "Extract all financial information, payment terms, and monetary amounts" "FINANCIAL TERMS"

analyze_contract_section "Identify potential risks, liabilities, and unfavorable terms in this contract" "RISK ASSESSMENT"
```

## Python Implementation

```python
import requests
import base64
import json
import time

class DocumentAnalysisAPI:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://generativelanguage.googleapis.com/v1beta"
        self.headers = {
            "x-goog-api-key": api_key,
            "Content-Type": "application/json"
        }
    
    def analyze_document(self, file_path, prompt):
        # Read and encode file
        with open(file_path, "rb") as f:
            file_data = base64.b64encode(f.read()).decode()
        
        # Determine MIME type
        mime_type = self.get_mime_type(file_path)
        
        # Analyze document
        url = f"{self.base_url}/models/gemini-2.5-flash:generateContent"
        
        data = {
            "contents": [{
                "parts": [
                    {
                        "inline_data": {
                            "mime_type": mime_type,
                            "data": file_data
                        }
                    },
                    {
                        "text": prompt
                    }
                ]
            }]
        }
        
        response = requests.post(url, headers=self.headers, json=data)
        result = response.json()
        
        if "error" in result:
            raise Exception(f"Analysis failed: {result['error']['message']}")
        
        return result["candidates"][0]["content"]["parts"][0]["text"]
    
    def get_mime_type(self, file_path):
        ext = file_path.lower().split('.')[-1]
        mime_types = {
            'pdf': 'application/pdf',
            'docx': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
            'txt': 'text/plain',
            'jpg': 'image/jpeg',
            'png': 'image/png'
        }
        return mime_types.get(ext, 'application/octet-stream')
    
    def extract_tables(self, file_path):
        return self.analyze_document(
            file_path,
            "Extract all tables and convert them to JSON format"
        )
    
    def extract_entities(self, file_path):
        return self.analyze_document(
            file_path,
            "Extract all named entities (people, organizations, dates, amounts)"
        )
    
    def analyze_financial_report(self, file_path):
        return self.analyze_document(
            file_path,
            """Analyze this financial report and extract:
            1. Revenue and profit figures
            2. Key financial ratios
            3. Year-over-year changes
            4. Cash flow information
            5. Balance sheet highlights
            6. Risk factors mentioned
            7. Management outlook"""
        )
    
    def document_qa(self, file_path, questions):
        results = {}
        
        for question in questions:
            answer = self.analyze_document(file_path, f"Question: {question}")
            results[question] = answer
        
        return results

# Usage
api = DocumentAnalysisAPI(os.environ["GEMINI_API_KEY"])

# Analyze document
summary = api.analyze_document("contract.pdf", "Provide a comprehensive summary")
print("Document Summary:")
print(summary)

# Extract tables
tables = api.extract_tables("financial_report.pdf")
print("\nExtracted Tables:")
print(tables)

# Document Q&A
questions = [
    "What is the main purpose of this document?",
    "Who are the key stakeholders mentioned?",
    "What are the important deadlines?"
]

answers = api.document_qa("contract.pdf", questions)
for question, answer in answers.items():
    print(f"\nQ: {question}")
    print(f"A: {answer}")
```

## Table Extraction

```json
{
  "contents": [{
    "parts": [
      {
        "inline_data": {
          "mime_type": "application/pdf",
          "data": "BASE64_ENCODED_PDF"
        }
      },
      {
        "text": "Extract all tables from this document and convert them to structured JSON format with headers and data clearly separated"
      }
    ]
  }]
}
```

## Financial Document Analysis

```bash
#!/bin/bash
# financial_analyzer.sh

REPORT_FILE="$1"

analyze_financial_section() {
    local prompt="$1"
    local section_name="$2"
    
    echo "=== $section_name ==="
    
    PDF_BASE64=$(base64 -i "$REPORT_FILE")
    
    curl -s -X POST \
        "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
        -H "x-goog-api-key: $GEMINI_API_KEY" \
        -H "Content-Type: application/json" \
        -d '{
            "contents": [{
                "parts": [
                    {
                        "inline_data": {
                            "mime_type": "application/pdf",
                            "data": "'"$PDF_BASE64"'"
                        }
                    },
                    {
                        "text": "'"$prompt"'"
                    }
                ]
            }]
        }' | jq -r '.candidates[0].content.parts[0].text'
    
    echo ""
}

# Financial analysis sections
analyze_financial_section "Analyze this financial report and extract: 1. Revenue and profit figures 2. Key financial ratios 3. Year-over-year changes 4. Cash flow information 5. Balance sheet highlights 6. Risk factors mentioned 7. Management outlook" "FINANCIAL OVERVIEW"

analyze_financial_section "Extract all financial tables and convert them to structured JSON format" "FINANCIAL TABLES"

analyze_financial_section "Calculate key financial ratios from this report: - Liquidity ratios (current ratio, quick ratio) - Profitability ratios (ROE, ROA, profit margin) - Leverage ratios (debt-to-equity, interest coverage) - Efficiency ratios (asset turnover, inventory turnover) Show calculations and explain what each ratio indicates" "FINANCIAL RATIOS"
```

## Scientific Paper Analysis

```bash
#!/bin/bash
# paper_analyzer.sh

PAPER_FILE="$1"
AUDIENCE="${2:-general}"

# Encode PDF
PDF_BASE64=$(base64 -i "$PAPER_FILE")

case $AUDIENCE in
    "general")
        PROMPT="Summarize this scientific paper for a general audience without technical jargon"
        ;;
    "technical")
        PROMPT="Provide a technical summary for researchers in the field"
        ;;
    "executive")
        PROMPT="Create an executive summary focusing on practical implications"
        ;;
    "student")
        PROMPT="Explain this paper in a way undergraduate students would understand"
        ;;
    *)
        PROMPT="Analyze this scientific paper and provide: 1. Title and authors 2. Abstract summary 3. Research methodology 4. Key findings and results 5. Conclusions and implications 6. References to key related work 7. Limitations and future work"
        ;;
esac

curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-pro:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
        "contents": [{
            "parts": [
                {
                    "inline_data": {
                        "mime_type": "application/pdf",
                        "data": "'"$PDF_BASE64"'"
                    }
                },
                {
                    "text": "'"$PROMPT"'"
                }
            ]
        }]
    }' | jq -r '.candidates[0].content.parts[0].text'
```

## Batch Document Processing

```bash
#!/bin/bash
# batch_document_processor.sh

DOCUMENT_DIR="$1"
ANALYSIS_TYPE="${2:-summary}"

# Create batch request
create_batch_request() {
    local documents=()
    local requests="["
    local first=true
    
    for doc in "$DOCUMENT_DIR"/*.pdf; do
        if [ -f "$doc" ]; then
            if [ "$first" = true ]; then
                first=false
            else
                requests+=","
            fi
            
            DOC_BASE64=$(base64 -i "$doc")
            
            requests+='{
                "model": "models/gemini-2.5-flash",
                "contents": [{
                    "parts": [
                        {
                            "inline_data": {
                                "mime_type": "application/pdf",
                                "data": "'"$DOC_BASE64"'"
                            }
                        },
                        {
                            "text": "Analyze this document and provide: '"$ANALYSIS_TYPE"'"
                        }
                    ]
                }],
                "metadata": {
                    "document_name": "'"$(basename "$doc")"'"
                }
            }'
        fi
    done
    
    requests+="]"
    echo "$requests"
}

# Submit batch
echo "Creating batch for documents in $DOCUMENT_DIR..."
BATCH_REQUESTS=$(create_batch_request)

BATCH_RESPONSE=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/batches" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
        "requests": '"$BATCH_REQUESTS"'
    }')

BATCH_ID=$(echo "$BATCH_RESPONSE" | jq -r '.name' | sed 's/batches\///')

if [ "$BATCH_ID" = "null" ]; then
    echo "Failed to create batch:"
    echo "$BATCH_RESPONSE" | jq '.error'
    exit 1
fi

echo "Batch created: $BATCH_ID"

# Monitor batch
while true; do
    STATUS=$(curl -s -X GET \
        "https://generativelanguage.googleapis.com/v1beta/batches/$BATCH_ID" \
        -H "x-goog-api-key: $GEMINI_API_KEY")
    
    STATE=$(echo "$STATUS" | jq -r '.state')
    COMPLETED=$(echo "$STATUS" | jq -r '.completedRequests // 0')
    TOTAL=$(echo "$STATUS" | jq -r '.totalRequests // 0')
    
    echo "Batch status: $STATE ($COMPLETED/$TOTAL)"
    
    if [ "$STATE" = "COMPLETED" ]; then
        break
    elif [ "$STATE" = "FAILED" ]; then
        echo "Batch failed!"
        exit 1
    fi
    
    sleep 30
done

# Get results
echo "Getting results..."
curl -s -X GET \
    "https://generativelanguage.googleapis.com/v1beta/batches/$BATCH_ID/results" \
    -H "x-goog-api-key: $GEMINI_API_KEY" | \
jq -r '.results[] | "=== " + .metadata.document_name + " ===\n" + .response.candidates[0].content.parts[0].text + "\n"'
```

## JavaScript Implementation

```javascript
class DocumentAnalysisAPI {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.baseUrl = "https://generativelanguage.googleapis.com/v1beta";
    }
    
    async analyzeDocument(filePath, prompt) {
        // Read file (Node.js)
        const fs = require('fs');
        const fileData = fs.readFileSync(filePath);
        const base64Data = fileData.toString('base64');
        
        const mimeType = this.getMimeType(filePath);
        
        const response = await fetch(
            `${this.baseUrl}/models/gemini-2.5-flash:generateContent`,
            {
                method: 'POST',
                headers: {
                    'x-goog-api-key': this.apiKey,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    contents: [{
                        parts: [
                            {
                                inline_data: {
                                    mime_type: mimeType,
                                    data: base64Data
                                }
                            },
                            {
                                text: prompt
                            }
                        ]
                    }]
                })
            }
        );
        
        const result = await response.json();
        
        if (result.error) {
            throw new Error(`Analysis failed: ${result.error.message}`);
        }
        
        return result.candidates[0].content.parts[0].text;
    }
    
    getMimeType(filePath) {
        const ext = filePath.toLowerCase().split('.').pop();
        const mimeTypes = {
            pdf: 'application/pdf',
            docx: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
            txt: 'text/plain',
            jpg: 'image/jpeg',
            png: 'image/png'
        };
        return mimeTypes[ext] || 'application/octet-stream';
    }
    
    async extractTables(filePath) {
        return await this.analyzeDocument(
            filePath,
            "Extract all tables and convert them to JSON format"
        );
    }
    
    async documentQA(filePath, questions) {
        const results = {};
        
        for (const question of questions) {
            const answer = await this.analyzeDocument(filePath, `Question: ${question}`);
            results[question] = answer;
        }
        
        return results;
    }
}

// Usage
const api = new DocumentAnalysisAPI(process.env.GEMINI_API_KEY);

const summary = await api.analyzeDocument("contract.pdf", "Provide a comprehensive summary");
console.log("Document Summary:", summary);
```

## Error Handling

```bash
# Check for document analysis errors
RESPONSE=$(curl -s -X POST \
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent" \
    -H "x-goog-api-key: $GEMINI_API_KEY" \
    -H "Content-Type: application/json" \
    -d "$REQUEST_DATA")

if echo "$RESPONSE" | jq -e '.error' > /dev/null; then
    ERROR_MSG=$(echo "$RESPONSE" | jq -r '.error.message')
    
    if [[ "$ERROR_MSG" == *"file too large"* ]]; then
        echo "Document too large - use File API for files >20MB"
    elif [[ "$ERROR_MSG" == *"unsupported format"* ]]; then
        echo "Unsupported document format - use PDF, DOCX, or TXT"
    elif [[ "$ERROR_MSG" == *"corrupted"* ]]; then
        echo "Document appears to be corrupted"
    else
        echo "Error: $ERROR_MSG"
    fi
    exit 1
fi

# Extract result
echo "$RESPONSE" | jq -r '.candidates[0].content.parts[0].text'
```

## Best Practices

1. **File Size**: Use File API for documents >20MB
2. **Format Support**: PDF works best, DOCX and images also supported
3. **Specific Prompts**: Be specific about what to extract or analyze
4. **Batch Processing**: Use batches for multiple documents
5. **Error Handling**: Handle corrupted or unsupported files
6. **Cleanup**: Delete uploaded files when done

## See Also
- [File API](file-api.md) - File upload details
- [Vision](vision.md) - Image-based document analysis
- [Batch](batch.md) - Batch document processing