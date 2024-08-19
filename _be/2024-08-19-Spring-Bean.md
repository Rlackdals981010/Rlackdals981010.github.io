---
title: Spring - Bean
layout: page
permalink: /be/spring/bean
---

<details>
<summary>Bean 수동 등록</summary>
<div markdown="1">

---

**Bean 수동 등록**
보통 @Component 사용시 @ComponentScan에 의해 해당 클래스는 자동으로 Bean으로 등록된다.
프로젝트의 규모가 커질 수록, 비즈니스 로직과 관련된 클래스가 많아질수록 이 방법이 개발 생산성에 유리해서 이게 일반적인 방법임

하지만, 기술적 문제나 공통적 관심사를 처리할 때는 수동으로 등록하는게 좋음
ex) 공통 로그처리와 같은 비즈니스 로직 지원 기능

비즈니스 Bean보다는 적어서 수동으로 할만하고, 수동 등록 Bean은 위치 파악에 유리하다는 장점이 있음

---

**수동 등록 방법**

{% highlight ruby %}
@Configuration
public class PasswordConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
{% endhighlight %}
위와 같이 Bean으로 등록하고 싶은 메소드에 직접 @Bean 어노테이션을 등록할 수 있다.
구체적으로는 해당 메소드가 반환하는 BCryptPasswordEncoder() 객체가 passwordEncoder라는 이름의 빈으로 등록된것.
passwordEncoder가 Bean으로 등록되어서 다음과 같이 사용이 가능해졌다.

{% highlight ruby %}
@Autowired
PasswordEncoder passwordEncoder;
~~
String encodePassword = passwordEncoder.encode(password);
{% endhighlight %}
그래서 BCryptPasswordEncoder의 내장 메소드인 encode를 위처럼 사용이 가능해진것이다.

---

</div>
</details>

<details>
<summary>동일 타입 Bean이 2개</summary>
<div markdown="1">

---
인터페이스를 사용하면 동일할 메소드 명을 갖는 2개의 클래스가 생성된다.

{% highlight ruby %}
public interface Food {
    void eat();
}

public class Chicken implements Food{~}
public class Pizza implements Food {~}
{% endhighlight %}

Food의 구현체인 Chicken과 Pizza를 @Component를 이용해서 Bean으로 등록했다고 가정하면

{% highlight ruby %}
@Autowired
Food food;
{% endhighlight %}

상황에서 어떤 Bean 객체를 주입해야할지 Spring에서 처리할 수 없다. 

이 경우 개발자가 직접 주입할 객체를 알려줘야 하는데 다음과 같다.

**등록된 Bean 이름 명시하기**

{% highlight ruby %}
@Autowired
Food pizza;

@Autowired
Food chicken;
{% endhighlight %}
그냥 이렇게 직접 Bean 이름을 명시하면 된다.

**@Primary 사용하기**

{% highlight ruby %}
@Component
@Primary
public class Chicken implements Food {
   @Override
   public void eat() {
      System.out.println("치킨을 먹습니다.");
      }
   }
{% endhighlight %}
위 처럼 @Primary를 이용해서 해당 Bean을 우선으로 주입시킬 수 있다.

**@Qualifer 사용하기**

{% highlight ruby %}
@Component
@Qualifier("pizza")
public class Pizza implements Food {
   @Override
   public void eat() {
      System.out.println("피자를 먹습니다."); 
   } 
}
~~
@Autowired
@Qualifier("pizza")
Food food;

{% endhighlight %}
위처럼 @Qualifier("pizza")를 이용해서 주입시킬 수 있다.

이때 Primary와 Qualifer가 동시에 존재한다면, Qualifer가 우선순위가 더 높다.
Primary는 전역에서 유효하고, Qualifer는 명시한 곳에서만 효과가 있다. 즉, Qualifer의 범위가 더 좁다고 할 수 있는데,
Spring에서는 범위가 좁은것이 대부분 우선순위가 높다.

따라서 범용적으로 사용되는 객체에는 Primary를 설정해두고, 특정지역에서만 사용할 Bean에 Qualifier를 사용하면 되겠다.
</div>
</details>
