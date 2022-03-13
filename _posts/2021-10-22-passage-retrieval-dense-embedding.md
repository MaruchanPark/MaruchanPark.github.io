---
layout: post
title: Passage retrieval을 위한 dense embedding의 구현에서 발생한 문제, 해결 과정
excerpt: Dense embedding 모델 훈련에서 발생할 수 있는 문제
date: 2021-10-22 14:32
tags: [nlp, ODQA, retrieval, mrc]
use_math: true
---
## Open domain question answering (ODQA)
ODQA는 문서에서 질문에 대한 답을 찾는 Reader, 질문과 관련있는 문서를 찾는 Retriever로 이루어져 있다.

### Dense Passage Retrieval
- 문서를 찾는 방법 중 하나로, 질문과 문서의 연관성을 계산, 학습하는 방법
- 하나의 질문에 대해 모델은 정답 문서와 오답 문서 여러 개를 입력 받고, 정답 문서와 질문의 유사도를 높이고, 오답 문서와의 유사도는 낮춘다.

구체적인 구현을 이해하기 위해 부스트캠프 강의에서 제공받은 실습코드를 분석해보았고, 이상한 점이 발견되어서 포스팅으로 남긴다.  

  
문서를 입력으로 받는 passage encoder에서 정답, 오답 문서 묶음을 batch로 받을 때, 정답 문서를 항상 0번째에 입력할 경우 3번 이내의 iteration 만에 loss가 0으로 수렴하는 현상이 나타났다.

모델은 huggingface의 pretrained bert(klue)를 사용했는데, batch 순서를 모델이 인식할 수 있게 입력받는다는 의심이 생겼다.  
임시방편으로 batch 내의 문서 순서를 섞어서 모델에 입력, 정답 문서의 index를 target으로 제공하니 1epoch 이상의 훈련 횟수 이후 loss 수렴, 충분히 훈련하면 loss가 0으로 떨어지는 정상적인 훈련을 확인할 수 있었다.  
  
구체적인 원인을 파악하기 위해서는 batch 구성에 따라 동일한 성분의 출력이 어떻게 바뀌는지 확인하면 좋을 것 같다.  

-----

view, transpose 관련 오류는 dense retrieval과 무관. 
Wiki, MRC 등을 이용해서 설명한다. 그림이 필요함.

훈련 질문에 대한 전체 문서내에서 score의 변화를 보여주는 것이 가장 의미있을 듯.
적은 갯수의 질문에 대해서 over fitting이 되며, 질문의 갯수가 많아질수록 어떻게 훈련이 변하는지 보여주는 것.
목요일에 전체 문서 내에서의 순위 확인. (평균?)
훈련 질문의 갯수를 줄이거나 훈련 epoch을 늘리면 순위를 높일 수 있어야 함.
훈련 질문의 갯수는 어떻게 조정하나?


[^fn-sample_footnote]: Handy! Now click the return link to go back.
