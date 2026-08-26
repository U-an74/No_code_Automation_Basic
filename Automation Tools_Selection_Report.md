# 📁자동화 도구 비교 분석 보고서

*노코드 자동화 기초: 프로젝트1 - 자동화 도구 비교 구현* 

---


## 1. 컨셉

- 목표: 제조업체 업무 자동화
- 주제: 품질팀向 타부서 **업무 요청의 실시간 관리 및 누락 방지**
- 방법: 타부서가 품질팀에 요청을 접수하면, **우선순위에 따라 자동 알림을 전송**하고 **처리 로그를 기록**한다.


## 2. 사용 도구 목록

1. Make (비교 대상 1) 🟠
2. Zapier (비교 대상 2) 🟢 
3. Google Spreadsheet
4. Discord


## **3. 구현 과정 및 워크플로우**

1. 각 부서의 사용자가 <요청 접수>시트에 업무 요청 등록 → 신규 행 추가를 Trigger로 자동화 시작
2. 자동화 도구는 새로 입력된 요청의 우선순위 값을 확인한 뒤, 이를 기준으로 조건/경로 분기
3. 우선순위 값에 따라 알림 전송 여부를 결정하고, <처리 로그>시트 상 상태 값을 달리 입력 
4. <처리 로그>시트에는 모든 요청 내용을 기록하고, 자동화 추적을 위한 처리 시간을 추가 입력 

*※ <처리 로그>시트는 후속 관리의 기준 문서로 활용, <요청 접수> 시트는 단순 접수 용도로만 사용*

<aside>
💡Workflow : 품질팀 협업 과제 관리 

- **[Trigger]** Google Sheets 신규 행(업무 요청) 등록/감지
- **[Filter/Router]** 업무 요청의 **우선순위 값**을 기준으로 분기
- 경로.1 우선순위가 **높음**인 경우
    
    > **[Action 1]** Discord로 즉시 접수 알림 전송
    
    > **[Action 2]** 요청 내용을 처리 로그 시트에 기록
    
    > **[Action 3]** 처리 로그의 상태 값을 **접수**로 저장
    > 
- 경로.2 우선순위가 **보통/낮음**인 경우
    
    > [Action 1] Discord 알림 전송 생략
    
    >[Action 2] 요청 내용을 처리 로그 시트에 기록
    
    >[Action 3] 처리 로그의 상태 값을 **접수대기**로 저장
    > 
- **[후속 관리]** <처리 로그>시트에 기록된 모든 요청 내용은 담당자가 직접 확인한 뒤, 
상태값을 **접수**, **처리중**, **완료**로 변경하며 Follow-up
</aside>


## 4. 실행 결과

- **Make** 🟠
  - Workflow 구성 화면     
  <img width="1252" height="666" alt="make_workflow" src="https://github.com/user-attachments/assets/608e108e-57b4-490d-8920-38d696e592bd" />

  - 실행 화면  
  <img width="1692" height="782" alt="make_요청접수" src="https://github.com/user-attachments/assets/f29d4ff3-57f7-4eb4-9f6c-887a8988890d" /> [Make/Trigger/Watch new rows/Google Sheets]
    
  <img width="1682" height="781" alt="make_discord" src="https://github.com/user-attachments/assets/18597507-8020-4374-9c05-c819999732f3" /> [Make/Action1/Send a message/Discord]

  <img width="1796" height="806" alt="make_처리로그" src="https://github.com/user-attachments/assets/588e3671-8487-49e8-b12a-f9115abda4fd" /> [Make/Action2&3/Add a row/Google Sheets]
  - 실행 방법 : 시나리오 저장 → activate(15분마다 자동 실행) → 신규 행 추가
  - 실행 결과 :  신규 행을 정상 감지하였고, 우선순위 분기에 따라 모든 작업이 설계 의도대로 수행됨. 
    (우선순위 높음/보통/낮음 3개 케이스 all 검증 완료)
  - 이슈 : Trigger 기준점 설정
    - activate 모드 실행 전에, 새 행을 추가하는 경우 동작하지 않는 상황 발생 → ex) 새 행 추가(A7) - Activate - 새 행 추가(A8) = A7 처리 누락
    - 편집 모드에서 수동으로 시작점을 지정해서 test를 실행하고 save한 뒤, Trigger 시트를 정리하고 같은 데이터를 재사용하는 경우, activate 모드 실행 시 동작하지 않는 상황 발생 → [Choose Where to start] 값을 다시 설정하면 정상 동작함을 확인함.

            
- **Zapier** 🟢
    - Workflow 구성 화면 
    <img width="749" height="709" alt="자피어_워크플로우" src="https://github.com/user-attachments/assets/04eeb99d-a99f-4134-aa3c-eb6622c92924" /> 
   
    - 실행 화면
    <img width="1350" height="840" alt="zapier_요청접수" src="https://github.com/user-attachments/assets/dd6a29d6-dbf2-4b6f-846e-522450fc50cc" /> [Zapier/Trigger/New Spreadsheet row/Google Sheets]

    <img width="1077" height="712" alt="image" src="https://github.com/user-attachments/assets/7badd399-92ac-41c0-855d-11805f3d5d07" /> [Zapier/Action1/Send Channel message/Discord]
    
    <img width="1710" height="842" alt="zapier_처리로그" src="https://github.com/user-attachments/assets/882b3624-3d95-407c-9245-8485c780f316" /> [Zapier/Action2&3/Create spreadsheet row/Google Sheets]  
    
    - 실행 방법: 시나리오 publish → Turn zap on(약 6분마다 자동 실행) → 신규 행 추가
    - 실행 결과:  신규 행을 정상적으로 감지하고, 우선순위 분기에 따라 설계된 모든 작업을 정확하게 처리하였음. (우선순위 높음/보통/낮음 3가지 case 검증 완료)
    - 이슈 : Trigger ****기준점 설정 및 단계별 테스트 데이터 참조 오류
        - Turn Zap on 모드 실행 전에, 새 행을 추가하는 경우 동작하지 않는 상황 발생 
        → ex) 새 행 추가(A7) - Turn Zap on - 새 행 추가(A8) = A7 처리 누락
        - trigger 단계에서 최초 불러온 data를 기준으로는 1개 path만 test를 통과하는 상황 발생
        → path 별 참조 data를 trigger 단계에서 각각 불러와서 2번 진행해야 정상 동작함을 확인


## 5. 결과 비교

- **비교 기준**
    1. UI/UX: 자동화 흐름을 화면에서 얼마나 직관적으로 확인하고 설정할 수 있는가?
    2. 설정 난이도: 트리거, 분기, 액션 설정 과정이 얼마나 쉽고 간단한가?
    3. 연동 서비스 범위: 다양한 외부 서비스와의 연결 가능 여부와 확장성
    4. 무료 플랜 범위: 무료 버전에서 제공되는 기능과 사용 제한
    5. 실행 로그 확인 방식: 자동화 실행 결과와 오류 내역을 얼마나 쉽게 확인할 수 있는가?
- **비교 결과표**
    
    3.에서 동일하게 설계한 Workflow를 Make와 Zapier에서 각각 구현한 뒤, 
    상기 5개 기준에 따라 정성적 비교를 진행했다. 
    
    | 비교 항목 | Make 🟠 | Zapier 🟢 |
    | --- | --- | --- |
    | UI/UX | 아이콘형, 매우 직관적, 수평 전개 | 배지형, 직관적, 수직 전개 |
    | 설정 난이도 | 보통(AI 도움O) | 높음(AI 도움O, 간단 코드 설계 포함) |
    | 연동 서비스 범위 | 디스코드, 구글시트 연동 가능 | 디스코드, 구글시트 연동 가능 |
    | 무료 플랜 범위 | 무료 1000 크레딧 제공, router기능 무료 플랜 범위 내 | 무료 1000 크레딧 제공, path기능 제한 제공(pro trial only) |
    | 실행 로그 확인 방식 | History, detail 정보 제공, 동작하지 않음 이력O | Zap runs, detail 정보 제공, 동작하지 않음 이력X |
    
    <img width="1777" height="842" alt="make_history" src="https://github.com/user-attachments/assets/b805a651-ebcd-4739-8aac-417b1f7d03c3" /> [Make/History 화면]
    
    <img width="1891" height="932" alt="zapier_run_detail" src="https://github.com/user-attachments/assets/edd82d18-7da3-4654-9bfd-47f33086c4d4" /> [Zapier/Zap runs 화면]


## 6. 결론

- **각 도구의 장단점**
    - Make 🟠
        - {{formatDate(now; "YYYY-MM-DD HH:mm"; "Asia/Seoul")}} 함수를 사용하면 간단하게 한국 기준 현재 시간을 불러올 수 있었음.
        - Trigger 설계 시, limit 수(한번에 읽을 행의 수)를 정할 수 있어 한번에 여러 새 행을 읽어야 하는 경우에 대해 안정성이 높았음.
        - 동작 하지 않은 경우에도 실행 로그 이력을 남겨 이슈 추적에 용이했음.
    - Zapier 🟢
        - 각 단계별 Test step이 있어 정상 작동을 확실하게 확인하고 넘어갈 수 있다는 장점이 있었지만, Path를 사용해 설계가 복잡해지자 단계별 테스트 데이터의 참조 이슈가 발생하여 초기 설정의 난이도가 상승함.
        - 기본 설정만으로는 한국 기준 현재 시간이 반영되지 않아, 추가적인 Formatter모듈 또는 Code step이 필요했음.
        - 동작 하지 않은 경우에는 실행 로그 이력을 남기지 않아 이슈 추적이 어려웠음.
        
        <br>
        
- **최종 결정 의견** : **Make 🟠**

우선 주제가 노코드 자동화인 만큼 **추가적 code step을 사용해야 했던 점에서 zapier에 감점**이 컸고, 설정과정의 이슈 해결 및 난이도, 연동 서비스 범위에 있어서는 비슷한 수준이라고 판단되나, **UI/UX 측면에서도 make가 조금 더 직관적**이고 초보자의 접근에 용이했다. 

특히 이번 구현 테스트에서는 Zapier가 Paths 기능(pro)을 기간제 무료 제공하여 하나의 Zap 안에서 분기할 수 있었지만(14일 후 만료예정), 다음 프로젝트2 구현에 Zapier를 쓰게 된다면, **장기적인 안정성을 위해 path 기능 없이 여러 Zap으로 분리 구성한 버전을 만들어 운용해야 한다는 점**이 가장 큰 결정 배제 요인이 되었다. 

또한 trigger의 기준점 설정 이슈는 두 도구 모두 발생했지만, **make는 이력을 남긴 반면 zapier는 별도 이력 남김 없이 동작하지 않았다**는 점 역시 make를 최종 결정하게 만든 이유가 되었다.
