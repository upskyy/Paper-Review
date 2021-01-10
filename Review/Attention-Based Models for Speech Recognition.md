# Attention-Based Models for Speech Recognition

### Abstract

- 기계 번역과 다르게 음성 인식에서는 input sequence가 길고 noisy가 많다.
- 이러한 문제점들에 robust한 모델을 만들기 위해 location-awareness attention mechanism을 제안한다. 

### General Framework
---
![general attention](https://user-images.githubusercontent.com/54731898/104130582-d94f3e80-53b4-11eb-99fc-c9450c1a5efb.PNG)  
(본 논문에서는 α를 alignment, attention weight, g를 glimpse라고 표현한다.)  
일반적으로 attention mechanism을 이용해서 output sequence를 뽑아내는 과정이다.  
위의 사진에 있는 Attend 함수를 통해 attention weights를 뽑아내고, 
encoder output과 곱해준 후 Generate 함수를 통해 output을 뽑아낸다.

![Attend 함수 내부](https://user-images.githubusercontent.com/54731898/104130630-216e6100-53b5-11eb-9872-8bb685c2a404.PNG)  
위의 사진은 Attend 함수를 더 자세하게 표현한 사진이다.
Score 함수를 통해 score를 구해주고 softmax 함수를 통해 normalizing 해준다.   
Score 함수는 여러 가지가 있는데, 
본 논문에서 새로운 attention mechanism을 제안한다는 것은 새로운 Score 함수를 제안한다는 뜻이다.
### Content-Based Attention
---
![content-based attention](https://user-images.githubusercontent.com/54731898/104130645-2e8b5000-53b5-11eb-9a66-00255d7b0192.PNG)  
본 논문은 실패한 방법부터 설명하고 있는데 그 중 첫 번째 방법이 content-based attention이다.  
이 방법은 decoder output에 weight를 곱해주고, encoder output에 weight를 곱해주고 bias와
모두 더한 후 Hyperbolic tangent에 넣어준다.  
그리고 나온 결과 값에 weight를 곱해주는 것이다.     
이 방법은 이전의 attention weights를 고려해주지 않기 때문에 sequence에서 위치를 고려해주지 못한다.
이러한 문제점을 "similar speech fragments" 라고 한다.
### Location-Based Attention
---
![location-based attention](https://user-images.githubusercontent.com/54731898/104130647-2fbc7d00-53b5-11eb-97b1-dc128ec2096b.PNG)  

두 번째 실패한 방법은 location-based attention이다.  
이전의 attention weight와 decoder output을 고려해줌으로써 연속적인 음소사이의 거리를 예측할 수 있다. 하지만 encoder의 output을 고려해주지 않기 때문에 한계점이 존재한다.

### Hybrid Attention
---
![hybrid attention1](https://user-images.githubusercontent.com/54731898/104130658-4b278800-53b5-11eb-92c2-76b45d9c65d2.PNG)
![hybrid attention2](https://user-images.githubusercontent.com/54731898/104130659-4bc01e80-53b5-11eb-8d38-f519c7eeed7d.PNG)  
위의 두 가지 방법을 섞어준 것이 hybrid attention이다.  
location-aware attention이라고도 불린다.
이전의 attention weight에 convolution을 취해주고 나온 결과값에 weight를 곱한 후
content-based attention식에 있는 Hyperbolic tangent 안에 같이 더해주는 방법이다.

### Three potential issues about Score Normalization
---
첫 번째는 input sequence가 길 때, glimpse는 encoder output으로부터 관련 없는 noisy한 정보들을 받게 된다. softmax함수는 모든 정보들을 확률 값으로 표현하여 합이 1이 되도록 해야되기 때문에 
noisy한 정보들도 일부 확률 값을 가지게 되고, 그 결과 관련있는 정보들에 집중하기가 어려워진다.  
두 번째는 input utterances가 길 때, 시간복잡도가 엄청나다는 것이다. output 길이가 T이고, 
decode할 때 각 시간당 L frames을 고려한다고 하면 시간복잡도가 O(LT)이다.  
세 번째는 softmax함수가 single vector에만 집중하기 때문에 여러 개의 top-scored frames을 고려하기가 어렵다.

### Sharpening
---
![sharpening1](https://user-images.githubusercontent.com/54731898/104130666-5ed2ee80-53b5-11eb-8d26-4f96e089df0a.PNG)  
(β > 1)  
첫 번째 문제점을 해결하기 위해 나온 방법이다.
softmax함수에 inverse temperature β를 적용한 방법이다. 또 다른 방법으로는 top-k frames만 다시 re-normalizing 하는 것이다.  
하지만 sharpening 방법도 시간복잡도가 O(LT)이어서 두 번째 문제점을 해결하지는 못한다.
그래서 windowing 기술을 제안한다. 이전 attention weight의 중간값을 기준으로 미리 정의된 window 사이즈만큼만 고려해주는 것이다. 이 방법을 사용하면 시간복잡도를 O(L + T)로 줄일 수 있다.  
이 windowing 기술은 top-k frames만 고려하는 것과 비슷하다.

### Smoothing
---
![smoothing](https://user-images.githubusercontent.com/54731898/104130671-60041b80-53b5-11eb-8f2c-1ed524cc0390.PNG)  
sharpening 방법이 long utterances에서의 문제는 해결했지만, short utterances에서는 성능이 좋지 않았다. 그래서 모델이 여러 개의 top-scored frames중 선택하는 것이 좋다는 가설을 세웠다.
위의 식처럼 softmax함수 식에 sigmoid를 추가해준 방법이다.

### Results
---
![result](https://user-images.githubusercontent.com/54731898/104130672-61354880-53b5-11eb-8020-5f45ab89d00d.PNG)  
Convolution과 smoothing방법을 모두 사용한 것이 성능이 제일 좋았다.

