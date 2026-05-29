# RAG Job Preparation Application — Technical Case Study

## TABLE OF CONTENTS

1. The Problem
2. Our Solution
3. System Architecture
4. What Was Actually Implemented
5. What Was Planned (Not Yet Done)
6. Evaluation & Testing
7. What I Personally Built
8. Limitations & Future Work
9. Conclusion

---

## 1. THE PROBLEM

### Challenge

Job candidates struggle to objectively assess skill gaps relative to target positions. Traditional keyword-matching approaches miss semantic equivalencies (e.g., "container orchestration" is not equal to "Kubernetes"), leading to generic or inaccurate recommendations.

### Why It Matters

Candidates need actionable, evidence-based feedback on which skills to develop. Current tools rely on surface-level text matching, not semantic understanding of skill requirements and candidate capabilities.

**Example:**
- Keyword tool: "No Python match" (even if resume says "Python 5 years")
- Semantic tool: "Python 5 years found" (with confidence score 0.92)

---

## 2. OUR SOLUTION (Brief)

We built a semantic skill-matching system using Retrieval-Augmented Generation (RAG) with 3-level hierarchical chunking to preserve resume context while enabling flexible retrieval.

### Key Design Decisions:

**3-Level Nested Chunking:**
- L1: Full sections (career narrative, 1000+ tokens)
- L2: Individual jobs/skills (500-700 tokens)
- L3: Specific accomplishments (50-100 tokens)

**Why This Matters:** Retrieves exact evidence (L3) plus contextual job data (L2) plus career overview (L1) — better than single-level chunking.

**Local Vector Indexing:**
- MVP uses ChromaDB
- FAISS considered for larger-scale local vector indexing

---

## 3. SYSTEM ARCHITECTURE

### High-Level Workflow

```
Resume Upload → Parse & Normalize → 3-Level Chunking → 
Embedding Generation → Vector Indexing → Job Requirement Vectorization →
Semantic Similarity Search → Skill Gap Detection → Report Generation
```

### Core Pipeline

**Ingestion Pipeline:**
```
Parse → Normalize → Section Detect → Semantic Chunk → Embed → Index
```

**Retrieval Pipeline:**
```
Vectorize Skill → Search → Rank → Multi-level Retrieval (L3/L2/L1) → 
Threshold → Extract Evidence
```

### Technology Stack
- Frontend: React.js
- Backend: Node.js/Express
- Processing: Python (LangChain)
- Vector DB: ChromaDB (MVP)
- Embeddings: Sentence-Transformers (384-1024 dims)
- Storage: SQLite (metadata)

---

## 4. WHAT WAS ACTUALLY IMPLEMENTED

### Completed Features

- Resume parser (PDF/DOCX/TXT)
- 3-level semantic chunking algorithm
- Embedding generation (sentence-transformers)
- ChromaDB vector indexing
- Multi-level semantic search
- Skill matching with confidence scoring
- Evidence extraction & ranking
- Report generation (JSON/PDF)
- API endpoints (Node.js/Express)
- React frontend (file upload, results display)
- Caching layer (memory-based, 2-5s retrieval)

### MVP Status

- Local deployment tested
- Handles 100-2000 token resumes
- Single-user architecture
- Processing time: 8-12s first run, 2-5s cached (75-80% faster on cache hits)
- Manual testing with 20+ resume samples
- Works with generic job descriptions

### Performance Metrics (Actual)

| Metric | Value | Notes |
|--------|-------|-------|
| Resume Processing (First) | 5-8 seconds | Includes parsing, chunking, embedding |
| Cache Hit Retrieval | 2-5 seconds | 75-80% faster than first run |
| Embedding Dimensions | 384-1024 | Sentence-transformers models |
| Skill Detection Accuracy | ~72-78% | Varies by skill domain |
| Confidence Scoring | 0.0-1.0 | Threshold: > 0.6 (high confidence) |
| Vector Database | ChromaDB (local) | FAISS for larger-scale considered |

---

## 5. WHAT WAS PLANNED (NOT YET DONE)

| Feature | Status | Note |
|---------|--------|------|
| Multi-user authentication | Planned (Week 12+) | Currently single-user only |
| Web scraping for job URLs | Planned (Week 8) | Manual job input only |
| FAISS production scale | Considered for scale | MVP uses ChromaDB locally |
| Redis distributed caching | Planned (Week 13+) | Local memory cache only |
| OCR for scanned resumes | Not implemented | Digital formats only (PDF/DOCX/TXT) |
| Domain-specific fine-tuning | Not implemented | Uses generic pre-trained models |
| A/B testing framework | Not implemented | Manual evaluation only |

---

## 6. EVALUATION & TESTING

### Dataset & Methodology

**Dataset:**
- 20 synthetic resume + job description pairs
- Manually annotated skill matches by domain experts
- Privacy: Tested using synthetic/anonymized resumes and job descriptions

**Testing Process:**
- Manual validation against ground truth labels
- Precision, recall, F1 metrics for skill detection
- Manual review of gap identification accuracy

### Observed Performance (MVP)

- **Skill detection accuracy:** ~72-78% on test set (varies by skill domain)
- **Nested chunking improvement:** 80% relevant context retrieval vs 55% single-level
- **False positive reduction:** Applied confidence thresholding (score > 0.6 = high confidence match)
- **Processing time:** 8-12s first run, 2-5s cached (75-80% faster on cache hits)

### Privacy Note

**All testing was conducted using synthetic and anonymized resumes and job descriptions. No real candidate data was used in development or evaluation.**

---

## 7. WHAT I PERSONALLY BUILT

### Core RAG Pipeline with Nested Chunking and Semantic Extraction

I built the complete Retrieval-Augmented Generation (RAG) system from scratch. This is the foundational technology powering the application. The core work centers on three critical areas:

#### 1. 3-Level Nested Chunking Architecture

Developed the hierarchical chunking algorithm that intelligently divides resumes into structured levels while preserving semantic meaning:

- Level 1 (L1): Full sections with complete career narrative (1000+ tokens) - entire Experience section, complete Skills section, full Education history
- Level 2 (L2): Focused subsections (500-700 tokens) - individual jobs, skill categories, each educational entry separately
- Level 3 (L3): Granular accomplishment chunks (50-100 tokens) - specific achievements, individual skills, project details

The critical innovation: Each chunk level maintains bidirectional references to its parent/child levels. When L3 chunks are retrieved, they automatically connect to L2 context and L1 career narrative. This prevents semantic loss that occurs with single-level chunking.

Implementation details:
- Section boundary detection using headers and patterns
- Smart splitting avoiding mid-sentence or mid-accomplishment breaks
- Metadata attachment (section type, original position, confidence score)
- Token counting using embedding model tokenizer to respect limits
- Hierarchy linking enabling multi-level context retrieval

#### 2. Semantic Extraction and Vector Embedding Pipeline

Built the embedding generation system using sentence-transformers:

- Text preprocessing and normalization (artifact removal, whitespace cleaning)
- Batch embedding generation processing 50-100 chunks simultaneously (2-10x performance improvement vs single processing)
- Content-hash based caching reducing redundant computation (first run 8-12s, cached run 2-5s)
- Cache invalidation logic on resume updates
- Memory-efficient storage of embeddings with associated metadata

The caching system is critical: when candidates analyze their resume multiple times, the cache hit reduces processing from 8-12 seconds to 2-5 seconds (75-80% faster).

#### 3. Multi-Level Semantic Search and Retrieval

Implemented the core semantic retrieval engine that searches across all three hierarchy levels:

- Parallel vector search using cosine similarity against L1, L2, and L3 embeddings simultaneously
- Confidence scoring with intelligent thresholding:
  - Score greater than 0.6: High confidence skill match (present in resume)
  - Score 0.4-0.6: Weak evidence (possible match, needs verification)
  - Score below 0.4: Skill gap identified (not found in resume)
- Result ranking prioritizing most relevant evidence
- Evidence attribution tracking exactly where skills were found (company, job title, dates)
- Deduplication removing redundant results across levels

This multi-level approach returns exact evidence (L3) plus full job context (L2) plus career overview (L1) for each skill match, providing comprehensive context unlike single-level retrieval.

### Resume Processing and Document Extraction

Built the document parsing system handling diverse resume formats:

- PDF extraction with text recovery using PyPDF2 (handles scanned text, embedded fonts)
- DOCX parsing with structure preservation using python-docx
- TXT format handling with intelligent section detection
- Text normalization removing formatting artifacts, extra whitespace, special characters
- Automatic section identification recognizing Experience, Skills, Education, Projects, Certifications
- Metadata attachment to every chunk (section type, confidence, original position)

### Skill Matching and Gap Identification

Developed the analysis engine comparing job requirements against resume content:

- Job requirement vectorization converting written requirements to embeddings
- Similarity-based matching across multi-level chunks (L1, L2, L3)
- Gap identification using similarity score thresholds
- Importance ranking of missing skills based on requirement priority and frequency
- Confidence scoring for every match and gap with reasoning

### Report Generation with Evidence

Built flexible report generation supporting multiple output formats:

- JSON structured reports with complete metadata and confidence scores
- Human-readable gap analysis showing exactly where evidence came from
- Evidence citations including company name, job title, employment dates
- Learning path recommendations ranked by skill importance
- Confidence score visualization for transparency

### Performance Optimization and Caching Strategy

Implemented caching layer reducing redundant computation:

- Content-hash based embedding cache with 24-hour TTL
- Result caching for identical resume-job combinations
- Job description template caching
- Reduced redundant vector similarity calculations
- Batch processing for multiple skill searches

Real impact: Processing time reduced 75-80% on repeated analyses through caching.

---

## 8. LIMITATIONS & FUTURE WORK

### Known Limitations

Accuracy: Current implementation achieves ~72-78% skill detection accuracy on test set; domain/role specificity affects results

Scale: MVP supports single-user only; requires authentication + distributed caching for multi-user deployment

Job Input: Manual entry or direct paste only; automated web scraping not yet implemented

Resume Format: Digital formats only (PDF/DOCX/TXT); scanned PDFs require OCR (not implemented)

Similarity Thresholds: Currently generic (0.4-0.6 range); requires tuning per industry/role for optimal results

Threshold Tuning: Generic thresholds used; manual adjustment per domain needed for production

### Future Improvements

- Implement fine-tuned models trained on job market data for domain-specific accuracy
- Add FAISS indexing for larger-scale local vector storage
- Build multi-user architecture with Redis caching and user authentication
- Develop automated job URL scraping with content extraction
- Add OCR support for scanned resume PDFs
- Implement adaptive threshold tuning based on industry benchmarks

---

## 9. CONCLUSION

The RAG Job Preparation Application demonstrates a viable approach to semantic skill matching using 3-level hierarchical chunking. While MVP accuracy (~72-78%) requires improvement through domain-specific tuning and fine-tuned models, the architecture successfully retrieves contextual evidence for skill gaps better than keyword-based alternatives.

### Technical Achievements
- Semantic retrieval outperforms keyword matching
- 3-level chunking preserves context better than single-level
- Local deployment viable for MVP scale
- Evidence-based skill matching (not generic recommendations)

### Next Steps for Production
1. Multi-user deployment with authentication
2. Automated job scraping
3. Domain-specific model fine-tuning
4. Production-scale vector indexing (FAISS)
5. Extended testing on larger resume datasets
6. Industry-specific threshold tuning



