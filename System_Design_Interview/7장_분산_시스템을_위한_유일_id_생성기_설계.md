# 분산 시스템을 위한 유일 ID 생성기 설계

- 분산 환경에서 `auto_increment`가 설정된 RDS의 primary key를 사용하는 것은 어렵다
  - 데이터베이스 서버 한 대로는 분산 환경에서의 요구를 감당하기 어렵다
  - 여러 데이터베이스 서버를 사용하면 지연 시간(delay)를 낮추기 어렵다

## TL;DR

- 분산 환경에서의 RDS `auto_increment`는 확장 지연 이슈 등으로 부적합하다.
- UUID 생성기 종류
  - 다중 마스터 복제: 규모 확장 가능, 유일성 보장, 다중 데이터센터에서의 규모 확장 어려움
  - UUID: 간단하고 독립적, 생성되는 ID 길이가 길고 문자 포함 가능
  - 티켓 서버: 간단하고 숫자 ID 사용, SPOF 발생 가능
  - 스노플레이크: 시간 정렬 및 수평 확장 가능
- 추가적으로 시계 동기화(NTP), 각 절 길이 최적화, 고가용성 구성 등을 고려할 수 있다.

## 다중 마스터 복제(multi-master replication)

- 데이터베이스의 `auto_increment` 기능 활용하되, 다음 ID 값은 기존 ID 값에서 k만큼 증가한다
  - k: 현재 사용 중인 데이터베이스 서버 수
- 장점
  - 데이터센터 내 규모 확장: 데이터베이스 수 늘리면 초당 생산 가능 ID 수를 늘릴 수 있다
- 단점
  - 여러 데이터센터에 걸쳐 규모 확장은 어렵다: 서버 추가/삭제 시 제대로 동작하지 않을 수도 있다
  - ID의 유일성은 보장되는 반면, 시간 흐름에 맞춰 커지도록 보장하기 어렵다

## UUID(Universally Unique Identifier)

- 컴퓨터 시스템에 저장된 정보를 식별하기 위한 128비트짜리 수
  - ex. 123e4567-e89b-12d3-a456-426614174000
- UUID 값은 유일성을 가지므로 충돌 가능성이 낮다
- 장점
  - UUID 생성 과정은 단순하다
  - 각 서버 사이에서 독립적으로 ID를 생성하므로 동기화 이슈가 없고, 규모 확장이 쉽다.
- 단점
  - ID가 128비트로 길며, 시간 순으로 정렬하기 어렵다
  - ID가 숫자(numeric)가 아닌 값이 포함될 수 있다

## 티켓 서버(ticket server)

- 티켓 서버(`auto_increment`가 설정된 데이터베이스 서버)를 중앙 집중형으로 하나만 사용한다
- 장점
  - 유일성이 보장되고 ID는 오직 숫자로만 구성되어 있다
  - 쉽게 생성할 수 있어 중소 규모 애플리케이션에 적합하다
- 단점
  - 티켓 서버를 하나만 사용하므로 SPOF(Single Point Of Failure)가 된다.
  - 해당 서버에 장애가 발생하면 티켓 서버를 사용하는 다른 시스템이 영향을 받을 수 있다
  - 티켓 서버를 여러 대 사용하면, 데이터 동기화 같은 문제가 발생할 수 있다

## 트위터 스노플레이크 (snowflake) 접근법

- ID를 바로 생성하지 않고, 생성해야 하는 ID를 여러 절(section)으로 분할하여 생성한다
- 분산 환경에서 규모 확장이 가능하다
- ex. 64비트 ID 생성기 기준

| 절                         | 설명                                                                                       | 할당된 비트(bit) |
| -------------------------- | ------------------------------------------------------------------------------------------ | ---------------- |
| 사인(sign) 비트            | 음수/양수 구분                                                                             | 1                |
| 타임스탬프(timestamp) 비트 | 기원 시각(epoch) 이후로 몇 밀리초(millisecond) 경과했는지 나타내는 값                      | 41               |
| 데이터센터 ID              | 시스템 시작할 때 결정, 시스템 운영 중에는 바뀌지 않음<br>2^5 = 32개의 데이터센터 지원 가능 | 5                |
| 서버 ID                    | 시스템 시작할 때 결정, 시스템 운영 중에는 바뀌지 않음<br>데이터센터 당 32개 서버 사용      | 5                |
| 일련번호                   | 각 서버에서 ID 생성할 때마다 일련번호 1만큼 증가<br>1밀리초 경과할 때마다 0으로 초기화     | 12               |

- 타임스탬프: 시간 값을 가지므로 시간 순으로 정렬 가능하다
  - 기원 시각을 현재에 가깝게 맞추어 오버플로우(overflow) 발생 시점을 늦추도록 설계할 수 있다

### 유일 ID 설계 이후로 생각해볼 수 있는 주제

- 시계 동기화(clock synchronization)
  - 지금까지의 설계는 'ID 생성 서버가 전부 같은 시계를 사용한다'고 가정하고 진행
  - 하나의 서버가 여러 코어에서 실행될 경우, 서버가 물리적으로 독립된 여러 서버에서 실행할 경우 추가적으로 고려 사항이 발생: NTP(Network Time Protocol) 사용 고려
- 각 절(section) 길이 최적화
  - 동시성(concurrency)이 낮고 수명이 길은 애플리케이션의 경우, 일련번호 절 길이를 줄이고 타임스탬프 절의 길이를 늘리는 것이 효과적을 수도 있다
- 고가용성(high availability): ID 생성기는 필수 불가결(mission critical)이므로 높은 가용성이 필요하다

## 참고문헌

- [Universally unique identifier](https://en.wikipedia.org/wiki/Universally_unique_identifier)
- [Ticket Servers: Distributed Unique Primary Keys on the Cheap](https://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)
- [Announcing Snowflake](https://blog.x.com/engineering/en_us/a/2010/announcing-snowflake)
- [Network Time Protocol](https://en.wikipedia.org/wiki/Network_Time_Protocol)