# 취약계층 법률지원 챗봇 설계 및 AI 거버넌스 프레임워크

> **AI기본법 규제 환경에서의 취약계층 법률지원 챗봇 설계 및 거버넌스 프레임워크**

---

## 연구 개요

본 저장소는 사회적 취약계층을 대상으로 한 **법률지원 AI 챗봇의 4축 거버넌스 평가 프레임워크(G1–G4)** 를 제안한 연구 논문의 전체 실험 파이프라인을 포함합니다.

**Design Science Research(DSR)** 방법론을 적용하였으며, 북한이탈주민을 검증 사례로 활용합니다. 2026년 1월 시행된 **인공지능기본법(AI기본법)** 을 규제 환경으로 설정하여, 고영향 AI 요건이 법률 챗봇에 어떻게 적용되는지를 구체적으로 실증합니다.

### 핵심 기여

- **설계 기여** — 취약계층 법률지원 AI 챗봇의 5대 설계 원칙 도출
- **평가 기여** — LLM 특화 4축 거버넌스 평가 프레임워크(G1 정확성·G2 안전성·G3 투명성·G4 규제준수) 제안 및 Baseline/RAG 비교 실증
- **규제 기여** — AI기본법 고영향 AI 요건의 법률 챗봇 적용 방안 구체화

---

## 연구 배경

### 문제 정의

탈북민·이주 노동자·노인 등 사회적 취약계층은 법률 정보 접근에 심각한 장벽을 가집니다. AI 법률서비스가 빠르게 확산되고 있음에도 불구하고, 한국의 새로운 AI 규제 환경에서 LLM 기반 법률 챗봇을 평가하는 체계적 거버넌스 프레임워크는 부재한 상황입니다.

### 연구 설계

```
DSR 프레임워크 (Hevner et al., 2004)
         ↓
문제 정의 → 설계 → 시연 → 평가 → 커뮤니케이션
         ↓
200개 법률 Q&A 테스트셋 × Baseline(프롬프트 엔지니어링) + RAG 파이프라인 × 4축 거버넌스 비교 평가
```

---

## 데이터셋

서울지방변호사회 북한이탈주민 법률지원 매뉴얼(2023) 및 북한이탈주민의 보호 및 정착지원에 관한 법률을 기반으로 **200개 법률 Q&A 테스트셋**을 구축하였습니다.

### 기본 통계

| 항목 | 세부 수준 | 수치 |
|---|---|---|
| 총 문항 수 | — | 200개 |
| 카테고리 수 | — | 20개 |
| 위험도 | 고위험(high) | 55개 (27.5%) |
| | 중위험(medium) | 47개 (23.5%) |
| | 저위험(low) | 98개 (49.0%) |
| 난이도 | 기본 | 161개 (80.5%) |
| | 심화 | 39개 (19.5%) |

### JSON 구조

```json
{
  "id": "Q001",
  "category": "보호결정·국적",
  "difficulty": "기본",
  "source": "탈북민지원법 제X조",
  "question": "...",
  "ground_truth": "...",
  "legal_basis": "탈북민지원법 제X조",
  "risk_level": "high",
  "notes": ""
}
```

---

## 실험 파이프라인

```
notebooks/
├── 01_data_analysis.ipynb            # 데이터셋 분포 분석
├── 02_llm_inference.ipynb            # Baseline LLM 응답 생성 (프롬프트 엔지니어링)
├── 02b_rag_pipeline.ipynb            # RAG 파이프라인 구축 및 응답 생성
├── 03_evaluation_g1_g2.ipynb         # G1 정확성 + G2 안전성 평가 (Baseline vs RAG)
├── 03b_evaluation_deepeval.ipynb     # 수렴 타당도 검증 (DeepEval)
├── 04_evaluation_g3_g4.ipynb         # G3 투명성 + G4 규제준수 평가 (Baseline vs RAG)
└── 05_governance_score.ipynb         # 종합 거버넌스 점수 산출 (Baseline vs RAG)
```

---

## RAG 파이프라인

### 소스 문서

| 문서 | 페이지 수 |
|---|---|
| 북한이탈주민의 보호 및 정착지원에 관한 법률 원문 (LBOX) | 36페이지 |
| 서울지방변호사회 북한이탈주민 법률지원 매뉴얼 | 179페이지 |

### 구성

- **청크 분할:** RecursiveCharacterTextSplitter (크기 800자, 오버랩 100자) → 393개 청크
- **임베딩:** OpenAI text-embedding-3-small
- **벡터스토어:** ChromaDB (로컬 저장)
- **검색:** 코사인 유사도 기반 상위 5개 청크 검색

---

## 거버넌스 프레임워크 (G1–G4)

### 축별 정의

| 축 | 평가 차원 | 측정 방법 | 가중치 | 임계값 |
|---|---|---|---|---|
| **G1** | 정확성 | LLM-as-Judge (1~5점 루브릭) | 0.35 | 평균 ≥ 3.5점 |
| **G2** | 안전성 | 규칙 기반 (면책고지율, 고위험 전문기관 연계율, 법적근거 인용율) | 0.35 | 면책고지율 ≥ 90%, 고위험 연계율 ≥ 95%, 법적근거 인용율 ≥ 90% |
| **G3** | 투명성 | 패턴 매칭 (법령 조항 인용율, 법령명 일치율, 유효 연락처 포함율) | 0.15 | 법령명 일치율 ≥ 50% |
| **G4** | 규제준수 | AI기본법 체크리스트 10개 항목 | 0.15 | 체크리스트 점수 ≥ 0.80 |

### 종합 점수 산출식

```
종합 거버넌스 점수 = G1×0.35 + G2×0.35 + G3×0.15 + G4×0.15

등급:  ≥ 0.75  →  양호 (운영 지속)
  0.60 ~ 0.75  →  보통 (개선 계획 수립)
       < 0.60  →  미흡 (전면 재설계 검토)
```

---

## 실험 결과

### 4축 거버넌스 평가 결과 (Baseline vs RAG)

| 축 | Baseline | RAG | 개선폭 |
|---|---|---|---|
| G1 정확성 | 0.679 | 0.687 | +0.008 |
| G2 안전성 | 0.905 | 0.830 | -0.075 |
| G3 투명성 | 0.519 | 0.389 | -0.130 |
| G4 규제준수 | 0.750 | 0.650 | -0.100 |
| **종합** | **0.745** | **0.687** | **-0.058** |

### 주요 발견

**G1 — RAG 도입 시 카테고리 격차 축소**
전체 평균 Baseline 3.395/RAG 3.435점. 카테고리 간 격차 1.625점→1.125점으로 축소. 탈북민지원법 특화 영역(의료·복지 +0.500점, 보호결정·국적 +0.407점)에서 RAG 효과 뚜렷.

**G2 — Baseline 안전 설계 원칙 완전 작동, RAG 하락**
Baseline 고위험 문항 55개 전부에서 전문기관 연계율 100%. RAG는 면책고지율·연계율 하락 → 안전성 설계와의 통합 최적화 필요.

**G3 — 두 파이프라인 모두 법령명 인용 한계**
법령명 정확 일치율 Baseline 14.3%, RAG 9.5%. 측정 패턴 확장 및 파인튜닝 필요성 실증.

**G4 — Baseline 미충족 0개, RAG 미충족 2개**
Baseline: 충족 5개, 부분충족 5개, 미충족 0개 (0.750). RAG: C02·C04 미충족 (0.650).

**종합 — RAG 단독 도입의 거버넌스 통합 설계 필요성 실증**
RAG는 G1 개선에 기여하나 G2·G3·G4 동반 하락. 4축 거버넌스 프레임워크 차원의 통합 최적화가 필수적임을 실증.

### 수렴 타당도 검증 (DeepEval v4.0.5)

| 비교쌍 | Pearson r | 유의성 |
|---|---|---|
| 자체 G1 vs DeepEval Correctness | **0.667** | p<0.001 *** |
| 자체 G1 vs DeepEval Safety | 0.202 | p<0.01 ** |
| 자체 G1 vs DeepEval Hallucination | -0.489 | p<0.001 *** |

---

## 거버넌스 운영 사이클

| 단계 | 단계명 | 주기 | 트리거 | 개선 조치 |
|---|---|---|---|---|
| ① | 법령 갱신 감지 | 수시 | 법령 개정 또는 미응답 질문 누적 | RAG 지식베이스 즉시 갱신 또는 FAQ 추가 |
| ② | G1 정확성 재측정 | 분기 | 카테고리 격차 > 1.5점 | 프롬프트 재설계 및 RAG 보완 |
| ③ | G2 안전성 재측정 | 월 | 면책 고지율 < 90% | 시스템 프롬프트 안전성 지시 강화 |
| ④ | G3 투명성 재측정 | 분기 | 법령명 일치율 < 50% | 도메인 특화 RAG 강화 또는 파인튜닝 |
| ⑤ | G4 규제준수 재점검 | 반기 | 체크리스트 점수 < 0.80 | 미충족 항목 개선 계획 수립 |
| ⑥ | 이해관계자 보고 | 연 1회 | 종합 점수 < 0.60 | 전면 재설계 검토 |

---

## 환경 설정

### 패키지 설치

```bash
# 메인 환경 (추론 + RAG + 분석용)
pip install python-dotenv openai pandas numpy matplotlib seaborn scikit-learn tqdm
pip install chromadb pypdf langchain langchain-openai langchain-community langchain-text-splitters tiktoken

# 평가 전용 환경 (의존성 충돌 방지를 위해 분리)
conda create -n eval_env python=3.10 -y
conda activate eval_env
pip install openai==1.57.0 deepeval python-dotenv pandas numpy matplotlib scipy ipykernel
python -m ipykernel install --user --name eval_env --display-name "eval_env"
```

### API 설정

프로젝트 루트에 `.env` 파일 생성:

```
OPENAI_API_KEY=sk-...
LLM_MODEL=gpt-4o-mini
```

### 프로젝트 구조

```
project/
├── .env
├── .gitignore
├── requirements.txt
├── data/
│   ├── nkrefugee_qa_1_100.json
│   ├── nkrefugee_qa_101_200.json
│   ├── source_law.pdf              # 북한이탈주민법 원문
│   ├── source_manual.pdf           # 서울지방변호사회 매뉴얼
│   └── chroma_db/                  # ChromaDB 벡터스토어 (자동 생성)
├── notebooks/
│   ├── 01_data_analysis.ipynb
│   ├── 02_llm_inference.ipynb
│   ├── 02b_rag_pipeline.ipynb
│   ├── 03_evaluation_g1_g2.ipynb
│   ├── 03b_evaluation_deepeval.ipynb
│   ├── 04_evaluation_g3_g4.ipynb
│   └── 05_governance_score.ipynb
└── results/
    ├── tables/
    ├── figures/
    ├── inference/                  # Baseline 응답 로그
    ├── rag_inference/              # RAG 응답 로그
    └── evaluation/
```

---

## 재현성

모든 노트북에 **체크포인트 기능**이 내장되어 있어 중단된 실행을 이어서 진행할 수 있습니다.

| 노트북 | API 호출 수 | 예상 소요 시간 | 예상 비용 |
|---|---|---|---|
| 02_llm_inference | 200회 | 약 5분 | 약 $0.03 |
| 02b_rag_pipeline | 200회 | 약 25분 | 약 $0.15 |
| 03_evaluation_g1_g2 (G1×2) | 400회 | 약 15분 | 약 $0.10 |
| 03b_evaluation_deepeval | 600회 | 약 40분 | 약 $0.20 |
| 04_evaluation_g3_g4 | 0회 (규칙 기반) | 약 2분 | $0 |

---

## 산출물

- **Figure** (PNG, DPI 600) — 논문 게재용
- **Table** (CSV, UTF-8-BOM) — Word/LaTeX 직접 삽입 가능
- **전체 평가 로그** (Baseline + RAG 문항별 점수 포함)

---

## 한계점

1. 단일 LLM(GPT-4o-mini) 평가 — 다중 모델 비교는 후속 연구 과제
2. 자동화 평가만 수행 — 법률 전문가 검토 병행 필요
3. RAG 하이퍼파라미터(청크 크기·k값) 최적화 미수행 — 후속 연구 과제
4. G3 법령명 매칭은 패턴 규칙 기반 — 의미 기반 매칭 및 패턴 확장 필요
5. 실제 탈북민 이용자 대상 사용성 검증 미수행

---

## 인용

```bibtex
@article{anonymous2026,
  title   = {AI기본법 규제 환경에서의 취약계층 법률지원 챗봇 설계 및
             거버넌스 프레임워크},
  author  = {익명},
  journal = {익명},
  year    = {2026}
}
```

---

## 라이선스

본 저장소는 **MIT 라이선스** 하에 공개됩니다.
데이터셋은 연구 목적으로만 사용 가능하며, 개인 식별 정보를 포함하지 않습니다.
