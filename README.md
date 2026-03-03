# 🤖 n8n AI Agentic Workflows

A collection of production-ready AI agentic workflows built on **n8n** — covering feedback classification, chatbots, email automation, RAG pipelines, and social media intelligence. All workflows are fully self-hosted and run locally via Docker.

---

## 📦 Workflows

| # | Workflow | Model | Status |
|---|----------|-------|--------|
| 01 | [Feedback Classifier](#1-feedback-classifier) | GPT-4o-mini | ✅ Live |
| 02 | [Webhook Chatbot](#2-webhook-chatbot) | Groq LLaMA 3 70B | ✅ Live |
| 03 | [Email Reply Agent](#3-email-reply-agent) | GPT-4o-mini | ✅ Live |
| 04a | [RAG Vector Store Builder](#4a-rag-vector-store-builder) | OpenAI Embeddings | ✅ Live |
| 04b | [RAG Chatbot](#4b-rag-chatbot) | GPT-4o-mini | ✅ Live |
| 05 | [MCP Agent](#5-mcp-agent) | GPT-4o-mini | 🔧 In Progress |
| 06 | [Twitter Trend Agent](#6-twitter-trend-agent) | GPT-4o-mini | 🔧 In Progress |

---

## 🚀 Quick Start

### Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- OpenAI API key
- Groq API key
- Pinecone account (for RAG workflows)
- Google Cloud project with Gmail + Drive APIs (for email/RAG workflows)

### 1. Clone & Configure

```bash
git clone https://github.com/YOUR_USERNAME/n8n-ai-workflows.git
cd n8n-ai-workflows

# Copy and fill in your API keys
cp .env.example .env
nano .env
```

### 2. Start n8n

```bash
docker compose up -d
```

Open **http://localhost:5678** — login with the credentials from your `.env` file.

### 3. Import Workflows

In n8n: **Workflows → ⋮ → Import from file** → select any `.json` from the `workflows/` folder.

### 4. Add Credentials

Go to **Settings → Credentials → New Credential** and add:
- `OpenAI API` — your `sk-...` key
- `Groq API` — your `gsk_...` key
- `Gmail OAuth2` — Google Cloud OAuth2 credentials
- `Google Drive OAuth2` — same Google Cloud project
- `Pinecone API` — your Pinecone key + index name

---

## 📋 Workflow Details

### 1. Feedback Classifier

Automatically categorizes customer feedback into **complaint**, **appreciation**, or **feature request** using an AI agent.

**Flow:**
```
Feedback Form → Classify Feedback (GPT-4o-mini) → Parse Classification → Route by Category → Send Response
```
<img width="1110" height="518" alt="image" src="https://github.com/user-attachments/assets/cd6d8c17-fbd2-4fac-b9cf-51a9d11658cb" />

**Key nodes:** Form Trigger · AI Agent · Code (JSON parser) · IF Router · Respond to Webhook

**Test it:**
1. Activate the workflow
2. Click the Form Trigger node → copy the form URL
3. Open the URL in your browser and submit feedback
4. Response returns: `{"status":"received","category":"appreciation","message":"Thank you for your feedback!"}`

---
<img width="526" height="163" alt="image" src="https://github.com/user-attachments/assets/915ed5a2-9ab1-486b-896b-33ded12715fa" />

### 2. Webhook Chatbot

A REST API chatbot powered by **Groq LLaMA 3 70B**. Send a POST request, get an AI reply instantly.

**Flow:**
```
Chat Webhook (POST) → Generate Reply (Groq LLaMA 3 70B) → Send Reply
```
<img width="984" height="443" alt="image" src="https://github.com/user-attachments/assets/88755477-1c20-47ab-aafd-cfd9ea67dbef" />

**Key nodes:** Webhook · AI Agent · Groq LLaMA 3 70B · Respond to Webhook

**Test it:**
```bash
curl -X POST http://localhost:5678/webhook/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello, what can you do?"}'

# Response:
# {"reply":"Hello! I can provide information and answer questions...","model":"llama3"}
```

---
<img width="1512" height="261" alt="Screenshot 2026-03-03 at 3 31 35 PM" src="https://github.com/user-attachments/assets/3cfd2b2f-09fe-48ae-aee1-a6724b0da129" />

### 3. Email Reply Agent

Runs every 5 hours, scans your Gmail inbox, and auto-generates personalized replies using GPT-4o-mini.

**Flow:**
```
Every 5 Hours → Get Unread Emails (Gmail) → Craft Email Reply (GPT-4o-mini) → Parse Reply → Send Reply (Gmail)
```
<img width="1064" height="612" alt="image" src="https://github.com/user-attachments/assets/9dd53da7-833d-4714-b9fe-364cfb8ed80d" />

**Key nodes:** Schedule Trigger · Gmail · AI Agent · Code (JSON parser) · Gmail Send

**Handles:** offers · advertisements · orders · general inquiries

**Note:** Activate the workflow and it runs automatically. Click **Execute workflow** to trigger manually.

---
<img width="860" height="424" alt="image" src="https://github.com/user-attachments/assets/d9e72510-f7b0-410f-87b8-05b184824ffe" />

### 4a. RAG Vector Store Builder

Monitors a Google Drive folder for new PDFs and automatically processes them into a Pinecone vector database for retrieval.

**Flow:**
```
Watch Google Drive → Download PDF → Store in Pinecone
                                         ↑             ↑
                                      Load PDF    OpenAI Embeddings
                                         ↑
                                   Split into Chunks
```
<img width="1142" height="665" alt="image" src="https://github.com/user-attachments/assets/382c67bb-d747-442c-8445-b18a94f28070" />

**Key nodes:** Google Drive Trigger · Google Drive Download · Pinecone Vector Store · Document Loader · Text Splitter · OpenAI Embeddings (`text-embedding-3-small`)

**Setup:** Set your Google Drive folder ID in the Watch Google Drive node. Any PDF dropped into that folder gets auto-embedded into Pinecone.

---

### 4b. RAG Chatbot

Chat interface that retrieves context from the Pinecone knowledge base and answers questions about your documents.

**Flow:**
```
Chat Interface → RAG Agent (GPT-4o-mini)
                      ↑           ↑            ↑
                 GPT-4o-mini  Chat Memory  Knowledge Base Tool
                                                ↑            ↑
                                        Pinecone Vector   GPT-4o-mini
                                             Store         (Tool)
                                              ↑
                                       OpenAI Embeddings
```

**Key nodes:** Chat Trigger · AI Agent (Tools Agent) · GPT-4o-mini · Memory Buffer · Vector Store Tool · Pinecone · OpenAI Embeddings

**Important:** The Knowledge Base Tool requires its **own GPT model node** (`GPT-4o-mini (Tool)`) connected to its `Model*` port — separate from the main agent's model.

**Test it:**
1. Activate workflow
2. Open the Chat Interface URL from the Chat Trigger node
3. Ask: *"What are my skills?"* or any question about documents you've uploaded

---
<img width="1287" height="843" alt="image" src="https://github.com/user-attachments/assets/8d2eaf40-3175-4b2a-8e28-5dfc6fffae0d" />

### 5. MCP Agent

> 🔧 Coming soon

AI agent using **Model Context Protocol (MCP)** to expose YouTube and Gmail APIs as tools.

---

### 6. Twitter Trend Agent

> 🔧 Coming soon

Searches and analyzes Twitter trends based on user queries using the Twitter v2 API.

---

## 🗂️ Repository Structure

```
n8n-ai-workflows/
├── docker-compose.yml          # Spin up n8n locally
├── .env.example                # API keys template
├── workflows/
│   ├── 01-feedback-classifier.json
│   ├── 02-webhook-chatbot.json
│   ├── 03-email-reply-agent.json
│   ├── 04a-rag-vector-store-builder.json
│   └── 04b-rag-chatbot.json
└── README.md
```

---

## 🛠️ Tech Stack

| Category | Technology |
|----------|------------|
| Automation Platform | [n8n](https://n8n.io) |
| AI Models | OpenAI GPT-4o-mini, Groq LLaMA 3 70B |
| Vector Database | [Pinecone](https://pinecone.io) |
| Embeddings | OpenAI text-embedding-3-small |
| Email | Gmail API (OAuth2) |
| Storage | Google Drive API |
| Social | Twitter/X API v2 |
| Infrastructure | Docker Compose |

---

## ⚙️ Environment Variables

```env
# AI Models
OPENAI_API_KEY=sk-...
GROQ_API_KEY=gsk_...

# n8n Auth
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=changeme123
N8N_ENCRYPTION_KEY=your-32-char-secret-key

# Pinecone
PINECONE_API_KEY=...
PINECONE_INDEX_NAME=ai-knowledge-base

# Twitter (for workflow 06)
TWITTER_BEARER_TOKEN=...
```

---

## 🧠 Key Concepts

### n8n Sub-node Connections
n8n AI nodes use special connection types that differ from regular `main` connections:

| Connection Type | Used For |
|----------------|----------|
| `ai_languageModel` | Connecting LLMs to agents |
| `ai_embedding` | Connecting embedding models to vector stores |
| `ai_document` | Connecting document loaders to vector stores |
| `ai_textSplitter` | Connecting text splitters to document loaders |
| `ai_tool` | Connecting tools to agents |
| `ai_memory` | Connecting memory to agents |
| `ai_vectorStore` | Connecting vector stores to tools |

### RAG Pipeline Architecture
```
[Google Drive] → [PDF] → [Chunks] → [Embeddings] → [Pinecone]
                                                         ↓
[User Question] → [Agent] → [Retrieve] → [GPT-4o-mini] → [Answer]
```

---

## 📄 License

MIT — feel free to use, fork, and build on these workflows.

---

## 🙋 Contributing

Pull requests welcome! If you build a new workflow, add the JSON to `workflows/` and update this README with the flow diagram and test instructions.
