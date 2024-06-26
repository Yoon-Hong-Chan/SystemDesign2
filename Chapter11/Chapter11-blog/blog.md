---
title: '네이버 페이, 카카오 페이, 페이코 등과 같은 결제 시스템들은 어떻게 만드는걸까? '
date: 2024-05-25T09:00:00+09:00
author: "YOON-HONG-CHAN"
cover:
  image: "cover.png"
---


# 💸 결제 시스템은 정확히 무엇을 말하는걸까요?
<img src="img_1.png" width="600px"/><br>
- 이번 블로그에서의 결제 시스템은 신용 카드, 은행 카드 등을 연동하여 결제 서비스를 제공하는 시스템을 말합니다.
- 요즘은 결제 시스템을 사용하시는 분들이 많기 때문에 '안정, 유연, 확장 가능'을 고려하여 시스템을 구축해야 합니다.
- 하지만 무엇보다도 '작은 실수로도 상당한 금액 손실이 발생'할 수 있다는 점을 간과해서는 안 됩니다.

# 💻 결제 시스템 흐름!
- 결제 시스템을 알기 위해서는 크게 '대금 수신 흐름'을 알아야 합니다.
#### 대금 수신 흐름
- 시스템 설계에서 필요한 주요 요소들이 있습니다.<br>
  <img src="img.png" width="600px"/><br>
  - 결제 서비스
      - 사용자로부터 결제 이벤트를 수락하고 결제 프로세스를 조율
      - AML/CFT와 같은 규정 준수 및 범죄 행위의 증거가 있는지 판별 등 위험 점검(risk check) -> 위험 확인을 통과한 결제만 진행
      - 위험 확인서비스는 매우 고도로 전문화되어 있음 -> 제 3자 제공업체 활용
  - 결제 실행자
      - 결제 서비스 공급자(PSP)를 통해 결제 주문 하나를 실행
      - 하나의 결제 이벤트에는 여러 결제 주문 포함 가능
  - 결제 서비스 공급자(PSP: Payment Service Provider)
      - A 계정에서 B 계정으로 돈을 옮기는 역할 담당
      - 예제로 구매자의 신용 카드 계좌에서 돈을 인출하는 역할
  - 카드 유형
      - 신용 카드 업무를 처리하는 조직
  - 원장
      - 결제 트랜잭션에 대한 금융 기록
      - 전자상거래 웹사이트의 총수익을 계산하거나 향후 수익을 예측하는 등, 결제 후 분석에서 매우 중요 역할
  - 지갑
      - 판매자의 계정 잔액을 기록
      - 특정 사용자가 결제한 총금액 기록
- 위와 같은 요소들을 통해 아래와 같은 흐름으로 결제가 진행되게 됩니다.
  - ① 사용자 주문하기 버튼 클릭 -> 결제 이벤트 생성 후 결제 서비스 전송
  - ② 결제 서비스는 결제 이벤트 DB에 저장
  - ③ 때로는 단일 결제 이벤트에 여러 결제 주문이 포함 가능
  - ④ 결제 실행자는 결제 주문을 DB에 저장
  - ⑤ 결제 실행자는 외부 PSP를 호출하여 신용 카드 결제를 처리
  - ⑥ 결제 실행자가 결제 성공적으로 처리 후, 결제서비스는 지갑을 갱신하여 특정 판매자의 잔고 기록
  - ⑦ 지갑 서버는 갱신된 잔고 정보를 DB에 저장
  - ⑧ 지갑 서비스가 판매자 잔고를 성공적으로 갱신 후 결제 서비스는 원장 호출
  - ⑨ 원장 서비스는 새 원장 정보를 DB 추가

# 🔎 결제 시스템 중요 세부 내용!
##### 복식부기 원장 시스템
- 모든 결제 시스템에 필수 요소이며, 정확한 기록을 남기는 데 핵심적 역할을 합니다.
  - 모든 결제 거래를 두 개의 별도 원장 계좌에 같은 금액으로 기록 -> 한 계좌에서 차감과 다른 계좌 입금

    | 계정  | 차감 | 증가 |
        |---|---|---|
    | 판매자  | $1  |   |
    | 구매자  |   | $1  |

- 이렇게 증가와 차감을 동시에 적기 때문에 모든 거래 항목의 합계는 0이어야 합니다. 만약 0이 되지 않는다면? 정산에 이슈가 있다고 판단하게 됩니다.
- 또한 이런 시스템은 자금 흐름을 추적할 수 있으며, 일관성 보장할 수 있습니다.
##### 조정
<img src="img_2.png" width="600px"/><br>
- PSP와의 결제 관련 연동 시, 결제 관련 내용들이 PSP와 완벽하게 동일하게 된다는 보장은 없다고 합니다.
- 이러한 불일치 되는 부분을 없애기 위해 주기적(보통 매일 밤)으로 PSP와의 정산 내용 비교를 통해 결제 내용을 교정한다고 합니다.
***
### 🌆(A사 사례)
 - PSP에서 결제 기준 하루가 지나 정산 데이터를 전달 또는 API를 조회하라고 가이드를 주면 해당 데이터를 조회하여 조정(대사)을 진행합니다.
 - PSP 쪽 거래번호(고유값)와 내부 데이터가 해당 거래번호와 같은지 여부를 체크합니다.
 - PSP 쪽 데이터와 내부 데이터가 맞지 않으면 월말 마감 전까지는 맞추려고 분석하면서 노력을 많이 합니다.
***

##### 결제 지연 처리
- 결제 요청은 몇 초 만에 처리되지만, 완료되거나 거부되는데 몇 시간 또는 며칠이 걸릴 수도 있다고 합니다.
  - PSP에서도 결제 위험성 검토를 하는 데 비정상적이라고 생각되면 담당자 검토를 요구한다고 하네요.
- 추후 완료가 되면 PSP에서는 등록된 Web-hook을 통해 결제 처리 결과를 알려준다고 합니다.
##### 정확히 한번 전달
- 네트워크 또는 시스템 이슈로 인해 고객에게 이중으로 청구하는 건 심각한 문제 중 하나라고 합니다.
- 결제 관련해서 정확하게 한번 전달하기 위해 아래와 같이 2가지 시점으로 나눠 해결해 봅니다.
  - 최소 한 번은 실행(재시도)<br>
    <img src="img_3.png" width="300px"/><br>
    - 재시도 매커니즘을 도입할 때는 얼마나 간격을 두고 재시도할지 정하는 것 중요
    - 적절한 재시도 전략을 결정하는 것은 어려움 -> 모든 상황에 맞는 해결책은 존재하지 않음
      - 일반적으로 적용 가능한 지침은 네트워크 문제는 단시간 해결 불가능 -> 지수적 백오프 사용 권고
      - 에러 코드 반환할 때는 'Retry-After' 헤더를 붙여 보낸 것이 바람직
  - 최대 한 번 실행(멱등성 검사)<br>
    <img src="img_4.png" width="400px"/><br>
    - 최대 한 번 실행을 보장하기 위한 핵심 개념 -> 여러번 반복해도 동일한 결과 처리
    - HTTP 헤더에 <명등키: 값>의 형태로 멱등 키를 추가
    - 처음 처리 가능 -> 두번째 처리 불가능 -> 동일한 멱등 키로 동시에 많은 요청 받으면 Status 429 내림
    - 데이터베이스의 고유 키 제약 조건 활용 (멱등성 지원 방법 중 하나)
      - 데이터 베이스 테이블의 기본 키를 멱등키로 활용 -> 중복 삽입 불가
- 이처럼 '최소 한번'과 '최대 한 번 실행' 메커니즘을 결제에 대해 정확하게 한번 전달 문제를 해결합니다.

# 🍵 마무리!
#### 결제 시스템에서 다루는 내용을 간략히 알아봤습니다. 저도 평소 많은 궁금증이 있었는데요(주변에서 금융 도메인을 접해보기는 쉽지 않으니까요 ㅎㅎ).
#### 이번 글이 저 말고도 결제 시스템에 대해 궁금한 점이 있으셨던 분들에게 도움이 되었기를 바랍니다.