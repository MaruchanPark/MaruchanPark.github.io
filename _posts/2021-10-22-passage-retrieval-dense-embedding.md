---
layout: post
title: Passage retrieval을 위한 dense embedding의 구현에서 발생한 문제, 해결 과정
date: 2021-10-22 14:32
tags: [nlp, ODQA, retrieval, mrc]
use_math: true
---

Passage retrieval에서 dense embedding은 훈련 가능한 모델로 질문과 문서의 연관성을 계산, 학습하는 방법이다. 하나의 질문에 대해 모델은 정답 문서와 오답 문서 여러 개를 입력 받는데, 구체적인 구현을 이해하기 위해 부스트캠프 강의에서 제공받은 실습코드를 분석해보았다.  

파이토치 nllloss는 출력 log softmax 값과 target index를 입력받는다. 모델에 입력되는 training batch에서는 정답 문서가 0번째에 위치하므로 target index는 모든 batch 성분이 0으로 주어지게 된다.  
  
그런데, 매우 짧은(5회 미만) iteration 만에 loss가 0으로 수렴하는 현상을 확인했다.  
  
모델은 huggingface의 pretrained bert(klue)를 사용했는데, batch 순서를 모델이 인식할 수 있게 입력받는다는 의심이 생겼다.  
임시방편으로 batch 내의 문서 순서를 섞어서 모델에 입력, 정답 문서의 index를 target으로 제공하니 1epoch 이상의 훈련 횟수 이후 loss 수렴, 충분히 훈련하면 loss가 0으로 떨어지는 정상적인 훈련을 확인할 수 있었다.  
  
구체적인 원인을 파악하기 위해서는 batch 구성에 따라 동일한 성분의 출력이 어떻게 바뀌는지 확인하면 좋을 것 같다.  

-----

[^fn-sample_footnote]: Handy! Now click the return link to go back.
