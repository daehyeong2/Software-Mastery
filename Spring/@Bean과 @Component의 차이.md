`@Bean`은 메소드 레벨에서 선언하며, 반환되는 객체(인스턴스)를 개발자가 수동으로 빈으로 등록하는 애노테이션입니다.

반면 `@Component`는 클래스 레벨에서 선언함으로써 스프링이 런타임시에 컴포넌트스캔을 하여 자동으로 빈을 찾고(detect) 등록하는 애노테이션입니다

#### @Bean 사용 예제
```java
@Configuration  
public class AppConfig {  
   @Bean  
   public MemberService memberService() {  
      return new MemberServiceImpl();  
   }  
}
```

#### @Component 사용 예제
```java
@Component  
public class Utility {  
   // ...  
}
```


참고
https://youngjinmo.github.io/2021/06/bean-component/