---
title: Project A: Kurly E-commerce Product Recommendation from a Recipe
categories: [Dev]
comments: true
---



# What
레시피를 입력하면 필요 재료들을 Kurly 상품에서 추천

# How

## 0. Data Preparation
Kurly 의 상품 페이지 일부를 상품 코드, 상세페이지 텍스트 등 세부 데이터를 수집한다.
동적 수집을 위해 BeautifulSoup 이 아닌 Selenuim 을 활용한다.
수집된 데이터의 카테고리 코드와 페이지 까지를 기억해둔다.

```python

```
kurly_crawler.py


## 1. Data Upload to Database
PineconeDB 라는 벡터DB를 Console로 생성한다.
수집한 데이터를 쌓아줄 index명을 지정한다.
문장 embedding 을 위한 모델을 정한다. 한국어로 된 자연어 문장을 우선으로 개발했기 때문에 다음과 같은 모델을 검토.
Pinecone 에서 유사도를 비교하여 제품 추천이 필요하기 때문에 문장 전체를 하나의 벡터로 표현할 필요가 있음.

### 1-1. RAW 데이터 토큰
Index 'kurlyproducts-openai' created or connected successfully
Index connected
Starting data upload
Error during Pinecone operation: Error code: 400 - {'error': {'message': "This model's maximum context length is 8192 tokens, however you requ
ested 10568 tokens (10568 in your prompt; 0 for the completion). Please reduce your prompt; or completion length.", 'type': 'invalid_request_error', 'param': None, 'code': None}}                                                                                                          Response status code: 400
Response content: {
  "error": {
    "message": "This model's maximum context length is 8192 tokens, however you requested 10568 tokens (10568 in your prompt; 0 for the comple
tion). Please reduce your prompt; or completion length.",                                                                                         "type": "invalid_request_error",
    "param": null,
    "code": null
  }
}

- 데이터 토큰 사용하려는 임베딩 text-embedding-ada-002 의 허용 토큰인 8192 보다 높은 상태
- 데이터 정제 필요


### 1-2. why text-embedding-ada-002? Sentence Model 후보
1. klue/roberta-base
BERT와 이에서 파생된 transformer 기반 언어모델에서는 가장 첫 위치에 CLS 라는 문장 공통 토큰을 둔다.
해당 위치의 임베딩 결과를 대표 임베딩으로 사용하게 됨.
CLS 토큰이 self-attention layer 을 거치면 문장 내에서 다른 토큰들을 골고루 반영한 weighted sum 이 대표값으로 나올 것이라 기대
but 최근 기조로 해상도가 높지 않다고 판단하여 drop

2. all-MiniLM-L6-v2
GPT (Generative Pre-trained Transformer) 모델 계열에도 BERT나 RoBERTa와 유사한 용도로 사용할 수 있는 모델들이 있습니다. 하지만 GPT 모델은 구조와 학습 방식에서 약간의 차이가 있음

GPT 모델의 특징:
단방향(left-to-right) 언어 모델
주로 텍스트 생성 작업에 특화
문맥을 이해하고 다음 단어를 예측하는 데 강점

문장 임베딩을 위한 GPT 모델:
GPT-2: OpenAI에서 개발한 모델로, 다양한 크기(small, medium, large, xl)
GPT-3: GPT-2의 더 큰 버전이지만, API를 통해서만 접근 가능
GPT-J: EleutherAI에서 개발한 오픈소스 GPT 모델
KoGPT: SK텔레콤에서 개발한 한국어 특화 GPT 모델

GPT vs BERT/RoBERTa 비교:

GPT는 단방향이고 BERT/RoBERTa는 양방향입니다.
GPT는 다음 단어 예측에 강하고, BERT/RoBERTa는 문맥 이해에 강합니다.
문장 임베딩 목적으로는 BERT/RoBERTa가 더 일반적으로 사용됩니다.

GPT 모델을 문장 임베딩에 사용할 때 주의할 점:
GPT 모델은 주로 생성 작업에 최적화되어 있어, 문장 임베딩 용도로 사용할 때 추가적인 fine-tuning이 필요할 수 있음
문장의 끝부분에 더 가중치를 둘 수 있으므로, 전체 문장의 의미를 균형있게 캡처하기 위해 평균이나 다른 풀링 방법을 신중히 선택해야 함.
결론적으로, 문장 임베딩 목적으로는 BERT/RoBERTa 계열 모델이 더 일반적으로 사용되지만, 특정 상황에 따라 GPT 모델도 효과적으로 사용될 수 있음


3. text-embedding-ada-002

[Sentence Embedding]

1. 문장 임베딩의 원리:
   문장 임베딩은 텍스트의 의미를 숫자 벡터로 표현하는 방법입니다. 이상적으로는 전체 문맥을 고려하여 텍스트의 의미를 잘 포착해야 합니다.
2. 청크 크기와 임베딩 품질의 관계:
   - 큰 청크: 더 넓은 문맥을 고려할 수 있어 전체적인 의미를 잘 포착할 수 있습니다.
   - 작은 청크: 지역적인 정보는 잘 포착할 수 있지만, 전체적인 문맥을 놓칠 수 있습니다.
3. 청크로 나누는 것의 영향:
   - 긍정적 측면: API 제한을 우회하고 매우 긴 텍스트를 처리할 수 있게 합니다.
   - 부정적 측면: 청크 간의 연결성이 끊어져 전체적인 문맥 파악이 어려워질 수 있습니다.
4. 이 접근법의 장단점:
   장점:
   - 긴 텍스트 처리 가능
   - API 제한 준수
   단점:
   - 전체적인 문맥 파악이 어려워질 수 있음
   - 청크 간 중요한 관계가 손실될 수 있음
5. 대안적 접근법:
   a. 요약 후 임베딩: 긴 텍스트를 먼저 요약한 후 임베딩하는 방법
   b. 계층적 임베딩: 문장, 단락, 전체 텍스트 수준의 임베딩을 결합
   c. 특수 목적 모델 사용: 긴 텍스트용으로 설계된 모델 활용
6. 현재 접근법의 개선 방안:
   - 청크 간 중복 허용: 연속성 유지를 위해 청크 간 일부 중복 허용
   - 중요 정보 보존: 제목, 주요 키워드 등을 각 청크에 포함
   - 후처리: 여러 청크의 임베딩을 더 정교한 방식으로 결합 (예: 가중 평균)

결론적으로, 청크로 나누는 방식이 임베딩 품질에 영향을 줄 수 있지만, API 제한 등의 현실적인 제약 하에서는 불가피한 선택일 수 있습니다. 이 방법의 한계를 인지하고, 가능한 한 큰 청크를 사용하거나 위에서 제시한 개선 방안을 적용하여 부정적 영향을 최소화하는 것이 좋습니다.

귀하의 특정 사용 사례와 데이터 특성에 따라 최적의 접근 방식이 달라질 수 있습니다. 예를 들어, 제품 설명 텍스트가 대체로 짧다면 청크로 나누는 것이 큰 문제가 되지 않을 수 있습니다. 반면, 긴 리뷰나 상세한 기술 문서를 다루고 있다면 다른 접근법을 고려해볼 수 있습니다.


```python

```
db_pinecone_uploader.py


### 1-3. Data 정제



## 2. Extract Products from User's Recipe
사용자의 레시피를 입력으로 받아 gpt-4o-mini 모델을 활용해 


## 



