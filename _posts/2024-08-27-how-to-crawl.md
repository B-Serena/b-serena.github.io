---
title: Project A: How to Crawl Data from a Website
categories: [Dev]
comments: true
---


# How to Crawl Data from a Website


## 1. 기초 

### 1-1. 데이터 수집 기술

- 내부 데이터

- 외부 데이터


### 1-2. 웹 크롤링 vs 웹 스크래핑

- 웹 크롤링:
- 웹 스크래핑: 


웹 클라이언트 - HTTP Request - 웹 서버 - HTTP Response - 웹 클라이언트
* HTTP: Hyper Text Transfer Protocol

### 1-3. Static (정적) vs Dynamic (동적) 크롤링


### 1-4. HTML 태그(Tag), 요소 (Element)
- DOM 구조

### 1-5. CSS Selector
Tip: 내가 필요한 가장 하위 항목 찾기: 마우스 올리고 그 영역 우클릭 -> Copy -> Copy selector 복사 후 가장 하위에서 class 등 명시해줘서 찾기
Tag 가 가지고 있는 selector을 한번에 찾아주는 것

- Tag Selector
- Class Selector
- ID Selector



## 2. BeautifulSoup 활용

### BeautifulSoup 

select, find, find_all
- select: 배열
- find: 
- find_all: 


## 3. Selenium 활용

### Selenium
- 웹 앱 테스트 도구로 개발됨
- 크롬 웹드라이버를 실행하여 모든 동작을 직접 제어 가능
- 동적 웹페이지가 실시간으로 변ㄴ동하는 내용을 중간단계에 저장 가능

find_element, find_elements

보통 액션 (클릭 등) 필요할 때는 Selenium 사용하고
BeautifulSoup 은 데이터 수집 시 사용이 편하다는 의견이 있음


- 페이지네이션(Pagination)


## 4. LangChain과 OpenAI API 활용

- LangChain: 대화형 모델을 쉽게 사용할 수 있는 파이썬 프레임워크
사용자의 질문을 입력으로 받아서 그 질문에 답변을 해줌
LLM 앱을 만드는 대표적인 프레임워크

- WebBaseLoader: 웹사이트에서 데이터를 로드하는 데 사용
텍스트만 수집하기에 손쉬운 편리함


### 4-1. Zero-shot 방식
별도의 예시를 보여주지 않고 LLM의 언어능력을 활용하여 답변을 바로 얻어냄
예시: 뉴스 키워드 추출

```python
# LLM 활용 - 기자 이름, 이메일 주소 추출
from langchain.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

# prompt
prompt_template = """다음 뉴스 본문에서 기자 이름, 이메일 주소를 추출합니다.

<뉴스 본문>
{text}
</뉴스 본문>

기자 이름:
이메일 주소:
"""

prompt = PromptTemplate.from_template(prompt_template)

# LLM
llm = ChatOpenAI(temperature=0, model_name="gpt-3.5-turbo-0125",
                 api_key=OPENAI_API_KEY)

# output parser
output_parser = StrOutputParser()

# Chain
llm_chain = prompt | llm | output_parser

response = llm_chain.invoke({"text": text})

```


### 4-2. HuggingFace 에서 지원하는 모델 사용 가능
- pipeline 지원됨
```python
from transformers import pipeline
sentiment_pipeline = pipeline(model="dudcjs2779/sentiment-analysis-with-klue-bert-base")
data = ["너무 좋아요", "조금 아쉬웠어요", "좋은지 나쁜지 모르겠어요", "최악이에요", "최고에요"]
sentiment_pipeline(data)
```

### 4-3. Few-shot 방식

예시: LangChain 활용한 LLM 감정 분석

```python
from langchain_community.document_loaders import DataFrameLoader

# DataFrameLoader: 데이터프레임을 로드하는 객체
loader = DataFrameLoader(all_etfs, page_content_column="summary")

print(docs[0].page_content)
print(docs[0].metadata)

```

page_content_column : LLM 분석
나머지 metadata : 참조값으로 활용 가능

- 임베딩 벡터 스토어에 저장
임베딩: 자연어(문장, 단어 등)을 벡터로 변환
벡터 스토어: 벡터 검색 가능

- 검색 쿼리 생성
```python
from langchain.chains.query_constructor.base import get_query_constructor_prompt

# 검색 쿼리 생성기
prompt = get_query_constructor_prompt(
    document_content_description,
    metadata_field_info,
)

output_parser = StructuredQueryOutputParser.from_components()
query_constructor = prompt | llm | output_parser

query_constructor.invoke({"query": "Find a fund that invests in equities of clean energy production companies."})

```

예시: 자연어 input을 받아서 LLM을 활용하여 DB에 저장된 summary 컨텐츠와 metadata를 이용해서 적절한 ETF를 추천