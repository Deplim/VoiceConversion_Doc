> voice conversion 프로젝트를 진행하며 작성한 문서들을 모은 리포지토리입니다.

Cotatron 논문 ▶
> [Cotatron: Transcription-Guided Speech Encoder for Any-to-Many Voice Conversion without Parallel Data. 2020](https://arxiv.org/abs/2005.03295) 

## Voice Conversion Architecture
 
![voice_conversion_architecture](https://gitlab.com/hanish3464/voice-conversion/uploads/0ad08d0c271c97f6d0ceac56fbcc3388/voice_conversion_architecture.png)

## Our project
이 프로젝트에서는 한국어 음성변조모델을 만들어 애니메이션 더빙에 활용하는 것을 목표로 한다.

* 사용할 모델 : Cotatron
* 목표 
	* 이 모델은 기존에 영어를 기준으로 개발되었으므로, 본 팀에서 사용할 한국어 데이터 셋에 맞추어 새로 학습시키고 하이퍼 파라미터를 찾는다. (현제 여기까지 진행)
	* 학습 결과 생성된 오디오의 품질을 평가하고, 성능을 프로젝트의 목적에 맞는 수준까지 끌어올린다. 

##  Any-to-many conversion
```
We propose Cotatron, a transcription-guided speech encoder for speaker-
independent linguistic representation. Cotatron is based on the multispeaker
TTS architecture and can be trained with conventional TTS datasets.
- Cotatron paper | Abstract 파트 초입부 -
```
Cotatron 에서는  **학습되지 않은 speaker 에 대해서도 작동할 수 있도록 tts decoder를 활용한 대사 기반 speech encoder 을 제안한다.**

-------
![voice_conversion_architecture2](https://gitlab.com/hanish3464/voice-conversion/uploads/48062a06512dbef141346bd0e8e4f1cf/voice_conversion_architecture2.png)
-------

voice conversion 부분은 이전 기술과 유사하나, 언어(linguistic) 적인 특성을 뽑는 모듈에서 새로운 제안을 한 것이 특징이며, 이 부분을 **cotatron** 이라 부른다. 

### LEARNING

`본 모델에서 학습해야 하는 타깃은 크게 다음과 같이 세가지로 나뉜다.`
* **학습 대상**

	▶ TTS Decoder

	▶ Residual Encoder

	▶ Voice conversion Decoder


## Speech encoder
![voice_conversion_architecture3](https://gitlab.com/hanish3464/voice-conversion/uploads/504b8a0a845c9061baa60dc2303c9db5/voice_conversion_architecture3.PNG)

`이 파트의 speech encoder 는 linguistic features 를 뽑기 위한 cotatron 부분을 지칭하는 것으로, 초기값 mel-spectrogram 을 전처리하는 모듈인 speaker encoder 와 다르다는 것을 주의할 것.`

위의 그림이 speech encoder 에서 speaker encoding 과 text encoding 를 입력으로 받아 언어적인 특징을 추출하는 과정이다. 
**linguistic features L** 이 여기서 뽑인 언어적인 특징인데, 이것은 conversion 모듈의 입력값으로 들어간다. 

이때 target speaker ID 도 함께 입력값으로 들어가는데 이 값에 따라 conversion 네트워크가
입력값을 어떤 사람의 목소리로 변환할지 결정된다.

## TTS Decoder 
`전체 아키텍처에서 tts 부분만을 확장한 그림이다.`

![](https://gitlab.com/hanish3464/voice-conversion/uploads/ef4ab8ce74c7a0eb842932d7f11e9671/taco2tron_4_.png)

tts는 대사 텍스트를 받아 음성을 출력하는 **text-to-speech** 모델을 의미한다. 여기서는 그 중에서도 **tacotron** 이라는 모델이 사용되었다.

이 모델의 구조는 크게 encoder 파트와 decoder 파트로 나뉘는데, cotatron 은 encoder output 을 그대로 사용하지 않고 여기에 **speaker encoding을 concat 하여 사용**한다. cotatron 아키텍처 이미지에서 text encoder 라고 되어 있는 모듈이 tacotron 의 encoder 와 동일한 계산을 하고, tts decoder 모듈이 tacotron 의 decoder 계산을 한다고 생각하면 된다. 

![cotatron2](https://gitlab.com/hanish3464/voice-conversion/uploads/32d272e00ce18c55eb0808787d91162a/cotatron2.png)

decoder 는 기본적으로 출력값 **mel-spectrogram** 을 계산하는데 이 과정에서 attention 계산이 진행된다. attention weight 는 각 디코더에서 각 타임스탭마다 출력값을 계산하는 데 입력 sequence 중 어떤 원소를 얼만큼 참고할지의 비중에 대한 값이다.  cotatron 에서 tts decoder 를 학습시키는 것이 바로 이 **attention weight 값을 이용하여 linguistic features 를 만들기** 위함이다. 

tts decoder 각 time step 에서 나온 attention weight 의 list 가 **alignment** 이며, 이것을 voice conversion module 의 입력값으로 사용한다.

`* tts decoder 에 대해 이해하려면 우선 다음 자료를 먼저 읽을 것을 추천함. `
* [Task 3 : Voice Conversion - RNN 을 이용한 인코더-디코더](https://gitlab.com/hanish3464/voice-conversion/-/issues/25)
* [Task 3 : Voice Conversion - Bahdanau Attention](https://gitlab.com/hanish3464/voice-conversion/-/issues/26)
* [Task 3 : Voice Conversion - Tacotron](https://gitlab.com/hanish3464/voice-conversion/-/issues/27)

### LEARNING
* **Loss** 

![](https://latex.codecogs.com/gif.latex?MSELoss%28melPred%2C%20melTarget%29%20&plus;%20MSELoss%28melPostNet%2C%20melTarget%29)

```
melTarget : source speaker audio 의 mel-spectrogram
melPred : tts decoder 에서 post net 을 통과하기 전 mel-spectrogram
melPostNet : tts decoder 에서 post net 을 통과한 후의 mel-spectrogram
```

* **classifier Loss** 

![](https://latex.codecogs.com/gif.latex?nllLoss%28speackerProb%2C%20speakers%29)

```
speackerProb : speacker classifier 를 통해 예측한 발화자 (softmax 를 거친 vector 형태)
speakers : 발화자 id (임베딩된 상태)
```

* **target Weight**

	▶ Go-Frame : 
	```
	tss decoder 에서는 이전 타입스탭의 결과값을 가져와 입력값으로 사용하는데, 첫 타임스탭
	에서는 그게 안되므로, 시작 frame 을 임의로 만들어내는데 이것이 go-frame.
	
	단순히 한번 초기화되는 것이 하니라 back-propagtion 의 학습 대상이 되어 업데이트가
	지속적으로 이루어진다.
	```	
	▶ Attention Weight 
	```
	tacotron 에서는 bahdanau attention 이 쓰이는데 여기에서 쓰이는 가중치.
	
	여기서 학습 대상이 되는 attention weight 와 계산 결과로써 출력되어 alignment 계산
	에 사용되는 attention weight 를 잘 구분해야 함.
	```
 	▶ Decoder Weight 
 	```
 	tts 의 decorder 부분 rnn 신경망에 들어가는 가중치들을 의미.
 	```

## Alignment
alignment 란 디코더의 특정 time step 이
**인코더의 sequence 중 어느 부분에 attention 을 많이 두는가** 에 대한 정보의 리스트라고 할 수 있다.

`다음은 alignment 를 그래프 상에 표현한 것이다.`
![alignment](https://gitlab.com/hanish3464/voice-conversion/uploads/ed664690553edc51f1940a5b6e512d84/alignment.PNG)

예를 들어서 디코더의 n 번째 타임 스탭에 대하여 attention weight 를 계산했을 때
인코더 output 중에 k 번째 원소에 대한 값이 크다면,
그래프에서 [Decoder time steps = n ,   Encoder states = k ] 인 위치에 색으로 표시가 되는것.

위 그래프의 경우 **weight 가 크면 클수록 밝은색으로 표시하도록** 되어있는 것을 볼 수 있다.

> 다른 자료들에서 alignment 에 대하여 **"음성과 대사를 매칭하여 유사한 부분을 계산"** 
> 한다는 표현이 자주 쓰이는 것이 바로 위의 이유 때문이다. 

따라서 그래프가 오른쪽 위로 끊기지 않고 연속적으로 증가하는 것이 가장 이상적이다.

![alignment2](https://gitlab.com/hanish3464/voice-conversion/uploads/86239843d9bd984194ff3af1d07e93a1/alignment2.PNG)

만약 위 그림과 같이 그래프가 깔끔하지 않고 흐려지는 부분이 있다면, 해당 time step 에서 어느 곳에 attention 을 줄지 명확하게 특정짓지 못하고 있다는 뜻이다. 

## Linguistic Feature L
feature L 은 alignment 와 text encoding 의 행렬곱, 즉

![](https://latex.codecogs.com/gif.latex?L%3Dmatmul%28A%2C%20%5Cunderset%7Btext%7D%7BEncoder%7D%28T%29%29)

이다.

feature L 은 사실상 디코더에서 계산되는 Attention value 와 동일한 값이다.
(Attention value 도 인코더 출력과 어텐션 weight의 가중합이므로)

## Voice Conversion Decoder

![VC_decoder](https://gitlab.com/hanish3464/voice-conversion/uploads/d22d0ed4c3fb5427cbe3e40902bfc659/VC_decoder.PNG)

### LEARNING

* **Loss**

![](https://latex.codecogs.com/gif.latex?MSELoss%28melSS%2C%20melTarget%29)

```
melTarget : source speaker audio 의 mel-spectrogram
melSS : conversion decoder 의 결과값 mel-spectrogram
```

* **Description**
    
    ![voice_conversion_learning](https://gitlab.com/hanish3464/voice-conversion/uploads/d4edc86f5c7f9b779c450ed1d4d65d62/voice_conversion_learning.png)

    **conversion decoder 학습에서 source speaker 와 target speaker 는 동일한 사람이다.**

    특정 speaker 에 대해 학습시키려면 
    > **특정 대사(1)** 에 대하여 **해당 target speaker(2)** 가 녹음한 **음성(3)** 은 다음과 같다

    라는 식으로 데이터가 형성되어있어야 한다. 

    target speaker 가 아닌 다른 speaker 가 같은 대사를 녹음한 것이 있다면 다른 speaker 의 음성 파일을 이용하여 linguistic feature 를 추출하여 사용해도  좋을 것이다. 
    (음성변조라는 목표를 생각하면 직관적으로 봤을때 이렇게 학습시키는게 더 좋을 것이라고 생각됨.)

    하지만 대부분의 경우 하나의 대사에 대하여 한 한명의 화자만 녹음을 하므로 그렇게 하긴는 힘들다. 때문에 특정 source data 에서 feature L 을 추출하여 target speaker 의 목소리로 변환하는 conversion 할 때에학습 레이블로 다시 source data 의 audio 가 들어가게끔 되어 있는 것이다.


## Vocoder
(작성예정)

