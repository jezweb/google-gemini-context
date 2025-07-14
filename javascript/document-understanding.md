# Document Understanding in JavaScript

*Analyze documents, PDFs, and structured content*

<!-- METADATA
Source: https://ai.google.dev/gemini-api/docs/document-processing
SDK: @google/genai (JavaScript)
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

```javascript
import { GoogleGenAI } from "@google/genai";
import * as fs from "node:fs";

const ai = new GoogleGenAI({});

// Analyze PDF document
const pdfData = fs.readFileSync("contract.pdf");

const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      inlineData: {
        mimeType: "application/pdf",
        data: pdfData.toString("base64")
      }
    },
    {
      text: "Analyze this document and provide a summary of key points"
    }
  ]
});

console.log(response.text);
```

## Large Document Processing (File API)

```javascript
import { GoogleAIFileManager } from "@google/genai/server";

const ai = new GoogleGenAI({});
const fileManager = new GoogleAIFileManager(process.env.GEMINI_API_KEY);

// Upload large document
console.log("Uploading document...");
const uploadResult = await fileManager.uploadFile("large_report.pdf", {
  mimeType: "application/pdf",
  displayName: "Large Report"
});

// Wait for processing
let file = await fileManager.getFile(uploadResult.file.name);
while (file.state === "PROCESSING") {
  process.stdout.write(".");
  await new Promise(resolve => setTimeout(resolve, 5000));
  file = await fileManager.getFile(uploadResult.file.name);
}

if (file.state !== "ACTIVE") {
  throw new Error(`Document processing failed: ${file.state}`);
}

console.log(`\nDocument ready: ${file.uri}`);

// Analyze document
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash",
  contents: [
    {
      fileData: {
        fileUri: uploadResult.file.uri,
        mimeType: uploadResult.file.mimeType
      }
    },
    {
      text: "Extract all tables and convert them to structured data"
    }
  ]
});

console.log(response.text);
```

## Document Analyzer Class

```javascript
class DocumentAnalyzer {
  constructor() {
    this.ai = new GoogleGenAI({});
    this.fileManager = new GoogleAIFileManager(process.env.GEMINI_API_KEY);
  }

  async analyzeDocument(filePath, analysisType = "summary") {
    const fileSize = fs.statSync(filePath).size;

    // Handle file size
    let contents;
    if (fileSize < 20 * 1024 * 1024) { // Under 20MB
      const documentData = fs.readFileSync(filePath);
      contents = [
        {
          inlineData: {
            mimeType: this.getMimeType(filePath),
            data: documentData.toString("base64")
          }
        },
        {
          text: this.getAnalysisPrompt(analysisType)
        }
      ];
    } else {
      // Upload large file
      const documentFile = await this.uploadDocument(filePath);
      contents = [
        {
          fileData: {
            fileUri: documentFile.uri,
            mimeType: documentFile.mimeType
          }
        },
        {
          text: this.getAnalysisPrompt(analysisType)
        }
      ];
    }

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents
    });

    return response.text;
  }

  getAnalysisPrompt(analysisType) {
    const prompts = {
      summary: "Provide a comprehensive summary of this document",
      tables: "Extract all tables and convert them to JSON format",
      keyPoints: "Extract key points, decisions, and action items",
      entities: "Extract all named entities (people, organizations, dates, amounts)",
      structure: "Analyze the document structure and create an outline",
      qa: "List potential questions this document answers",
      compliance: "Check for compliance-related content and requirements",
      financial: "Extract all financial information, numbers, and calculations"
    };
    return prompts[analysisType] || prompts.summary;
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

  async uploadDocument(filePath) {
    console.log(`Uploading ${filePath}...`);
    const uploadResult = await this.fileManager.uploadFile(filePath, {
      mimeType: this.getMimeType(filePath),
      displayName: filePath
    });

    let file = await this.fileManager.getFile(uploadResult.file.name);
    while (file.state === "PROCESSING") {
      process.stdout.write(".");
      await new Promise(resolve => setTimeout(resolve, 5000));
      file = await this.fileManager.getFile(uploadResult.file.name);
    }

    if (file.state !== "ACTIVE") {
      throw new Error(`Upload failed: ${file.state}`);
    }

    console.log("\nUpload complete!");
    return file;
  }

  async extractTables(filePath) {
    return await this.analyzeDocument(filePath, "tables");
  }

  async extractEntities(filePath) {
    return await this.analyzeDocument(filePath, "entities");
  }

  async getDocumentQA(filePath, questions) {
    const fileSize = fs.statSync(filePath).size;

    let baseContents;
    if (fileSize < 20 * 1024 * 1024) {
      const documentData = fs.readFileSync(filePath);
      baseContents = [{
        inlineData: {
          mimeType: this.getMimeType(filePath),
          data: documentData.toString("base64")
        }
      }];
    } else {
      const documentFile = await this.uploadDocument(filePath);
      baseContents = [{
        fileData: {
          fileUri: documentFile.uri,
          mimeType: documentFile.mimeType
        }
      }];
    }

    const results = {};
    for (const question of questions) {
      const contents = [...baseContents, { text: `Question: ${question}` }];

      const response = await this.ai.models.generateContent({
        model: "gemini-2.5-flash",
        contents
      });

      results[question] = response.text;
    }

    return results;
  }
}

// Usage
const analyzer = new DocumentAnalyzer();

// Analyze contract
const summary = await analyzer.analyzeDocument("contract.pdf", "summary");
console.log("Contract Summary:");
console.log(summary);

// Extract tables
const tables = await analyzer.extractTables("financial_report.pdf");
console.log("\nExtracted Tables:");
console.log(tables);

// Document Q&A
const questions = [
  "What is the total contract value?",
  "When does this contract expire?",
  "Who are the parties involved?"
];

const answers = await analyzer.getDocumentQA("contract.pdf", questions);
for (const [question, answer] of Object.entries(answers)) {
  console.log(`Q: ${question}`);
  console.log(`A: ${answer}\n`);
}
```

## Contract Analysis

```javascript
class ContractAnalyzer {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async analyzeContract(contractPath) {
    const contractData = fs.readFileSync(contractPath);
    const analyses = {};

    // 1. Basic summary
    let response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: [
        {
          inlineData: {
            mimeType: "application/pdf",
            data: contractData.toString("base64")
          }
        },
        {
          text: `Analyze this contract and provide:
          1. Contract type and purpose
          2. Parties involved
          3. Key terms and conditions
          4. Important dates and deadlines
          5. Financial terms
          6. Termination clauses
          7. Risk factors or unusual terms`
        }
      ]
    });
    analyses.summary = response.text;

    // 2. Extract key dates
    response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        {
          inlineData: {
            mimeType: "application/pdf",
            data: contractData.toString("base64")
          }
        },
        {
          text: "Extract all dates mentioned in this contract and their significance"
        }
      ]
    });
    analyses.dates = response.text;

    // 3. Financial analysis
    response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        {
          inlineData: {
            mimeType: "application/pdf",
            data: contractData.toString("base64")
          }
        },
        {
          text: "Extract all financial information, payment terms, and monetary amounts"
        }
      ]
    });
    analyses.financial = response.text;

    // 4. Risk assessment
    response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: [
        {
          inlineData: {
            mimeType: "application/pdf",
            data: contractData.toString("base64")
          }
        },
        {
          text: "Identify potential risks, liabilities, and unfavorable terms in this contract"
        }
      ]
    });
    analyses.risks = response.text;

    return analyses;
  }

  async compareContracts(contract1Path, contract2Path) {
    const contract1Data = fs.readFileSync(contract1Path);
    const contract2Data = fs.readFileSync(contract2Path);

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: [
        { text: "Compare these two contracts and highlight key differences:" },
        { text: "Contract 1:" },
        {
          inlineData: {
            mimeType: "application/pdf",
            data: contract1Data.toString("base64")
          }
        },
        { text: "Contract 2:" },
        {
          inlineData: {
            mimeType: "application/pdf",
            data: contract2Data.toString("base64")
          }
        },
        {
          text: "Provide a detailed comparison focusing on terms, conditions, and financial aspects"
        }
      ]
    });

    return response.text;
  }
}

// Usage
const contractAnalyzer = new ContractAnalyzer();

// Analyze single contract
const analysis = await contractAnalyzer.analyzeContract("service_agreement.pdf");
for (const [section, content] of Object.entries(analysis)) {
  console.log(`=== ${section.toUpperCase()} ===`);
  console.log(content);
  console.log();
}

// Compare contracts
const comparison = await contractAnalyzer.compareContracts("old_contract.pdf", "new_contract.pdf");
console.log("Contract Comparison:");
console.log(comparison);
```

## Scientific Paper Analysis

```javascript
class PaperAnalyzer {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async analyzePaper(paperPath) {
    const paperData = fs.readFileSync(paperPath);

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: [
        {
          inlineData: {
            mimeType: "application/pdf",
            data: paperData.toString("base64")
          }
        },
        {
          text: `Analyze this scientific paper and provide:
          1. Title and authors
          2. Abstract summary
          3. Research methodology
          4. Key findings and results
          5. Conclusions and implications
          6. References to key related work
          7. Limitations and future work`
        }
      ]
    });

    return response.text;
  }

  async extractCitations(paperPath) {
    const paperData = fs.readFileSync(paperPath);

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        {
          inlineData: {
            mimeType: "application/pdf",
            data: paperData.toString("base64")
          }
        },
        {
          text: "Extract all citations and references from this paper in a structured format"
        }
      ]
    });

    return response.text;
  }

  async summarizeForAudience(paperPath, audience = "general") {
    const paperData = fs.readFileSync(paperPath);

    const audiencePrompts = {
      general: "Summarize this paper for a general audience without technical jargon",
      technical: "Provide a technical summary for researchers in the field",
      executive: "Create an executive summary focusing on practical implications",
      student: "Explain this paper in a way undergraduate students would understand"
    };

    const prompt = audiencePrompts[audience] || audiencePrompts.general;

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        {
          inlineData: {
            mimeType: "application/pdf",
            data: paperData.toString("base64")
          }
        },
        {
          text: prompt
        }
      ]
    });

    return response.text;
  }
}

// Usage
const paperAnalyzer = new PaperAnalyzer();

// Analyze research paper
const analysis = await paperAnalyzer.analyzePaper("research_paper.pdf");
console.log("Paper Analysis:");
console.log(analysis);

// Create summaries for different audiences
const audiences = ["general", "technical", "executive"];
for (const audience of audiences) {
  const summary = await paperAnalyzer.summarizeForAudience("research_paper.pdf", audience);
  console.log(`\n${audience.toUpperCase()} SUMMARY:`);
  console.log(summary);
}
```

## Financial Document Processing

```javascript
class FinancialDocumentProcessor {
  constructor() {
    this.ai = new GoogleGenAI({});
  }

  async analyzeFinancialReport(reportPath) {
    const reportData = fs.readFileSync(reportPath);

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: [
        {
          inlineData: {
            mimeType: "application/pdf",
            data: reportData.toString("base64")
          }
        },
        {
          text: `Analyze this financial report and extract:
          1. Revenue and profit figures
          2. Key financial ratios
          3. Year-over-year changes
          4. Cash flow information
          5. Balance sheet highlights
          6. Risk factors mentioned
          7. Management outlook`
        }
      ]
    });

    return response.text;
  }

  async extractFinancialTables(reportPath) {
    const reportData = fs.readFileSync(reportPath);

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        {
          inlineData: {
            mimeType: "application/pdf",
            data: reportData.toString("base64")
          }
        },
        {
          text: "Extract all financial tables and convert them to structured JSON format"
        }
      ]
    });

    return response.text;
  }

  async calculateRatios(reportPath) {
    const reportData = fs.readFileSync(reportPath);

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-pro",
      contents: [
        {
          inlineData: {
            mimeType: "application/pdf",
            data: reportData.toString("base64")
          }
        },
        {
          text: `Calculate key financial ratios from this report:
          - Liquidity ratios (current ratio, quick ratio)
          - Profitability ratios (ROE, ROA, profit margin)
          - Leverage ratios (debt-to-equity, interest coverage)
          - Efficiency ratios (asset turnover, inventory turnover)
          Show calculations and explain what each ratio indicates`
        }
      ]
    });

    return response.text;
  }
}

// Usage
const financialProcessor = new FinancialDocumentProcessor();

// Analyze quarterly report
const analysis = await financialProcessor.analyzeFinancialReport("q4_report.pdf");
console.log("Financial Analysis:");
console.log(analysis);

// Extract tables
const tables = await financialProcessor.extractFinancialTables("q4_report.pdf");
console.log("\nFinancial Tables:");
console.log(tables);

// Calculate ratios
const ratios = await financialProcessor.calculateRatios("q4_report.pdf");
console.log("\nFinancial Ratios:");
console.log(ratios);
```

## Batch Document Processing

```javascript
async function processDocumentBatch(documentPaths, analysisType = "summary") {
  const ai = new GoogleGenAI({});
  const requests = [];

  for (const docPath of documentPaths) {
    // Prepare document data
    const docData = fs.readFileSync(docPath);
    const mimeType = getMimeType(docPath);

    const request = {
      model: "gemini-2.5-flash",
      contents: [
        {
          inlineData: {
            mimeType,
            data: docData.toString("base64")
          }
        },
        {
          text: `Analyze this document and provide: ${analysisType}`
        }
      ],
      metadata: {
        documentPath: docPath,
        analysisType
      }
    };
    requests.push(request);
  }

  // Submit batch
  const batch = await ai.batches.create({ requests });

  // Wait for completion
  while (true) {
    const status = await ai.batches.get({ name: batch.name });
    if (status.state === "COMPLETED") {
      break;
    } else if (status.state === "FAILED") {
      throw new Error("Batch processing failed");
    }
    await new Promise(resolve => setTimeout(resolve, 30000));
  }

  // Get and organize results
  const results = await ai.batches.getResults({ name: batch.name });

  const processedDocs = {};
  for (const result of results) {
    const docPath = result.metadata.documentPath;
    processedDocs[docPath] = result.response.text;
  }

  return processedDocs;
}

function getMimeType(filePath) {
  const ext = filePath.toLowerCase().split('.').pop();
  const mimeTypes = {
    pdf: 'application/pdf',
    docx: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    txt: 'text/plain'
  };
  return mimeTypes[ext] || 'application/octet-stream';
}

// Usage
const documentPaths = [
  "contract1.pdf",
  "contract2.pdf",
  "report1.pdf",
  "report2.pdf"
];

const results = await processDocumentBatch(documentPaths, "key points and financial terms");

for (const [docPath, analysis] of Object.entries(results)) {
  console.log(`=== ${docPath} ===`);
  console.log(analysis);
  console.log();
}
```

## Web-based Document Analysis

```javascript
// Client-side document analysis
class WebDocumentAnalyzer {
  constructor(apiKey) {
    this.ai = new GoogleGenAI({ apiKey });
  }

  async analyzeFileUpload(fileInput, analysisType = "summary") {
    const file = fileInput.files[0];
    if (!file) {
      throw new Error("No file selected");
    }

    // Convert file to base64
    const base64Data = await this.fileToBase64(file);

    const response = await this.ai.models.generateContent({
      model: "gemini-2.5-flash",
      contents: [
        {
          inlineData: {
            mimeType: file.type,
            data: base64Data
          }
        },
        {
          text: this.getAnalysisPrompt(analysisType)
        }
      ]
    });

    return response.text;
  }

  async fileToBase64(file) {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = () => {
        const base64 = reader.result.split(',')[1];
        resolve(base64);
      };
      reader.onerror = error => reject(error);
    });
  }

  getAnalysisPrompt(analysisType) {
    const prompts = {
      summary: "Provide a comprehensive summary of this document",
      tables: "Extract all tables and convert them to JSON format",
      keyPoints: "Extract key points, decisions, and action items",
      entities: "Extract all named entities (people, organizations, dates, amounts)"
    };
    return prompts[analysisType] || prompts.summary;
  }

  async streamAnalysis(file, analysisType, onChunk) {
    const base64Data = await this.fileToBase64(file);

    const response = await this.ai.models.generateContentStream({
      model: "gemini-2.5-flash",
      contents: [
        {
          inlineData: {
            mimeType: file.type,
            data: base64Data
          }
        },
        {
          text: this.getAnalysisPrompt(analysisType)
        }
      ]
    });

    for await (const chunk of response) {
      onChunk(chunk.text);
    }
  }
}

// Usage in browser
const analyzer = new WebDocumentAnalyzer(API_KEY);

// Handle file upload
document.getElementById('fileInput').addEventListener('change', async (event) => {
  const analysis = await analyzer.analyzeFileUpload(event.target, 'summary');
  document.getElementById('results').textContent = analysis;
});

// Stream analysis with progress
await analyzer.streamAnalysis(
  file,
  'keyPoints',
  (chunk) => {
    document.getElementById('results').textContent += chunk;
  }
);
```

## Error Handling

```javascript
try {
  const documentData = fs.readFileSync(filePath);

  const response = await ai.models.generateContent({
    model: "gemini-2.5-flash",
    contents: [
      {
        inlineData: {
          mimeType: "application/pdf",
          data: documentData.toString("base64")
        }
      },
      {
        text: "Analyze this document"
      }
    ]
  });

  console.log(response.text);

} catch (error) {
  if (error.code === 'ENOENT') {
    console.log(`Document not found: ${filePath}`);
  } else if (error.message.includes("file too large")) {
    console.log("Document too large - use File API for files >20MB");
  } else if (error.message.includes("unsupported format")) {
    console.log("Unsupported document format - use PDF, DOCX, or TXT");
  } else if (error.message.includes("corrupted")) {
    console.log("Document appears to be corrupted");
  } else {
    throw error;
  }
}
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