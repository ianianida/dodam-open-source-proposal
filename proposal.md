# 도담도담 프로젝트 제안서

## 1. 주제

Open Food Facts SDK 확장을 통한 임산부 맞춤형 식품 확인 서비스 도담도담

분반: [분반]  /  팀: [팀 번호]  /  학번: [학번]  /  이름: 김이안

## 2. 요약

도담도담은 임산부가 스마트폰으로 식품을 구매할 때 자신에게 해당하는 주의사항을 쇼핑 중인 화면에서 확인하도록 돕는 서비스다. Android 앱의 플로팅 버튼으로 현재 상품 정보를 읽고, Open Food Facts Python SDK의 조회 기능을 확장해 임신 주차·건강상태·알레르기에 해당하는 안내와 출처를 제공한다. 사용자가 허용한 화면에서 상품 정보를 추출하고, AI는 정보를 구조화하고 근거에 따른 설명을 정리하는 데 활용한다. 이를 통해 다른 앱으로 이동해 자료를 첨부하고 질문하는 수고를 줄이고자 한다.

## 3. 대표 그림

> 사용자 모바일 캡처를 추가할 위치. [캡처 배치·설명 초안](mobile_capture_note.md)을 참고한다.

그림 1. 모바일 식품 쇼핑 앱의 상품 상세 화면 비교 — 상품 이미지, 상품명·가격과 구매 영역을 중심으로 구성한 자료

출처: [캡처한 앱 이름과 상품 링크] / 캡처일: [실제 캡처일]

## 6. 결론

본 프로젝트는 오픈소스 식품 조회 기능에 사용자 조건별 안내와 출처를 연결하고, Android의 모바일 쇼핑 흐름 안에서 결과를 보여주는 설계안이다. 화면 접근과 상품 식별 가능성을 먼저 확인한 뒤 시제품을 구현하고, 정보 누락 처리와 확인 과정의 개선 여부를 검증한다.

## 4. 서론

### 배경 설명 및 사례 분석

임신 중에는 식품의 종류뿐 아니라 성분, 조리 방법과 개인 조건에 따라 확인할 정보가 달라질 수 있다. 공식 자료에서도 주의할 식품·조리 조건, 임신 초기의 영양 안내와 임신성 당뇨 관련 식품 안내를 구분한다[3], [4], [5]. 스마트폰으로 식품을 고르다가 상품 화면을 캡처하거나 링크를 다른 AI 앱으로 옮겨 질문하면, 정보를 확인한 뒤 쇼핑 앱으로 돌아와야 한다. 이런 확인 과정은 구매 흐름을 끊고 반복 입력을 늘릴 수 있다.

분석 대상인 Open Food Facts Python SDK는 상품 조회·검색 기능을 제공한다[1]. 원재료와 영양 정보를 얻을 수 있지만[2], 쇼핑 중인 상품과 조회 결과를 연결하고 사용자에게 해당하는 주의사항과 근거를 제시하는 과정은 추가 설계가 필요하다. 본 프로젝트는 원본의 조회 구조와 반환 데이터를 분석하고 이 연결 과정을 개선한다.

### 문제 정의

상품 정보와 개인별 주의사항이 따로 있으면 사용자가 직접 대조해야 한다. 알레르기가 있는 사용자는 특정 성분을, 임신성 당뇨가 있다고 등록한 사용자는 탄수화물 정보를 먼저 확인할 수 있다[5], [6]. 화면에서 상품을 잘못 인식하거나 정보가 빠진 경우에도 충분한 근거 없이 안내를 만들기 어렵다. 따라서 쇼핑 화면에서 필요한 안내와 근거를 확인하고, 부족한 정보도 구분할 수 있어야 한다.

### 극복 방안

사용자가 화면 접근을 허용한 세션에서 플로팅 버튼을 누르면 현재 상품 정보를 읽고, SDK 조회 결과와 사용자 조건을 대조한다. 임신 주차·건강상태·알레르기는 적용 근거가 있는 안내에 반영한다. 결과는 쇼핑 화면 위의 하단 안내 패널에 주의사항, 적용 이유, 출처와 추가 확인 조건으로 표시한다. 초기 시제품은 Android의 쇼핑 앱 한 곳과 확인 가능한 예시 상품을 대상으로 구성한다. 구매 후 식재료 관리·레시피·일일 영양 리포트는 향후 확장으로 남긴다.

## 5. 본론

### 시스템 개요

![Android 상품 화면 확인과 SDK 확장을 연결하는 구조](02_시스템개요_Android.png)

그림 2. Android의 상품 화면 확인과 SDK의 맞춤 안내 기능을 연결하는 구조

### 필요한 기술 요소

- Android 앱: MediaProjection과 해당 포그라운드 서비스로 사용자가 허용한 상품 화면을 읽고[7], 다른 앱 위에 표시할 수 있는 권한을 확인해 플로팅 버튼과 하단 안내 패널을 구성한다[8].
- OCR·AI: 화면에서 상품명·브랜드·규격 등 표시된 정보를 추출하고[9], 조회 결과와 채택된 근거를 설명으로 정리한다. 누락 정보나 안내 기준을 임의로 만들지 않는다.
- Open Food Facts Python SDK·API 서버: 원본 조회·검색 코드와 반환 구조를 분석하고, 상품 정보·프로필을 받아 주의사항·이유·출처·정보 누락 상태를 반환하는 기능을 추가한다.
- 안내 기준: 임신 주차 범위, 건강 조건, 성분·영양 항목과 근거 URL을 기준표로 관리한다.
- Ubuntu·Git/GitHub: 서버의 실행 환경을 구성하고 이슈·브랜치·Pull Request로 변경과 리뷰를 관리한다.

### 구현 방법 및 개발 방향

1. SDK를 fork해 조회 구조와 라이선스를 분석한다. MIT 코드와 ODbL 식품 데이터의 이용 조건을 구분하고 원본 고지를 유지한다[1], [2].
2. Android에서 화면 접근·플로팅 표시를 먼저 검증하고 상품 정보를 추출한다. 브랜드·규격을 대조해 같은 상품인지 확인하며, 정보 누락·권한 거부·상품 오인은 확인 필요 상태로 처리한다.
3. 사용자 조건별 안내와 출처를 반환하는 SDK 기능을 추가하고 API 서버를 통해 모바일 화면과 연결한다. 검토한 안내 기준을 바탕으로 AI의 설명 기능을 연동한다.
4. 예시 상품 10~15개와 가상 프로필로 안내·근거 연결, 정보 누락과 지원하지 않는 조건을 검증한다.

4명은 원본 코드·서버, 안내 기준·기능, Android 화면 연동, 검증·문서화를 나누고 서로의 변경을 리뷰한다. 역할은 팀 주제 선정 후 조정한다. 원본 조회 결과와 확장 결과를 비교하고, 별도 AI 앱 방식과 시제품의 화면 전환·입력 횟수 및 확인 시간을 비교한다. 설치 방법과 원본과의 차이, 지원 범위와 출처를 README에 기록한다.

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
