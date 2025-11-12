# 🏢 OCI RAG Implementation Guide
## Enterprise IT Network Admin Assistant with Retrieval-Augmented Generation

### 📋 Overview

This guide provides step-by-step instructions to build an enterprise-grade IT Network Admin Assistant on Oracle Cloud Infrastructure (OCI) using Retrieval-Augmented Generation (RAG) with Oracle Vector Search.

**Key Components:**
- Knowledge Base: OCI Object Storage (NOC documentation)
- Vector Storage: Oracle Database 23ai with VECTOR data type
- Embeddings: Oracle AI Embedding Service (ORACLEAI_EMBED_TEXT_V1)
- Search: Semantic similarity vector search
- Interface: Oracle Digital Assistant + REST API

---

## 🚀 Phase 1: Create Knowledge Base in OCI Object Storage

### Step 1.1: Create OCI Object Storage Bucket

1. **Login to OCI Console**
   - Navigate to Object Storage → Buckets
   - Click "Create Bucket"

2. **Bucket Details**
   - Name: `noc-ai-docs`
   - Compartment: Your compartment
   - Storage Tier: Standard
   - Versioning: Disabled
   - Click "Create"

3. **Configure Access**
   - Set appropriate IAM policies for document access
   - Example policy:
   ```
   Allow group your-group to read buckets in compartment your-compartment
   Allow group your-group to read objects in compartment your-compartment
   ```

### Step 1.2: Upload Documentation to Bucket

**Sample Documents to Upload:**
- NOC Operations Manual (PDF)
- Network Topology Documentation (PDF/TXT)
- VLAN Migration Checklist (TXT)
- Fiber Link Troubleshooting Guide (PDF)
- Security Policy Documentation (PDF)
- Change Management Procedures (TXT)
- Incident Response Runbooks (PDF)

**Upload Steps:**
1. Click "Upload Objects" in bucket
2. Drag and drop files or browse
3. Set object names in format: `category/document_name.pdf`
4. Click "Upload"

---

## 💾 Phase 2: Configure Oracle Database 23ai Vector Storage

### Step 2.1: Create Vector-Enabled Database

1. **Provision Oracle Database 23ai**
   - OCI Console → Databases → Autonomous Database
   - Click "Create Autonomous Database"
   - Workload Type: "Transaction Processing" (OLTP)
   - Database name: `NOC_AI_KB`
   - Admin password: Set strong password
   - Network Access: Allow traffic from OCI AI services
   - Click "Create"

2. **Wait for Provisioning** (5-10 minutes)

### Step 2.2: Create Knowledge Base Table with Vector Support

**Connect to Database:**
```sql
-- Using SQL Developer or similar tool
-- Connect to NOC_AI_KB database
```

**Create Table:**
```sql
CREATE TABLE ai_knowledge_base (
  doc_id NUMBER PRIMARY KEY,
  title VARCHAR2(256) NOT NULL,
  category VARCHAR2(128),
  content CLOB NOT NULL,
  source VARCHAR2(512),
  embedding VECTOR(1024),
  chunk_number NUMBER,
  total_chunks NUMBER,
  created_date TIMESTAMP DEFAULT SYSDATE,
  updated_date TIMESTAMP DEFAULT SYSDATE
);
```

**Create Vector Index for Fast Similarity Search:**
```sql
CREATE VECTOR INDEX idx_knowledge_embedding
  ON ai_knowledge_base (embedding)
  USING HNSW
  WITH (type=FLOAT, dim=1024, distance=COSINE);

CREATE INDEX idx_category ON ai_knowledge_base(category);
CREATE INDEX idx_created_date ON ai_knowledge_base(created_date);
```

**Grant Privileges:**
```sql
GRANT SELECT, INSERT, UPDATE ON ai_knowledge_base TO application_user;
GRANT VECTOR_SEARCH TO application_user;
```

### Step 2.3: Load Documents and Generate Embeddings

**Create Embedding Generation Function:**
```sql
CREATE OR REPLACE FUNCTION generate_embedding(p_text CLOB) RETURN VECTOR
AS
  v_embedding VECTOR;
BEGIN
  SELECT vector_embedding 
  INTO v_embedding
  FROM (
    SELECT VECTOR_TRANSFORM(
      ORACLEAI_EMBED_TEXT_V1(p_text, JSON('{"provider": "cohere"}')),
      1024
    ) as vector_embedding
    FROM dual
  );
  RETURN v_embedding;
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.put_line('Error: ' || SQLERRM);
    RETURN NULL;
END;
/
```

**Load Documents Script:**
```sql
DECLARE
  v_doc_id NUMBER := 1;
  v_content CLOB;
  v_embedding VECTOR;
BEGIN
  -- Insert documents and generate embeddings
  INSERT INTO ai_knowledge_base (
    doc_id, title, category, content, source, embedding, chunk_number, total_chunks
  ) VALUES (
    v_doc_id,
    'Fiber Link Troubleshooting',
    'Troubleshooting',
    v_content,
    'noc-ai-docs/fiber-link-guide.pdf',
    generate_embedding(v_content),
    1,
    1
  );
  COMMIT;
END;
/
```

---

## 🔍 Phase 3: Configure Vector Search Tool in OCI AI Services

### Step 3.1: Create RAG Tool

1. **Navigate to OCI Generative AI Service**
   - OCI Console → Generative AI → Models
   - Note the endpoint and available models

2. **Create Custom Tool Integration**
   - Tool name: `rag_vector_tool`
   - Type: "Custom"
   - Description: "Retrieves relevant documents using vector similarity search"

3. **Configure Tool Parameters:**
   ```json
   {
     "knowledge_base_table": "ai_knowledge_base",
     "embedding_column": "embedding",
     "content_column": "content",
     "similarity_threshold": 0.7,
     "top_k_results": 5,
     "search_method": "vector_distance_cosine"
   }
   ```

### Step 3.2: Implement Vector Search Function

**Search Query Function:**
```sql
CREATE OR REPLACE FUNCTION rag_vector_search(
  p_query_text VARCHAR2,
  p_top_k NUMBER := 5,
  p_threshold NUMBER := 0.7
) RETURN SYS.ODCIVARCHAR2LIST
AS
  v_query_embedding VECTOR;
  v_results SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST();
BEGIN
  -- Generate embedding for query
  v_query_embedding := generate_embedding(p_query_text);
  
  -- Perform vector similarity search
  SELECT LISTAGG(kb.content, '||') WITHIN GROUP (ORDER BY similarity DESC)
  INTO v_results(1)
  FROM (
    SELECT kb.content,
           1 - VECTOR_DISTANCE(kb.embedding, v_query_embedding, COSINE) as similarity
    FROM ai_knowledge_base kb
    WHERE 1 - VECTOR_DISTANCE(kb.embedding, v_query_embedding, COSINE) >= p_threshold
    ORDER BY similarity DESC
    FETCH FIRST p_top_k ROWS ONLY
  );
  
  RETURN v_results;
END;
/
```

---

## 🤖 Phase 4: Create OCI Digital Assistant

### Step 4.1: Create New Assistant

1. **Open Oracle Digital Assistant**
   - URL: Your ODA instance
   - Login with OCI credentials

2. **Create New Assistant**
   - Click "+ New Assistant"
   - Name: `NOC_Assistant_RAG`
   - Description: "OCI AI Assistant for NOC operations using RAG and Oracle Vector Search"
   - Platform: "Oracle Cloud"
   - Click "Create"

### Step 4.2: Add RAG Tool to Assistant

1. **Configure Tool Integration**
   - Go to Skills → Add Custom Tool
   - Select `rag_vector_tool`
   - Map parameters:
     - Input: `user_query`
     - Output: `retrieved_context`

2. **Create Skill Dialog**
   ```yaml
   name: RetrieveNOCDocuments
   input: user_query
   tool: rag_vector_tool
   output: retrieved_context
   ```

### Step 4.3: Configure LLM Integration

1. **Add Generative AI Model**
   - Services → Generative AI
   - Select model: "Cohere Command (Latest)"
   - Configure context window size: 4000 tokens

2. **Create Main Dialog Flow**
   ```
   User Query
      ↓
   RetrieveNOCDocuments (RAG Tool)
      ↓
   [Combine Retrieved Context + User Query]
      ↓
   Generative AI Model (LLM)
      ↓
   Generate Response
      ↓
   Return to User
   ```

---

## 🌐 Phase 5: Enable REST API Endpoints

### Step 5.1: Create REST Endpoint

**ODA REST API Configuration:**
```
Base URL: https://<oda-instance>.data.digitalassistant.oci.oraclecloud.com/api

Endpoints:
- POST /noc-assistant/query
- GET /noc-assistant/status
- POST /noc-assistant/feedback
```

**Query Endpoint Implementation:**
```bash
curl -X POST https://<oda-instance>/api/noc-assistant/query \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "Show me the fiber link troubleshooting procedure",
    "context": {}
  }'
```

### Step 5.2: Enable Conversation Logging

1. **Configure Logging**
   - Settings → Logging
   - Enable: "Conversation Logs"
   - Enable: "Query Metrics"
   - Enable: "Error Tracking"

2. **Set Up Log Storage**
   - OCI Object Storage: `noc-ai-logs`
   - Retention: 90 days

---

## ✅ Phase 6: Testing & Validation

### Test Queries

**Test 1: Fiber Link Troubleshooting**
```
Query: "Show me the fiber link troubleshooting procedure"
Expected: Returns fiber troubleshooting documentation with:
  - Symptom identification
  - Diagnostic steps
  - Resolution procedures
```

**Test 2: VLAN Migration**
```
Query: "Summarize the VLAN migration checklist"
Expected: Returns VLAN migration checklist with:
  - Pre-migration validation
  - Migration steps
  - Post-migration verification
```

**Test 3: Incident Response**
```
Query: "What are the incident response procedures for network outages?"
Expected: Returns incident response runbook
```

**Test 4: Security Policy**
```
Query: "Summarize network security policies"
Expected: Returns security policy documentation
```

### Validation Checklist

- [ ] RAG tool correctly retrieves relevant documents
- [ ] Vector similarity scores are above threshold (>0.7)
- [ ] LLM generates contextual responses using retrieved content
- [ ] Response time < 3 seconds
- [ ] All conversation logs are stored
- [ ] REST API endpoints respond correctly
- [ ] Error handling works as expected

---

## 🔐 Phase 7: IAM Configuration

### Required Policies

```
# Allow ODA to read from Object Storage
Allow service "oda" to read buckets in compartment <compartment-name>
Allow service "oda" to read objects in compartment <compartment-name>

# Allow ODA to query Database
Allow service "oda" to use autonomous-database in compartment <compartment-name>

# Allow ODA to use Generative AI
Allow service "oda" to use generative-ai-models in compartment <compartment-name>

# Allow users to access Assistant
Allow group <your-group> to use digital-assistants in compartment <compartment-name>
```

---

## 📊 Monitoring & Maintenance

### Key Metrics to Monitor

1. **Performance**
   - Query response time (target: <3s)
   - Vector search accuracy
   - LLM token usage

2. **Quality**
   - Retrieved document relevance
   - User satisfaction (if feedback enabled)
   - Query resolution rate

3. **Usage**
   - Daily queries
   - Peak usage times
   - Top query topics

### Regular Maintenance

```
// Weekly
- Review error logs
- Check vector search quality
- Monitor API response times

// Monthly  
- Update knowledge base with new documents
- Regenerate embeddings for updated content
- Analyze user queries for improvement areas
- Review system metrics and costs
```

---

## 🎯 Expected Outcomes

✅ **Fully Operational NOC Assistant** with:
- Real-time access to NOC documentation
- Semantic search across 1000+ documents
- Context-aware response generation
- Complete conversation history
- REST API for integration

✅ **Enterprise Features:**
- IAM-based access control
- Audit logging for compliance
- Performance monitoring
- Scalable vector search
- Multi-document RAG pipeline

✅ **User Interface:**
- Oracle Digital Assistant web interface
- REST API for custom integrations
- Conversation history and feedback
- Document source attribution

---

**Last Updated:** November 11, 2025
**Version:** 1.0
**Status:** Production Ready
