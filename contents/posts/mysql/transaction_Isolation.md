---
title: "트랜잭션이란?"
description: ""
date: "2022-12-13 17:00:00"
update: "2022-12-13"
tags:
  - RealMySQL.8.0
---


## 학습 동기

프로그램 내부에서 예외가 발생했을 때, 진행되고 있던 데이터 변경 사항들을 모두 성공시키거나 실패시켜야 할 필요성을 느꼈어요. 

면담 예약 서비스로 예를 든다면  면담을 예약하기 위해서는 1) 멘토의 시간도 사용중으로 바꿔야하고, 2)면담 요청 내용도 저장해야하고, 3)면담 테이블에 면담도 생성해야하는 3가지 DB 변경 작업이 필요해요. 세가지 중 하나라도 정상적으로 실행이 안되면 진행된 내역을 전부 취소 시켜서 프로그램 오류를 막는것이 안전한 서비스겠죠? 

## 트랜잭션

더이상 나눌 수 없는 작업 단위를 말합니다. **데이터의 정합성을 보장하기 위한 작업이기도 해요.**  위에서 면담을 생성하기 위해서 해야하는 3가지는 반드시 필요한 작업이라 하나의 트랜잭션에 묶이듯이요!  이 트랜잭션을 통해서 지키고 싶은 특징이 4가지가 있어요.  각 영단어에 앞 글자를 따서 ACID 원칙이라고도 불려요

### 원자성(Atomicity)

> 작업이 하나의 쿼리로 실행되더라도, 해당 쿼리로 바뀌는 데이터가 1개 이상이라면 그 데이터가 모두 정상적으로 동작했을 때만 변경 사항이 저장되는 특징을 말해요. 모두  성공하거나 모두 실패하거나 all or nothing 이라고도 표현합니다!
>

**1) Commit 연산**

- 트랜잭션에 대한 작업이 성공적으로 끝났을 때 수행
- 트랜잭션이 행한 갱신 연산이 완료된 것을 트랜잭션 관리자에게 알려주는 연산

**2) Rollback 연산**

- 트랜잭션 처리가 비정상적으로 종료되었을 때 수행
- 트랜잭션의 원자성을 구현하기 위해 트랜잭션이 행한 모든 연산은 취소하는 연산

### 일관성(Consistency)

> 트랜잭션이 성공한 후에도 일관성을 유지해야한다는 것을 의미합니다.
> 첫번째로는 트랜잭션이 커밋되면 데이터베이스에 설정한 제약사항을 위반하지 않는다는 뜻인데요. 만약 면담 예약 시간에 유니크 제약조건을 걸었는데, 동일한 시간이 두개가 
> 생기게 되면 일관성을 해치게 됩니다. 따라서 저장할 수 없어야하는거죠. 
> 두번째로는 어플리케이션 단에서 의도한대로 정상적으로 동작한다는 의미이기도 해요.  판매할 제품의 재고가 없다면, 판매하지 못하게 막아야한다 정도로 설명드릴 수 있을 것 
> 같습니다


 두번째로는 어플리케이션 단에서 의도한대로 정상적으로 동작한다는 의미이기도 해요.  판매할 제품의 재고가 없다면, 판매하지 못하게 막아야한다 정도로 설명드릴 수 있을 것 같습니다. 

### 독립성(Isolation)

> 트랜잭션이 실행되고 있을 때, 다른 트랜잭션의 연산 작업이 끼어들지 못하도록 독립적인 행위를 보장하는 것을 의미합니다. 독립성이 잘 지켜지지 않았을 때 동시성 이슈 문제도 
> 있고, 독립성이 너무 높으면 동시 처리량이 줄어들어 성능 저하가 발생하기도 해요.

### 지속성(Durability)

> 성공적으로 실행된 트랜잭션은 영구적으로 반영되야하는 것을 의미합니다. 만약에 디비 복제 작업이 계속 진행되다가 갑자기 디비가 올라가 있는 서버가 꺼져 시스템이 다운 됐다면? 다시 처음 부터해야할까요? 이런 문제를 방지하기 위해서 Mysql 에서는 [리두로그](https://sudal.site/undo/)라는 것을 사용하여 작업중에 내역을 지속적으로 백업해두기도 합니다!
> 

## ACID 원칙이 무조건 지켜질까?

이 원칙을 지키려면 DB 에 접근하는 작업은 무조건 하나일 수밖에 없고 그러면 동시성이 떨어져 성능 저하가 심각하게 발생하게 될 것 같아요. 그렇기 때문에 DB 엔진은 이 원칙을 희생해서 트랜잭션 격리 수준으로 동시성을 조금이나마 얻을 수 있는 방법을 제공합니다. 

## 트랜잭션 격리수준(Isolation level)

트랜잭션을 실행하는 중에 다른 트랜잭션의 영향을 받지 않는 독립성을 조금은 덜 지키더라도 동시성을 챙기기 위한 것인데요. ANSI/ISO SQL 표준으로 4개를 정의하고 있습니다.

### READ UNCOMMITTED - 커밋되지 않은 읽기

트랜잭션이 두개 있다고 가정하면, 첫번째 트랜잭션에서 데이터 변경을 진행중일 떄, 그 데이터를 가지고 다른 트랜잭션에서 읽어와 작업을 할 수 있다는 것을 의미하는데요. 

- A-트랜잭션 작업중
- B-트랜잭션 A 작업 내역 읽어와서 작업시작
- A-트랜잭션 문제 생겨 `롤백`
- B-트랜잭션은 성공되어 저장

만약에 리드 언 커밋 상태일 때 면담 신청을 한다고 가정해보면, 누군가 면담을 신청해서 1)시간이 사용중으로 변하고 2)면담 내역 저장 작업을 하다가 다른 누군가가 같은 내용으로 면담을 신청했다고 합시다! 그러면 그 시간을 읽어올테고, 시간이 사용중이기 때문에 면담을 신청하지 못한다는 예외를 던지겠죠? 

그런데 이때 첫번째 면담 신청 내역에 문제가 있어서 첫번째 내역이 전체 롤백이 되었어요. 그러면 두번째 면담 신청은 사실 가능한 상황인데 못하게 되는 문제가 발생합니다 ㅜㅜ 

이렇게 커밋되지 않은 데이터를 볼 수 있는 현상을 **더티 리드(dirty read)** 라고 합니다. 이렇게 되면 데이터 정합성이 맞지 않을 수 있고, 프로그램에 많은 혼란을 야기하기 떄문에 권장되지 않는 격리 레벨입니다.

### READ COMMITTED - 커밋된 읽기

커밋된 내역만 다른 트랜잭션에 보여지는 것을 의미합니다.  작업중에 해당 데이터가 필요하다면 레코드를 읽는게 아니라 언두로그에 백업된 데이터를 읽어 일관된 읽기를 보장합니다.  작업이 끝나면 다시 레코드를 읽을 수 있습니다.

이 읽기도 문제가 하나 있는데요.  아래 상황을 통해 설명하도록 하겠습니다

- A- 트랜잭션  2:00시에 면담 생성 작업 시작
- B- 트랜잭션 2시에 면담 잡힌거 조회
    select * from interview where interview_time = 2:00; - 언두로그 읽어서 5개 조회
    
- A- 트랜잭션  면담 생성 완료
- B- 트랜잭션 2시에 면담 잡힌거 조회 → 6개 조회

이렇게 한 트랜잭션에서  다른 결과를 읽어드리는 것을  **NON-REPEATABLE READ 라고 합니다.**  하나의 트랜잭션내에서 **동일한 SELECT 쿼리를 실행했을 때 항상 같은 결과를 보장해야 한다는 정합성에 어긋나는 상황입니다. (단일 상황으로 가정하고 이해하시면 좋을 것 같습니다.)**

### REPEATABLE READ - 반복 읽기

MySQL 의 InnoDB 스토리지 엔진에서 기본으로 사용되는 격리 수준입니다. DDL로 데이터가 변경되기 전에 이전 데이터를 언두로그에 백업해두고 실제 레코드 값을 변경합니다. 이러한 변경 방식을 MVCC 라고 합니다. 

> **MVCC(Multi Version Concurrency Control) - 잠금을 사용하지 않는 일관된 읽기를 제공**
> **MVCC**는 다중 버전 병행수행 제어의 약자로 DBMS에서는 쓰기(Write) 세션이 읽기(Read) 세션을 블로킹하지 않고, 읽기 세션이 쓰기 세션을 블로킹하지 않게 서로 
> 다른 세션이 동일한 데이터에 접근했을 때 각 세션마다 스냅샷 이미지를 보장**해주는 메커니즘이는 RDBMS에서 **동시성을 높이기 위해 등장**한 기술입니다. MySQL 에서는 > 언두로그가 MVCC 인거죠! 


### 일관된 읽기와 반복 읽기의 차이는 뭘까?

언두 영역에 백업된 레코드의 여러 버전 가운데 몇 번째 이전 버전까지 찾아 들어가야하느냐에 차이가 있어요. 모든 트랜잭션은 고유한 트랜잭션 번호와 순서를 가지고 있고, 언두 영역에 백업된 모든 레코드에는 변경을 발생시킨 트랜잭션의 번호가 포함되어 있습니다.  따라서 일관된 읽기는 트랜잭션이 사용한 데이터가 사용중일 때는 언두를 읽지만, 사용이 완료되면 레코드를 읽게 되고, 반복 읽기에 경우는 한번 언두로그를 읽었으면 해당 트랜잭션이 종료될 때까지 그 언두로그에 있는 데이터를 읽게 됩니다. 그래서 서비스가 종료될 때 까지 같은 데이터를 읽을 수 있게 보장하는거죠! 하나의 레코드에는 여러개의 언두(백업)가 존재할 수 있어요.

여기서 문제는 언두로그에는 락을 걸 수 없다는 것인데요. 만약에 Select for update(조회 후 수정) 를 위해서 데이터 락을 걸고 싶다면 언두가 아닌 레코드를 읽어내야합니다. 레코드는 다른 트랜잭션도 변경할 수 있기 때문에 일관된 읽기를 보장할 수 없게되는거죠.

그래서 조건절로 점수가 80점 이상인 사람을 조회하고, 그 사람의 레벨을 2로 올리고 싶을 때 조회와 변경이 함께 일어날 거니까 그 데이터를 락 걸고 조회하게 된다면? 그 전에 누군가 점수가 80점 인 사람에 데이터를 추가하는 등 변경이 일어나면 조회 갯수가 3개에서 4개로 바뀐다던지, 같은 내용의 데이터를 읽을 수 없게 되는 거죠. 이런 현상을 PHANTOM READ(팬텀 리드)라고 합니다. (다중 결과를 조회하는 상황이라고 이해하시면 좋을 것 같습니다)

mysql innoDB 에서는 데이터를 변경할 때 넥스트 키락을 걸어 조회된 데이터 중간에 어떤 삽입이 일어나는 것을 방지하기 때문에 팬텀리드가 발생하지 않습니다! 

## SERIALIZABLE - 읽기 불가능 (직렬화)

가장 엄격한 격리 수준입니다.  InnoDB 엔진에서는 기본적으로 순수한 작업은 아무런 레코드 잠금도 설정하지 않고 실행되는데요. 해당 수준으로 격리 레벨을 올리면 읽기 중에도 다른 트랜잭션은 그 데이터에 접근할 수 없게 됩니다.  mysql 을 사용한다면 REPEATABLE READ 만으로도 충분히 안정성 있는 프로그램이 운영이 가능하기 때문에 가급적이면 SERIALIZABLE 은 지양하는 것이 좋을것 같다고 생각됩니다. 

## 느낀점

프로젝트를 하면서 동시성 이슈가 발생해서 무턱대고 SERIALIZABLE 로 격리 레벨을 올리며 해결하려고 했던 적도 있었어요. 지금 생각해보면 엄청난 성능 저하가 발생 했을 것 같아요 ㅋㅋ 동시에 데이터를 처리하기 위한 여러 고민들을 하다보니 트랜잭션을 공부하게 되었는데, 안정성있는 프로그램을 운영하는 개발자가 되기 위해서는 필수적으로 알아야하는 내용이라고 생각되면서 이 마음을 가지고 다음에 할 프로젝트에서 트랜잭션에 대해 깊이있게 고민하고 작업해보면 좋겠다고 생각되었습니다. 

## 참고

- Real MySQL 8.0  5장