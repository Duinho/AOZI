# 💻 Developer — 기술 문서화가 완료된 데이터 전처리 가이드라인과 표준화 템플릿을 기반으로, 사내 문서를 인덱싱하고 질문을 받아 검색 및 답변을 생성하는 RAG(Retrieval-Augmented Generation) 시스템의 프로토타입 파이프라인을 구축하고, API 연결 지점을 설계할 것.

💻 Developer: 작업 시작합니다.

# 🚀 AOZI Knowledge Core: RAG 프로토타입 파이프라인 구축 설계 및 API 설계

**수신:** CEO, 관련 에이전트
**발신:** Developer (Lead Engineer)
**일자:** 2024. MM. DD.
**주제:** 사내 문서 기반 검색 증강 생성(RAG) 시스템의 엔드투엔드 프로토타입 파이프라인 설계 및 API 인터페이스 정의.

---

## Ⅰ. 시스템 아키텍처 개요 (RAG Pipeline Architecture)

요구사항을 충족하기 위해, 단순히 텍스트 검색을 넘어 **'메타데이터 필터링'**과 **'유사 사례 검색'**이 가능한 3단계 파이프라인을 설계합니다.

```mermaid
graph LR
    A[Raw Documents (PDF, DOCX, etc.)] --> B{1. Ingestion Pipeline};
    B --> C[Data Preprocessor (Writer Guide)];
    C --> D[Metadata Extractor (Key Fields)];
    D --> E[Chunking & Embedding];
    E --> F[Vector Store (e.g., ChromaDB)];
    
    G[User Query] --> H{2. Retrieval Module};
    H --> I[Query Embedding];
    I --> F;
    F --> J[Relevant Chunks + Metadata];
    
    J --> K{3. Generation Module (LLM)};
    K --> L[Final Answer (Source Cited)];
    
    subgraph API Gateway
        G --> H;
        J --> K;
        L --> M(Client/Frontend);
    end
```

### 핵심 기능 구현 전략

1.  **Source Citation (출처 명시):** 모든 청크(Chunk)는 원본 파일의 `Document ID`, `Page Number`, `Section Title`을 메타데이터로 반드시 포함해야 합니다.
2.  **Similarity Search (유사 사례 검색):** 질문과 함께 **핵심 메타데이터 필터(예: `Project Type = '마케팅'`, `Industry = 'IT'`)**를 전달하여 검색 범위를 좁힙니다.
3.  **Structured Output:** 답변 생성 단계에서, LLM에게 **"결과보고서의 만족도 점수를 언급할 때는 반드시 [Source: Report ID]를 포함하라"**는 System Prompt 지침을 명확히 부여합니다.

## Ⅱ. 기술 스택 및 컴포넌트 정의

| 컴포넌트 | 역할 | 기술 스택 (예시) | 책임 |
| :--- | :--- | :--- | :--- |
| **데이터 전처리** | 비정형 데이터를 구조화된 텍스트/메타데이터로 변환. | Python, `pypdf`, `python-docx`, Regex, OCR Library | 지식 카드 생성 및 표준 용어 매핑. |
| **임베딩/벡터 DB** | 텍스트 청크를 벡터화하고 저장. 유사도 검색 제공. | OpenAI/Cohere API, **ChromaDB / Pinecone** | 임베딩 모델 호출 및 벡터 인덱싱. |
| **오케스트레이션/API** | 전체 파이프라인의 흐름 제어 및 API 엔드포인트 제공. | **Python, FastAPI** | 전체 로직 관리 및 요청/응답 처리. |
| **LLM (Generation)** | 검색된 문맥(Context)을 바탕으로 답변 생성. | OpenAI API (GPT-4), Anthropic API | 답변 생성 및 근거 문서 인용 포맷팅. |

## Ⅲ. 프로토타입 코드 스켈레톤 (Python / FastAPI)

가장 핵심이 되는 **1. 인덱싱 파이프라인 (Ingestion Pipeline)**의 구조와, **2. 검색 API (Query Endpoint)**의 기본 뼈대를 제공합니다.

### 1. 인덱싱 파이프라인 스켈레톤 (`ingest_pipeline.py`)

```python
# ingest_pipeline.py - 문서 로드 및 벡터 저장
from typing import List
# 가상의 라이브러리 (실제 구현 시 LangChain, LlamaIndex 등을 활용)
# from chromadb import Client 

# --- 환경변수 설정 ---
VECTOR_DB_CLIENT = "chroma"
EMBEDDING_MODEL = "text-embedding-ada-002"

def preprocess_document(file_path: str) -> List[dict]:
    """
    [Writer/Researcher 가이드 적용] 파일을 로드하고, 지식 카드 구조에 맞춰 청크화 및 메타데이터 추출.
    :param file_path: 원본 문서 파일 경로
    :return: 청크 및 메타데이터 리스트
    """
    print(f"-> Processing file: {file_path}")
    # 1. OCR/Parsing: 원본 텍스트 추출 (PDF/DOCX)
    raw_text = extract_text_from_file(file_path) 
    
    # 2. Chunking & Metadata Extraction: 문맥 단위로 분할하고 메타데이터 추가
    chunks = []
    # (실제로는 문맥적 분할 로직이 들어갑니다.)
    for i in range(0, len(raw_text), 500):
        chunk_text = raw_text[i:i+500]
        metadata = {
            "document_id": os.path.basename(file_path),
            "page_number": f"Page_{i//500 + 1}",
            "source_type": "Report" if "report" in file_
