# How to improve API performance?

<img width="404" alt="api-performance" src="https://github.com/user-attachments/assets/2d9a0186-990b-437f-8221-e1c828fea2ce" />

## **API 성능을 개선하기 위한 5가지 방법**

## **Pagination**

### **개념**

API 요청 시 대규모 데이터를 한 번에 전송하면 네트워크 부하가 크고 응답 시간이 길어집니다.

Pagination은 데이터를 일정한 크기의 페이지 단위로 나눠서 클라이언트에게 순차적으로 전달합니다.

Pagination을 통해서 클라이언트는 필요할 때마다 다음 페이지를 요청하고, 서버는 한 번에 처리하는 데이터 양이 줄어들어서 처리 효율이 상승하는 효과를 볼 수 있습니다.

### **장점**

- 성능
    - 데이터를 한 번에 반환하는 대신에 페이지 단위로 나누어서 서버의 부하를 줄여줍니다.
- 속도
    - 사용자가 필요한 부분만 요청이 가능해서 초기 응답 속도 향상에 도움을 줍니다.
- 안정성
    - 메모리 부족이나 타임아웃 문제를 방지할 수 있습니다.

### **구현 방법**

<img width="404" alt="스크린샷 2024-12-18 오후 10 55 43" src="https://github.com/user-attachments/assets/9ef9d52b-63ab-471a-8019-c960575795df" />

- 일반적으로 page, page_size 파라미터를 받아서 적절하게 쿼리한다.
- API 응답 구조
    - 현재 페이지, 총 페이지 수, 다음 페이지 링크 등을 응답 메타데이터로 제공하면 클라이언트 측 로직이 용이해집니다.
- 응답 메타데이터
    1. **페이징(paging)**: 데이터를 여러 페이지로 나눠서 제공할 때, 총 페이지 수, 현재 페이지 번호, 다음 페이지 링크 등을 포함합니다. 이를 통해 클라이언트는 더 많은 데이터를 요청하거나, 페이지를 탐색할 수 있습니다.
    2. **정렬/필터링 정보**: 데이터가 어떻게 정렬되었는지, 어떤 필터가 적용되었는지 등의 정보가 포함될 수 있습니다. 예를 들어, 요청된 데이터가 날짜별로 정렬되었는지 알 수 있습니다.
    3. **상태 코드 및 메시지**: 응답 상태 코드와 함께 오류 메시지나 상태 설명이 제공될 수 있습니다. 예를 들어, 요청이 성공했는지, 실패했는지, 추가적인 오류 정보 등을 알 수 있습니다.
    4. **총 데이터 개수**: 전체 데이터의 수를 알려줍니다. 예를 들어, 검색된 데이터가 100개이고, 클라이언트가 한 번에 10개씩 요청할 경우, “총 100개 중 10개가 반환됨”과 같은 정보가 제공됩니다.
    5. **인증 및 권한 관련 정보**: API가 인증된 사용자에게만 특정 데이터를 제공할 때, 현재 사용자의 권한 수준 등을 포함할 수 있습니다.

## **Async Logging**

### **개념**

- 일반적으로 API 서버는 각 요청에 대한 로그를 기록합니다. 하지만 로그 기록은 상대적으로 느린 디스크 쓰기를 포함하는 경우가 있어서 요청 처리 흐름을 지연시킵니다.
- Async logging은 요청 처리 로직과 로그 기록 로직을 분리해서 로그 기록 자체를 별도 thread나 비동기 큐를 통해 처리해서 메인 요청 thread가 기다리지 않고 빠르게 응답을 반환하는 방식입니다.

## **장점**

- 속도
    - 로그를 비동기로 처리해서 다른 요청 응답을 빠르게 할 수 있습니다.
- 성능
    - async logging은 동시에 많은 요청이 들어와도 즉각적으로 큐에 쌓이고 나중에 묶음으로 기록해서 서버 처리량이 증가합니다.
    - 더 쉽게 말하자면 디스크에 기록하는 시간을 최소화해서 I/O 작업을 최소화하는 방식입니다. 또한 로그를 기록하는 작업이 메인 요청 처리 흐름을 막지 않고 백그라운드에서 진행한다고 생각하시면 됩니다.
- 자원 효율
    - 별도의 logging thread, 버퍼를 사용해서 CPU나 I/O자원을 효율적으로 분배할 수 있습니다.
    - thread와 버퍼의 용도
        
        thread: 하나의 thread는 사용자의 요청을 처리하고, 다음 thread는 데이터를 백그라운드에서 처리합니다.
        
        buffer: 버퍼는 데이터를 일시적으로 저장하는 메모리 공간입니다. 그래서 데이터를 한 번에 여러 개를 읽어서 버퍼에 저장하고 나중에 한 번에 처리합니다.
        

### **구현 방법**

<img width="426" alt="스크린샷 2024-12-18 오후 11 01 19" src="https://github.com/user-attachments/assets/689a267c-8d51-492f-9641-5aca644fbc2d" />

- 버퍼 사용
    - 로그를 바로 디스크에 쓰는 대신 메모리 상의 버퍼나 큐에 적재한 뒤 일정 시간 간격 또는 일정량이 쌓이면 일괄적으로 디스크에 기록합니다.
- 로그 백엔드 선택
    - 비동기 로깅 지원 라이브러리나 프레임워크를 활용할 수 있습니다.

## **Caching**

### **개념**

API 요청에 자주 조회되는 데이터를 매번 DB에서 읽으면 느려지겠지요, 그래서 빈번히 요청되는 데이터를 메모리 저장소에 저장하고 요청 시 DB대신 Cache에서 바로 반환하는 방식입니다.

### **장점**

- 속도
    - 메모리 기반 캐시는 디스크 기반 DB보다 훨씬 빠른 읽기 속도를 가지고 있습니다.
- DB 부하 감소
    - DB 접근 횟수가 줄어들어서 DB서버의 부하가 감소합니다.
- 비용
    - DB 스케일링 비용을 절감합니다.

### **구현 방법**

<img width="419" alt="스크린샷 2024-12-18 오후 11 15 48" src="https://github.com/user-attachments/assets/d91c7f9b-bd94-4832-a291-4d9e93e91d8c" />

- TTL (Time-to-Live) 설정을 통해서 데이터를 최신으로 유지해서, 오래된 데이터를 지속적으로 제공하는 문제를 방지합니다.
- 캐시 우선순위: 자주 사용되는 데이터, 변경 빈도가 낮은 데이터부터 캐싱해서 효율을 높일 수 있습니다.
- 캐시 미스 처리: 캐시에 데이터가 없으면 DB를 조회해서 데이터를 받아온 뒤 캐시에 올리는 로직을 구현합니다.

## **Payload Compression**

### **개념**

API 응답 데이터, 즉 payload가 클라이언트와 서버 간 전송 시 크기가 크면 전송 시간이 길어지고 비용이 증가합니다. 그래서 각종 압축 기법을 사용해서 전송해야 할 데이터 양을 줄여 응답 속도를 높이는 방식입니다.

### **장점**

- 속도
    - 압축된 데이터는 크기가 많이 줄어들어 전송 시간이 감소합니다. 이러한 부분은 UX에도 도움을 줄 수 있겠지요.
- 비용
    - 클라우드 환경에서 트래픽 비용 절감에 도움을 줍니다.

### **구현 방법**

<img width="252" alt="스크린샷 2024-12-18 오후 11 22 37" src="https://github.com/user-attachments/assets/cdf95b29-8044-4f45-8790-136fbdb9483c" />

- 웹 서버나 API Gatewat에서 압축 설정을 지원합니다.
- 클라이언트 요청 헤더와 서버 응답 헤더를 통해 적합한 압축 방식을 결정합니다.
- 압축 알고리즘은 CPU를 사용하므로 CPU와 네트워크 trade-off를 고려해야 합니다. (일반적으로는 압축으로 얻는 이점이 더 크다고 합니다.)

## **Connection Pool**

### **개념**

API 서버가 DB, 외부 서비스와 연결할 때마다 TCP 커넥션을 새로 생성하고 닫는 과정은 비싸고 시간이 많이 걸립니다. 그래서 connection pool은 재사용 가능한 연결들을 미리 만들어두고, 요청 시 즉시 할당해서 커넥션의 생성/해제 overhead를 줄이는 방식입니다.

### **장점**

- 비용
    - 매 요청마다 새로운 connection을 열지 않아도 됩니다, 그래서 네트워크 인증 과정을 반복하지 않아서 속도가 빨라집니다.
- 리소스 활용
    - DB 또는 외부 서비스에 대한 요청 처리 효율이 상승하는데 이는 일정 수의 connection을 효율적으로 재사용하면서 얻은 이점입니다.
- 안정성
    - pool 안의 connection 상태를 모니터링하고 문제가 있는 연결은 제거해서 안정적인 서비스 제공이 가능합니다.

### **구현 방법**

<img width="255" alt="스크린샷 2024-12-18 오후 11 28 32" src="https://github.com/user-attachments/assets/92c57e25-2248-49d0-aef4-e336f26edc86" />

- pool 크기 설정
    - 트래픽 패턴과 시스템 자원을 기반으로 connection poll 크기를 조정합니다.
    - 너무 작으면 요청 대기 시간이 길어집니다.
    - 너무 크면 리소스 낭비가 일어납니다.
- pool 관리 전략 사용
    - Idle Timeout, Validation Query 등을 사용해서 오래되거나 비정상적인 connection을 정리합니다.