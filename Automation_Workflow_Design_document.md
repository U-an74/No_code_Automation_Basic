# 📁 자동화 워크플로우 설계 문서

*노코드 자동화 기초: 프로젝트2 - 자유 주제 자동화 설계 및 구현*

---

## 1. 자동화 대상 반복 업무 정의

- 목표 : 제조업체 업무 자동화
- 주제 : 불량 발생 현황의 실시간 관리 자동화, 처리 지연 방지
- 방법 : 신규 불량 정보가 입력되면, 자동으로 처리 可否를 분류하고 관련 자료를 작성한다.

## 2. 자동화 도구 선정 및 이유

- 선정 자동화 도구 : Make
- 선정 이유 :  초보자向 진입 장벽이 낮았던 점과 장기적 안정성 측면의 강점
    - 100% 노코드 설계 가능, 보다 직관적인 UI/UX
    - 넓은 무료 플랜 범위, 월간 크레딧 리셋
- 추가 사용 도구 : Google Spreadsheet, Discord

## 3. 워크플로우 설계

1. 불량 발생을 확인한 사용자가 <Raw data>시트에 내용을 등록하면 신규 행 추가를 Trigger로 자동화가 시작된다. 
2. 자동화 도구는 새로 입력된 불량의 유형을 확인한 뒤, 홀 버 불량 포함 여부를 기준으로 경로 분기한다. 
3. 홀 버 불량 포함 여부에 따라 시트를 달리하여 기록하고, 자동화 추적을 위한 처리 일시는 trigger 시트인 <Raw data>시트에 입력하여 활용한다.  
4. <처리 가능 불량> 시트에서 타흔/홀 메카스 불량만 필터링하여 <선별 일지> 시트를 추가로 작성한다.
5. 실패 알림 재시도 전략으로써 경로3을 추가로 설계한다.   

<aside>
💡

**Workflow : 불량 발생 현황 실시간 관리 자동화** 

- **[Trigger] <Raw data>** 신규 행(불량 내용) 등록/감지
- **[Router]** **홀 버 불량 포함 여부**를 기준으로 분기
→ 처리 可否 판단 규칙: 홀 버를 포함하면 처리 불가
- 경로.1 홀 버 불량 **포함**인 경우
    
    > **[Action 1]** <처리 **불가** 불량> 시트에 기록(1차 분류)
    > **[Action 2]** 자동 분류 완료 일시 정보를 <Raw data> 시트에 업데이트 
    
- 경로.2 홀 버 불량 **미포함**인 경우
    
    > **[Action 1]** <처리 **가능** 불량> 시트에 기록(1차 분류)
    > 
    > **[Action 2]** 자동 분류 완료 일시 정보를 <Raw data> 시트에 업데이트
    > 
    > **[Filter] 타흔/홀 메카스 불량만 필터링, 다음 단계로**
    > 
    > **[Action 3]** <선별 일지> 시트에 기록(2차 분류) 및 담당 자동 배정
    > 
    > → 배정 규칙: {{if(2.`1` = "타흔"; "현장 작업자"; if(2.`1` = "홀 메카스"; "사무직"; ""))}}

- 경로.3 불량 유형 입력 누락인 경우(error handler)
    
    > **[Action 4]** 자동 분류 실패 정보를 Discord 알림 메세지로 발송
    > 
    > **[Action 2]** 자동 분류 실패 정보를 <Raw data> 시트에 업데이트
    
</aside>

## 4. 구현 결과

- 구현 화면

<img width="1332" height="736" alt="image" src="https://github.com/user-attachments/assets/8dc18777-efcd-494f-8c98-daee54ff6b09" /> [Make/Workflow 구성 화면]


- 실행 결과 화면
    - 실행 방법 : 시나리오 저장 → activate(15분마다 자동 실행) → 신규 행 추가
    - 결과 요약 :  신규 행을 정상 감지하였고, 불량 유형 분기와 필터에 따라 모든 작업이 설계 의도대로 수행됨.
<img width="1426" height="843" alt="test_data_" src="https://github.com/user-attachments/assets/73c6e469-34af-4521-aa40-469708ffef1e" /> [경로 1/2/3 총 12번 검증 test 기록 - all pass]

<img width="1370" height="849" alt="image 1" src="https://github.com/user-attachments/assets/d30e1f67-4186-4bb9-a9b8-d1ceda982b68" /> [trigger] watch new rows - 신규 행(불량 내용) 감지 / [Action 2] update a row - 자동 분류 완료 일시 기록 

<img width="1440" height="817" alt="스크린샷_2026-08-31_112801" src="https://github.com/user-attachments/assets/803990bb-49e2-4f84-b2ef-282b7bc3ba43" />[Action 1] add a row - 1차 불량 분류(처리 불가)

<img width="1452" height="811" alt="image 2" src="https://github.com/user-attachments/assets/d287d8aa-25d1-4d6c-8311-1110f8c08d3f" /> [Action 1] add a row - 1차 불량 분류(처리 가능)

<img width="1399" height="847" alt="image 3" src="https://github.com/user-attachments/assets/856482f6-f211-4fe8-bae7-29eba4372793" /> [Action 3] add a row - 2차 불량 분류(처리 가능 불량 중 선별 가능) 및 자동 담당 배정

<img width="962" height="707" alt="image 4" src="https://github.com/user-attachments/assets/308ccba2-215a-4eb9-8ec4-cd75311b3c01" /> [Action 4] send a message - 자동 분류 실패 정보 알림

</aside>

## 5. 모듈화 전략

### (1) 개요
현재 하나의 시나리오에 집중된 로직을 역할 기준 아래 3개 모듈로 분리하여 유지보수성과 재사용성을 높인다.

### (2) 구성
| 모듈 | 시나리오 | 역할 |
|------|----------|------|
| 🔵A | 입력 검증 | 데이터 수신 및 필수값 검증 |
| 🟡B | 분류·저장 | 홀버 유무 판단 및 시트 저장 |
| 🔴C | 알림·배정 | 자동배정 및 Discord 알림 |
 
### (3) 재사용 방안
- 🔵 시나리오 A (검증 모듈)
  필수값 항목만 수정하면 다른 불량 관련 자동화에 그대로 적용 가능
  공통 검증 로직으로 재사용 가능
- 🔴 시나리오 C (알림 모듈)
  알림 채널(Discord → Email 등)만 교체하면 타 프로젝트에 즉시 적용 가능
  배정 조건만 수정해 다양한 업무에 확장

### (4) 기대 효과
- 수정 용이성 : 알림 방식 변경 시 C만 수정, 나머지 영향 없음
- 오류 추적 : 어느 모듈에서 오류 발생했는지 즉시 파악 가능
- 재사용성 : A·C 모듈은 타 프로젝트에 독립적으로 재사용 가능
- 확장성 : 새로운 분류 조건 추가 시 B만 수정하면 됨
