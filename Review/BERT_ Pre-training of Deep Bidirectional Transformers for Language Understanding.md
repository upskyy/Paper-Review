# BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding
https://arxiv.org/pdf/1810.04805.pdf  
![a](https://user-images.githubusercontent.com/54731898/109606223-b0078f00-7b69-11eb-8fea-a0562c4ab7c4.jpg)  


## Abstract
---
- 새로운 언어 모델 BERT(**B**idirectional **E**ncoder **R**epresentations from **T**ransformers)를 소개합니다.
- 트랜스포머의 인코더로 이루어져 있습니다.
- 레이블 되지 않은 데이터로 pre-train을 하고, 레이블 된 데이터로 fine-tuning을 합니다.
- 모든 자연어 처리 분야에서 좋은 성능을 내고 있습니다.  

## Introduction
---
OpenAI GPT 같은 경우는 left-to-right 구조를 사용합니다. 즉, self-attention layers에서 이전 토큰의 정보들을 통해 다음 토큰을 예측합니다. 본 논문에서는 이러한 단방향성이 양방향의 정보들을 통합해야 하는 question answering 같은 태스크에 fine-tuning 어렵게 한다고 말하고 있습니다.  
따라서 masked language model(MLM)을 사용하여 단방향성의 제약을 해결합니다.  
MLM 방법은 input의 일부 토큰을 랜덤으로 마스킹하고 양방향의 토큰 정보들을 통해 마스킹 된 토큰의 id를 예측하는 방법입니다.  
이외에도 next sentence prediction(NSP)방법을 사용합니다.  

BERT의 코드와 사전 학습된 모델은 [여기](https://github.com/google-research/bert)에서 확인하실 수 있습니다.  

## BERT
---
![b](https://user-images.githubusercontent.com/54731898/109619922-a89db100-7b7c-11eb-96f2-b722bfd3d580.PNG)  

Pre-training과 fine-tuning, 2개의 스텝으로 나눌 수 있습니다.  
Pre-training에서는 unlabeled data로 학습을 합니다.  
Fine-tuning에서는 처음에 사전 학습된 모델의 파라미터로 초기화된 후, labeled data로 fine-tuning을 합니다.  

### Model Architecture  
L : layer 개수  
H : 히든 디멘션  
A : 셀프 어텐션 헤드 개수  

||L|H|A|Total Parameters|
|:---:|:---:|:---:|:---:|:---:|
|**BERT(base)**|12|768|12|1.1억|
|**BERT(large)**|24|1024|16|3.4억|  

BERT(base) 같은 경우는 OpenAI GPT와 비교하기 위해 모델 사이즈를 같게 하였습니다.  

### Input/Output Representations  
![c](https://user-images.githubusercontent.com/54731898/109622145-22369e80-7b7f-11eb-8285-12539cbdb732.PNG)  
 
모든 문장의 시작에는 항상 [CLS] 토큰을 사용합니다. 그리고 문장을 구분하기 위해서 두 가지 방법을 사용하는데,   첫 번째 방법은 문장 사이의 [SEP] 토큰을 사용하는 것입니다.
두 번째 방법은 A문장인지 B문장인지를 나타내주는 segment embedding을 추가하는 것 입니다.
따라서 input data는 토큰에 대한 임베딩 값과 세그먼트 임베딩 값과 위치를 알려주는 포지션 임베딩 값을 모두 더해서 표현해줍니다.



## Pre-training BERT
---
![d](https://user-images.githubusercontent.com/54731898/109630846-b9542400-7b88-11eb-8d63-63eda6384e0e.PNG)  

### Task #1: Masked LM
랜덤하게 인풋 토큰의 몇 퍼센트를 마스킹하고, 마스킹 된 토큰을 예측합니다. 마스킹 된 토큰의 마지막 히든 벡터에 소프트맥스를 취하여 예측합니다. 본 논문에서는 15% 마스킹을 진행하였습니다.  

### Task #2: Next Sentence Prediction (NSP)
Question Answering(QA)과 Natural Language Inference(NLI) 같은 태스크들은 두 문장 사이의 관계를 이해하는 것이 중요합니다. 그런데 이러한 것들은 앞서 말씀드린 language modeling만으로는 조금 부족합니다. 그래서 다음 문장을 예측하는 태스크를 사전 학습 합니다.  
예를 들어, 50% 정도는 IsNext로 레이블링 되어 실제 다음 문장이 들어가고 나머지 50% 정도는 NotNext로 레이블링 되어 랜덤한 문장이 들어갑니다.


## Fine-tuning BERT
---
사전 학습된 BERT 모델의 파라미터로 초기화하고, 각 태스크의 구체적인 인풋 데이터와 결과값으로 fine-tuning을 진행합니다.  
Pre-training과 비교했을 때, fine-tuning은 상대적으로 비용과 시간이 더 적게 걸립니다.


## Experiments
---
![e](https://user-images.githubusercontent.com/54731898/109631476-54e59480-7b89-11eb-8a17-7b4d0b7cebca.PNG)  
**G**eneral **L**anguage **U**nderstanding **E**valuation(GLUE) 벤치마크는 다양한 자연어 이해 능력을 평가하기 위해 고안 되었습니다.  
GLUE 테스트 결과, BERT base 모델과 BERT large 모델이 좋은 성능을 보이는 것을 확인 할 수 있었습니다.
특히, OpenAI GPT 모델과 BERT(base) 모델은 어텐션에서 마스킹을 제외하면 모델 구조와 파라미터 수가 거의 비슷하지만 BERT(base) 모델이 더 좋은 성능을 보이는 것을 확인 할 수 있었습니다.  

## Conclusion
---
Unsupervised pre-training은 데이터가 적은 태스크에 많은 도움을 주었습니다.  
본 논문에서 제안한 양방향 구조의 BERT는 많은 NLP 태스크에서 좋은 성능을 보이고 있습니다.  


