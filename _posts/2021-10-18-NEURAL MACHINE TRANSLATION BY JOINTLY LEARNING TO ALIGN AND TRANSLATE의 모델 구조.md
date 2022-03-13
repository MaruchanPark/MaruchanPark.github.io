---
layout: post
title: NEURAL MACHINE TRANSLATION BY JOINTLY LEARNING TO ALIGN AND TRANSLATE의 모델 구조
excerpt: seq2seq의 additive attention 연산 과정
date: 2021-10-18 21:07
tags: [nlp, attention]
use_math: true
---

### 2014년 attention 논문의 모델 구조
(NEURAL MACHINE TRANSLATION BY JOINTLY LEARNING TO ALIGN AND TRANSLATE)

Attention is all you need 논문에 나오는 transformer로 attention을 처음 접하게 되었는데, 다른 논문을 읽다보니 다른 attention 연산을 찾게 되었다.  

transformer와 다른 구조라서 정확한 이해를 위해 정리해봤다!  

![image](https://user-images.githubusercontent.com/48475993/138714997-43768ebb-e214-4872-a636-382b95bba02c.png){: width="80%" height="80%"}

scaled dot-product attention과의 차이점은 Q,K,V 벡터 내적이 FCN으로 대체되었다는 것이다.  
해당 논문에서는 alignment model이라고 표현하는데, 최근에는 additive attention이라고 부르는 것 같다.  


Scaled dot-product attention 대비 연산이 더 복잡한데, 성능이 더 좋을까? 연산량은 얼마나 차이가 날까? 하는 궁금함이 든다.  



-----

[^fn-sample_footnote]: Handy! Now click the return link to go back.
