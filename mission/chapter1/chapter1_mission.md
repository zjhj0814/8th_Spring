# 미션
### 1. 주어진 IA(기획 플로우)와 와이어 프레임(디자인 프로토타입)을 보고 직접 데이터베이스를 설계해오기
### 이때, 위에서 언급한 경우를 모두 만족해야 함
1차] ERD
![Image](https://github.com/user-attachments/assets/2ffe489b-7659-4a56-b2c8-f5df745b4a73)
2차] 개인 피드백 반영
![Image](https://github.com/user-attachments/assets/ef20dcc2-c17f-4549-9d5b-8aabe58a291b)
- user -> member 수정
- phone_auth 테이블 조회 성능을 위해 반정규화
- member와 review 일대다 관계 맵핑
- 가게 사진 저장하도록 store_picture 테이블 생성 후 store과 다대일 관계 맵핑
- 하나의 가게는 하나의 카테고리만 설정할 수 있다고 가정하고 store과 food_type 다대일 관계 맵핑
- user 테이블에 point 컬럼, mission 테이블에 deadline 컬럼, terms_conditions에 is_essential 컬럼 추가