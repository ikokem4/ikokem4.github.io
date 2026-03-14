---
title: "Hugging Face가 뭔지 — AI계의 GitHub라고 불리는 이유"
date: 2026-02-10
draft: false
tags: ["AI", "Hugging Face", "오픈소스"]
categories: ["AI 활용법"]
description: "AI 모델 공유 플랫폼 Hugging Face의 개념과 활용법을 소개합니다."
ShowToc: true
---

## Hugging Face?

이름이 좀 귀엽습니다. 🤗 이모지가 로고인 회사인데, AI 분야에서는 모르면 안 되는 플랫폼입니다.

한마디로 정의하면 **AI 모델을 공유하고 다운받을 수 있는 플랫폼**입니다. GitHub이 코드를 공유하는 곳이라면, Hugging Face는 AI 모델을 공유하는 곳입니다.

## 뭘 할 수 있나

### 1. AI 모델 다운로드
오픈소스 AI 모델을 다운받을 수 있습니다. Llama, Mistral, Stable Diffusion 등 유명한 모델들이 다 여기에 올라와 있습니다.

### 2. 데이터셋 공유
AI 학습에 필요한 데이터셋도 공유됩니다. 이미지, 텍스트, 음성 등 다양한 데이터를 찾을 수 있습니다.

### 3. 데모 체험 (Spaces)
다른 사람이 만든 AI 앱을 웹에서 바로 체험할 수 있습니다. 설치 없이 브라우저에서 AI 모델을 돌려볼 수 있어서 편합니다.

### 4. 모델 학습/배포
자신이 파인튜닝한 모델을 업로드하고 공유할 수 있습니다. API로 배포하는 것도 가능합니다.

## 왜 중요한가

### AI 민주화의 핵심
대기업이 만든 AI 모델을 개인도 쓸 수 있게 해주는 플랫폼입니다. Hugging Face가 없었다면 오픈소스 AI 생태계가 지금만큼 발전하지 못했을 겁니다.

### 사실상의 표준
AI 관련 튜토리얼이나 논문에서 "모델은 Hugging Face에서 다운받으세요"라는 안내가 기본입니다. AI를 다루는 사람이라면 반드시 접하게 되는 플랫폼입니다.

## 모델 카드 읽는 법

각 모델 페이지에는 **모델 카드**라는 설명서가 있습니다.

확인해야 할 항목:
- **모델 크기** — 7B, 13B 등 (B = 10억 파라미터)
- **라이선스** — 상업적 사용 가능 여부
- **지원 언어** — 한국어 지원 여부
- **사용 방법** — 코드 예시
- **벤치마크** — 다른 모델 대비 성능

## transformers 라이브러리

Hugging Face가 만든 Python 라이브러리입니다. 이걸로 모델을 쉽게 불러와서 사용할 수 있습니다.

```python
from transformers import pipeline

generator = pipeline("text-generation", model="모델이름")
result = generator("AI의 미래는")
print(result)
```

몇 줄이면 AI 모델을 실행할 수 있습니다. 이 편의성이 Hugging Face가 인기 있는 이유 중 하나입니다.

## 처음 방문하면 뭘 봐야 하나

1. **Trending Models** — 요즘 인기 있는 모델 확인
2. **Spaces** — 다른 사람이 만든 AI 앱 체험
3. **Datasets** — 어떤 데이터셋이 있는지 구경
4. **관심 분야 검색** — "korean", "image generation" 등으로 검색

## 마치며

Hugging Face는 AI에 관심 있는 사람이라면 꼭 알아야 할 플랫폼입니다. 직접 모델을 돌리지 않더라도, Spaces에서 다양한 AI를 체험해보는 것만으로도 재밌습니다.

한 번 방문해서 둘러보세요. AI 생태계가 얼마나 활발한지 느낄 수 있을 겁니다.
