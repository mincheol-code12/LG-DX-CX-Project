# 🐾 [KI:UM] 나를 케어해DOG! — 반려동물 CX 분석 기반 스마트 펫케어 제품 기획

> **LG전자 DX School 3기 핵심 역량 프로젝트 (우수상)**
> 팀 프로젝트 (5인) | 본인 기여: 데이터 수집 ~ 분석 파이프라인 전체 설계·구현, 제품 컨셉 기획 참여

---

## 📌 프로젝트 요약

반려동물 보호자 커뮤니티 리뷰 **3.6만 건**을 NLP 기반으로 분석하여 **고객 유형(Actor) × 행동 주제(Action)**를 도출하고, 감성분석과 Opportunity Score를 통해 **개선 우선순위가 높은 고객 경험**을 데이터로 발견한 프로젝트입니다.

분석 결과를 바탕으로 타겟 페르소나를 정의하고, LG전자의 강점(ThinQ 생태계, H/W 경쟁력)을 활용한 **모듈형 스마트 IoT 펫케어 제품 [KI:UM]**을 기획했습니다.

### 핵심 성과
- 5개 고객 유형 × 3~4개 행동 주제로 구조화된 **Customer Activity Map** 도출
- Opportunity Score 기반 **개선 우선순위 영역 2개(헬스케어형·초보 보호자)** 발견
- 데이터 기반 페르소나 → 제품 컨셉 → 기대 성과지표까지 연결

---

## 🔍 프로젝트 배경

| 구분 | 내용 |
|------|------|
| **시장 기회** | 글로벌 반려동물 시장 2022년 3,200억$ → 2030년 4,930억$ 전망 |
| **고객 트렌드** | 펫팸족 확산, 반려견 1마리당 양육비 연 17.8만원(2025) |
| **LG 강점** | ThinQ 생태계, H/W 경쟁력, 브랜드 신뢰도, 글로벌 유통망 |
| **문제 인식** | 보호자들이 겪는 관리 페인포인트를 정량적으로 파악할 데이터가 부재 |

### 가설 설정
1. 외출 후 **청결관리**에 불편함을 느낄 것이다
2. 반려동물과 외출 시 **외부인, 동물**을 만나는데 불편함이 있을 것이다
3. 반려동물의 **건강관리** 어려움을 느끼는 반려인이 많을 것이다

---

## 📊 분석 파이프라인

```
데이터 수집 → 텍스트 전처리 → 임베딩 → 클러스터링 → LDA 토픽 모델링 → CAM → 제품 기획
  (1)          (2)            (3)        (4)            (5)       (6)      (7)
```

### (1) 데이터 수집
- **타겟 사이트**: 네이버 블로그, 카페, 지식인, 블라인드, 브런치 스토리 등
- **주요 키워드**: 외출, 산책, 여행, 케이지, 건강, 케어, 병원 등 26개
- **수집량**: 팀원 총 데이터 수집10만건 → 전처리 후 **36,245건**

### (2) 텍스트 전처리
```python
# HTML 태그, URL, 이메일, 특수문자, 이모지 제거
# 15자 미만 짧은 리뷰 제거
# Okt 형태소 분석 (명사/형용사/동사 추출, 불용어 제거)
```

### (3) 텍스트 임베딩
- **모델**: Ko-SBERT (`snunlp/KR-SBERT-V40K-klueNLI-augSTS`)
- **출력**: 768차원 벡터 (L2 정규화 적용)

### (4) 클러스터링 — 방법론 선택 과정

여러 클러스터링 방법을 실험한 뒤, 데이터 특성에 맞는 최적 조합을 선택했습니다.

| 방법 | Silhouette | 한계 |
|------|-----------|------|
| AGG (Ward) | -0.015 | 고차원 임베딩에서 유클리드 거리 비적합 |
| PCA + AGG | 0.035~0.04 | 선형 차원축소의 한계 |
| MiniBatch + AGG | 0.06~0.07 | 클래스 불균형 심함 |
| AGG (Average + Cosine) | 0.30~0.32 | 가장 양호하나 불균형 잔존 |
| **UMAP + HDBSCAN** | **0.378~0.53** | **채택** |

**최종 선택: UMAP + HDBSCAN**
- UMAP: PCA와 달리 **비선형 구조를 보존**하여 군집 경계를 명확히 표현
- HDBSCAN: 덴드로그램을 위한 **계층적 구조 + 노이즈 처리** 강점, DBSCAN보다 안정도 높음

```
UMAP 설정: n_components=15, metric='cosine', n_neighbors=50, min_dist=0.0
HDBSCAN 설정: min_cluster_size=1400, min_samples=7, metric='euclidean'
→ 클러스터 5개, 노이즈 37.8% (데이터 보존 우선)
```

### (5) LDA 토픽 모델링
- 클러스터별 Gensim LDA 적용 (Coherence + Perplexity 기반 토픽 수 결정)
- 클러스터(Actor) × 토픽(Action) 매핑으로 **고객 유형별 행동 주제** 정의

| Actor | 소제목 | Action 예시 |
|-------|--------|------------|
| Actor 0 | 헬스케어형 펫 보호자 | 일상 건강관리, 수술/재활, 영양제/보조제 |
| Actor 1 | 초보 펫 보호자 | 예방 중심 건강관리, 외출/여행, 용품 구매, 위생 관리 |
| Actor 2 | 펫과 동반 여행을 즐기는 보호자 | 여행 준비, 배변 관리 |
| Actor 3 | 매일 산책을 원하는 보호자 | 산책 루틴, 반려동물 용품, 사회화 |
| Actor 4 | 펫의 행동 교정을 원하는 보호자 | 반려인 훈련, 행동 교정, 스트레스 관리 |

### (6) Customer Activity Map (CAM)

각 Actor × Action 조합에 대해 두 축을 정량화했습니다.

**Satisfaction (만족도)**
- 감성사전(SentiWord) 기반 리뷰별 감성 점수 산출
- Actor × Action별 **평균** 점수로 계산 (초기 합계 방식에서 몰림 현상 발견 → 평균으로 수정)
- MinMaxScaler로 -10 ~ 10 정규화

**Importance (중요도)**
- Actor × Action별 리뷰 빈도 비율
- MinMaxScaler로 0 ~ 10 정규화

**Opportunity Score**
```
Opportunity = Importance + Max(Importance - Satisfaction, 0)
```
→ 중요도는 높지만 만족도가 낮은 영역 = **개선 기회가 큰 영역**

### (7) 분석 결과 → 비즈니스 제안

**핵심 발견**: Actor 0(헬스케어형)과 Actor 1(초보 보호자)에서 Opportunity Score가 가장 높음

**타겟 페르소나**: 김민정 (31세, 혼자 반려견 관리)
- Pain Point: 건강 정보 부족, 혼자 외출 시 불안, 맞춤형 케어 방법 부재

**솔루션 도출 (관찰 → 고찰 → 통찰)**:
- 건강 상태 추적 예측 시스템
- 실시간 모니터링 시스템
- 청결 관리 시스템

**제품 컨셉**: **[KI:UM] 모듈형 스마트 IoT HOME**
- LG ThinQ 연동, 모듈형 커스터마이징, 건강 데이터 통합 관리
- 기대 효과: 양육비 약 35% 감소, 양육 만족도 13.6% 증가

---

## 📁 프로젝트 구조

```
📦 LG-DX-CX-Project/
├── 📄 README.md
├── 📂 notebooks/
│   ├── 00_crawling.ipynb               # 네이버 카페 Selenium 크롤링
│   ├── 01_preprocessing.ipynb          # 텍스트 전처리 (정제, 광고 제거, 중복 제거)
│   ├── 02_embedding.ipynb              # Ko-SBERT 임베딩
│   ├── 03_clustering_experiments.ipynb  # KMeans, HAC 등 실험 과정
│   ├── 04_umap_hdbscan.ipynb           # UMAP + HDBSCAN 최종 클러스터링
│   ├── 05_lda_topic_modeling.ipynb      # LDA 토픽 모델링
│   ├── 06_cam_analysis.ipynb           # 감성분석 + CAM + Opportunity
│   └── 07_dendrogram.ipynb             # 클러스터 구조 시각화
├── 📂 results/
│   ├── cam_opportunity_chart.png       # Opportunity Area 산점도
│   ├── clustering_comparison.png       # 클러스터링 방법별 성능 비교
│   ├── dendrogram.png                  # 블록 덴드로그램
│   └── lda_visualization/              # pyLDAvis HTML 파일
└── 📂 docs/
    └── presentation_summary.pdf        # 발표자료 요약
```

---

## 🛠 기술 스택

| 구분 | 도구 |
|------|------|
| **언어** | Python |
| **크롤링** | Selenium |
| **전처리** | re, emoji, Pandas |
| **형태소 분석** | Okt (KoNLPy), Kiwi |
| **임베딩** | Sentence-Transformers (Ko-SBERT) |
| **차원축소** | UMAP, PCA |
| **클러스터링** | HDBSCAN, MiniBatchKMeans, AgglomerativeClustering |
| **토픽 모델링** | Gensim LDA, pyLDAvis |
| **감성분석** | SentiWord 감성사전 |
| **시각화** | Matplotlib |
| **데이터 처리** | Pandas, NumPy, Scikit-learn |

---

## 💡 배운 점

- **클러스터링 방법론 선택의 중요성**: 고차원 텍스트 임베딩에 전통적 클러스터링을 적용했을 때의 한계를 직접 경험하고, 데이터 특성에 맞는 비선형 차원축소(UMAP) + 밀도 기반 클러스터링(HDBSCAN)의 필요성을 근거 기반으로 도출
- **정량화 방식의 영향**: CAM 감성 점수를 합계 → 평균으로 바꾸는 것만으로 결과의 해석이 완전히 달라진 경험. 지표 설계가 분석 결론을 좌우한다는 것을 체감
- **데이터 분석 → 비즈니스 제안 연결**: 클러스터링과 토픽 모델링 결과를 Opportunity Score로 정량화하여 "어디를 개선해야 하는가"라는 비즈니스 질문에 답할 수 있었음
