###유의사항

#AOP 적용을 위해 Application 위에 @EnableAspectJAutoProxy 어노테이션 부착

```java
@EnableAspectJAutoProxy
@SpringBootApplication
public class HelloSpringApplication {

	public static void main(String[] args) {
		SpringApplication.run(HelloSpringApplication.class, args);
	}

}

```

###AOP 부착법 2가지

1. #AOP 클래스에 @Component 부착법

```java
@Aspect
@Component
public class TimeTraceAop {
    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());
        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString()+ " " + timeMs +
                    "ms");
        }
    }
```

2. #Spring Config에서 @Bean으로 컨테이너 등록
> AOP @Around시에 Config에 있는 TimeTraceAop도 check하는데 여기서 TimeTraceAop (자기자신) 을 생성하기 때문에 순환참조오류가 난다.
> 그래서 execution에서 해당 file은 aop 제거

```java
@Configuration
public class SpringConfig {
    @Bean
    public TimeTraceAop timeTraceAop() {
        return new TimeTraceAop();
    }
}

```
```java
@Aspect
public class TimeTraceAop {
    @Around("execution(* hello.hellospring..*(..)) && !target(hello.hellospring.SpringConfig)")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());
        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString()+ " " + timeMs +
                    "ms");
        }
    }
}
```
