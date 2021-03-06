> **트랜잭션이란?**

**하나의 작업을 수행**하기 위해 **DB의 연산들은 모아놓은 것**입니다. DB에서 논리적인 작업의 단위이며 장애가 발생했을 때 데이터를 복구하는 작업의 단위입니다.

DB 연산이란 SELECT, INSERT, UPDATE, DELETE 라는 질의어를 통해 DB에 접근하는 것을 말합니다.

작업의 단위란 무엇일까? 예를 들어 **사용자 A가 사용자 B에게 만원을 전송**한다고 가정 해보자.
> 이때 DB 작업  
>> - 1. 사용자 A의 계좌에서 만원을 차감한다 : UPDATE 문을 사용해 사용자 A의 잔고를 변경  
>> - 2. 사용자 B의 계좌에 만원을 추가한다 : UPDATE 문을 사용해 사용자 B의 잔고를 변경  

> 현재 작업 단위 : 출금 UPDATE문 + 입금 UPDATE문 → 이를 통틀어 하나의 트랜잭션이라고 한다.  
>> - Commit: 위 두 쿼리문 모두 성공적으로 완료되어야만 "하나의 작업(트랜잭션)"이 완료되는 것이다.  
>> - Rollback: 작업 단위에 속하는 쿼리 중 하나라도 실패하면 모든 쿼리문을 취소하고 이전 상태로 돌려놓아야한다.

> **트랜잭션의 특징**

- **원자성**

1.  트랜잭션 연산은 데이터베이스에 모두 반영되든지 아니면 전혀 반영되지 않아야 한다.
2.  트랜잭션의 모든 명령들은 완벽하게 수행되어야 하며, 모두가 완벽히 수행되지 않고, 어느하나라도 오류가 발생하면 트랜잭션 전부가 취소되어야 한다.

- **일관성**

1.  트랜잭션이 실행을 성공적으로 완료하면 언제나 일관성 있는 데이터베이스 상태로 유지하는 것을 의미합니다.
2.  시스템이 가지고 있는 고정요소는 트랜잭션 수행 전과 트랜잭션 수행 완료 후의 상태가 같아야 합니다.
3.  트랜잭션 수행 전후의 DB 상태는 각각 일관성이 보장되는 서로 다른 상태가 됩니다. 트랜잭션 수행이 보존해야 할 일관성은 기본 키, 외래 키, 제약과 같은 명시적인 무결성 제약 조건과 자금 이체 예에서 두 계좌 잔고의 합은 이체 전후가 같아야 한다는 사항과 같은 비명시적인 일관성 조건도 있습니다.

- **독립성**

1.  둘 이상의 트랜잭션이 동시에 병행 실행되는 경우 어느 하나의 트랜잭션 실행중에 다른 트랜잭션의 연산이 끼어들 수 없습니다.
2.  수행중인 트랜잭션은 완전히 완료될 때까지 다른 트랜잭션에서 수행결과를 참조할 수 없습니다.

- **지속성**

1.  성공적으로 완료된 트랜잭션의 결과는 시스템이 고장나더라도 영구적으로 반영되어야 합니다.

> **트랜잭션 연산 및 상태**

- Commit 연산

1.  하나의 트랜잭션이 성공적으로 끝났고, 데이터베이스가 일관성있는 상태에 있을 때, 하나의 트랜잭션이 끝났다라는 것을 알려주기위해 사용하는 연산입니다.

- Rollback 연산

1.  하나의 트랜잭션 처리가 비정상적으로 종료되어 트랜잭션 원자성이 깨진 경우, 이 트랜잭션의 일부가 정상적으로 처리되었더라고 트랜잭션의 원자성을 구현하기 위해 이 트랜잭션이 행한 **모든 연산을 취소(Undo)**하는 연산입니다.
2.  Rollback 시에는 해당 트랜잭션을 재시작하거나 폐기합니다.

> **트랜잭션의 상태**

![1](https://user-images.githubusercontent.com/63203480/132120641-8111ac3e-a054-4e40-aa20-9abac66ce1af.PNG)

-   Active(활동) : 트랜잭션이 실행중인 상태
-   Failed(실패) : 트랜잭션 실행에 오류가 발생하여 중단된 상태
-   Aborted(철회) : 트랜잭션이 비정상적으로 종료되어 Rollback 연산을 수행한 상태
-   Partially Committed(부분 완료) : 트랜잭션의 마지막 연산까지 실행했지만, Commit 연산이 실행되기 직전의 상태
-   Committed(완료) : 트랜잭션이 성공적으로 종료되어 Commit 연산을 실행한 후의 상태

> 트랜잭션 관리를 위한 DBMS의 전략

**1. DBMS 의 구조**

데이터베이스 시스템은 보통 **비휘발성 저장 장치인 디스크에 데이터를 저장**하며 전체 데이터베이스의 **일부분을 메인 메모리에 유지**합니다. DBMS는 **데이터를 고정 길이의 페이지(page)로 저장**하며, 디스크에서 데이터를 읽거나 쓸 때에 페이지 단위로 입출력이 이루어집니다.

**2. 페이지 버퍼 관리자, 버퍼관리자**

**메인 메모리에서 유지하는 페이지들을 관리하는 모듈**을 보통 Page Buffer 관리자 또는 버퍼 관리자라고 부릅니다.

Buffer 관리 정책에 따라, **UNDO 복구와 REDO 복구가 요구**되거나 그렇지 않게 되므로, 트랜잭션 관리에 매우 중요한 결정을 가져옵니다.

**3. UNDO 가 필요한 이유**

작업을 수행하는 중에 **수정된 페이지들이 버퍼 관리자의 버퍼 교체 알고리즘에 따라서 디스크에 출력**될 수 있습니다. 버퍼 교체는 트랜잭션과는 무관하게 전적으로 버퍼의 상태에 따라서 결정됩니다.

즉 아직 완료되지 않은 트랜잭션이 수정한 페이지들도 디스크에 출력될 수 있으므로, 만약 해당 **트랜잭션이 어떤 이유든 정상적으로 종료될 수 없게 되면 트랜잭션이 변경한 페이지들은 원상 복구** 되어야 합니다. 이러한 복구를 UNDO라고 합니다.

수정된 페이지를 디스크에 쓰는 시점으로 2개의 정책을 분류해볼 수 있습니다.

-   **STEAL** : 수정된 페이지를 **언제든지 디스크에 쓸 수 있는** 정책, 대부분의 DBMS가 채택하는 정책, UNDO logging과 복구를 필요로 함.
-   **¬STEAL** : 수정된 페이지들을 **최소한 트랜잭션 종료시점까지는 버퍼에 유지**하는 정책, UNDO 작업이 필요하지 않지만 매우 큰 메모리 버퍼가 필요함.

**4. REDO 가 필요한 이유**

**이미 commit한 트랜잭션의 수정을 재반영하는 복구작업**을 말합니다.

UNDO 복구와 마찬가지로 버퍼 관리 정책에 영향을 받습니다.

트랜잭션이 종료되는 시점에 해당 트랜잭션이 수정한 페이지를 디스크에 쓸 것인가 아닌가를 기준으로 2개의 정책을 분류합니다.

-   **FORCE**: 수정했던 모든 페이지를 트랜잭션 커밋 시점에 디스크에 반영하는 정책, 트랜잭션이 커밋 되었을 때 수정된 페이지들이 디스크에 반영되므로 **REDO가 필요없습니다**.
-   **¬FORCE**: 수정했던 페이지를 트랜잭션 커밋 시점에 디스크에 반영하지 않는 정책, 트랜잭션이 DB에 반영되지 않을 수 있기 때문에 **REDO 작업이 필요**합니다. 대부분의 DBMS가 채택하는 정책입니다.