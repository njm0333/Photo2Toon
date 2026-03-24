# Photo2Toon 

OpenCV를 활용하여 일반 사진을 만화(Cartoon) 스타일로 변환하는 이미지 프로세싱 프로그램입니다. 

## 1. 주요 기능 (Features)
컴퓨터 비전 기술을 이용하여 주어진 이미지를 애니메이션 스틸컷과 같은 만화 스타일로 변환합니다.
* **Edge Detection**: `cv2.adaptiveThreshold`와 `cv2.medianBlur`를 사용하여 피사체의 뚜렷한 윤곽선을 추출합니다.
* **Color Simplification**: `cv2.bilateralFilter`를 적용합니다. 이 필터는 비선형적이며 엣지를 보존하는 특성이 있어, 경계선은 살리면서도 질감과 색상을 단순화하여 물감으로 칠한 듯한 효과를 냅니다.
* **Image Merging**: 추출된 윤곽선 마스크와 단순화된 컬러 이미지를 병합(`cv2.bitwise_and`)하여 최종 만화 렌더링을 완성합니다.

---

## 2. 알고리즘 데모 및 결과 분석 

### 2-1. 성공 데모 
경계가 뚜렷하고, 색상 대비가 명확하며, 조명이 밝은 이미지에서는 알고리즘이 정상적으로 작동합니다

| Original Image | Cartoon Rendered Image |
| :---: | :---: |
| ![image2](https://github.com/user-attachments/assets/d713bd5a-eed3-40e2-84ea-eba9e822cf38) |![cartoon_result2](https://github.com/user-attachments/assets/70e9f893-659b-417c-961f-1f985f4338d4)|
|     ![image](https://github.com/user-attachments/assets/38f35fc2-aaac-4922-bb69-ce8da8d4e698) |   ![cartoon_result1](https://github.com/user-attachments/assets/3e715425-a5d8-4199-83de-a9eef5901dea)



* **결과 분석**: 피사체의 외곽선이 깔끔하게 추출되었으며, Bilateral Filter가 내부 색상을 부드럽게 밀어주어 만화적인 특징이 잘 살아났습니다.

### 2-2. 실패 데모 
잔가지가 많은 나무, 복잡한 털을 가진 동물, 혹은 저조도 노이즈가 많은 사진에서는 변환 품질이 떨어집니다.

| Original Image | Cartoon Rendered Image |
| :---: | :---: |
| <img width="760" height="922" alt="fail" src="https://github.com/user-attachments/assets/7d3afcfd-37df-4d96-be9c-54c7db54cb5d" />| ![failed](https://github.com/user-attachments/assets/53173349-229f-4e8b-a1e0-142f0df84cc5) |

* **결과 분석**: 화면 전체에 검은색 선이 낀 것처럼 지저분하게 변환되며, 피사체의 형태를 알아보기 힘들어집니다. 
* **원인**: 윤곽선 추출에 사용된 Adaptive Thresholding은 local image region을 기준으로 임계값을 조정합니다. 따라서 복잡한 텍스처나 미세한 명암 변화가 있는 곳에서 불필요한 노이즈까지 모두 Edge로 과도하게 검출해버리는 현상이 발생합니다.

---

## 3. 알고리즘의 한계점 

1. **Adaptive Thresholding 방식의 한계**:
   코드 내 `cv2.adaptiveThreshold(gray, 255, ..., 9, 9)`는 주변 9x9 픽셀 블록의 평균을 기준으로 윤곽선을 추출합니다. Global Thresholding보다 조명 변화에는 강하지만, 국소적인 텍스처나 미세한 노이즈마저 모두 엣지(Edge)로 과도하게 검출해버리는 문제가 발생합니다. 이로 인해 복잡한 이미지에서는 화면 전체에 검은색 선이 지저분하게 깔립니다.
2. **전문적인 엣지 검출 알고리즘의 부재**:
   만화의 깔끔한 스케치 라인을 따기 위해서는 Canny나 Sobel과 같은 방향성 기반의 엣지 검출이 유리합니다. 하지만 현재 코드는 단순히 이미지를 이진화(Binarization)하는 방식에 의존하고 있어, 진짜 물체의 경계와 텍스처 노이즈를 구분하지 못합니다.
3. **과도한 스무딩 파라미터와 연산 병목**:
   색상 단순화를 위해 `cv2.bilateralFilter(img, 9, 300, 300)`가 적용되었습니다. Bilateral Filter는 비선형적이며 엣지를 보존하는 좋은 필터이지만  `sigmaColor`와 `sigmaSpace`가 300이라는 매우 큰 값으로 설정되어 있어 고해상도 이미지 처리 시 연산 속도가 느려집니다.

---

## 4. 향후 개선 방향
위와 같은 한계점들을 극복하고 렌더링 품질과 성능을 향상시키기 위해 다음과 같은 컴퓨터 비전 기법의 도입을 고려할 수 있습니다.

1. **Canny Edge Detector의 도입**:
   현재의 `cv2.adaptiveThreshold` 대신 다단계 엣지 검출기인 Canny 알고리즘(`cv2.Canny`)을 사용할 수 있습니다. 가우시안 스무딩을 통한 노이즈 제거와 이력 임계값(Hysteresis Thresholding) 적용 과정을 거치면 불필요한 잔선을 억제하고 훨씬 정확하고 부드러운 만화 외곽선을 얻을 수 있습니다.
2. **형태학적 연산(Morphological Operations)을 통한 마스크 정제**:
   추출된 엣지 마스크에 자잘한 점 형태의 노이즈가 남을 경우, 침식(Erosion)과 팽창(Dilation)을 결합한 열기(Opening, `cv2.MORPH_OPEN`) 연산을 적용하여 윤곽선의 굵기는 유지하면서 미세한 노이즈만 깔끔하게 제거할 수 있습니다
3. **이미지 피라미드 및 병렬 연산(CUDA)을 통한 최적화**:
   연산량이 높은 Bilateral Filter의 속도를 개선하기 위해, 원본 이미지의 크기를 Downsampling한 후 필터를 적용하고 다시 Upsampling하는 방식을 통해 연산량을 극적으로 줄일 수 있습니다. 또한, 향후 OpenCV의 CUDA 모듈을 적용해 GPU 가속을 활용하면 비디오 스트리밍 수준의 실시간 렌더링도 구현할 수 있다고 생각합니다.
