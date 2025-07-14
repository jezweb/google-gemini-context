# Document Understanding in Python

*Analyze documents, PDFs, and structured content*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/document-processing
SDK: google-genai (Python)
Verified: 2025-01-14
Models: gemini-2.5-flash, gemini-2.5-pro
Key Features: PDF analysis, table extraction, document structure
-->

## Quick Reference
- **File types**: PDF, DOCX, TXT, images with text
- **Max size**: 20MB inline, 2GB via File API
- **Use cases**: Contract analysis, report extraction, data mining
- **Features**: Table extraction, structure analysis, Q&A

## Basic Document Analysis

```python
from google import genai
import pathlib
import base64

client = genai.Client()

# Analyze PDF document
pdf_file = pathlib.Path("contract.pdf")

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {"mime_type": "application/pdf", "data": pdf_file.read_bytes()},
        {"text": "Analyze this document and provide a summary of key points"}
    ]
)

print(response.text)
```

## Large Document Processing (File API)

```python
import time

# Upload large document
print("Uploading document...")
document_file = client.files.upload(path="large_report.pdf")

# Wait for processing
while document_file.state == "PROCESSING":
    print(".", end="", flush=True)
    time.sleep(5)
    document_file = client.files.get(name=document_file.name)

if document_file.state != "ACTIVE":
    raise ValueError(f"Document processing failed: {document_file.state}")

print(f"\nDocument ready: {document_file.uri}")

# Analyze document
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        document_file,
        "Extract all tables and convert them to structured data"
    ]
)

print(response.text)
```

## Document Analyzer Class

```python
class DocumentAnalyzer:
    def __init__(self):
        self.client = genai.Client()
    
    def analyze_document(self, file_path, analysis_type="summary"):
        """Analyze documents with different focus areas"""
        file_size = pathlib.Path(file_path).stat().st_size
        
        # Handle file size
        if file_size < 20 * 1024 * 1024:  # Under 20MB
            document_data = pathlib.Path(file_path).read_bytes()
            contents = [
                {"mime_type": self._get_mime_type(file_path), "data": document_data},
                {"text": self._get_analysis_prompt(analysis_type)}
            ]
        else:
            # Upload large file
            document_file = self._upload_document(file_path)
            contents = [
                document_file,
                {"text": self._get_analysis_prompt(analysis_type)}
            ]
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=contents
        )
        
        return response.text
    
    def _get_analysis_prompt(self, analysis_type):
        """Get analysis prompts for different types"""
        prompts = {
            "summary": "Provide a comprehensive summary of this document",
            "tables": "Extract all tables and convert them to JSON format",
            "key_points": "Extract key points, decisions, and action items",
            "entities": "Extract all named entities (people, organizations, dates, amounts)",
            "structure": "Analyze the document structure and create an outline",
            "qa": "List potential questions this document answers",
            "compliance": "Check for compliance-related content and requirements",
            "financial": "Extract all financial information, numbers, and calculations"
        }
        return prompts.get(analysis_type, prompts["summary"])
    
    def _get_mime_type(self, file_path):
        """Determine MIME type from file extension"""
        ext = pathlib.Path(file_path).suffix.lower()
        mime_types = {
            '.pdf': 'application/pdf',
            '.docx': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
            '.txt': 'text/plain',
            '.jpg': 'image/jpeg',
            '.png': 'image/png'
        }
        return mime_types.get(ext, 'application/octet-stream')
    
    def _upload_document(self, file_path):
        """Upload and wait for document processing"""
        print(f"Uploading {file_path}...")
        document_file = self.client.files.upload(path=file_path)
        
        while document_file.state == "PROCESSING":
            print(".", end="", flush=True)
            time.sleep(5)
            document_file = self.client.files.get(name=document_file.name)
        
        if document_file.state != "ACTIVE":
            raise ValueError(f"Upload failed: {document_file.state}")
        
        print("\nUpload complete!")
        return document_file
    
    def extract_tables(self, file_path):
        """Extract tables from document"""
        return self.analyze_document(file_path, "tables")
    
    def extract_entities(self, file_path):
        """Extract named entities from document"""
        return self.analyze_document(file_path, "entities")
    
    def get_document_qa(self, file_path, questions):
        """Answer specific questions about document"""
        file_size = pathlib.Path(file_path).stat().st_size
        
        if file_size < 20 * 1024 * 1024:
            document_data = pathlib.Path(file_path).read_bytes()
            base_contents = [
                {"mime_type": self._get_mime_type(file_path), "data": document_data}
            ]
        else:
            document_file = self._upload_document(file_path)
            base_contents = [document_file]
        
        results = {}
        for question in questions:
            contents = base_contents + [{"text": f"Question: {question}"}]
            
            response = self.client.models.generate_content(
                model="gemini-2.5-flash",
                contents=contents
            )
            
            results[question] = response.text
        
        return results

# Usage
analyzer = DocumentAnalyzer()

# Analyze contract
summary = analyzer.analyze_document("contract.pdf", "summary")
print("Contract Summary:")
print(summary)

# Extract tables
tables = analyzer.extract_tables("financial_report.pdf")
print("\nExtracted Tables:")
print(tables)

# Document Q&A
questions = [
    "What is the total contract value?",
    "When does this contract expire?",
    "Who are the parties involved?"
]

answers = analyzer.get_document_qa("contract.pdf", questions)
for question, answer in answers.items():
    print(f"Q: {question}")
    print(f"A: {answer}\n")
```

## Contract Analysis

```python
class ContractAnalyzer:
    def __init__(self):
        self.client = genai.Client()
    
    def analyze_contract(self, contract_path):
        """Comprehensive contract analysis"""
        document_data = pathlib.Path(contract_path).read_bytes()
        
        # Multi-step analysis
        analyses = {}
        
        # 1. Basic summary
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=[
                {"mime_type": "application/pdf", "data": document_data},
                {"text": """Analyze this contract and provide:
                1. Contract type and purpose
                2. Parties involved
                3. Key terms and conditions
                4. Important dates and deadlines
                5. Financial terms
                6. Termination clauses
                7. Risk factors or unusual terms"""}
            ]
        )
        analyses["summary"] = response.text
        
        # 2. Extract key dates
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=[
                {"mime_type": "application/pdf", "data": document_data},
                {"text": "Extract all dates mentioned in this contract and their significance"}
            ]
        )
        analyses["dates"] = response.text
        
        # 3. Financial analysis
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=[
                {"mime_type": "application/pdf", "data": document_data},
                {"text": "Extract all financial information, payment terms, and monetary amounts"}
            ]
        )
        analyses["financial"] = response.text
        
        # 4. Risk assessment
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=[
                {"mime_type": "application/pdf", "data": document_data},
                {"text": "Identify potential risks, liabilities, and unfavorable terms in this contract"}
            ]
        )
        analyses["risks"] = response.text
        
        return analyses
    
    def compare_contracts(self, contract1_path, contract2_path):
        """Compare two contracts"""
        contract1_data = pathlib.Path(contract1_path).read_bytes()
        contract2_data = pathlib.Path(contract2_path).read_bytes()
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=[
                {"text": "Compare these two contracts and highlight key differences:"},
                {"text": "Contract 1:"},
                {"mime_type": "application/pdf", "data": contract1_data},
                {"text": "Contract 2:"},
                {"mime_type": "application/pdf", "data": contract2_data},
                {"text": "Provide a detailed comparison focusing on terms, conditions, and financial aspects"}
            ]
        )
        
        return response.text

# Usage
contract_analyzer = ContractAnalyzer()

# Analyze single contract
analysis = contract_analyzer.analyze_contract("service_agreement.pdf")
for section, content in analysis.items():
    print(f"=== {section.upper()} ===")
    print(content)
    print()

# Compare contracts
comparison = contract_analyzer.compare_contracts("old_contract.pdf", "new_contract.pdf")
print("Contract Comparison:")
print(comparison)
```

## Scientific Paper Analysis

```python
class PaperAnalyzer:
    def __init__(self):
        self.client = genai.Client()
    
    def analyze_paper(self, paper_path):
        """Analyze scientific papers"""
        paper_data = pathlib.Path(paper_path).read_bytes()
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=[
                {"mime_type": "application/pdf", "data": paper_data},
                {"text": """Analyze this scientific paper and provide:
                1. Title and authors
                2. Abstract summary
                3. Research methodology
                4. Key findings and results
                5. Conclusions and implications
                6. References to key related work
                7. Limitations and future work"""}
            ]
        )
        
        return response.text
    
    def extract_citations(self, paper_path):
        """Extract citations and references"""
        paper_data = pathlib.Path(paper_path).read_bytes()
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=[
                {"mime_type": "application/pdf", "data": paper_data},
                {"text": "Extract all citations and references from this paper in a structured format"}
            ]
        )
        
        return response.text
    
    def summarize_for_audience(self, paper_path, audience="general"):
        """Create audience-specific summaries"""
        paper_data = pathlib.Path(paper_path).read_bytes()
        
        audience_prompts = {
            "general": "Summarize this paper for a general audience without technical jargon",
            "technical": "Provide a technical summary for researchers in the field",
            "executive": "Create an executive summary focusing on practical implications",
            "student": "Explain this paper in a way undergraduate students would understand"
        }
        
        prompt = audience_prompts.get(audience, audience_prompts["general"])
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=[
                {"mime_type": "application/pdf", "data": paper_data},
                {"text": prompt}
            ]
        )
        
        return response.text

# Usage
paper_analyzer = PaperAnalyzer()

# Analyze research paper
analysis = paper_analyzer.analyze_paper("research_paper.pdf")
print("Paper Analysis:")
print(analysis)

# Create summaries for different audiences
audiences = ["general", "technical", "executive"]
for audience in audiences:
    summary = paper_analyzer.summarize_for_audience("research_paper.pdf", audience)
    print(f"\n{audience.upper()} SUMMARY:")
    print(summary)
```

## Financial Document Processing

```python
class FinancialDocumentProcessor:
    def __init__(self):
        self.client = genai.Client()
    
    def analyze_financial_report(self, report_path):
        """Analyze financial reports and statements"""
        report_data = pathlib.Path(report_path).read_bytes()
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=[
                {"mime_type": "application/pdf", "data": report_data},
                {"text": """Analyze this financial report and extract:
                1. Revenue and profit figures
                2. Key financial ratios
                3. Year-over-year changes
                4. Cash flow information
                5. Balance sheet highlights
                6. Risk factors mentioned
                7. Management outlook"""}
            ]
        )
        
        return response.text
    
    def extract_financial_tables(self, report_path):
        """Extract and structure financial tables"""
        report_data = pathlib.Path(report_path).read_bytes()
        
        response = self.client.models.generate_content(
            model="gemini-2.5-flash",
            contents=[
                {"mime_type": "application/pdf", "data": report_data},
                {"text": "Extract all financial tables and convert them to structured JSON format"}
            ]
        )
        
        return response.text
    
    def calculate_ratios(self, report_path):
        """Calculate financial ratios from report"""
        report_data = pathlib.Path(report_path).read_bytes()
        
        response = self.client.models.generate_content(
            model="gemini-2.5-pro",
            contents=[
                {"mime_type": "application/pdf", "data": report_data},
                {"text": """Calculate key financial ratios from this report:
                - Liquidity ratios (current ratio, quick ratio)
                - Profitability ratios (ROE, ROA, profit margin)
                - Leverage ratios (debt-to-equity, interest coverage)
                - Efficiency ratios (asset turnover, inventory turnover)
                Show calculations and explain what each ratio indicates"""}
            ]
        )
        
        return response.text

# Usage
financial_processor = FinancialDocumentProcessor()

# Analyze quarterly report
analysis = financial_processor.analyze_financial_report("q4_report.pdf")
print("Financial Analysis:")
print(analysis)

# Extract tables
tables = financial_processor.extract_financial_tables("q4_report.pdf")
print("\nFinancial Tables:")
print(tables)

# Calculate ratios
ratios = financial_processor.calculate_ratios("q4_report.pdf")
print("\nFinancial Ratios:")
print(ratios)
```

## Batch Document Processing

```python
def process_document_batch(document_paths, analysis_type="summary"):
    """Process multiple documents in batch"""
    client = genai.Client()
    requests = []
    
    for doc_path in document_paths:
        # Prepare document data
        doc_data = pathlib.Path(doc_path).read_bytes()
        mime_type = get_mime_type(doc_path)
        
        request = {
            "model": "gemini-2.5-flash",
            "contents": [
                {"mime_type": mime_type, "data": doc_data},
                {"text": f"Analyze this document and provide: {analysis_type}"}
            ],
            "metadata": {
                "document_path": str(doc_path),
                "analysis_type": analysis_type
            }
        }
        requests.append(request)
    
    # Submit batch
    batch = client.batches.create(requests=requests)
    
    # Wait for completion
    while True:
        status = client.batches.get(name=batch.name)
        if status.state == "COMPLETED":
            break
        elif status.state == "FAILED":
            raise Exception("Batch processing failed")
        time.sleep(30)
    
    # Get and organize results
    results = client.batches.get_results(name=batch.name)
    
    processed_docs = {}
    for result in results:
        doc_path = result.metadata["document_path"]
        processed_docs[doc_path] = result.response.text
    
    return processed_docs

def get_mime_type(file_path):
    """Helper function to get MIME type"""
    ext = pathlib.Path(file_path).suffix.lower()
    mime_types = {
        '.pdf': 'application/pdf',
        '.docx': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
        '.txt': 'text/plain'
    }
    return mime_types.get(ext, 'application/octet-stream')

# Usage
document_paths = [
    "contract1.pdf",
    "contract2.pdf",
    "report1.pdf",
    "report2.pdf"
]

results = process_document_batch(document_paths, "key points and financial terms")

for doc_path, analysis in results.items():
    print(f"=== {doc_path} ===")
    print(analysis)
    print()
```

## Error Handling

```python
try:
    document_data = pathlib.Path(file_path).read_bytes()
    
    response = client.models.generate_content(
        model="gemini-2.5-flash",
        contents=[
            {"mime_type": "application/pdf", "data": document_data},
            {"text": "Analyze this document"}
        ]
    )
    
    print(response.text)
    
except FileNotFoundError:
    print(f"Document not found: {file_path}")
except Exception as error:
    if "file too large" in str(error):
        print("Document too large - use File API for files >20MB")
    elif "unsupported format" in str(error):
        print("Unsupported document format - use PDF, DOCX, or TXT")
    elif "corrupted" in str(error):
        print("Document appears to be corrupted")
    else:
        raise
```

## Best Practices

1. **File Size**: Use File API for documents >20MB
2. **Format Support**: PDF works best, DOCX and images also supported
3. **Specific Prompts**: Be specific about what to extract or analyze
4. **Batch Processing**: Use batches for multiple documents
5. **Error Handling**: Handle corrupted or unsupported files
6. **Cleanup**: Delete uploaded files when done

## See Also
- [Vision](vision.md) - Image-based document analysis
- [File Handling](file-handling.md) - File upload management
- [Batch](batch.md) - Batch document processing