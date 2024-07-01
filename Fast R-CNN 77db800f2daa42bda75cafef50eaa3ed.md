# Fast R-CNN

## Intro

- Limit of R-CNN
    - RoI (Region of Interest) 마다 CNN연산을 하기 때문에 속도가 느리다.
    - multi-stage pipelines으로써, 모델을 한번에 학습시키지 못한다.
- Fast R-CNN
    - CNN 특징 추출부터 classification, bounding box regression을 모두 하나의 모델에서 학습 시킴.
    - RoI pooling으로 한계점을 극복.

## Process of Fast R-CNN

![[Process] [https://arxiv.org/pdf/1504.08083](https://arxiv.org/pdf/1504.08083)](Fast%20R-CNN%2077db800f2daa42bda75cafef50eaa3ed/Untitled.png)

[Process] [https://arxiv.org/pdf/1504.08083](https://arxiv.org/pdf/1504.08083)

- Process
    - R-CNN에서와 같이 Selective Search를 통해 RoI를 찾는다.
    - 전체 이미지를 CNN에 통과시켜 feature map을 추출한다.
    - Selective Search로 찾았었던 RoI를 feature map크기에 맞춰서 projection시킨다.
    - projection시킨 RoI에 대해 RoI Pooling을 진행하여 고정된 크기의 feature vector를 얻는다.
    - feature vector는 FC layers를 통과한 뒤, 두 갈래로 나뉜다.
    - 하나는 softmax를 통과하여 RoI에 대해 object classification을 한다.
    - 다른 하나는 bounding box regression을 통해 selective search로 찾은 bbox의 위치를 조정한다.
- 핵심적으로 사용된 도구는 RoI pooling.
    - R-CNN에서는,  CNN output이 FC layer의 input으로 들어가므로 CNN input size를 동일하게 맞춰줘야만 했다.
    - 따라서 원래 이미지에서 추출한 RoI를 warp해 동일 size로 조정했다.
    - 그러나, 실제로 **FC layer의 input이 고정이지 CNN input은 고정이 아니다.**
        - 따라서 CNN의 input에는 입력 이미지의 size에 관계없이  들어갈 수 있고, **FC layer의 input으로 들어갈 때만 size를 맞춰주면 된다.**
        - 여기서부터 **Spatial Pyramid Pooling(SPP)**이 제안 된다.

## Spatial Pyramid Pooling(SPP)

![[SPP] [https://arxiv.org/pdf/1406.4729](https://arxiv.org/pdf/1406.4729)](Fast%20R-CNN%2077db800f2daa42bda75cafef50eaa3ed/Untitled%201.png)

[SPP] [https://arxiv.org/pdf/1406.4729](https://arxiv.org/pdf/1406.4729)

- Process
    - 이미지를 CNN에 통과시켜 feature map을 추출한다.
    - 미리 정해진 4x4, 2x2, 1x1 영역의 피라미드로 feature map을 나눠준다. ( 피라미드 한 칸 = bin )
    - bin내에서 max pooling을 적용하여 각 bin마다 하나의 값을 추출한다.
    - 최종적으로 피라미드 크기만큼 max값을 추출하여 3개의 피라미드의 결과를 모두 이어 붙이고 고정된 크기 vector를 생성한다.
    - 생성된 크기 vector는 FC layer의 input으로 들어간다.
- 결론적으로, CNN을 통과한 feature map에서 2천개의 region proposal을 만들고 region proposal마다 SPPNet에 넣어 고정된 크기의 feature vector를 얻어냈다.
    - 이는 이전에 region proposal마다 했던 2000번의 CNN 연산을 1번으로 줄였다.

## **RoI Pooling Layer**

![[RoI Pooling] [https://jamiekang.github.io/2017/05/28/faster-r-cnn/](https://jamiekang.github.io/2017/05/28/faster-r-cnn/)](Fast%20R-CNN%2077db800f2daa42bda75cafef50eaa3ed/Untitled%202.png)

[RoI Pooling] [https://jamiekang.github.io/2017/05/28/faster-r-cnn/](https://jamiekang.github.io/2017/05/28/faster-r-cnn/)

- RoI 영역에 해당하는 부분만 max-pooling을 통해feature map으로 부터 고정된 길이의 저차원 벡터로 축소하는 단계를 의미한다.
    - Fast R-CNN은 1개의 피라미드(7*7)를 적용 시킨 SPP로 구성되어 있다.

![Untitled](Fast%20R-CNN%2077db800f2daa42bda75cafef50eaa3ed/Untitled%203.png)

- Process
    - 미리 설정된 H*W 크기에 대해 grid를 RoI 위에 생성한다.
    - RoI를 grid 크기로 split 시킨 뒤 max pooling 을 적용해 각 grid 칸마다 하나의 값을 추출한다.
    - 고정된 feature vector로 변환됨.
- 이를 통해 CNN 연산을 1번으로 줄인 것이다.
- fast R-CNN은 object detection 모델이기 때문에 위치 정보를 담는 것이 중요하지는 않다.
    - 그러나 RoI Pool 방식은 RoI 가 소수점 좌표를 가지면 반올림 하기 때문에 input image의 위치 정보가 왜곡된다는 특징이 있다.
    - 따라서 classification에서는 문제가 없지만 pixel 별로 detection 하는 경우에는 문제가 있다.       (→ RoI Align)

## Loss Function

![Untitled](Fast%20R-CNN%2077db800f2daa42bda75cafef50eaa3ed/Untitled%204.png)

![Untitled](Fast%20R-CNN%2077db800f2daa42bda75cafef50eaa3ed/Untitled%205.png)

- classification과 localization loss를 합친 function으로 한 번에 학습 시키고 있다.

## Result

![Untitled](Fast%20R-CNN%2077db800f2daa42bda75cafef50eaa3ed/Untitled%206.png)

- VOC 2012 test

## Limitation

- R-CNN과 SPPNet에 비해 빠른 연산 속도와 정확도를 나타낼 수 있었다.
    - 그러나 여전히 region proposal을 selective search로 수행하기 때문에 region proposal 연산이 느리다는 단점이 있다.
        - selective search 알고리즘은 GPU에 부적합하다.
- Preview
    - faster R-CNN
        - Region Proposal Network (RPN)