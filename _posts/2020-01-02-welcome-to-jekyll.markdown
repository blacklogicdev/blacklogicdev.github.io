---
layout: post
title:  "[논문리뷰] BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension"
date:   2021-10-06
---

# [논문리뷰]
**BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension**
([논문보기](https://arxiv.org/abs/1910.13461))

<br>

## 1. Introduction

Self-supervised 방식은 광범위한 NLP 과제에서 주목할 만한 성공을 거두었다.

그 중 가장 성공적인 접근은 Denoising Autoencoder.

이는 임의로 마스킹된 텍스트를 재구성하도록 사전학습된 마스킹 언어 모델의 변형이다.

**그러나 기존의 마스킹 사전훈련 모델은** 특정 타입의 과제 (예: span prediction, generation, etc.)에 초점을 두어 적용성이 제한된다.

**반면 BART는** NLG & NLU tasks에 모두 적용 가능하다. 

**BART는** Bidirectional과 Auto-Regressive Transformers를 결합한 사전학습을 위한 모델이다.

<br>

**BART의 사전 학습 단계**는 다음과 같다.
  1) 임의의 Noise Function으로 텍스트를 손상시킨 후 
  2) Sequence-to-Sequence 모델이 학습하여 원본 텍스트를 재구성한다.

<br>

**BERT / GPT / BART는 어떻게 다를까?**

(a) **BERT:** Bidirectional Encoder Representation from Transformers (양방향 인코더 표현)

- 2018년 구글이 공개한 사전훈련 모델
- 양방향으로 인코딩되어 마스킹된 토큰이 독립적으로 예측된다.
- Text Understanding에 사용될 수 있다. Text Generation에는 사용되기 어렵다.

![Untitled](/public/img/bart1.png)

(b) **GPT:** Generative Pre-trained Transformer (생성적 사전훈련 트랜스포머) 

- 2018년 OpenAI가 공개한 사전훈련 모델
- 왼쪽의 컨텍스트 조건만 사용하기 때문에 양방향 상호 작용은 어렵다.
- Text Generation에 사용될 수 있다. Text Understanding은 어렵다.

![Untitled](/public/img/bart2.png)

(c) **BART:** Bidirectional and Auto-Regressive Transformer (양방향 자동회귀 트랜스포머)

- 2019년 Facebook AI가 공개한 사전훈련 모델
- 손상된 문서를 양방향 모델로 encoding하고 원본 문서에 대한 가능성을  자동회귀 decoder로 계산한다.
- 미세 조정을 위해 손상되지 않은 문서가 encoder/decoder에 모두 입력되며 decoder의 최종 hidden state를 사용한다.
- Text Generation에 특히 효과적이지만 Text Understanding에도 적용이 가능하다.

![Untitled](/public/img/bart3.png)

## 2. Model

BART는 (Vaswani et al., 2017)의 Sequence-to-Sequence Transformer 구조를 사용하지만, 

- ReLU 활성화 함수를 GeLUs (Hendrycks & Gimpel, 2016)로 수정하고
- N(0, 0.02)로 매개 변수를 초기화 했다.

Base model에는 6개의 레이어를, 

Large Model에는 12개의 레이어를 각각 encoder와 decoder에 사용했다.

## 3. Fine-tuning BART

BART로 생성된 representations은 다음과 같은 tasks에 적용(Fine-tuning)될 수 있다.

- Sequence Classification
- Token Classification
- Sequence Generation
- Machine Translation

## 4. Comparing Pre-training Objectives

Base model를 사용해 다양한 옵션을 비교해본 결과:

(참고: 모든 모델은 비슷한 크기이고, 책과 위키피디아 데이터로 1M steps를 학습한 결과)

- 과제에 따라 성능이 좋은 모델이 달랐지만,
- Text Infilling이 포함된 BART 모델이 여러 과제에서 일관되게 좋은 성능을 보였다.

![Untitled](/public/img/bart4.png)

## 5. Large-scale Pre-traning Experiments

RoBERTa 모델과 동일한 스케일 (8000 batch size & 500000 steps)로 

BART(w/ Text Infilling + Sentence Shuffling) 모델을 훈련시켜 본 결과는 다음과 같다:

1) **Discriminative Tasks**

- SQuAD와 GLUE에서 RoBERTa 및 XLNet과 유사한 성능
- 즉, 단방향 decoder 계층이 분류 과제의 성능을 저하시키지 않음을 시사
    
    ![Untitled](/public/img/bart5.png)
    

2) **Generation Tasks**

- 두 개의 표준 요약 데이터 세트에 대한 결과
- BART가 두 개의 과제 및 모든 메트릭에서 모두 기존 모델들보다 좋은 성능을 보임
- 특히 XSum(매우 추상적인 뉴스 요약 데이터 세트)에서 더 좋은 성능을 나타냄

![Untitled](/public/img/bart6.png)

- BART는 대화 응답 생성 과제에서도 기존 연구보다 성능이 우수

![Untitled](/public/img/bart7.png)

- 까다로운 ELI5 추상적 질문 응답 데이터 세트에서는 SOTA를 달성
- 그러나 응답이 질문에 의해 weakly specified되기 때문에 여전히 challenging한 과제

![Untitled](/public/img/bart8.png)

3) **Translation**

- BART는 단일 언어 영어 사전학습을 통해 강력한 역번역(Back Translation) 성능을 보임
- 그러나 역번역 데이터가 없으면 과적합(Overfitting)되기 쉬움
- 따라서 정규화 기법에 대한 추가적인 연구가 필요

![Untitled](/public/img/bart9.png)

### 6**. Qualitative Analysis**

4) **Summarization**

- BART는 특히 텍스트 요약에서 강력한 성능을 보임
- 아래 샘플 요약 텍스트는 BART가 자연어의 이해와 생성의 강력한 조합으로 학습되었음을 보여줌

![Untitled](/public/img/bart10.png)

## 8. Conclusion

- BART는 Discriminative 과제에서 RoBERTa와 유사한 성능을 보이는 동시에
- 다양한 Generation 과제에서 SOTA를 달성
- End task에 따라 조정할 수 있도록, document를 손상시키는 새로운 방법에 대한 추가 연구가 필요
