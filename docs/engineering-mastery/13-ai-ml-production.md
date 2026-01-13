# Chapter 13: AI/ML in Production

> "Building ML models is 5% of the work. Deploying them is the other 95%."

---

## 🤖 ML System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    ML System in Production                      │
│                                                                 │
│  Data Pipeline:                                                 │
│  ┌─────────┐  ┌───────────┐  ┌────────────┐  ┌──────────────┐  │
│  │ Collect │─►│ Transform │─►│   Store    │─►│   Feature    │  │
│  │  Data   │  │   (ETL)   │  │(Data Lake) │  │    Store     │  │
│  └─────────┘  └───────────┘  └────────────┘  └──────────────┘  │
│                                                                 │
│  Training Pipeline:                                             │
│  ┌─────────┐  ┌───────────┐  ┌────────────┐  ┌──────────────┐  │
│  │Features │─►│   Train   │─►│  Evaluate  │─►│   Register   │  │
│  │         │  │   Model   │  │   Model    │  │    Model     │  │
│  └─────────┘  └───────────┘  └────────────┘  └──────────────┘  │
│                                                                 │
│  Serving Pipeline:                                              │
│  ┌─────────┐  ┌───────────┐  ┌────────────┐  ┌──────────────┐  │
│  │ Request │─►│  Feature  │─►│  Predict   │─►│   Response   │  │
│  │         │  │  Lookup   │  │  (Model)   │  │              │  │
│  └─────────┘  └───────────┘  └────────────┘  └──────────────┘  │
│                                                                 │
│  Monitoring:                                                    │
│  ┌─────────┐  ┌───────────┐  ┌────────────┐                    │
│  │ Metrics │─►│   Drift   │─►│  Retrain   │                    │
│  │         │  │ Detection │  │  Trigger   │                    │
│  └─────────┘  └───────────┘  └────────────┘                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 MLOps

### Model Versioning

```
Model Registry:
┌─────────────────────────────────────────────────────────────┐
│ Model: recommendation_v1                                     │
│ ├── Version 1.0 (staging) - accuracy: 0.85                  │
│ ├── Version 1.1 (production) - accuracy: 0.87               │
│ └── Version 1.2 (development) - accuracy: 0.89              │
│                                                             │
│ Metadata:                                                   │
│ - Training data hash                                        │
│ - Hyperparameters                                           │
│ - Metrics (accuracy, latency, size)                         │
│ - Dependencies (framework versions)                         │
└─────────────────────────────────────────────────────────────┘

Tools: MLflow, Weights & Biases, DVC
```

### Feature Store

```
Purpose: Single source of truth for features

┌─────────────────────────────────────────────────────────────┐
│                     Feature Store                           │
│                                                             │
│  Feature: user_purchase_count_30d                           │
│  ├── Definition: Count of purchases in last 30 days         │
│  ├── Entity: user_id                                        │
│  ├── Type: Integer                                          │
│  └── Updated: Hourly batch                                  │
│                                                             │
│  Online Store (Redis):                                      │
│  - Low latency (<10ms)                                      │
│  - Real-time serving                                        │
│                                                             │
│  Offline Store (Data Lake):                                 │
│  - Historical data                                          │
│  - Training datasets                                        │
└─────────────────────────────────────────────────────────────┘

Tools: Feast, Tecton, AWS Feature Store
```

---

## 🚀 Model Serving

### Serving Patterns

```
1. Batch Prediction:
   - Run periodically (hourly/daily)
   - Pre-compute predictions
   - Store in database
   - Good for: Recommendations, risk scores
   
2. Real-time Prediction:
   - On-demand inference
   - Sub-100ms latency
   - Good for: Search ranking, fraud detection
   
3. Streaming Prediction:
   - Process event streams
   - Near real-time
   - Good for: Anomaly detection, live scoring
```

### Model Serving Infrastructure

```python
# FastAPI model serving
from fastapi import FastAPI
import joblib
import numpy as np

app = FastAPI()
model = joblib.load('model.pkl')

@app.post("/predict")
async def predict(features: dict):
    X = np.array([features['values']])
    prediction = model.predict(X)
    probability = model.predict_proba(X)
    
    return {
        "prediction": int(prediction[0]),
        "probability": float(probability[0].max()),
        "model_version": "1.2.0"
    }

# With model caching and batching
from functools import lru_cache

@lru_cache(maxsize=1000)
def get_cached_prediction(features_tuple):
    return model.predict([list(features_tuple)])
```

### Deployment Options

```
1. Kubernetes + Custom Container
   - Full control
   - Complex setup
   
2. AWS SageMaker
   - Managed endpoints
   - Auto-scaling
   - A/B testing built-in
   
3. Serverless (Lambda + API Gateway)
   - Pay per request
   - Cold start issues
   - Model size limits
   
4. Edge Deployment
   - TensorFlow Lite, ONNX
   - Low latency
   - Privacy (data stays local)
```

---

## 🧠 LLMs in Production

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   LLM Application Stack                      │
│                                                             │
│  Application Layer:                                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Chat UI  │  API  │  Agents  │  Workflows           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Orchestration Layer:                                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  LangChain / LlamaIndex / Custom                    │   │
│  │  - Prompt management                                │   │
│  │  - Chain composition                                │   │
│  │  - Memory management                                │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Retrieval Layer (RAG):                                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Vector DB  │  Embeddings  │  Reranking             │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Model Layer:                                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  OpenAI  │  Claude  │  Llama  │  Custom Fine-tuned  │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### RAG (Retrieval Augmented Generation)

```python
# RAG Pipeline Example
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Pinecone
from langchain.llms import OpenAI
from langchain.chains import RetrievalQA

# 1. Create embeddings and store
embeddings = OpenAIEmbeddings()
vectorstore = Pinecone.from_documents(
    documents, 
    embeddings,
    index_name="my-index"
)

# 2. Create retrieval chain
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(model="gpt-4"),
    chain_type="stuff",
    retriever=vectorstore.as_retriever(k=5),
    return_source_documents=True
)

# 3. Query
result = qa_chain({"query": "What is our refund policy?"})
print(result["result"])
print(result["source_documents"])

# Benefits:
# - Ground responses in your data
# - Reduce hallucinations
# - Keep knowledge up to date
```

### Prompt Engineering

```python
# System prompts
SYSTEM_PROMPT = """You are a helpful customer service agent for CareCircle.
Your goal is to assist caregivers with questions about the platform.

Guidelines:
- Be empathetic and patient
- Provide accurate information from the knowledge base
- If unsure, say "I'll need to check on that"
- Never make up information about medical topics

Context from knowledge base:
{context}
"""

# Few-shot examples
FEW_SHOT_PROMPT = """
User: How do I add a medication?
Assistant: To add a medication, go to the Medications tab and click "+ Add Medication". 
Enter the medication name, dosage, and schedule. You can also add instructions for when to take it.

User: {question}
Assistant:"""

# Chain of thought
COT_PROMPT = """
Question: {question}

Let me think through this step by step:
1. First, I'll identify what the user is asking about
2. Then, I'll find relevant information
3. Finally, I'll provide a clear answer

Thinking:
"""
```

---

## 📊 ML Monitoring

### What to Monitor

```
Model Performance:
□ Accuracy/Precision/Recall over time
□ Prediction distribution
□ Latency (p50, p95, p99)
□ Throughput (predictions/sec)
□ Error rates

Data Quality:
□ Feature drift (distribution changes)
□ Missing values
□ Data freshness
□ Schema changes

Business Metrics:
□ Click-through rate
□ Conversion rate
□ User engagement
□ Revenue impact
```

### Drift Detection

```python
# Detect distribution drift
from scipy import stats

def detect_drift(reference_data, production_data, threshold=0.05):
    """
    Use KS test to detect distribution drift
    """
    statistic, p_value = stats.ks_2samp(reference_data, production_data)
    
    if p_value < threshold:
        return {
            "drift_detected": True,
            "p_value": p_value,
            "statistic": statistic
        }
    return {"drift_detected": False}

# Alert on drift
if detect_drift(training_features, production_features)["drift_detected"]:
    send_alert("Feature drift detected! Consider retraining.")
```

---

## 🛡️ ML Security

### Threats

```
1. Data Poisoning
   - Malicious training data
   - Model learns wrong patterns
   
2. Model Extraction
   - Querying model to recreate it
   - Stealing intellectual property
   
3. Adversarial Attacks
   - Inputs designed to fool model
   - Small perturbations, big impact
   
4. Prompt Injection (LLMs)
   - Malicious prompts override instructions
   - Data exfiltration
```

### Mitigations

```python
# Rate limiting
from slowapi import Limiter
limiter = Limiter(key_func=get_remote_address)

@app.post("/predict")
@limiter.limit("100/minute")
async def predict(request: Request, input: PredictionInput):
    return model.predict(input)

# Input validation
from pydantic import BaseModel, validator

class PredictionInput(BaseModel):
    features: List[float]
    
    @validator('features')
    def validate_features(cls, v):
        if len(v) != 10:
            raise ValueError('Expected 10 features')
        if any(f < -1000 or f > 1000 for f in v):
            raise ValueError('Feature out of range')
        return v

# Output filtering (LLMs)
def filter_response(response: str) -> str:
    # Remove potential PII
    response = re.sub(r'\b[\w.-]+@[\w.-]+\.\w+\b', '[EMAIL]', response)
    # Remove potential secrets
    response = re.sub(r'sk-[a-zA-Z0-9]+', '[API_KEY]', response)
    return response
```

---

## 📖 Further Reading

- "Designing Machine Learning Systems" by Chip Huyen
- "Machine Learning Engineering" by Andriy Burkov
- MLOps.org Community
- Google ML Best Practices

---

**Next:** [Chapter 14: Career Path & Resources →](./14-career-path.md)


