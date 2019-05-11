# Image Segmentation

## 다루게 될 특정 개념들
> 처리 과정에서, 우리는 다음 개념을 중심으로 실제 경험을 쌓아 나갈 것입니다

* **[Functional API](https://keras.io/getting-started/functional-api-guide/)** - 우리는 **Functional API**로 생체 이미지 분할에 사용되는 컨볼루션 네트워크 모델인  U-Net을 구현할 것입니다. 
  * 이 모델은 복수의 입력 / 출력을 요구합니다.
  * 이 모델은 Functional API를 사용해야 합니다.
* Custom Loss Functions and Metrics : 우리는 custom loss function을 크로스 엔트로피와 다이스 로스를 이용하여 구현할 것입니다. 또한, 우리는 `dice coeefficient  `(우리의 loss를 위해 사용 되는것)와 `Mean Intersection Over Union`을 구현할 것입니다. 이것은 우리가 학습 절차를 어떻게 동작하는지 알아가는 것을 도와줄 것입니다.
* 케라스를 이용한 모델 저장 및 불러오기 : 우리는 모델을 디스크에 저장할 것입니다. 우리가 모델을 이용해 추측 / 평가할때, 우리는 디스크에서 모델을 불러 올 것입니다.

## 우리가 따라갈 절차
1. 데이터를 시각화하고, 탐색적 데이터 분석을 실행
2. 데이터 파이프라인 구축 및 전처리
3. 모델 생성
4. 모델 훈련
5. 모델 발전 (최적화)
6. 반복

##### binary cross entropy
두 개의 class 중 하나를 예측하는 경우에 대한 크로세 엔트로피

##### dice loss
유사도를 측정하기 위해 사용

##### IOU (Intersection Over Union)
두 지역에서 겹쳐지는 부분

## tf.data를 이용한 입력 파이프라인 구성
파일명으로 시작할때 부터,우리가 모델을 좋게 구동할 수 있게 줄  견고하고 확장 가능한 데이터 파이프라인을 구성해야할 필요가 있습니다. 

### 우리의 입력 파이프라인이 따라갈 절차
1. 파일 명으로 부터 파일의 byte를 읽어오기  - 이미지와 레이블을 동시에. 우리의 레이블은 각각의 픽셀에 대해서 차인지 배경인지를 (1,0) 으로 나타낸 것임을 상기하라. 
2. byte를 이미지 형식으로 디코드 한다.
3. 이미지 변형 적용: 선택, 입력된 파라미턴에 따라서)
  * `resize` - 우리의 이미지를 표준사이즈로 리사이징 시킨다. (eda나 메모리 제한, 계산에 의해 결정된 것으로)
    * 이것이 선택인 이유는 `U-Net`이 `Fully Convolutional Network` 이기 때문이다. (Fc 레이어가 없기 때문에) 또한, 그럼으로 인해 입력 값 크기에 의존적이지 않다. 그러나, 이미지를 리사이징 하지 않으면,이미지 크기를 함께 가변시킬 수 없으므로 batch size를 1로 설정해야한다
    * 대안으로, mini-batch마다 이미지를 묶고 리사이즈 할 수 있다. 이미지 리사이징을 너무 많이 하는 것은 성능에 안좋은 영향을 준다.
  * `hue_delta` - 임의의 요소로 RGB 이미지의 색조를 조정해라. 이것은 실제 이미지에만 적용된다 (라벨된 이미지가 아니라).`hue_delta` 는 반드시 `[0, 0.5]` 사이어야 한다. 
  * `horizontal_flip` - 50%의 확률로 중심 축을 따라 이미지를 수평으로 뒤집어라. 이 변환은 레이블과 실제 이미지 모두에 적용되어야 한다. 
  * `width_shift_range` 과`height_shift_range`는 이미지를 가로 또는 세로로 무작위로 변환하는 범위 (전체 너비 혹은 높이의 일부)이다. 이 변환은 레이블과 실제 이미지 모두에 적용되어야 한다.
  * `rescale` - 특정 요인에 따라 이미지를 재조정 하라.  예) 1/ 255.
4. 데이터를 섞고, 반복하라 (이를 통해 여러 epoch에 걸쳐 여러번 반복할 수 있음), 효율성을 위해, 데이터를 일괄처리 하고  처리한 것을 미리 가져온다.

이미지를 변환시키는 이유는 data augmentation를 위함이다.

data augmentation : 같은 양의 데이터를 여러 방법으로 변환(수평반전, 수직반전, 색반전 등등)하여 더 많은 데이터 셋을 만들기 위한 방법이다. 이를 통해 다양한 사진들에서도 accuracy를 증가시킬 수 있다.


## Build a Model
우리는 U-Net 모델을 빌드할 것이다. U-Net은 고해상도 세그멘테이션 마스크를 제공하기 위해 잘 현지화 될 수 있으므로 세분화 작업에 특히 좋다. 게다가, U-Net은 작은 데이터센에서도 잘 작동하며 학습 데이터가 이미지 내의 패치 수와 관련하여 overfitting에 상대적으로 강하다. 이는 학습 이미지 자체의 수보다 크다. 원래 모델과 달리, 각 블록에 `batch normarlization`을 추가한다.

#### conv_block
> Convolution을 진행하는 함수, padding = 'same', (3,3) 필터를 사용
Overfitting 방지를 위하 `DropOut`이 아니라 `batch normalization`을  사용하였다

#### encoder_block
> Downsampling을 진행하는 함수, stride=(2,2), poolSize = (2,2)로 maxPooling

#### decoder_block
> Up Sampling을 진행하는 함수. padding='same' (2,2) 필터를 사용하였고, concatenate 함수를 이용하여 downsampling 이전의 벡터 정보로 픽셀의 위치를 복원함.