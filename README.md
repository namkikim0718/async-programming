# 프로젝트 목적
- `Spring`을 활용해 비동기 프로그래밍을 구현하고, 올바르게 동작하는지 확인<br/><br/><br/>

# 비동기 프로그래밍이란?
- `Main Thread`에서 `Task`를 처리하는 것이 아닌, Sub Thread에게 Task를 위임하여 처리하는 행위
> 그렇다면 `Java`에서 `Sub Thread`를 어떻게 생성하고 어떻게 관리할까?

<br/><br/><br/>

# Java의 ThreadPool
- 자바에서는 `ThreadPool`을 생성하여 `Async` 작업을 처리

<br/><br/><br/>

### `ThreadPool`의 장점  
  
  - 자원 효율성
    - 쓰레드 풀은 미리 정해진 개수의 스레드를 생성하여 관리하기 때문에, 쓰레드 생성 및 삭제에 따른 오버헤드를 줄일 수 있음
    - 이는 시스템의 자원을 효율적으로 관리할 수 있고, 불필요한 자원 소모를 방지
  
  - 응답성 및 처리량 향상
    - 쓰레드 풀은 작업을 대기 상태로 유지하여 작업 처리 속도를 향상
    - 즉, 작업이 발생하면 대기 중인 쓰레드 중 하나를 선택하여 작업을 할당하므로, 작업 처리를 병렬로 진행 가능
      
  - 작업 제어
    - 쓰레드 풀을 사용하면 동시에 처리할 수 있는 작업의 개수를 제한 가능
    - 즉, 쓰레드 풀의 크기를 조절하여 시스템의 부하를 조절하고, 과도한 작업 요청으로 인한 성능 저하를 방지
      
  - 쓰레드 관리
    - 쓰레드 풀을 사용하면 쓰레드의 생명 주기를 관리 가능
    - 쓰레드 풀은 스레드의 생성, 재사용, 종료 등을 관리하므로, 스레드의 안전한 운영을 도와줌

<br/><br/><br/>
### ThreadPool 생성 옵션
1. `CorePoolSize` (최소 쓰레드 개수)
2. `MaxPoolSize` (최대 쓰레드 개수)
3. `WorkQueue` (CorePoolSize보다 쓰레드가 많아진 경우, 해당 큐에 담아둠)
4. `KeepAliveTime` (CorePoolSize보다 쓰레드가 많아져 maximumPoolSize까지 쓰레드가 생성됐을 때 keepAliveTime시간만큼 유지하고 다시 corePoolSize로 돌아움)

- ThreadPool 생성 시 주의해야하는 부분  
  <img width="494" alt="스크린샷 2024-04-24 16 24 03" src="https://github.com/namkikim0718/async-programming/assets/113903598/e4f7111f-c091-4dce-bcb6-ef4b6f0d3a71">
<br/><br/><br/>
# 테스트 내용
- 총 3가지 상황에서 비동기적으로 동작을 잘 하는지 확인
  1. 의존관계 주입을 통해 실행  
     <img width="533" alt="스크린샷 2024-04-24 16 28 20" src="https://github.com/namkikim0718/async-programming/assets/113903598/e7ba0a08-96b4-4863-9677-94b90fca2cb9">
  2. 인스턴스를 직접 생성해 실행  
     <img width="463" alt="스크린샷 2024-04-24 16 28 30" src="https://github.com/namkikim0718/async-programming/assets/113903598/b810fbde-ecac-4b88-bf1e-a024c2cc9483">
  3. 메서드를 생성해 실행  
     <img width="351" alt="스크린샷 2024-04-24 16 28 39" src="https://github.com/namkikim0718/async-programming/assets/113903598/7c7bd941-ad0a-4558-9e71-eab242fbf5ee">

위의 스크린샷은 각각 `Thread.currentThread().getName()`을 출력한 결과  
1번 상황에서는 각각 다른 쓰레드를 사용한 것이 확인되었지만, 2, 3번 상황에서는 그렇지 않으므로 의존관계 주입을 통해서만 비동기적으로 올바르게 동작한다는 사실을 알 수 있다.

<br/>

### 의존관계 주입을 통해서만 비동기적으로 동작하는 이유
- `@Async`는 Spring AOP에 의해 프록시 방식으로 동작
- `Spring context`에 등록되어 있는 `Async Bean`이 호출되면 Spring이 개입하여 해당 빈을 프록시 객체로 `Wrapping`  
  <img width="434" alt="스크린샷 2024-04-24 16 43 10" src="https://github.com/namkikim0718/async-programming/assets/113903598/d1ba4860-77f3-42c3-b7c3-0342c5c48eea">

- 2, 3번 상황에서는 Spring이 `Wrapping`한 객체를 호출하는 것이 아니고, 직접 호출하므로 비동기적으로 동작하지 않음  
  <img width="270" alt="스크린샷 2024-04-24 16 44 29" src="https://github.com/namkikim0718/async-programming/assets/113903598/581d1cd2-e902-4c08-a46e-6f402063a5e8">
