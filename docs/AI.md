# AI Features for CareCircle -- Gemini + pgvector (Free)

CareCircle integrates three AI-powered features using **Google Gemini** (free tier) and **pgvector** on Neon PostgreSQL. Zero additional infrastructure cost.

| Feature | What it does | Where in the app |
|---------|-------------|-----------------|
| **AI Care Summaries** | Generates daily/weekly care summaries from timeline, medications, appointments, and shifts | Dashboard sidebar card |
| **Smart Data Entry** | Caregivers type naturally, AI parses into structured timeline entries with vitals | Timeline page ("Smart Entry" toggle) |
| **RAG -- Ask AI** | Family members ask questions and get answers sourced from their care data | Dashboard header (sparkles icon) |

---

## Core Concepts Explained

Before diving into the code, here is what each technology does and why we chose it.

### What is Google Gemini?

Google Gemini is Google's family of large language models (LLMs). Think of it as a general-purpose AI brain that can read text, understand it, and generate new text. We use two specific Gemini capabilities:

- **Gemini 2.0 Flash** -- A fast, lightweight text generation model. We send it a prompt ("Here is care data, write a summary") and it returns a well-written response. "Flash" means it is optimized for speed over raw intelligence, which is perfect for our use case.
- **gemini-embedding-001** -- An embedding model (explained below). It turns text into numbers so we can do math-based search.

**Why Gemini?** It has a generous free tier (15 requests/minute, 1 million tokens/day), a simple API, supports structured JSON output natively, and is from Google so the SDK is well-maintained. Alternatives like OpenAI (GPT-4o) or Anthropic (Claude) would work but cost money from the first request.

> **Token** = roughly 4 characters of English text. "Hello world" is about 2 tokens. A 500-word care summary uses ~600 tokens.

### What are Embeddings?

An **embedding** is a list of numbers (a "vector") that represents the meaning of a piece of text. For example:

```
"Mom took her blood pressure medication" → [0.12, -0.45, 0.78, ..., 0.33]  (768 numbers)
"Mother's BP meds were administered"     → [0.11, -0.44, 0.77, ..., 0.34]  (768 numbers, very similar!)
"The weather is sunny today"             → [0.89, 0.22, -0.56, ..., -0.71] (768 numbers, very different)
```

Even though the first two sentences use different words, their embeddings are almost identical because they mean the same thing. This is what makes semantic search (search by meaning) possible. We use Google's `gemini-embedding-001` model which produces vectors of 768 dimensions.

### What is pgvector?

**pgvector** is a PostgreSQL extension that lets you store and search embedding vectors directly in your existing database. Without it, you would need a separate vector database (Pinecone, Weaviate, etc.) which adds complexity and cost.

With pgvector, our `ai_embeddings` table has a column of type `vector(768)` that stores the 768-dimensional embedding. We can then run SQL queries like:

```sql
SELECT * FROM ai_embeddings
WHERE family_id = 'fam_123'
ORDER BY embedding <=> $query_embedding  -- cosine distance (lower = more similar)
LIMIT 5;
```

The `<=>` operator computes **cosine similarity** between two vectors. The **HNSW index** makes this fast even with thousands of rows (it is an approximate nearest-neighbor algorithm that avoids comparing every single row).

### What is RAG (Retrieval-Augmented Generation)?

**RAG** is a pattern that makes an LLM answer questions using YOUR data, not its training data. The problem with just asking Gemini "How is Mom's medication adherence?" is that Gemini has no idea who Mom is. RAG solves this in three steps:

```
1. EMBED the question     → "How is Mom's medication adherence?" → [0.23, -0.11, ...]
2. RETRIEVE relevant data → Search ai_embeddings for similar vectors → find 5 matching records
3. GENERATE an answer     → Send Gemini: "Given these 5 records, answer the question" → grounded answer
```

This way, Gemini only uses your family's actual data. It cannot hallucinate about medications that do not exist in your records because we explicitly give it only your records as context.

### What is Structured Output?

Normally, an LLM returns free-form text. **Structured output** forces the model to return valid JSON matching a schema you define. We use this for:

- **Care Summaries** -- Gemini must return `{ summary, highlights, concerns, medications }` not just a paragraph
- **Smart Entry** -- Gemini must return `{ type, title, description, severity, vitals }` not just "I think this is a vitals entry"

This is done by passing a JSON schema to Gemini's `responseMimeType: "application/json"` mode. If the model cannot fit its answer into the schema, it returns a best-effort match. We always validate the response and fall back to defaults if parsing fails.

### What is AI Text Summarization?

**AI text summarization** is the process of feeding a large amount of raw data to an LLM and asking it to produce a shorter, human-readable narrative. There are two types:

- **Extractive summarization** -- Picks the most important sentences from the source text verbatim. Like highlighting a document.
- **Abstractive summarization** -- Generates *new* text that captures the meaning of the source, paraphrased in the model's own words. This is what Gemini does and what CareCircle uses.

In CareCircle, summarization is not just "shrink this text." It is **multi-source aggregation + synthesis**. The LLM receives structured data from 4 different database tables (timeline entries, medication logs, appointments, caregiver shifts), understands the relationships between them, and writes a cohesive narrative:

```
INPUT (raw data from 4 tables):
  - 12 timeline entries (vitals, meals, mood notes)
  - 9 medication logs (8 given, 1 missed)
  - 1 appointment (Dr. Smith, cardiology)
  - 2 caregiver shifts (Sarah morning, David evening)

OUTPUT (AI-generated summary):
  "Mom had a good day overall. 8 of 9 medications were given on time — the evening
   blood pressure pill was missed. Sarah noted her appetite was good at breakfast and
   lunch. The cardiology appointment with Dr. Smith went well. David is on duty tonight."
```

The key insight is that a human could write this summary by reading all 24 records, but it would take 5-10 minutes. The LLM does it in 1-2 seconds.

### How Does the Summarization Pipeline Work?

CareCircle's summarization follows a **4-step pipeline**: Aggregate → Format → Generate → Parse. Here is each step in detail.

**Step 1: Data Aggregation (Database Queries)**

`CareSummaryService` runs 5 parallel Prisma queries to fetch everything relevant for the time window:

```
Daily summary → last 24 hours
Weekly summary → Monday through Sunday of the current week
```

The queries are:

| Query | Table | What it fetches |
|-------|-------|-----------------|
| Care recipient profile | `CareRecipient` | Name, preferred name, conditions |
| Timeline entries | `TimelineEntry` | Notes, incidents, vitals, meals, mood -- ordered by time |
| Medication logs | `MedicationLog` | Each scheduled dose and its status (GIVEN, MISSED, SKIPPED) + who administered it |
| Appointments | `Appointment` | Scheduled visits with doctor name and specialty |
| Caregiver shifts | `CaregiverShift` | Who was on duty, check-in status |

All 5 queries run with `Promise.all()` so they execute in parallel (total ~50-100ms instead of ~250-500ms sequentially). The results are plain JavaScript objects -- no AI has been called yet.

**Step 2: Context Formatting (Prompt Construction)**

Raw database records are not useful as-is for an LLM. The `buildDayContext()` method converts them into a human-readable text block:

```
Timeline entries (12):
  - [8:15 AM] MEAL: Breakfast — Ate well, had oatmeal and juice
  - [8:30 AM] VITALS: Morning vitals — BP 130/85, pulse 72
  - [10:00 AM] ACTIVITY: Morning walk — 15 minutes around the block
  ...

Medications: 8 given, 1 missed out of 9 scheduled
  - Lisinopril 10mg: GIVEN by Sarah
  - Metformin 500mg: GIVEN by Sarah
  - Amlodipine 5mg: MISSED
  ...

Appointments (1):
  - Cardiology checkup with Dr. Smith (COMPLETED)

Caregiver shifts (2):
  - Sarah: COMPLETED (checked in)
  - David: IN_PROGRESS (checked in)
```

This formatting is critical. The LLM does not understand Prisma objects or JSON database rows. It understands text. The better the formatting, the better the summary. Some key decisions:

- **Timeline entries are capped at 20** per summary to stay within token limits
- **Medication logs are capped at 15** for the same reason
- **Times are formatted as human-readable** ("8:15 AM" not "2026-02-12T08:15:00.000Z")
- **Medication stats are pre-calculated** (given/missed/skipped counts) so the LLM does not have to count

**Step 3: LLM Generation (Gemini Structured Output)**

The formatted context is combined with a **system instruction** and a **user prompt**, then sent to Gemini:

- **System instruction**: Tells Gemini *who it is* and *how to behave* -- "You are CareCircle AI, a compassionate care summary assistant. Use plain language. Be concise but thorough." This instruction persists across the conversation and shapes the model's tone.
- **User prompt**: Contains the actual data and the specific request -- "Generate a daily care summary for Mom on February 12, 2026. [context data]. Respond in this JSON format: { summary, highlights, concerns }."
- **Response schema**: Gemini's `responseMimeType: "application/json"` mode with a `responseSchema` forces the model to return valid JSON matching our exact structure. The model cannot return a paragraph -- it *must* return `{ summary: string, highlights: string[], concerns: string[] }`.

This is where the actual AI "thinking" happens. Gemini reads all the context, identifies patterns (medication adherence rate, notable incidents, positive trends), and writes a narrative that a family member can read in 30 seconds.

**Step 4: Parse, Merge, and Return**

After Gemini returns JSON, the service:

1. **Parses** the JSON response (`JSON.parse(text)`)
2. **Merges** it with the pre-calculated medication stats (Gemini does not calculate medication counts -- we do that from the database, which is 100% accurate)
3. **Adds metadata** like the period string ("February 12, 2026") and the generation timestamp
4. **Returns** the complete `CareSummary` object to the controller

If anything goes wrong (Gemini timeout, invalid JSON, rate limit), the service catches the error and returns a **basic fallback** -- a simple sentence with medication counts and entry totals, no AI required. The user still sees something useful.

### What is a System Instruction?

A **system instruction** (sometimes called a "system prompt") is a special instruction given to the LLM before any user content. It shapes the model's personality, tone, and behavior for the entire interaction.

```
System instruction: "You are CareCircle AI, a compassionate care summary
assistant. Use plain language. Be concise but thorough. Focus on what matters
most: medication adherence, health changes, and anything that needs attention."

User prompt: "Generate a daily care summary for Mom..."
```

Think of it like giving a new employee their job description before their first task. The system instruction is the job description; the user prompt is the specific task.

In CareCircle we use different system instructions for different features:

| Feature | System instruction tone |
|---------|------------------------|
| Care Summaries | Compassionate, concise, caregiver-focused |
| Smart Entry | Clinical, precise, classification-focused |
| RAG (Ask AI) | Helpful, evidence-based, citation-focused |

The system instruction is **not part of the visible prompt** -- it is a separate parameter to the Gemini API. This separation also helps with prompt injection security: even if a user types "ignore your instructions" in a Smart Entry, the system instruction remains authoritative.

### What is Prompt Engineering?

**Prompt engineering** is the practice of carefully writing the text instructions you send to an LLM to get the best possible output. It is less "engineering" and more "clear communication" -- the same skills you use to write a good email to a colleague.

Key techniques we use in CareCircle:

| Technique | What it means | Example from our code |
|-----------|---------------|----------------------|
| **Role assignment** | Tell the model who it is | "You are CareCircle AI, a compassionate care summary assistant" |
| **Explicit formatting** | Show the exact output format you want | "Respond in JSON: { summary, highlights, concerns }" |
| **Context injection** | Give the model all the data it needs | "Timeline entries (12): [8:15 AM] MEAL: Breakfast..." |
| **Constraint setting** | Set boundaries on behavior | "Only use the provided context. Do not make up information." |
| **Few-shot examples** | Show example inputs and outputs (not currently used, but effective) | "Example: Input 'BP 130/85' → Output { type: VITALS }" |
| **Output length guidance** | Tell the model how long the answer should be | "A 2-3 sentence overview of the day" |

Bad prompts get bad results. Compare:

```
❌ Bad prompt:  "Summarize this data"
✅ Good prompt: "Generate a daily care summary for Mom on Feb 12, 2026.
                 Given the following care data, write a warm 2-3 sentence
                 overview highlighting medication adherence, any health
                 concerns, and positive moments. [data follows]"
```

The difference in output quality is dramatic. Prompt engineering is the reason CareCircle's summaries feel helpful rather than generic.

### What is Graceful Degradation (AI Fallbacks)?

**Graceful degradation** means the app continues to work when AI is unavailable -- just without the AI-enhanced features. This is a design pattern, not a technology.

In CareCircle, every AI service checks `geminiService.enabled` before calling the API. If the Gemini API key is not set, is invalid, or the API is rate-limited/down, each feature falls back:

| Feature | With AI | Without AI (fallback) |
|---------|---------|----------------------|
| Care Summary | Rich narrative with highlights and concerns | Basic stats: "8/9 medications given, 5 timeline entries" |
| Smart Entry | Parsed structured entry (type, severity, vitals) | Raw text saved as a NOTE type |
| RAG / Ask AI | Grounded answer with source citations | "AI features are not configured" message |

The fallback is always *functional* -- it never throws an error or shows a blank screen. The app was built to work without AI first, and AI is layered on top.

This design means:
- New contributors can develop without a Gemini API key
- Tests do not require Gemini (the EmbeddingIndexer is injected with `@Optional()`)
- Production stays healthy even if Google's API goes down
- Users on free tier who hit rate limits get a degraded but usable experience

### What is BullMQ?

**BullMQ** is a job queue library built on Redis. Instead of processing slow tasks (like calling Gemini to generate embeddings) inside the API request, we:

1. **Enqueue** a job: `{ action: "upsert", resourceType: "timeline_entry", resourceId: "te_123", content: "..." }`
2. **Return** the API response immediately (user does not wait)
3. A **worker** process picks up the job, calls Gemini, stores the embedding

This is called **asynchronous processing**. Benefits:
- API stays fast (< 200ms response times)
- If Gemini is slow or down, jobs retry automatically with exponential backoff
- Workers can be scaled independently from the API
- If the worker crashes, jobs survive in Redis and process when it restarts

### What is Cosine Similarity?

When comparing two embedding vectors, we need a way to measure "how similar are they?" **Cosine similarity** measures the angle between two vectors:

- **1.0** = identical meaning
- **0.0** = completely unrelated
- **-1.0** = opposite meaning (rare in practice)

In our RAG search, we return the top 5 results with similarity > 0.7 (a threshold that filters out noise). The `<=>` operator in pgvector computes cosine *distance* (1 - similarity), so lower distance = more similar.

### What is HNSW?

**HNSW** (Hierarchical Navigable Small World) is an algorithm for fast approximate nearest-neighbor search. Without an index, finding the most similar vectors requires comparing your query against every single row (O(n) -- slow with thousands of records). HNSW builds a graph structure that lets you find approximate nearest neighbors in O(log n) time.

Our migration creates this index:

```sql
CREATE INDEX idx_embeddings_vector ON ai_embeddings
  USING hnsw (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);
```

- `m = 16` -- number of connections per node (higher = more accurate but slower builds)
- `ef_construction = 64` -- search width during index build (higher = better quality index)

These defaults are good for up to ~100K embeddings. You would only tune them for millions of rows.

---

## How Data Flows (End-to-End)

### Flow 1: Caregiver Creates a Timeline Entry

```
1. Caregiver submits "Mom had lunch, BP 120/80"
2. API saves the TimelineEntry to PostgreSQL (normal flow)
3. API calls EmbeddingIndexerService.indexTimelineEntry()
4. EmbeddingIndexer adds a job to the "ai-embeddings" BullMQ queue
5. API returns 201 Created immediately (user does not wait for AI)
6. [Background] AI Embedding Worker picks up the job
7. Worker calls Gemini gemini-embedding-001 to generate a 768-dim vector
8. Worker stores the vector + metadata in the ai_embeddings table
9. This entry is now searchable via RAG
```

### Flow 2: Family Member Asks "How has Mom's blood pressure been?"

```
1. Frontend sends POST /ai/ask { question, careRecipientId }
2. API (RagService) calls Gemini to embed the question → [0.23, -0.11, ...]
3. RagService queries ai_embeddings with cosine similarity, filtered by family_id
4. Top 5 most relevant records are retrieved (e.g., timeline entries with BP readings)
5. RagService builds a prompt: "Given these records, answer: How has Mom's BP been?"
6. Gemini generates a grounded answer using only the provided records
7. API returns { answer, sources } with citations back to the frontend
8. Frontend displays the answer and clickable source links
```

### Flow 3: Dashboard Loads Care Summary

```
1. Dashboard mounts CareSummaryCard, which calls GET /ai/summary/daily/:id
2. API (CareSummaryService) queries the last 24 hours of:
   - TimelineEntry (notes, incidents, vitals)
   - MedicationLog (given/missed/skipped counts)
   - Appointment (any scheduled today)
   - CaregiverShift (who was on duty)
3. CareSummaryService builds a prompt with all this data
4. Gemini returns structured JSON { summary, highlights, concerns, medications }
5. API returns it to the frontend
6. CareSummaryCard renders the narrative, stats, and highlights
```

### Flow 4: Smart Entry Parsing

```
1. Caregiver types: "Dad fell in the bathroom, small bruise on his arm"
2. Frontend sends POST /ai/smart-entry/parse { text }
3. API (SmartEntryService) sends the text to Gemini with a schema:
   { type: enum, title: string, description: string, severity: enum, vitals: object }
4. Gemini returns: { type: "INCIDENT", title: "Fall in bathroom", severity: "MEDIUM", ... }
5. Frontend shows a preview card with the parsed fields
6. Caregiver reviews, optionally edits, and clicks "Confirm & Save"
7. Frontend saves via the existing POST /care-recipients/:id/timeline endpoint
8. (Flow 1 triggers -- the new entry gets embedded for RAG)
```

---

## Architecture Overview

```
Frontend (Next.js)
  ├── CareSummaryCard     → GET /ai/summary/daily/:id
  ├── SmartEntryInput     → POST /ai/smart-entry/parse
  └── AskAiPanel          → POST /ai/ask

API (NestJS -- AiModule)
  ├── GeminiService       → Google Gemini 2.0 Flash + gemini-embedding-001
  ├── EmbeddingService    → pgvector storage & cosine similarity search
  ├── EmbeddingIndexer    → Enqueues BullMQ embedding jobs on data changes
  ├── CareSummaryService  → Aggregates care data → Gemini structured output
  ├── SmartEntryService   → Natural language → structured timeline entry
  └── RagService          → Embed question → vector search → Gemini answer

Workers (BullMQ)
  ├── ai-embedding.worker → Generates & stores embeddings asynchronously
  └── ai-summary.worker   → Background summary generation

Infrastructure
  ├── Neon PostgreSQL + pgvector (existing, free)
  ├── Upstash Redis / BullMQ (existing, free)
  └── Google Gemini API (free tier)
```

---

## Setup Instructions

### 1. Get a Gemini API Key (Free)

1. Go to [Google AI Studio](https://aistudio.google.com/apikey)
2. Sign in with your Google account
3. Click **Create API Key**
4. Copy the key -- you will need it for both local development and production

### 2. Local Development Setup

Add the following to your `apps/api/.env.local` (or `.env`):

```env
# AI (Google Gemini) -- Optional, AI features are disabled if not set
GEMINI_API_KEY=your-gemini-api-key-here

# Optional overrides (defaults are fine for most cases)
# GEMINI_MODEL=gemini-2.0-flash
# GEMINI_EMBEDDING_MODEL=gemini-embedding-001
```

Then run the database migration to create the `ai_embeddings` table and enable pgvector:

```bash
pnpm --filter @carecircle/database db:migrate
```

> **Note**: If you are using Neon (free tier), pgvector is already available. For local PostgreSQL, you may need to install the pgvector extension separately. The migration handles `CREATE EXTENSION IF NOT EXISTS vector;` automatically.

### 3. Production Deployment (Render)

1. Go to your **Render Dashboard** > API service > **Environment**
2. Add `GEMINI_API_KEY` with the value from step 1
3. Deploy -- the migration runs automatically via `preDeployCommand` (`pnpm db:migrate`)
4. Workers will automatically pick up the `ai-embeddings` and `ai-summaries` queues

No other infrastructure changes are needed. No new Docker containers, no new databases, no new services.

---

## File Structure

```
apps/api/src/ai/
  ├── ai.module.ts                          # NestJS module (registers queues, exports services)
  ├── controllers/
  │   └── ai.controller.ts                  # REST endpoints for all AI features
  └── services/
      ├── gemini.service.ts                 # Gemini SDK wrapper (text, structured, embeddings)
      ├── embedding.service.ts              # Vector storage & cosine similarity search
      ├── embedding-indexer.service.ts       # Enqueues embedding jobs via BullMQ
      ├── care-summary.service.ts           # Daily/weekly summary generation
      ├── smart-entry.service.ts            # Natural language → structured entry parsing
      └── rag.service.ts                    # RAG pipeline (embed → search → answer)

apps/web/src/
  ├── components/ai/
  │   ├── care-summary-card.tsx             # Dashboard summary card (daily/weekly toggle)
  │   ├── smart-entry-input.tsx             # NL input → preview → confirm flow
  │   └── ask-ai-panel.tsx                  # Slide-out chat panel with source citations
  ├── hooks/use-ai.ts                       # React Query hooks for all AI endpoints
  └── lib/api/ai.ts                         # API client functions

apps/workers/src/workers/
  ├── ai-embedding.worker.ts                # Processes embedding upsert/delete jobs
  └── ai-summary.worker.ts                  # Background summary generation

packages/database/prisma/migrations/
  └── 20260212000000_add_ai_embeddings/
      └── migration.sql                     # pgvector extension + ai_embeddings table + HNSW index
```

---

## API Reference

All AI endpoints require authentication (JWT). All care-recipient-scoped endpoints verify family membership.

### Care Summaries

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/ai/summary/daily/:careRecipientId` | Generate today's care summary |
| `GET` | `/ai/summary/weekly/:careRecipientId` | Generate this week's care summary |

**Query parameters**: `?date=2026-02-12` (optional, defaults to today/this week)

**Response** (`CareSummary`):

```json
{
  "summary": "Mom had a good day. 8 of 9 medications given, one walk completed...",
  "highlights": ["Medication adherence at 89%", "Completed morning walk"],
  "concerns": ["Missed evening blood pressure medication"],
  "medications": { "total": 9, "given": 8, "missed": 1, "skipped": 0 },
  "period": "February 12, 2026",
  "generatedAt": "2026-02-12T15:30:00.000Z"
}
```

### Smart Data Entry

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/ai/smart-entry/parse` | Parse natural language into a structured timeline entry |

**Request body**:

```json
{ "text": "Mom had breakfast at 8am, blood pressure 130/85, seemed tired" }
```

**Response** (`ParsedTimelineEntry`):

```json
{
  "type": "VITALS",
  "title": "Morning vitals and breakfast",
  "description": "Mom had breakfast at 8am, blood pressure 130/85, seemed tired",
  "severity": "LOW",
  "vitals": {
    "bloodPressureSystolic": 130,
    "bloodPressureDiastolic": 85
  }
}
```

The user reviews the parsed result on the frontend before confirming. The confirm action saves via the existing `POST /care-recipients/:id/timeline` endpoint.

### RAG -- Ask Questions

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/ai/ask` | Ask a natural language question about care history |

**Request body**:

```json
{
  "question": "How has medication adherence been this week?",
  "careRecipientId": "cr_abc123"
}
```

**Response** (`RagAnswer`):

```json
{
  "answer": "Based on the medication logs, adherence has been 92% this week...",
  "sources": [
    { "type": "medication", "title": "Lisinopril 10mg", "date": "2026-02-10", "id": "med_1", "similarity": 0.87 },
    { "type": "timeline_entry", "title": "Missed evening meds", "date": "2026-02-12", "id": "te_5", "similarity": 0.82 }
  ]
}
```

---

## Usage Guidelines

### For Developers

1. **AI features are opt-in.** If `GEMINI_API_KEY` is not set, all AI services gracefully degrade -- summaries return basic stats, smart entry returns the raw text as a NOTE, RAG returns a "not configured" message. The app works perfectly fine without it.

2. **Embedding is asynchronous.** When a timeline entry, medication, appointment, or document is created/updated/deleted, an embedding job is enqueued to BullMQ. The AI embedding worker processes it in the background. This means:
   - The main API request is never blocked by AI processing
   - If the worker is down, embeddings queue up and process when it restarts
   - If the Gemini API is unreachable, the job retries with exponential backoff

3. **All vector searches are family-scoped.** The `family_id` column in `ai_embeddings` is always filtered in queries, so family A can never see family B's data through RAG. This is enforced at the database query level, not just the application level.

4. **Rate limiting.** Gemini free tier allows 15 RPM (requests per minute) and 1 million tokens/day. The AI endpoints are already covered by the app-wide throttler. For high-traffic deployments, consider:
   - Caching summaries (they don't change often)
   - Adding a dedicated rate limit for `/ai/*` endpoints
   - Upgrading to Gemini paid tier

5. **Error handling.** All AI services catch Gemini API errors and return sensible fallbacks. The frontend components show retry buttons on failure. No AI error should crash the API.

### For Users (Feature Descriptions)

**AI Care Summary** (Dashboard):

- Appears as a card in the dashboard sidebar when a care recipient is selected
- Toggle between "Today" and "Week" views
- Shows an AI-generated narrative summary, medication stats, positive highlights, and items needing attention
- Refreshes on each page load (cached for 5 minutes)

**Smart Entry** (Timeline page):

- Click "Smart Entry -- type naturally, AI parses it" above the timeline filters
- Type a natural note like "Dad had lunch, BP 120/80, complained of headache"
- Click "Parse with AI" to see a structured preview with detected type, severity, title, and vitals
- Review and click "Confirm & Save" to add it to the timeline, or "Discard" to start over

**Ask CareCircle AI** (Dashboard):

- Click the sparkles icon in the dashboard header bar
- A slide-out panel opens with suggested questions
- Type any question about the care recipient's history
- AI answers using only the family's actual care records, with source citations
- Click a source to navigate to the original record

---

## Security Considerations

1. **Family data isolation.** All AI queries (summaries, RAG) verify that the requesting user is an active member of the care recipient's family. Vector search is always filtered by `family_id`.

2. **No data leaves the family scope.** Gemini receives de-identified context snippets (no user IDs, no emails). The prompts include only care-relevant data like medication names, timeline descriptions, and appointment details.

3. **Prompt injection mitigation.** User input is always placed in a clearly delineated section of the prompt (e.g., `Caregiver note: "{text}"`). System instructions explicitly tell the model to only use provided context.

4. **Input validation.** Smart entry text is capped at 2,000 characters. RAG questions are capped at 500 characters. Both are validated server-side.

5. **No PII in embeddings metadata.** The `metadata` JSONB column stores only titles and types, never user emails or passwords.

---

## Free Tier Limits & Cost

| Resource | Free Tier Limit | CareCircle Usage |
|----------|----------------|------------------|
| Gemini 2.0 Flash | 15 RPM, 1M tokens/day | ~50-100 requests/day typical |
| Gemini gemini-embedding-001 | 1,500 RPM, 100M tokens/day | Minimal (one embedding per data change) |
| Neon PostgreSQL (pgvector) | 512 MB storage, 0.25 CU | ~1KB per embedding row, thousands fit easily |
| Upstash Redis (BullMQ) | 10K commands/day | 2 new queues, minimal overhead |

**Estimated cost: $0/month** for typical family use (1-10 family members, <100 timeline entries/day).

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| "AI features are not configured" in responses | `GEMINI_API_KEY` not set | Add the env variable and restart the API |
| Summaries show basic stats instead of AI text | Gemini API key invalid or rate-limited | Check the key in Google AI Studio; check API logs for 429 errors |
| RAG returns "not enough care records" | No embeddings stored yet | Ensure the AI embedding worker is running; check BullMQ dashboard |
| Smart Entry always returns type NOTE | Gemini structured output failed | Check API logs for parse errors; the service falls back to NOTE on failure |
| `CREATE EXTENSION vector` fails | pgvector not installed on PostgreSQL | Use Neon (has pgvector built-in) or install pgvector locally |
| Embedding worker jobs stuck in queue | Worker not started or Redis unreachable | Check worker health endpoint `/health`; verify Redis connection |

---

## All AI Feature Suggestions (Brainstormed)

Before building, we brainstormed 10 AI features that fit naturally into CareCircle. **3 are implemented**, the rest are documented here for future reference. Each includes what it does, why it matters, and what tech it would use.

### Status Overview

| # | Feature | Status | Complexity |
|---|---------|--------|------------|
| 1 | RAG -- Ask Questions About Care History | **Implemented** | High |
| 2 | AI Care Summaries & Daily Briefings | **Implemented** | Medium |
| 3 | AI Chat Assistant (CareBot) | Future | Medium |
| 4 | Smart Data Entry -- Natural Language | **Implemented** | Medium |
| 5 | Vitals & Health Trend Analysis | Future | Low-Medium |
| 6 | Document Intelligence -- OCR & Extraction | Future | Medium |
| 7 | Medication Interaction Checker (AI-Powered) | Future | Low |
| 8 | Care Plan Suggestions & Proactive Recommendations | Future | Medium |
| 9 | Voice Notes with AI Transcription | Future | Low |
| 10 | Smart Notifications & Priority Ranking | Future | Medium |

---

### 1. RAG -- Ask Questions About Care History & Documents (Implemented)

**What it does**: Family members can ask natural-language questions and get answers sourced from their own data -- uploaded documents, timeline entries, medications, appointments, vitals.

**Example queries**:
- "What's Mom's insurance policy number?"
- "When was Dad's last blood sugar reading?"
- "What did the doctor say about the medication change last month?"
- "List all the medications Mom is currently taking and their dosages"

**How it works**: Documents get chunked and embedded into a vector database (pgvector in the existing PostgreSQL). Timeline entries, medications, and appointments are also embedded. When a user asks a question, it retrieves relevant chunks and feeds them to Gemini for a grounded answer with citations.

**Tech**: Gemini embeddings + pgvector (Neon) + Gemini 2.0 Flash for answers

---

### 2. AI Care Summaries & Daily Briefings (Implemented)

**What it does**: Auto-generates summaries at different granularities:
- **Daily Briefing**: "Today: 3 medications given on time, blood pressure was 130/85 (slightly elevated), Sarah completed the pharmacy pickup, 1 new timeline note from David"
- **Weekly Care Report**: A summary of the entire week's care activity
- **Appointment Prep** (future): Before a doctor visit, generates a summary of recent symptoms, vitals trends, medication changes, and suggested questions to ask

**Why it matters**: Long-distance caregivers can't check the app constantly. A summary catches them up in 30 seconds.

**Tech**: Gemini 2.0 Flash, BullMQ scheduled job (existing worker infrastructure), Brevo for email delivery (future)

---

### 3. AI Chat Assistant (CareBot) -- Future

**What it does**: An AI assistant available inside the family chat (or as a separate chat interface) that family members can @mention or message directly. It has context about the care recipient and can:
- Answer questions about the care plan
- Explain medication side effects in plain language
- Suggest what to do if a symptom appears
- Summarize what happened while someone was away ("@CareBot what did I miss this week?")

**Why it matters**: Not every family member is medically literate. An AI assistant bridges the knowledge gap without requiring a nurse in the chat.

**Tech**: Stream Chat (already integrated) with a bot user + Gemini API, or a standalone chat UI component

**Implementation notes**: The existing RAG service can be reused as the knowledge backbone. The main work is creating a bot user in Stream Chat that listens for @mentions and responds via the Gemini API with RAG context.

---

### 4. Smart Data Entry -- Natural Language to Structured Data (Implemented)

**What it does**: Instead of filling forms, caregivers type or speak naturally:
- "Mom ate well today, blood pressure 120/80, she was in a good mood" --> Creates a structured timeline entry with vitals (BP: 120/80), mood (good), meal (ate well)
- "Give Mom her Metformin at 2pm" --> Logs the medication as given
- Upload a photo of a prescription --> OCR extracts medication name, dosage, frequency, and pre-fills the medication form (future -- requires Gemini Vision)

**Why it matters**: Caregivers are busy and often on their phones. Reducing form friction means more data gets logged, which means better coordination.

**Tech**: Gemini 2.0 Flash for NL parsing with structured output (JSON mode)

---

### 5. Vitals & Health Trend Analysis -- Future

**What it does**: Analyzes vitals data logged in the timeline to:
- Detect trends ("Blood pressure has been rising over the past 2 weeks")
- Flag anomalies ("Heart rate of 110 is significantly above the usual range of 65-80")
- Generate weekly health trend charts with AI-written commentary
- Send proactive alerts when something looks off

**Why it matters**: Individual readings are hard to interpret. Patterns across time tell the real story, and most family caregivers aren't trained to spot them.

**Tech**: Simple statistical analysis (no LLM needed for detection) + Gemini for human-readable explanations, Tremor charts (already in the stack) for visualization

**Implementation notes**: Collect vitals data from `TimelineEntry` records where `type = VITALS`. Calculate rolling averages, standard deviations, and flag readings > 2 standard deviations from the mean. Send the flagged data to Gemini for a plain-language explanation. Could run as a daily BullMQ job.

---

### 6. Document Intelligence -- OCR, Extraction & Classification -- Future

**What it does**: When a document is uploaded (medical record, lab result, prescription, insurance card):
- **Auto-classify**: Detect what type of document it is
- **Extract key data**: Pull out medication names, lab values, dates, provider names
- **Summarize**: "Lab results from Jan 15: Cholesterol 210 (borderline high), A1C 6.8 (pre-diabetic range), kidney function normal"
- **Link to care**: Auto-suggest creating medications, appointments, or timeline entries from extracted data

**Why it matters**: Families upload documents but rarely read them thoroughly. AI makes the documents actionable.

**Tech**: Gemini with vision capabilities (for scanned PDFs/images), Gemini Flash for text-based PDFs, Cloudinary (already used) for image processing

**Implementation notes**: The document service already handles uploads. Add a post-upload BullMQ job that sends the document to Gemini Vision for analysis. Store extracted data in document metadata. The embedding pipeline already indexes documents -- extend it to include extracted text.

---

### 7. Medication Interaction Checker (AI-Powered) -- Future

**What it does**: Replaces the current hardcoded interaction list (`medication-interactions.service.ts`) with an AI-powered checker that:
- Checks new medications against all current medications for interactions
- Explains the severity and what to watch for in plain language
- Suggests alternatives or flags when to consult the doctor
- Can also check against the care recipient's conditions and allergies

**Why it matters**: The hardcoded list covers common interactions but misses edge cases. An AI-powered version is more comprehensive and can explain in context.

**Tech**: Gemini function calling + a medical knowledge base (OpenFDA API for drug data + LLM for explanation), or RAG over a drug interaction dataset

**Implementation notes**: The existing `medication-interactions.service.ts` already has the interface. Replace the hardcoded lookup with a Gemini call that takes the full medication list + conditions as context. Add a disclaimer that this is informational, not medical advice.

---

### 8. Care Plan Suggestions & Proactive Recommendations -- Future

**What it does**: Based on the care recipient's profile (conditions, allergies, age, medications), the AI proactively suggests:
- Daily care activities ("Patients with dementia benefit from structured routines -- consider adding a morning walk task")
- Dietary considerations ("Metformin works better when taken with food")
- Preventive measures ("Based on Mom's age and conditions, consider scheduling a flu shot")
- Task suggestions when gaps are detected ("No one has been assigned to the Wednesday evening shift")

**Why it matters**: Most families are reactive, not proactive. AI can gently guide better care practices.

**Tech**: Gemini Flash with care recipient context, scheduled analysis via BullMQ

**Implementation notes**: Run weekly via a BullMQ job. Collect the care recipient's profile, recent activity, upcoming gaps (unassigned shifts, overdue appointments). Send to Gemini with a "proactive care advisor" system prompt. Store suggestions as a special type of notification or dashboard card.

---

### 9. Voice Notes with AI Transcription -- Future

**What it does**: Caregivers record a voice note from their phone. The AI:
- Transcribes it to text
- Extracts structured data (vitals, mood, symptoms)
- Creates the appropriate timeline entry, medication log, or note
- Optionally shares it in the family chat

**Why it matters**: Typing on a phone while caring for someone is hard. Speaking is natural and fast.

**Tech**: Google Cloud Speech-to-Text (free tier available) or OpenAI Whisper API for transcription, Gemini for parsing (reuse the Smart Entry service)

**Implementation notes**: The Smart Entry service already handles natural language to structured data. The only new piece is audio transcription. Add a microphone button to the Smart Entry input that records audio, transcribes it, then pipes the text through the existing parsing flow.

---

### 10. Smart Notifications & Priority Ranking -- Future

**What it does**: Instead of bombarding everyone with every notification, AI:
- Prioritizes notifications based on urgency and the user's role
- Groups related notifications ("3 medications given on time this morning")
- Writes natural-language notification summaries instead of generic templates
- Learns which notifications each family member actually acts on

**Why it matters**: Notification fatigue is real. Caregivers get desensitized and miss the important ones.

**Tech**: Gemini Flash for summary generation, simple scoring logic for priority

**Implementation notes**: The notification system already exists with multi-channel delivery (email, SMS, push, in-app). Add a preprocessing step in the notification worker that batches related notifications, scores priority (HIGH for emergencies/missed meds, LOW for routine updates), and generates natural language summaries. No new models needed -- just smarter orchestration.

---

### Quick Comparison Chart

| Feature | Impact | Complexity | Cost | Uniqueness |
|---------|--------|------------|------|------------|
| RAG (docs + care history) | High | High | Free (Gemini) | Very High |
| AI Care Summaries | High | Medium | Free (Gemini) | High |
| AI Chat Assistant (CareBot) | High | Medium | Free (Gemini) | High |
| Smart Data Entry (NL) | High | Medium | Free (Gemini) | Very High |
| Vitals Analysis | Medium | Low-Medium | Free | Medium |
| Document Intelligence | High | Medium | Free (Gemini Vision) | High |
| Medication Interactions | Medium | Low | Free (Gemini) | Medium |
| Care Plan Suggestions | Medium | Medium | Free (Gemini) | High |
| Voice Notes + Transcription | Medium | Low | Free or Low | Medium |
| Smart Notifications | Medium | Medium | Free (Gemini) | Medium |

---

## What CareCircle Already Has (Beyond AI)

CareCircle is a full-featured caregiving coordination platform. AI is one layer on top of a comprehensive set of features that work without any AI at all. Here is everything the platform currently offers:

### Core Platform Features

| Area | What it does | Key capabilities |
|------|-------------|------------------|
| **Authentication & Security** | Secure access for all users | JWT login, registration, email verification, password reset, session management, account lockout, role-based access (ADMIN / CAREGIVER / VIEWER) |
| **Family Management** | Organize caregiving teams | Create families, invite members via email/link, assign roles, accept invitations, manage multiple families |
| **Care Recipients** | Profile for the person receiving care | Medical info (conditions, allergies, blood type), doctors list, emergency contacts, preferred name, date of birth |
| **Medications** | Full medication lifecycle | Add medications with dosage/frequency/schedule, log doses (given/missed/skipped), supply tracking with refill alerts, medication interaction warnings |
| **Appointments** | Healthcare scheduling | Create appointments with doctor details, recurring appointments (iCal rrule), transport assignment (who drives), appointment reminders |
| **Timeline** | Comprehensive health journal | 10+ entry types (notes, vitals, incidents, mood, meals, sleep, activity, symptoms, medication, appointments), severity levels, caregiver attribution |
| **Documents** | Secure family document vault | Upload medical records, insurance cards, prescriptions, legal documents; Cloudinary cloud storage; organized by document type |
| **Family Chat** | Real-time communication | Stream Chat integration, family-scoped channels, real-time messaging, read receipts |
| **Caregiver Shifts** | Shift scheduling | Create/assign shifts, check-in/check-out, shift status tracking, shift reminders |
| **Emergency System** | Emergency alerts | Emergency button (fall, medical, fire, missing, other), alert all family members instantly, emergency contact display |
| **Notifications** | Multi-channel alerts | In-app notifications, browser push (Web Push / VAPID), email (Brevo/SMTP), SMS (Twilio), notification preferences per user |
| **Admin Dashboard** | Platform administration | User management, family management, HIPAA-style audit logs, analytics dashboard, system monitoring, application logs |

### Technical Infrastructure (Already Built)

| Component | Technology | What it provides |
|-----------|-----------|------------------|
| **Real-time updates** | Socket.io WebSockets | Live notifications, timeline updates, shift status changes |
| **Domain events** | RabbitMQ | Decoupled event publishing (e.g., medication logged → trigger notification → trigger audit log) |
| **Background jobs** | BullMQ + Redis | Medication reminders, appointment reminders, shift reminders, refill alerts, notification delivery, dead-letter queue |
| **Offline support** | Service Worker + LocalForage | Offline fallback page, cached assets, local data persistence |
| **Internationalization** | i18n | Multi-language support (English, French, extensible) |
| **Monitoring** | Prometheus metrics + Sentry | Application metrics, error tracking, performance monitoring |
| **Analytics** | Vercel Analytics + Speed Insights | Page views, user engagement, Core Web Vitals |
| **Database** | PostgreSQL (Neon) + Prisma ORM | 20+ data models, migrations, type-safe queries |
| **File storage** | Cloudinary | Document and image uploads, secure delivery, transformations |
| **Caching** | Redis | API response caching, session storage, rate limiting |

### Background Workers (Automated Jobs)

| Worker | Schedule | What it does |
|--------|----------|-------------|
| Medication Reminder | Runs on scheduler interval | Checks upcoming medication doses, sends reminders before scheduled time |
| Appointment Reminder | Runs on scheduler interval | Sends reminders for upcoming appointments (24h and 1h before) |
| Shift Reminder | Runs on scheduler interval | Notifies caregivers before their shifts start |
| Refill Alert | Hourly | Monitors medication supply levels, alerts when running low |
| Notification Delivery | Event-driven | Delivers notifications across all channels (push, email, SMS, in-app) |
| AI Embedding | Event-driven | Generates and stores vector embeddings for RAG search |
| AI Summary | On-demand / scheduled | Generates daily/weekly care summaries |
| Dead Letter | Continuous | Captures and logs failed jobs for debugging |

> **Key takeaway**: CareCircle is not an "AI app" -- it is a full caregiving platform with AI layered on top. Every feature works without AI. The 3 AI features (summaries, smart entry, RAG) enhance the experience but are not required.

---

## Free vs Paid -- What You Get at Each Tier

CareCircle is built entirely on free-tier services. Here is what each service provides for free, and what upgrading would unlock.

### Current Free Tier Stack

| Service | Free Tier | What CareCircle uses | Limit that matters |
|---------|-----------|---------------------|--------------------|
| **Neon PostgreSQL** | 0.5 GB storage, 1 project, branching | Main database + pgvector embeddings | 0.5 GB total (thousands of families fit) |
| **Upstash Redis** | 10K commands/day, 256 MB | BullMQ queues, caching | 10K commands/day (enough for ~50 active families) |
| **Google Gemini** | 15 RPM (Flash), 1M tokens/day | Summaries, smart entry, RAG, embeddings | 15 requests/minute for text generation |
| **Stream Chat** | 100 monthly active users, 5 channels | Family chat | 100 MAU (fine for early stage) |
| **Cloudinary** | 25 credits/month (~25 GB storage) | Document uploads, images | 25 GB storage + 25 GB bandwidth |
| **Brevo (Email)** | 300 emails/day | Email verification, notifications, summaries | 300 emails/day |
| **Twilio (SMS)** | Trial credit (~$15) | SMS notifications | Runs out after ~150 SMS |
| **Vercel** | 100 GB bandwidth, serverless | Frontend hosting | 100 GB bandwidth/month |
| **Render** | 750 hours/month (free instance) | API + workers hosting | Instance sleeps after 15 min idle |

**Total monthly cost: $0** for a small deployment (1-10 families, <50 active users).

### What Paid Tiers Unlock

| Upgrade | Cost | What it unlocks | When you need it |
|---------|------|----------------|------------------|
| **Neon Pro** | $19/month | 10 GB storage, autoscaling, more compute | >500 families or >100K embeddings |
| **Upstash Pro** | $10/month | 100K commands/day, 1 GB | >50 active families sending frequent reminders |
| **Gemini Pay-as-you-go** | ~$0.075/1M tokens (Flash) | No RPM limits, higher throughput | >100 AI requests/day or >15 simultaneous users |
| **Stream Chat Startup** | $99/month | 10K MAU, 25 channels, message history | >100 active users using chat |
| **Cloudinary Plus** | $89/month | 225 credits, more storage/bandwidth | Lots of document uploads |
| **Brevo Starter** | $25/month | 20K emails/month, no daily limit | Scheduled email summaries for many families |
| **Twilio Pay-as-you-go** | ~$0.0079/SMS | Unlimited SMS | Regular SMS notifications |
| **Render Starter** | $7/month per service | No sleep, better performance | Production deployment (API must not sleep) |
| **Vercel Pro** | $20/month | 1 TB bandwidth, analytics | High-traffic frontend |

### Realistic Growth Scenarios

| Stage | Users | Monthly cost | Key upgrades needed |
|-------|-------|-------------|---------------------|
| **Development / MVP** | 1-5 | **$0** | Nothing -- free tier covers everything |
| **Friends & Family** | 5-50 | **$0** | Still free, monitor Redis commands |
| **Early Adopters** | 50-200 | **~$30-50** | Render Starter ($7 x2), Upstash Pro ($10), maybe Brevo Starter ($25) |
| **Growing** | 200-1,000 | **~$150-250** | + Neon Pro ($19), Stream Chat ($99), Gemini paid tier |
| **Scale** | 1,000-10,000 | **~$500-1,000** | + Cloudinary Plus, dedicated infrastructure, CDN |

> **Bottom line**: You can build, launch, and serve your first 50 users at $0/month. The first dollar you spend should be on Render Starter ($7) so the API does not sleep.

---

## Additional AI & Automation Features (Trending Ideas)

Beyond the 10 AI features already brainstormed, here are additional ideas inspired by current industry trends in healthcare tech, AI agents, and workflow automation.

### AI Agents & Autonomous Workflows

| # | Feature | What it does | Trend | Complexity |
|---|---------|-------------|-------|------------|
| 11 | **AI Care Coordinator Agent** | An autonomous agent that monitors all care data daily and takes actions: reschedules missed medications, suggests shift swaps for uncovered times, drafts appointment follow-ups. Runs as a daily BullMQ job, generates a "suggested actions" list for the family admin to approve or dismiss. | AI Agents (2025-2026 trend) | High |
| 12 | **Automated Shift Scheduling** | AI analyzes caregiver availability, care recipient needs (medication times, appointment schedule), and historical patterns to auto-generate optimal weekly shift schedules. Family admin reviews and publishes. | Workforce automation | Medium |
| 13 | **Smart Escalation Chains** | When a critical event happens (missed critical medication, fall detected, abnormal vitals), AI decides who to notify first based on availability, proximity, and role. If no response in 5 minutes, auto-escalates to the next person. If no response in 15 minutes, calls emergency contacts. | Incident response automation | Medium |
| 14 | **Medication Adherence Predictor** | Learns from historical medication logs to predict which doses are likely to be missed (e.g., "Evening Metformin is missed 40% of the time on weekends"). Proactively sends reminders earlier or suggests schedule adjustments. | Predictive AI | Medium |

### Workflow Automation (No AI Required)

| # | Feature | What it does | Trend | Complexity |
|---|---------|-------------|-------|------------|
| 15 | **If-This-Then-That Care Rules** | Family admins create automation rules: "IF blood pressure > 140/90 THEN notify all family members" or "IF medication missed THEN send SMS to caregiver on duty." A simple rule engine that evaluates conditions on timeline entries and triggers actions. | Low-code automation (Zapier-like) | Medium |
| 16 | **Auto-Generated Care Reports (PDF)** | Weekly/monthly PDF reports with charts, medication adherence graphs, vitals trends, and AI narrative. Auto-emailed to family members or downloadable for doctor visits. Uses Puppeteer or @react-pdf for generation. | Report automation | Low-Medium |
| 17 | **Appointment Auto-Booking Reminders** | When a recurring appointment pattern is detected (e.g., cardiology every 3 months), the system reminds the family to book the next one 2 weeks before the interval is up. Tracks the recurrence pattern from historical appointments. | Calendar automation | Low |
| 18 | **Automatic Medication Refill Orders** | When supply tracking shows <7 days of medication remaining, auto-generate a pharmacy refill request (formatted message or email) that the caregiver just needs to approve and send. Future: integrate with pharmacy APIs. | Supply chain automation | Low |
| 19 | **Shift Handoff Automation** | When a caregiver checks out from a shift, auto-generate a handoff note summarizing what happened during their shift (medications given, incidents, vitals) using the AI summary pipeline. The incoming caregiver sees it immediately. | Process automation | Medium |

### Trending Health Tech Features

| # | Feature | What it does | Trend | Complexity |
|---|---------|-------------|-------|------------|
| 20 | **Wearable Device Integration** | Connect Apple Health, Google Fit, or Fitbit to automatically import vitals (heart rate, steps, sleep quality, blood oxygen). Data flows into the timeline automatically, no manual entry. Uses Apple HealthKit / Google Health Connect APIs. | IoT + health data | High |
| 21 | **Telehealth Integration** | One-click video call with the care recipient's doctor directly from the appointment page. Integrate with Twilio Video or Daily.co. After the call, AI auto-generates a visit summary from the transcript. | Telehealth boom | High |
| 22 | **Family Wellness Score** | A daily score (0-100) combining medication adherence, vitals stability, activity level, mood trends, and caregiver coverage. Displayed as a dashboard widget. AI provides a one-sentence explanation ("Score dropped because evening meds were missed twice this week"). | Gamification + health metrics | Medium |
| 23 | **Caregiver Burnout Detection** | Track caregiver activity patterns: hours worked, response times, shift frequency. AI flags potential burnout ("Sarah has worked 12 of the last 14 days with no break"). Suggests redistributing shifts or adding respite care. | Caregiver wellness (trending in healthcare) | Medium |
| 24 | **Multi-Language AI Responses** | AI summaries, smart entry parsing, and RAG answers in the user's preferred language. Gemini natively supports 40+ languages. Uses the existing i18n language preference to set the prompt language. | Multilingual AI | Low |
| 25 | **Emergency Fall Detection via Phone** | Use the device's accelerometer (via a PWA or native companion app) to detect falls. Auto-trigger an emergency alert if a sudden impact + no movement is detected for 30 seconds. | Mobile health / IoT | High |

### AI-Enhanced Existing Features

| # | Feature | What it does | Enhances | Complexity |
|---|---------|-------------|----------|------------|
| 26 | **Smart Appointment Prep** | Before a doctor appointment, AI generates a one-page summary: recent vitals trends, medication changes, new symptoms, and suggested questions for the doctor. Shareable as PDF or link. | Appointments | Medium |
| 27 | **Intelligent Document Filing** | When a document is uploaded, AI auto-classifies it (insurance card, lab result, prescription, legal document) and suggests the correct category, tags, and associated care recipient. No manual filing. | Documents | Low-Medium |
| 28 | **AI-Powered Search** | Replace keyword search across the app with semantic search. "Mom's doctor" finds the cardiologist even if you never typed "doctor." Uses the existing embedding infrastructure. | Global search | Medium |
| 29 | **Care Recipient Mood Tracker** | AI analyzes mood mentions across timeline entries ("seemed happy," "was irritable," "great mood today") and generates a weekly mood chart with AI commentary. | Timeline | Low-Medium |
| 30 | **Automated Insurance Claim Helper** | AI reads uploaded insurance documents (via OCR) and medical bills, cross-references them, and flags discrepancies or suggests what to submit for reimbursement. | Documents + Finance | High |

### Comparison: All Features (Existing + Future)

| # | Feature | Status | AI? | Automation? | Free? | Impact |
|---|---------|--------|-----|-------------|-------|--------|
| 1 | RAG -- Ask AI | **Implemented** | Yes | No | Yes | High |
| 2 | AI Care Summaries | **Implemented** | Yes | No | Yes | High |
| 3 | AI Chat Assistant (CareBot) | Future | Yes | No | Yes | High |
| 4 | Smart Data Entry | **Implemented** | Yes | No | Yes | High |
| 5 | Vitals Trend Analysis | Future | Yes | No | Yes | Medium |
| 6 | Document Intelligence / OCR | Future | Yes | No | Yes | High |
| 7 | Medication Interaction Checker | Future | Yes | No | Yes | Medium |
| 8 | Care Plan Suggestions | Future | Yes | No | Yes | Medium |
| 9 | Voice Notes + Transcription | Future | Yes | No | Free/Low | Medium |
| 10 | Smart Notifications | Future | Yes | Yes | Yes | Medium |
| 11 | AI Care Coordinator Agent | Future | Yes | Yes | Yes | Very High |
| 12 | Automated Shift Scheduling | Future | Yes | Yes | Yes | High |
| 13 | Smart Escalation Chains | Future | Yes | Yes | Yes | High |
| 14 | Medication Adherence Predictor | Future | Yes | No | Yes | Medium |
| 15 | Care Rules Engine (IFTTT) | Future | No | Yes | Yes | High |
| 16 | Auto PDF Care Reports | Future | Yes | Yes | Yes | High |
| 17 | Appointment Auto-Booking | Future | No | Yes | Yes | Medium |
| 18 | Auto Medication Refill | Future | No | Yes | Yes | Medium |
| 19 | Shift Handoff Automation | Future | Yes | Yes | Yes | High |
| 20 | Wearable Device Integration | Future | No | Yes | Yes | Very High |
| 21 | Telehealth Integration | Future | Yes | No | Paid | High |
| 22 | Family Wellness Score | Future | Yes | No | Yes | Medium |
| 23 | Caregiver Burnout Detection | Future | Yes | No | Yes | High |
| 24 | Multi-Language AI | Future | Yes | No | Yes | Medium |
| 25 | Emergency Fall Detection | Future | No | Yes | Yes | High |
| 26 | Smart Appointment Prep | Future | Yes | Yes | Yes | High |
| 27 | Intelligent Document Filing | Future | Yes | Yes | Yes | Medium |
| 28 | AI-Powered Search | Future | Yes | No | Yes | Medium |
| 29 | Mood Tracker | Future | Yes | No | Yes | Low-Medium |
| 30 | Insurance Claim Helper | Future | Yes | No | Yes | Medium |

> **26 out of 30 features are completely free** using Gemini free tier + existing infrastructure. The only features that might require paid services are telehealth (video API) and high-volume wearable data ingestion.

### Implementation Priority Recommendation

If building features beyond the current 3 implemented AI features, here is the recommended order based on impact, complexity, and user demand:

**Phase 1 -- Quick Wins (1-2 weeks each)**
1. Multi-Language AI Responses (#24) -- Just change the prompt language, minimal code
2. Intelligent Document Filing (#27) -- Reuse existing Gemini structured output
3. Auto PDF Care Reports (#16) -- High demand from families visiting doctors
4. Appointment Auto-Booking Reminders (#17) -- Simple cron job logic

**Phase 2 -- High Impact (2-4 weeks each)**
5. AI Chat Assistant / CareBot (#3) -- Reuses RAG, high user engagement
6. Care Rules Engine / IFTTT (#15) -- No AI needed, pure automation
7. Shift Handoff Automation (#19) -- Reuses summary pipeline
8. Smart Appointment Prep (#26) -- Reuses summary pipeline + PDF generation

**Phase 3 -- Differentiators (4-8 weeks each)**
9. Vitals Trend Analysis (#5) -- Charts + AI commentary
10. Document Intelligence / OCR (#6) -- Gemini Vision, high impact
11. Caregiver Burnout Detection (#23) -- Unique in the market
12. Family Wellness Score (#22) -- Engagement driver

**Phase 4 -- Advanced (8+ weeks each)**
13. AI Care Coordinator Agent (#11) -- Autonomous workflows, cutting-edge
14. Wearable Device Integration (#20) -- Hardware APIs, high complexity
15. Telehealth Integration (#21) -- Video calling, third-party dependency

---

## Why We Chose Gemini (and Not OpenAI/Claude)

| Factor | Google Gemini | OpenAI | Anthropic (Claude) |
|--------|--------------|--------|-------------------|
| **Free tier** | 15 RPM, 1M tokens/day, embeddings free | No free tier (pay from first request) | No free tier |
| **Embeddings** | Built-in, free (gemini-embedding-001) | ada-002 costs $0.10/1M tokens | Not available |
| **Vision/OCR** | Included in free tier | GPT-4o Vision costs extra | Claude Vision costs extra |
| **Structured output** | Native JSON mode | Function calling | Tool use |
| **SDK** | `@google/generative-ai` (well-maintained) | `openai` (excellent) | `@anthropic-ai/sdk` (good) |
| **Deployment** | Single API key, no server needed | Same | Same |

**Bottom line**: Gemini's free tier covers everything CareCircle needs for a typical family (1-10 members, <100 entries/day). If the app grows beyond free limits, switching to OpenAI or Claude is a config change -- the service abstraction (`GeminiService`) isolates all provider-specific code.

---

## Future Enhancements (Implementation Priorities)

Beyond the 10 features above, here are specific enhancements to the existing implemented features:

- **Appointment prep summaries** -- generate a pre-visit summary of recent symptoms, vitals, and medication changes to bring to the doctor
- **Backfill worker** -- a one-time job that embeds all existing data for families that enable AI after having existing records
- **Document OCR** -- extract text from uploaded PDFs/images and embed it for RAG (requires Gemini Vision)
- **Conversation persistence** -- save Ask AI chat history to the database for reference
- **Scheduled email summaries** -- daily/weekly summary emails sent to family members via the summary worker + Brevo
- **Anomaly detection** -- flag unusual patterns in vitals or medication adherence using Gemini analysis

---

## How Each Service Works (Code Walkthrough)

### GeminiService (`apps/api/src/ai/services/gemini.service.ts`)

This is the lowest-level service -- a thin wrapper around Google's `@google/generative-ai` SDK. It exposes three methods:

| Method | What it does | Used by |
|--------|-------------|---------|
| `generateText(prompt)` | Sends a prompt, gets back free-form text | RagService (final answer) |
| `generateStructured(prompt, schema)` | Sends a prompt with a JSON schema, gets back typed JSON | CareSummaryService, SmartEntryService |
| `generateEmbedding(text)` | Sends text, gets back a `number[768]` vector | EmbeddingService, AI Embedding Worker |

If `GEMINI_API_KEY` is not set, all methods return `null` and log a warning. This is why the app works fine without the key -- every caller checks for `null` and falls back gracefully.

### EmbeddingService (`apps/api/src/ai/services/embedding.service.ts`)

Manages the `ai_embeddings` PostgreSQL table. Does **not** call Gemini directly -- it just stores/queries vectors.

- `storeEmbedding(content, embedding, metadata)` -- inserts a row with the vector
- `search(queryEmbedding, familyId, limit)` -- runs the cosine similarity query scoped to a family
- `deleteByResource(resourceType, resourceId)` -- removes embeddings when source data is deleted

Uses raw SQL via Prisma's `$queryRawUnsafe` because Prisma does not have native pgvector support yet.

### EmbeddingIndexerService (`apps/api/src/ai/services/embedding-indexer.service.ts`)

The glue between feature modules and the AI system. When any CRUD happens:

```typescript
// In timeline.service.ts, after creating an entry:
this.embeddingIndexer?.indexTimelineEntry(entry);

// This enqueues a BullMQ job -- does NOT block the request
```

It constructs a human-readable text representation of the resource (e.g., "Timeline Entry: Morning vitals check. Blood pressure 130/85. Severity: LOW.") and sends it to the queue. The worker then embeds this text.

### CareSummaryService (`apps/api/src/ai/services/care-summary.service.ts`)

Collects data from 4 Prisma models for a given care recipient and time period, then asks Gemini to summarize it. The prompt is carefully structured:

```
You are a care assistant summarizing care activities for a family caregiver.
Given the following care data for [period], generate a summary.

Timeline Entries: [list of entries with dates and descriptions]
Medication Logs: [given: 8, missed: 1, skipped: 0, details...]
Appointments: [list]
Caregiver Shifts: [who was on duty and when]

Return JSON matching this schema: { summary, highlights, concerns, medications }
```

### SmartEntryService (`apps/api/src/ai/services/smart-entry.service.ts`)

Takes free-form text and uses Gemini structured output to parse it. The schema tells Gemini exactly what fields to extract:

- `type` -- one of: NOTE, VITALS, INCIDENT, MOOD, ACTIVITY, MEAL, SLEEP, MEDICATION, APPOINTMENT, OTHER
- `title` -- a short title (max 100 chars)
- `description` -- cleaned up version of the input
- `severity` -- LOW, MEDIUM, HIGH, CRITICAL
- `vitals` -- optional object with bloodPressureSystolic, bloodPressureDiastolic, heartRate, temperature, weight, bloodSugar, oxygenSaturation

If parsing fails, it falls back to `{ type: "NOTE", title: first 100 chars, description: original text }`.

### RagService (`apps/api/src/ai/services/rag.service.ts`)

Orchestrates the full RAG pipeline in one method:

1. Call `GeminiService.generateEmbedding(question)` to vectorize the question
2. Call `EmbeddingService.search(vector, familyId, 5)` to find top 5 relevant records
3. Build a prompt with the retrieved records as context
4. Call `GeminiService.generateText(prompt)` for the final answer
5. Return `{ answer, sources }` where sources include the record type, title, date, and similarity score

The prompt explicitly says: "Only use the provided context. If the context does not contain relevant information, say so. Do not make up information."

---

## Key Design Decisions

| Decision | Why |
|----------|-----|
| **pgvector instead of Pinecone/Weaviate** | Zero cost, zero new infrastructure. We already have PostgreSQL. pgvector handles our scale (thousands of records) easily. |
| **BullMQ for async embedding** | API stays fast. Embedding generation takes 200-500ms per item -- too slow for synchronous requests. BullMQ gives us retry logic, dead-letter queues, and backpressure for free. |
| **Gemini Flash over GPT-4o/Claude** | Free tier is sufficient for small-to-medium families. The quality difference is negligible for care summaries and simple Q&A. Easy to swap later by changing the model name in config. |
| **768-dim embeddings** | Google's gemini-embedding-001 outputs 768 dimensions. This is a good balance of accuracy vs. storage size (~3KB per embedding row). |
| **Family-scoped vector search** | Security-first. We filter by `family_id` at the SQL level, not application level. Even if application code had a bug, the query physically cannot return another family's data. |
| **Optional `@Inject` for EmbeddingIndexer** | Feature modules (timeline, medications, etc.) inject EmbeddingIndexer as `@Optional()`. If AiModule is not loaded (e.g., in tests), indexing silently skips. No coupling between core features and AI. |
| **Structured output over free-text parsing** | Gemini's JSON mode guarantees valid JSON matching our schema. No need for fragile regex parsing of LLM output. |

---

## Learning Resources

### Google Gemini

| Resource | Link | What you will learn |
|----------|------|-------------------|
| Google AI Studio (playground) | https://aistudio.google.com | Test prompts interactively, manage API keys, see token usage |
| Gemini API Quickstart | https://ai.google.dev/gemini-api/docs/quickstart | How to call Gemini from Node.js with the SDK |
| Structured Output Guide | https://ai.google.dev/gemini-api/docs/structured-output | How `responseMimeType: "application/json"` and schemas work |
| Embeddings Guide | https://ai.google.dev/gemini-api/docs/embeddings | How gemini-embedding-001 works, embedding dimensions, best practices |
| Free Tier Limits | https://ai.google.dev/pricing | Exact RPM, TPM, and daily limits for each model |

### pgvector & Vector Search

| Resource | Link | What you will learn |
|----------|------|-------------------|
| pgvector GitHub | https://github.com/pgvector/pgvector | Installation, data types, operators (`<=>`, `<#>`, `<->`), index types |
| pgvector on Neon | https://neon.tech/docs/extensions/pgvector | How to enable pgvector on Neon free tier, performance tips |
| HNSW Explained (blog) | https://www.pinecone.io/learn/series/faiss/hnsw/ | Visual explanation of how HNSW indexing works |
| Cosine Similarity (Wikipedia) | https://en.wikipedia.org/wiki/Cosine_similarity | The math behind vector similarity (with diagrams) |

### RAG (Retrieval-Augmented Generation)

| Resource | Link | What you will learn |
|----------|------|-------------------|
| RAG Explained (AWS) | https://aws.amazon.com/what-is/retrieval-augmented-generation/ | Plain-English explanation of the RAG pattern |
| RAG with LangChain (tutorial) | https://js.langchain.com/docs/tutorials/rag/ | Step-by-step RAG implementation (we did it without LangChain for simplicity) |
| Prompt Engineering Guide | https://ai.google.dev/gemini-api/docs/prompting-strategies | How to write better prompts for Gemini (applies to all our services) |

### BullMQ & Job Queues

| Resource | Link | What you will learn |
|----------|------|-------------------|
| BullMQ Documentation | https://docs.bullmq.io | Queues, workers, job lifecycle, retry strategies, events |
| BullMQ + NestJS | https://docs.nestjs.com/techniques/queues | How NestJS integrates with BullMQ (we use this in AiModule) |
| Redis Basics | https://redis.io/docs/get-started/ | Understanding the backbone that powers BullMQ |

### NestJS (Backend Framework)

| Resource | Link | What you will learn |
|----------|------|-------------------|
| NestJS Modules | https://docs.nestjs.com/modules | How AiModule is structured and why services are exported |
| NestJS Config | https://docs.nestjs.com/techniques/configuration | How `registerAs` and `@Inject(aiConfig.KEY)` work |
| NestJS Guards | https://docs.nestjs.com/guards | How `FamilyAccessGuard` protects AI endpoints |

### Prisma (Database ORM)

| Resource | Link | What you will learn |
|----------|------|-------------------|
| Prisma Raw Queries | https://www.prisma.io/docs/orm/prisma-client/using-raw-sql/raw-queries | Why we use `$queryRawUnsafe` for pgvector (Prisma does not support vector types natively yet) |
| Prisma Migrations | https://www.prisma.io/docs/orm/prisma-migrate | How the `20260212000000_add_ai_embeddings` migration works |

### React Query (Frontend Data Fetching)

| Resource | Link | What you will learn |
|----------|------|-------------------|
| React Query Docs | https://tanstack.com/query/latest | How `useQuery` and `useMutation` work (used in our `use-ai.ts` hooks) |
| React Query + Next.js | https://tanstack.com/query/latest/docs/framework/react/guides/advanced-ssr | SSR considerations for our dashboard summary queries |

---

## Glossary

| Term | Definition |
|------|-----------|
| **Abstractive Summarization** | A summarization approach where the AI generates new sentences that capture the meaning of the source data, rather than copying sentences verbatim. This is what Gemini does for care summaries. |
| **Data Aggregation** | Collecting data from multiple database tables (timeline, medications, appointments, shifts) into a single context block before sending it to the LLM. The summarization pipeline aggregates 4-5 Prisma queries in parallel. |
| **LLM** | Large Language Model -- an AI model trained on vast text data that can generate, summarize, and understand text (Gemini, GPT, Claude) |
| **Token** | The basic unit of text for LLMs. ~4 characters of English. "Hello world" = ~2 tokens. Pricing and limits are measured in tokens. |
| **Embedding** | A list of numbers (vector) representing the meaning of text. Similar meanings = similar numbers. Used for semantic search. |
| **Vector** | An ordered list of numbers, like `[0.12, -0.45, 0.78]`. Embeddings are vectors with 768 dimensions. |
| **Dimension** | The length of an embedding vector. gemini-embedding-001 uses 768 dimensions. Higher dimensions = more nuance but more storage. |
| **Cosine Similarity** | A measure of how similar two vectors are. 1.0 = identical, 0.0 = unrelated. Used to find "nearby" embeddings. |
| **HNSW** | Hierarchical Navigable Small World -- a fast algorithm for finding approximate nearest neighbors in high-dimensional vector space. |
| **pgvector** | A PostgreSQL extension that adds vector data types and similarity search operators. Lets us store embeddings in our existing DB. |
| **RAG** | Retrieval-Augmented Generation -- a pattern: embed the question, retrieve relevant data, generate an answer grounded in that data. |
| **Structured Output** | Forcing an LLM to return valid JSON matching a specific schema, instead of free-form text. |
| **BullMQ** | A Node.js job queue library backed by Redis. We use it to process embedding and summary jobs asynchronously. |
| **Queue** | A list of jobs waiting to be processed. Jobs are added by the API and consumed by workers. |
| **Worker** | A background process that pulls jobs from a queue and processes them. Runs in `apps/workers/`. |
| **Dead Letter Queue (DLQ)** | Where failed jobs go after exhausting all retries. Useful for debugging. |
| **RPM** | Requests Per Minute -- Gemini free tier allows 15 RPM for Flash and 1,500 RPM for embeddings. |
| **Prompt** | The text instruction sent to an LLM. Good prompts = good outputs. We carefully craft prompts in each service. |
| **Prompt Engineering** | The practice of carefully writing LLM instructions to get optimal output. Includes techniques like role assignment, context injection, explicit formatting, and constraint setting. |
| **Prompt Injection** | An attack where user input tricks the LLM into ignoring its instructions. We mitigate this by clearly separating user input from system instructions. |
| **Graceful Degradation** | When AI is unavailable (no key, rate limited, error), the app continues to work normally -- just without AI features. Nothing breaks. |
| **Semantic Search** | Searching by meaning rather than exact keywords. "blood pressure meds" finds results about "hypertension medication" because their embeddings are similar. |
| **System Instruction** | A special instruction given to the LLM before any user content that shapes the model's personality, tone, and behavior. Separate from the user prompt. Also called a "system prompt." |
| **Summarization Pipeline** | The 4-step process CareCircle uses to generate care summaries: (1) Aggregate data from multiple DB tables, (2) Format into human-readable context, (3) Send to Gemini with structured output, (4) Parse response and merge with pre-calculated stats. |
| **Neon** | A serverless PostgreSQL provider with a free tier. Supports pgvector out of the box. Used in production. |
| **Upstash** | A serverless Redis provider with a free tier. Used for BullMQ queues in production. |
