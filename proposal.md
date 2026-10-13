# 도담도담 프로젝트 제안서

## 1. 주제

Open Food Facts SDK 확장을 통한 임산부 맞춤형 식품 확인 서비스 도담도담

분반: [분반]  /  팀: [팀 번호]  /  학번: [학번]  /  이름: 김이안

## 2. 요약

도담도담은 임산부가 모바일 쇼핑 중 식품 주의사항을 찾는 시간과 불확실성을 줄이기 위한 서비스다. Android 앱의 플로팅 버튼으로 사용자가 허용한 상품 화면을 읽고, Open Food Facts Python SDK를 확장해 상품 정보와 임신 주차·건강상태·알레르기에 해당하는 안내를 연결한다. 주의사항과 적용 이유·출처를 같은 화면의 하단 패널에 보여주며, 정보가 부족하면 추가로 확인할 항목을 표시한다. AI는 정보 구조화와 근거에 따른 설명 정리에 활용한다. 이를 통해 앱을 옮겨 자료를 첨부하고 질문하는 수고를 줄이고, 사용자가 근거를 살펴보고 구매 여부를 판단하도록 돕고자 한다.

## 3. 대표 그림

> 사용자 모바일 캡처를 추가할 위치. [캡처 배치·설명 초안](mobile_capture_note.md)을 참고한다.

그림 1. 모바일 식품 쇼핑 앱의 상품 상세 화면 비교 — 상품 이미지, 상품명·가격과 구매 영역을 중심으로 구성한 자료

출처: [캡처한 앱 이름과 상품 링크] / 캡처일: [실제 캡처일]

## 6. 결론

본 프로젝트는 임산부가 모바일 쇼핑 중 식품 주의사항을 확인하는 데 드는 시간과 불확실성을 줄이기 위해, Open Food Facts Python SDK에 사용자 조건별 안내 기능을 추가하는 제안이다. 상품 조회 결과에 사용자에게 해당하는 주의사항과 적용 이유·출처를 연결하고, Android 쇼핑 화면에서 확인하도록 한다. 임신 주차·건강상태·알레르기는 근거가 있는 안내에만 반영하며, 상품을 잘못 식별하거나 정보가 부족한 경우에는 추가로 확인할 부분을 표시한다. 이를 통해 다른 앱으로 이동해 자료를 첨부하고 질문하는 수고를 줄이고, 사용자가 근거를 참고해 구매 여부를 판단하도록 돕고자 한다.

향후에는 대상 쇼핑 앱의 화면 접근과 상품 조회 가능성을 먼저 확인하고, 예시 상품과 가상 프로필로 시제품을 구현·검증할 계획이다. 안내와 근거가 올바르게 연결되는지, 정보 누락과 지원하지 않는 조건이 구분되는지 확인하고, 같은 상품·조건으로 별도 AI 앱 방식과 화면 전환·입력 횟수 및 확인 시간을 비교한다. 개발 과정과 원본 SDK의 변경 내용은 GitHub에 기록하고, 추가 기능의 사용 방법과 지원 범위를 문서화한다. 이후 사용자 피드백과 확보한 데이터에 따라 지원 상품·앱을 확대하고, 구매 후 식품 관리 기능의 필요성을 검토한다.

## 4. 서론

### 배경 설명 및 사례 분석

임신 전에 즐겨 먹던 식품도 임신 중에는 확인해야 할 정보가 달라질 수 있다. 예를 들어 NHS는 치즈의 종류와 살균·조리 여부에 따라 주의사항을 구분하며[3], 임신 초기의 영양 안내와 임신성 당뇨 식품 안내도 따로 제공한다[4], [5]. 스마트폰 쇼핑 중 이런 정보를 알아보기 위해 상품 화면이나 링크를 다른 AI 앱으로 옮겨 질문하는 상황을 가정하면, 확인 후 쇼핑 앱으로 돌아와야 한다. 이 반복을 줄일 방법으로 도담도담을 기획했다.

분석 대상인 Open Food Facts Python SDK의 api.product.get()은 바코드로 상품을 조회하고, api.product.text_search()는 상품명 등으로 검색한다[1]. 호출자는 이 결과를 받아 사용자 조건별 안내를 별도로 구성해야 한다. 도담도담은 원본의 조회 기능을 유지하면서, 상품 정보에 맞춤 주의사항·근거·누락 상태를 연결하는 기능을 fork한 SDK에 추가하고자 한다.

### 문제 정의

문제는 상품의 원재료·영양 정보와 개인 조건별 주의사항을 사용자가 직접 대조해야 한다는 점이다[2], [5], [6]. 상품을 잘못 식별하거나 성분 누락을 구분하지 못하면 충분한 정보 없이 판단하게 될 수도 있다. 따라서 구매 흐름 안에서 상품을 확인하고, 근거가 있는 안내와 추가 확인이 필요한 부분을 구분하는 기능이 필요하다.

### 극복 방안

사용자가 화면 접근을 허용한 세션에서 플로팅 버튼을 누르면 상품명·브랜드·규격을 추출하고 조회 후보와 대조한다. 확인된 상품 정보에 임신 주차·건강상태·알레르기별 안내 기준을 적용하되, 근거가 확인된 조건만 반영한다. 결과는 쇼핑 화면 위의 하단 패널에 주의사항·이유·출처와 추가 확인 항목으로 표시하며, 상품을 식별하지 못하면 사용자에게 확인을 요청한다. 초기 범위는 Android의 쇼핑 앱 한 곳과 구매 전 확인으로 정한다. 구매 후 관리·레시피·일일 영양 리포트는 향후 확장으로 남긴다.

## 5. 본론

### 시스템 개요

![Android 상품 화면 확인과 SDK 확장을 연결하는 구조](02_시스템개요_Android.png)

그림 2. Android의 상품 화면 확인과 SDK의 맞춤 안내 기능을 연결하는 구조

Android 앱이 추출한 상품 정보와 사용자 조건을 Python API 서버로 보내면, 확장 SDK가 상품 조회·조건 대조·근거 연결을 수행한다. 반환된 주의사항·출처·누락 항목은 앱의 하단 안내 패널에 표시한다.

### 필요한 기술 요소

- Android 앱: MediaProjection과 해당 포그라운드 서비스로 사용자가 허용한 상품 화면을 읽고[7], 다른 앱 위에 표시할 수 있는 권한을 확인해 플로팅 버튼과 하단 안내 패널을 구성한다[8].
- OCR·AI: OCR로 상품명·브랜드·규격을 추출하고[9], AI로 조회 결과와 채택된 근거의 설명을 정리한다. 안내 기준과 누락 정보를 임의로 만들지 않는다.
- Python SDK·API 서버: fork한 SDK에 상품 정보·프로필·안내 기준을 받아 주의사항·이유·출처·누락 항목을 반환하는 모듈을 추가하고, 앱과 연결한다.
- 안내 기준: 임신 주차 범위, 건강 조건, 성분·영양 항목과 근거 URL을 기준표로 관리한다.
- Ubuntu·Git/GitHub: 서버의 실행 환경을 구성하고 이슈·브랜치·Pull Request로 변경과 리뷰를 관리한다.

### 구현 방법 및 개발 방향

1. SDK를 fork해 조회·검색 코드와 반환 구조를 분석한다. MIT 코드와 ODbL 식품 데이터의 이용 조건을 구분하고 원본 고지를 유지한다[1], [2].
2. 대상 쇼핑 앱의 화면 접근·플로팅 표시를 확인한다. 상품명으로 검색한 후보를 브랜드·규격과 대조해 조회 가능한 예시 상품 10~15개를 선정한다. 불일치·미조회 상태는 사용자 확인이 필요한 상태로 처리한다.
3. 기존 조회 함수를 유지하고, 사용자 조건을 기준표와 대조하는 안내 모듈을 추가한다. API 서버와 앱을 연결하고, AI로 채택된 근거의 설명을 정리한다.
4. 가상 프로필로 조건 일치·미지원 조건·정보 누락·권한 거부·상품 오인 사례를 검증한다. 안내와 채택된 출처가 올바르게 연결되고, 누락 항목이 표시되는지 확인한다.

팀 주제로 선정되면 4명이 원본 코드·서버, 안내 기준·기능, Android 연동, 검증·문서화를 나누고 서로의 변경을 리뷰한다. 원본과 확장 결과를 비교하고, 같은 상품·조건으로 별도 AI 앱 방식과 시제품의 화면 전환·입력 횟수 및 확인 시간을 측정한다. 설치 방법·변경 내용·지원 범위·출처는 README에 정리한다.

## 7. 출처

[1] Open Food Facts, “Open Food Facts Python SDK,”
https://github.com/openfoodfacts/openfoodfacts-python

[2] Open Food Facts, “Introduction to Open Food Facts API documentation,”
https://openfoodfacts.github.io/openfoodfacts-server/api/

[3] NHS, “Foods to avoid in pregnancy,”
https://www.nhs.uk/pregnancy/keeping-well/foods-to-avoid/

[4] NHS, “Pregnancy vitamins and supplements,”
https://www.nhs.uk/pregnancy/keeping-well/pregnancy-vitamins-and-supplements/

[5] University Hospitals Sussex NHS Foundation Trust, “Healthy eating with gestational diabetes,”
https://www.uhsussex.nhs.uk/resources/healthy-eating-with-gestational-diabetes/

[6] U.S. Food and Drug Administration, “Food Allergies,”
https://www.fda.gov/food/nutrition-food-labeling-and-critical-foods/food-allergies

[7] Android Developers, “Media projection,”
https://developer.android.com/media/grow/media-projection

[8] Android Developers, “WindowManager.LayoutParams — TYPE_APPLICATION_OVERLAY,”
https://developer.android.com/reference/android/view/WindowManager.LayoutParams#TYPE_APPLICATION_OVERLAY

[9] Google for Developers, “Text recognition v2,”
https://developers.google.com/ml-kit/vision/text-recognition/v2

웹 자료 확인일: 2026년 10월 13일.

[1]: https://github.com/openfoodfacts/openfoodfacts-python
[2]: https://openfoodfacts.github.io/openfoodfacts-server/api/
[3]: https://www.nhs.uk/pregnancy/keeping-well/foods-to-avoid/
[4]: https://www.nhs.uk/pregnancy/keeping-well/pregnancy-vitamins-and-supplements/
[5]: https://www.uhsussex.nhs.uk/resources/healthy-eating-with-gestational-diabetes/
[6]: https://www.fda.gov/food/nutrition-food-labeling-and-critical-foods/food-allergies
[7]: https://developer.android.com/media/grow/media-projection
[8]: https://developer.android.com/reference/android/view/WindowManager.LayoutParams#TYPE_APPLICATION_OVERLAY
[9]: https://developers.google.com/ml-kit/vision/text-recognition/v2
