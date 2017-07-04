GoF에 의해 정의된 디자인 패턴 32가지 중 하나이자 가장 많이 쓰이면서 편리하지만, 동시에 여러 단점들을 가지고 있는 소프트웨어 디자인 패턴이다. [1]

> 이 싱글톤이라는 용어의 유래는 수학용어 싱글톤에 있다.
>
> 우리말로는 한원소(집합) 이라고 하는데 하나의 원소만을 갖는 집합을 말할 때 쓰는 용어이다. 예를 들면 {0} 는 한원소 집합(singleton set)이다.

위의 유래를 보면 감이 오겠지만 이 패턴을 써야하는 상황은 아래와 같다.

- 시스템 전체에서 해당 객체가 1개만 존재해야 할 때
- 객체가 1개만 존재해야 시스템의 동작이 효율적일 때
- 전역 변수와 같은 개념으로 클래스의 값이 공유되어 사용해야 할 때

위의 상황에서 이 패턴을 사용하면 유용하지만,

이 패턴이 딱히 이득이 없을때 남용되거나 1개의 객체만을 사용하게하는 제약이 필요없는 상황에서 사용된다면 안티 패턴이 될 수도 있다.

 다른 디자인패턴(추상 팩토리, 빌더, 프로토타입, 퍼사드)에서 싱글톤이 사용되기도 한다.

 

그렇다면 구현은 어떻게 해야할까?

1. 위의 설명과 같이 싱글톤 패턴은 해당 클래스의 객체가 영원히 1개만 존재해야 하고,
2. 전역 접근이 가능해야 한다.

 그러기 위해서는

1. 클래스의 생성자들이 private여야 한다 (외부에서 객체 생성이 불가능하도록)
2. static 매소드를 제공해야 한다 (객체 없이 데이터에 접근 할 수 있도록)
3. 객체는 private static 인 변수여야 한다

 이렇게 구현하면 객체는 static 매서드가 처음 호출되기 전, 변수가 초기화 될 때 생성된다.

 

아래는 싱글톤의 예시이다.

```
public final class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

 여기까지가 기본 싱글톤 패턴에 대한 설명이다. 싱글톤만 이해하고자 한다면 여기서 멈춰도 좋다.

 그러나 몇가지 다른 형태의 싱글톤 패턴도 존재한다. 기본 싱글톤 패턴은 객체를 사용하기도 전에 미리 생성되어 자원을 차지하는 문제가 있다. 만약 이 객체의 크기가 상당 할 경우 효율적이지 않기 때문에 객체의 생성을 사용하기 직전까지 미루는 형태의 싱글톤이 있다.

그것을 lazy initialization 싱글톤 패턴이라고 하는데 아래에서 설명하였다.

------

**Lazy Initialization Singleton Pattern**

이 패턴은 클래스의 static 매소드가 처음 호출될 때 객체가 생성되는 싱글톤 패턴이다.

만약 그 호출된 static 매소드가 여러개의 쓰레드에서 동시에 호출되면 그 쓰레드에서 한번에 여러개의 객체가 생성되어버리는 문제가 발생 할 수 있다.

 

그래서 그 문제를 해결하는 수단이 필요한데, 해결하기 위한 선택지는 아래와 같다.

- 기본 싱글톤 패턴을 사용하라 (그러나 자원낭비가 있을 수 있다)
- getInstance()매소드를 동기화 하라 (그에따른 오버헤드가 존재할 수 있다)
- Double-checked locking 패턴을 사용하라 (메모리를 공유하면서 문제가 발생할 수 있다)

 

아래는 Double-checked locking이라는 디자인 패턴으로 그 문제를 해결한 예시이다.

```
public final class Singleton {
    private static volatile Singleton instance = null;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

 이런 lazy initialization형 싱글톤 패턴은 위에서 언급한 것과 같이 여러 문제가 있기 때문에 사용하는것을 지양하는게 좋다.

 싱글톤 패턴을 사용할 때는 이왕이면 enum 싱글톤 패턴을 따르는 것이 좋다[2]

```
public enum EnumInitialization {
    INSTANCE;
    static String test = "";
    public static EnumInitialization getInstance() {
        test = "test";
        return INSTANCE;
    }
}
```

간결하고, 직렬화가 자동으로 처리되며, 멀티 쓰레드로부터 안전하고 리플렉션으로 인한 객체 중복생성이 없기 때문이다. 

스프링과 사용하는 경우, 싱글톤이 멀티 쓰레드로 부터 안전하지 않기 때문에 bean 객체로 등록하고 사용하는 경우가 있는데, 이 상황에서도 싱글톤의 대체로 enum 싱글톤을 사용할 수 있다.

  

Reference

------

[1] https://en.wikipedia.org/wiki/Singleton_pattern

[2] Effective java 2/E - JOSHUA BLOCH



Description

------

**INSTANCE; 란?**

public static final EnumInitialization INSTANCE = new EnumInitialization(); 과 같다