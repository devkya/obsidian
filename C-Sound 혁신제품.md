
## 판단
### 통합될 수 없는 이유
1. 추후 C-Biz 기획과 통합될 수 없음
	1. 전시회 선택
	2. 미팅기록
	3. 포럼 및 비즈니스 미팅 추천
2. 상위 노출, 등의 BM과 C-Sound 혁신제품과 연관성이 없음
3. 라이센스 개념이 다름
4. Auth
## 전제
1. 혁신제품 앱 리뉴얼 기간이 반드시 필요
2. OTA 업데이트 및 추후 앱 스토어 배포가 일어나면, 기존 
## Split Mode
> 현재 구현되어 있는 기능 외 추가 기능 없다고 판단됨
### Server
- 서버 복제 후, Auth만 B2G에 맞게 변경
	- Facepro와 같은 구조
- `deprecate endpoint` -> `normal endpoint`로 전환
- 현재 사용량 체크 등의 부가 기능 전부 제외
### Mobile
- 로그인 추가
- 도메인 및 `endpoint` 변경
- OTA 업데이트
## Dual Mode
> 정보 패널
### Server
- `Split Mode`와 동일

### Mobile
- `Split Mode`와 동일
## 궁금해
1. OTA 업데이트 기능을 추가하는데 걸리는 시간 -> 모바일팀 답변
## 추후 예정
1. C-Sound 혁신제품 앱 통합
2. 플레이 스토어 배포(앱 스토어X)