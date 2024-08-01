# README.md

## 핵심 내용 정리

### RAG(Retrieval-Augmented Generation) 개념

1. **기본 원리**: 질문에 관련된 정보를 외부 데이터베이스에서 검색하여 LLM의 컨텍스트로 제공합니다.
2. **장점**:
    - 최신 정보 반영 가능
    - 특정 도메인에 대한 전문성 향상
    - 소스 추적 가능성 제공
3. **구현 방식**:
    - 기본 RAG
    - 다중 쿼리 RAG
    - 재귀적 RAG
    - 에이전트 기반 RAG

### 프로젝트 구현 세부사항

1. **데이터 수집 및 전처리**:
    - 사용한 데이터 소스: 기본 RAG, 다중 쿼리 RAG - txt 파일 이용 / 에이전트 RAG - notion 임원진 소개 링크를 url로 구글 서치 이용
    - 전처리 방법: text split을 사용해 chunk size로 정보를 쪼개서 전달
2. **벡터 데이터베이스**:
    - 사용한 벡터 데이터베이스: FAISS
    - 인덱싱 방법:
        1) Embeddings : text data를 vector 공간으로 변환. 유사도에 따라 벡터 공간에서 위치가 달라짐
        2) Vector DB 생성 : FAISS를 시용해 유사한 벡터를 검색할 수 있도록 데이터베이스 생성. 'from_documents' method; pages와 embedding을 사용해 벡터 db 생성
3. **RAG 파이프라인**:
    - 선택한 RAG 방식: 기본 RAG, 다중 쿼리 RAG, 에이전트 RAG
    - 구현 세부사항:
        1) 기본 RAG
           -제공된 txt 파일을 바탕으로 FAISS 벡터 DB 이용
        3) 다중쿼리 RAG
           -텍스트를 분할하여 벡터 DB 구축 후 다중 쿼리 retriever를 불러와 생성하고 검색함
           -유사한 문서를 불러와 LLM 입력으로 사용
        4) 에이전트 RAG
           -google-search를 이용하여 키를 발급받아 구현
           -API ID 생성 시 노션 페이지를 참조하는 URL로 넣어 해당 정보만 식별하도록 했으나 적용이 완벽히 된 것 같지 않음 
4. **프롬프트 엔지니어링**:
    - 설계한 프롬프트 구조:
      1) 시스템 메시지 템플릿: 주어진 자료만을 바탕으로 답할 수 있도록, 추가적인 정보를 제공하지 않도록 명령
      2) 사용자 질문 템플릿을 사용해 프롬프트 구축
    - 최적화 전략: 임의의 정보를 배제하기 위해, 창의성이 발휘되지 않기 위해 정확한 명령 설정 및 temperature를 0으로 설정해 창의성 배제제
5. **평가 및 최적화**:
    - 평가 메트릭: 정확성, 관련성, 일관성 등
    - 최적화 과정: 복잡한 쿼리를 생성하여 테스트 진행

## 향후 개선 방향

- 구글 서치를 이용한 부분에서 해당 url만을 참조하는지 확인할 필요가 있어 보임
- 구글 서치를 이용한 부분에서 정보량에 따라 time elapsed가 발생하는 것 같은데, 해당 사항에 대해 확인이 필요해 보임
- 멀티 쿼리 RAG를 활용한 부분에서 정확한 답변을 생성하는지 더 많은 쿼리로 확인이 필요해 보임

## 결과 및 성능

- 정확성: 모델을 돌리는 횟수에 따라 답변의 정확도가 달라지는 것을 확인함. 답변할 수 있는 정보와 답변할 수 없는 정보를 하나의 쿼리로 전달했을 때 각 범주에 따라 답변하고 있음.
- 관련성: 레퍼런스 파일에 따른 내용만 답변하도록 설정하고, temperature를 0으로 설정하여 답변의 창의성을 최대한으로 줄임. 실제 답변에서는 불필요한 정보까지 제공하지 않았음.
- 일관성: 모델을 돌리는 횟수에 따라 답변이 달라지기도 하지만, 정답을 제공한 후로는 같은 답변을 제공하고 있음.

## 사용 기술 및 도구

- FAISS
  유사성 검색을 활용하는 벡터 DB 도구; 추후 인덱싱 방법을 사용할 수 있다면 적용하면 더 좋은 성능으로 프로젝트 진행할 수 있을 것 같음
- LLM
  Large Language Model: 대량의 텍스트 데이터 학습을 이용한 자연어 처리 모델
- google-searh agent: 구글 맞춤 검색을 사용하는 도구로, 이벤트에 대한 질문에 답해야 할 때 유용하게 사용 가능. 실시간 정보 검색이 가능하여 정확도가 높은 최신의 정보를 반영할 수 있음

## 추가 고려사항 (Optional)

### 실시간 성능

- RAG 시스템의 평균 응답 시간:
- 성능 최적화 전략:

### 확장성

- 다양한 주제 처리 능력:
- 스케일링 전략:

### 설명 가능성

- 정보 선택 이유 설명 방법: 정보의 출처와 참조한 도메인 링크를 제공함. (코드에 구현되어 있지 않음)
- 투명성 향상 전략: 참조 자료의 출처와 더불어 쿼리와 자료의 코사인 유사도를 제공하여 투명성을 향상할 수 있을 것이라고 생각.

### 관련 논문 리뷰

- 논문 리스트
    - "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020)
        - RAG의 기본 개념을 소개한 원본 논문
        - https://arxiv.org/abs/2005.11401
    - "Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection" (2023)
        - RAG 시스템의 자가 개선 방법을 제시
        - https://arxiv.org/abs/2310.11511
    - "REPLUG: Retrieval-Augmented Black-Box Language Models" (2023)
        - 블랙박스 LLM에 RAG를 적용하는 방법 제안
        - https://arxiv.org/abs/2301.12652
    - "Chain-of-Note: Enhancing Robustness in Retrieval-Augmented Language Models" (2023)
        - RAG의 견고성을 높이는 새로운 접근법 제시
        - https://arxiv.org/abs/2311.09210
    - "Benchmarking Large Language Models for News Summarization" (2023)
        - LLM의 할루시네이션 문제를 다루며, RAG의 효과성을 검증
        - https://arxiv.org/abs/2301.13848
    - "RAFT: Adapting Language Model to Domain Specific RAG" (2023)
        - 특정 도메인에 RAG를 적용하는 방법 제안
        - https://arxiv.org/abs/2309.11733
    - "In-Context Retrieval-Augmented Language Models" (2023)
        - RAG를 인-컨텍스트 학습과 결합하는 방법 제시
        - https://arxiv.org/abs/2302.00083
1. [논문 제목](논문 링크)
    - 핵심 아이디어:
    - 본 프로젝트에 적용한 점:
  
2. 기타
   - 이번 프로젝트 때 논문을 읽어 보지는 못했지만, 구글 서치를 활용해 특정 도메인을 참조하는 것에 있어 "RAFT: Adapting Language Model to Domain Specific RAG" (2023) 논문이 도움이 되는지 확인해 볼 필요가 있어 보임.
