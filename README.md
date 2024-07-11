# 한국어 챗봇 모델링

## 개요
> **아이펠 Project & Main Quest** <br/> 
> **프로젝트 기간: 2024.06.21** <br/>

## ⭐️ 프로젝트 소개
- 한국어 챗봇 데이터를 바탕으로 한국어 챗봇을 만들어보는 프로젝트틀 진행했습니다.

- `Transformer for ChatBot From Scratch`
  - 트랜스포머 구조를 학습하고 직접 구현하는 시간을 가졌습니다.
  - [korean-chatbot-Transformer.ipynb](Ko-Chatbots-From-Scratch/korean-chatbot-Transformer.ipynb)
- `GPT for ChatBot From Scratch`
  - 앞서 구현한 Transformer 구조를 수정하여 GPT 구조를 만들었습니다.
  - [korean-chatbot-GPT.ipynb](Ko-Chatbots-From-Scratch/korean-chatbot-GPT.ipynb)

### 데이터셋 출처
- chatbot 데이터 출처는 아래 데이터를 사용했습니다
  ```text
  Youngsook Song.(2018). Chatbot_data_for_Korean v1.0)[Online].
  Available : https://github.com/songys/Chatbot_data (downloaded 2022. June. 29.)
  ```
## 사용용도
- [korean-chatbot-Transformer.ipynb](Ko-Chatbots-From-Scratch/korean-chatbot-Transformer.ipynb)
  - Transformer에 대한 자세한 구조를 코드를 통해 확인해보고 싶다면 이 코드를 통해 학습할 수 있습니다
  - 또한, 이 파일에는 custom model을 저장하는 코드가 추가되어 있습니다
  - learning rate, model 구조를 customize 했을 때 어떻게 모델을 저장하고 불러오는지 참고하기 좋을 것 같습니다
- [korean-chatbot-GPT.ipynb](Ko-Chatbots-From-Scratch/korean-chatbot-GPT.ipynb)
  - 위 transformer notebook을 보고 GPT를 이해하고 싶다면 이 파일을 통해 쉽게 이해할 수 있을 것입니다

----
## 📚 STACKS

### Environment
![Pycharm](https://img.shields.io/badge/PyCharm-000000.svg?&style=for-the-badge&logo=PyCharm&logoColor=white)
![Github](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=GitHub&logoColor=white)

### Development
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![keras](https://img.shields.io/badge/Keras-D00000?style=for-the-badge&logo=Keras&logoColor=white)
![Tensorflow](https://img.shields.io/badge/TensorFlow-FF6F00?style=for-the-badge&logo=tensorflow&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-2C2D72?style=for-the-badge&logo=pandas&logoColor=white)
![Numpy](https://img.shields.io/badge/Numpy-777BB4?style=for-the-badge&logo=numpy&logoColor=white)
![scikit](https://img.shields.io/badge/scikit_learn-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)

## 변경사항 (Transformer 에서 GPT로)
- 자세한 코드는 [korean-chatbot-GPT.ipynb](Ko-Chatbots-From-Scratch/korean-chatbot-GPT.ipynb) 참고

### 전처리 단계
    - Unsupervised Pre-training
        - 모델이 다음 토큰을 예측하도록 하나의 문장 데이터를 학습하는 구조
        - 질문, 답변 문장을 연결하지 않고 학습을 위해 row 방향으로 붙여주었다
        - 답변에서 중복 문장들이 많았기에 총 19436개의 데이터를 학습하였다
        - train data의 토큰 길이를 바탕으로 Pre-training에서의 max length는 40으로 설정해줬다
    - Supervised Fine-tuning
        - 질문과 답변 데이터를 구분자 토큰과 함께 하나의 시퀀스로 결합
        - delimiter token 을 추가
        - Supervised Fine-tuning dataset은 기존 pre-training 방식과 데이터셋이 다르게 들어가야한다
        - [[start] 질문 [delimiter] 답 [end]] 구조이므로 길이도 좀 더 길게 설정해줘야한다
        - train data의 토큰 길이를 바탕으로 finetuning에서의 max length는 70으로 설정해줬다

### 모델 구현 단계
    - GPT는 디코더만으로 구성된 트랜스포머 모델이다
    - Encoder-Decoder Attention 제거 :
        - GPT는 디코더만으로 구성되기 때문에 encoder와 비교하는 encoder-decoder Attention Layer가 필요가 없다.
        - 기존 코드에서 두 번째 서브 레이어를 제거한다
        ---------------------------------------------------------------------------------------
                attention2 = MultiHeadAttention(
                    d_model, num_heads, name="attention_2")(inputs={
                          'query': attention1,
                          'key': enc_outputs,
                          'value': enc_outputs,
                          'mask': padding_mask
                      })

                # 마스크드 멀티 헤드 어텐션의 결과는
                # Dropout과 LayerNormalization이라는 훈련을 돕는 테크닉을 수행
                attention2 = tf.keras.layers.Dropout(rate=dropout)(attention2)
                attention2 = tf.keras.layers.LayerNormalization(
                  epsilon=1e-6)(attention2 + attention1)
       -----------------------------------------------------------------------------------------
    - 인코더 출력 제거 : encoder_outputs 제거
    - 마스킹 방식 : padding mask 제거, look_ahead_mask 만 사용하면됨



회고 및 결론
---
### 회고
<details>
  <summary><b>Transformer 구현 회고</b></summary>
  <div markdown="1">
    <li> 배운 점
      <li>transformer의 구조에 대해 좀 더 명확히 이해할 수 있었다 </li>
      <li>custom 모델 저장하는 방법을 배웠다 </li>
      <li>숫자를 제거하는 전처리 제거만으로도 대답이 확연히 달라지는 것을 볼 수 있었다 </li>
      <li>underfitting 상황을 생각해서 epoch을 높였더니 성능이 향상되었다</li>
    </li>
    <li>아쉬운 점
      <li>프로젝트에서 한글 토큰을 잘 만들지 못해서 아쉬웠다</li>
      <li>토큰화를 잘 하지 못해서 띄어쓰기에 따라서 답변이 달라진다</li>
    </li>
    <li>느낀 점
      <li>어려운 개념이라도 노력하면 이해할 수 있다는 것을 느꼈다</li>
      <li>챗봇도 결국 어떤 데이터를 학습하냐에 따라 대답이 달라진다</li>
    </li>
    <li>어려웠던 점
      <li>transformer의 구조를 이해하는데 어려웠다</li>
      <li>custom 모델 저장하는 데 config 설정하는 것이 어려웠다</li>
    </li>
  </div>
</details>
<details>
  <summary><b>GPT 구현하기 회고</b></summary>
  <div markdown="1">
    <li> 배운 점
      <li>논문을 읽고 이를 바탕으로 GPT 모델을 직접 구현해볼 수 있었다</li>
    </li>
    <li>아쉬운 점
      <li>Unsupervised pre-training 단계에서 좀 더 다양한 문장들을 실험해보면 좋을 것 같았다</li>
      <li>데이터셋이 적어서 pretrain weight를 불러오는게 오히려 성능이 좋지 않았다</li>
    </li>
    <li>느낀 점
      <li>transformer보다도 일반화 성능이 잘 되는 것 같았다</li>
    </li>
    <li>어려웠던 점
      <li>직접 구현하는 과정에서 모델 shape를 맞추는 것이 생각보다 까다로웠다</li>
      <li>기존 코드를 수정하는 과정에서 오류가 많이 났다</li>
    </li>
  </div>
</details>

### 결론
- 직접 Transformer와 GPT를 구현하면서 자세한 구조에 대해서 배울 수 있었습니다
- 이번 경험을 통해 Transformer, GPT 구조에 대해 잘 알 수 있게 되었습니다

## 디렉토리 구조
```bash
├── README.md
├── data : 데이터 셋을 여기에 다운 받아서 넣어줍니다
├── korean-chatbot-Transformer.ipynb
└── korean-chatbot-GPT.ipynb
```

