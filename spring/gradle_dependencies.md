# api vs implementation
## Gradle 의존성 관리
의존성 전파(Transitive Dependency)
- 내 라이브러리가 내부적으로 사용하는 다른 라이브러리(A)를, 내 라이브러리의 사용자(B)에게 얼마나, 어떻게 노출할지 결정
- 명확한 가이드 제공 -> 사용자가 직접 필요한 의존성을 추가하도록 안내
- starter 패키지 제공 -> 관련 의존성을 모두 포함하는 starter를 만들어 사용자가 편리하게 사용할 수 있도록 함

### api vs implementation 선택 가이드
내가 만든 라이브러리의 Public API(메서드 반환 타입, 파라미터 등)에 내부 라이브러리의 타입이 노출된다면,  
Gradle 의존성 선언 시 `api`로 지정해야 함
- 예: `return new LibraryA.Data` -> LibraryA 타입이 Public API
- `api`로 지정 시, 내 라이브러리를 사용하는 사용자(B)도 LibraryA에 접근 가능
- 만약 `implementation`으로 지정하면, 사용자(B)가 LibraryA 타입을 인식하지 못해 컴파일 오류 발생

현재 구현 중인 batch-common 라이브러리에서,
```java
public JobParameters launchJob(String jobName, Map<String, Object> params) {
    // ...
}
```
- `BatchJobLauncher` 클래스의 `launchJob` 메서드가 `JobParameters` 타입을 반환
- `JobParameters` 타입이 Spring Batch 라이브러리의 Public API에 해당
- 따라서, batch-common 라이브러리의 `build.gradle` 파일에서 Spring Batch 의존성을 `api`로 선언해야 함

