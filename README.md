# Kotlin-Coroutines-_DeepDive


# 📌 1부 코틀린 코루틴 이해하기

<details>
<summary><strong>1. 코틀린 코루틴을 배워야 하는 이유</strong></summary>
  
### **왜 코틀린 코루틴을 배워야 하는가?**

- **기존 방식 한계**: RxJava나 콜백 기반의 비동기 처리 방식은 복잡하고, 중단(resume)이 불가능하거나 가독성이 낮음.
- **코루틴의 장점**:
    - 코드가 순차적으로 보이지만 실제로는 비동기로 작동함.
    - suspend 키워드 하나로 중단/재개 가능.
    - 백그라운드 작업 중에도 UI 스레드를 차단하지 않음.
    - 기존 구조 대부분 유지 가능 (Java/Rx 대비 적은 수정)
    - async/await, launch, scope 등을 통해 직관적인 병렬 처리 구현 가능.
- **다양한 플랫폼 지원**: Android, iOS, Backend 등 모든 플랫폼에서 활용 가능.

### **기존 방식들과의 비교**

1. **콜백 기반**
    - 중첩 콜백 구조 (Callback Hell)
    - 예외 처리 어려움, 가독성 및 유지보수 악화
    - 코드 예: getConfig -> getNews -> getUser -> showNews
2. **RxJava 기반**
    - subscribeOn, observeOn, map, zip, flatMap 등 다양한 연산자 사용
    - 비동기 처리 및 스트림 처리에 유리하나 러닝커브 높고 코드 복잡
    - 리소스 정리용 Disposable 관리 필요
3. **스레드 기반**
    - 직접 스레드 생성 및 UI 전환 필요
    - 스레드 생성 비용 높고 메모리 누수 위험

### **코루틴의 실제 적용**

- **Android 예제**:
    - viewModelScope.launch를 통한 UI와 분리된 백그라운드 작업 처리
    - async/await를 이용한 병렬 호출 (getConfig, getNews, getUser 동시 호출)
    - scope.launch로 중단지점 이후 재개 가능

```kotlin
viewModelScope.launch {
    val config = async { getConfigFromApi() }
    val news = async { getNewsFromApi(config.await()) }
    val user = async { getUserFromApi() }
    view.showNews(user.await(), news.await())
}
```

- **컬렉션 처리 예시**:
    - 페이지별 API 호출을 병렬/순차로 처리 가능

### **백엔드에서의 활용**

- **효율성**: 스레드를 명시적으로 생성하지 않고 동시성 처리가 가능
- **비용 절감**: 수만 개의 코루틴도 수백 MB 내에서 관리 가능
- **예시**: 10만 명 사용자 요청을 처리할 때, 스레드 방식은 OOM 발생 가능하지만 코루틴은 안전함

```kotlin
fun main() = runBlocking {
    repeat(100_000) {
        launch {
            delay(1000L)
            print(".")
        }
    }
}
```

### **요약 정리**

- 코루틴은 **단순한 비동기 처리를 넘어**, **최신 동시성 프로그래밍을 구현하기 위한 핵심 도구**임.
- 복잡한 Rx/Callback 구조보다 **간결하고 직관적**이며 **범용성이 뛰어남**.
- 이후 장에서는 suspend, 중단점 복원, 예외 처리, 구조화된 동시성 등에 대해 다룰 예정.
</details>

<details>
<summary><strong>2. 시퀀스 빌더</strong></summary>
  
### **코루틴과 시퀀스 빌더의 관계**

- **JavaScript, Python**과 같이 일부 언어는 제한된 형태의 코루틴을 지원 (예: async/await, 제너레이터).
- 코틀린에서는 제너레이터 대신 시퀀스 빌더(sequence builder) 제공.
- sequence { ... } 블록 내부에서 yield()를 호출하여 값을 순차적으로 반환함.
- 이 방식은 **lazy evaluation(지연 계산)** 기반으로, 필요한 시점에만 값을 생성.

### **시퀀스 빌더의 장점**

- 최소한의 연산 수행
- 무한 시퀀스 구현 가능
- 메모리 효율적

### **시퀀스 동작 방식**

- yield() 호출 시 해당 위치에서 멈췄다가 다음 호출에서 이어서 실행됨.
- main() 함수는 시퀀스를 반복문으로 순회하거나 .iterator()로 값을 하나씩 꺼냄.
- **중단지점**을 기억하고 이어서 실행되기 때문에 효율적이고 직관적임.

```kotlin
val seq = sequence {
    yield(1)
    yield(2)
    yield(3)
}
```

```kotlin
val seq = sequence {
    println("Generating first")
    yield(1)
    println("Generating second")
    yield(2)
    println("Generating third")
    yield(3)
    println("Done")
}
```

```kotlin
for (num in seq) {
    println("The next number is $num")
}
```

### **Flow와의 비교**

- sequence는 **yield를 사용한 빌더** 방식.
- flow는 **suspend 함수로 구성**되며, 더 많은 코루틴 기능과 **중단점 재개**, 예외 처리, 병렬 처리 지원.
- flow는 네트워크 요청 같은 suspend 작업을 포함할 수 있음:

```kotlin
fun allUsersFlow(api: UserApi): Flow<User> = flow {
    var page = 0
    do {
        val users = api.takePage(page++)
        emitAll(users)
    } while (!users.isNullOrEmpty())
}
```

### **결론**

- 시퀀스 빌더는 가볍고 직관적인 반복/지연 계산 구현에 적합.
- **중단점 기반으로 lazy 처리**가 가능하여 복잡한 로직을 간결하게 구성할 수 있음.
- **suspend 함수가 필요한 경우에는 Flow 사용**을 고려해야 함.
- 다음 장에서는 **중단(suspension)** 이 실제로 어떻게 동작하는지 다룸.
</details>
