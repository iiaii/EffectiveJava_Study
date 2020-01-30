# #EffectiveJava/2장_객체생성과파괴/2빌더

## 2. 생성자에 매개변수가 많다면 빌더를 고려하라

정적 팩토리와 생성자에는 똑같은 제약이 하나 있다. 선택적 매개변수가 많을때 적절히 대응하기 어렵다는 점이다. 식품포장의 영양정보를 표현하는 클래스를 생각해보면 정보는 많지만 대부분의 값은 0으로 이루어 진다.

이런 클래스용 생성자 혹은 정적 팩토리를 구현할때 프로그래머 들은 점층적 생성자 패턴을 즐겨 사용했다. 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개까지 받는 생성자 .. 형태로 매개변수를 전부다 받는 생성자 까지 늘려가는 방식이다.

```java
public NutritionFacts(int servingSize, int servings) {
	this(servingSize, servings, 0);
}

public NutritionFacts(int servingSize, int servings, int calories) {
	this(servingSize, servings, calories, 0);
}
…
```

보통 이런 생성자는 사용자가 설정하고 싶지 않은 매개변수까지 포함하기 쉽다.
-> 점층적 생성자 패턴도 쓸수는 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다. 

두번째 대안인 자바빈즈 패턴을 보면 매개변수가 없는 생성자로 객체를 만든후 setter메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다. 점층적 생성자 패턴의 단점들이 보이지 않는다. 코드가 길어졌지만 인스턴스 만들기가 쉽고 읽기 쉬운 코드가 되었다.

하지만 자바빈즈 패턴에서는 객체 하나를 만들려면 메서드를 여러개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다. 점층적 생성자 패턴에서는 매개변수들이 유효한지를 생성자에서만 확인하면 일관성을 유지할 수 있었는데, 그 장치가 완전히 사라진 것이다. 일관성이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없으며 스레드 안정성을 얻으려면 프로그래머가 추가 작업을 해줘야만 한다.

이러한 단점을 완화하고자 생성이 끝난 객체를 수동으로 얼리고 얼리기 전에는 사용할 수 없도록 하기도 한다. (실전에서는 안쓰임)

세번째 대안은 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비한 빌더 패턴이다.

클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. 그런다음 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다. 마지막으로 매객변수가 없는 build 메서드를 호출해 필요한 (보통은 불변인) 객체를 얻는다. 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어 두는게 보통이다.

```java
/*
    Item 2 
    생성자에 매개변수가 많다면 빌더를 고려하라
*/
class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}	

// 클라이언트 코드(실행)
// NutritionFacts cocaCola = new NutritionsFacts.Builder(240,8).calories(100).sodium(35).carbohydrate(27).build();
```

빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다.
이런 방식을 메서드 호출이 흐르듯 연결된다는 뜻으로 플루언트 API 혹은 메서드 연쇄라 한다. 

클라이언트 코드는 쓰기 쉽고 읽기 쉽다. 빌더 패턴은 (파이썬과 스칼라에 있는) 명명된 선택적 매개변수를 흉내낸 것이다.

유효성검사 코드는 생략했지만 잘못된 매개변수를 최대한 일찍 발견하려면 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고, build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식을 검사하는 것이 좋다.

공격에 대비해 이런 불변식을 보장하려면 빌더로 부터 매개변수를 복사한 후 해당 객체 필드들도 검사해야 한다. 검사해서 잘못된 점을 발견하면 어떤 매개변수가 잘못되었는지 자세히 알려주는 메시지를 담아 IllegalArgumentException을 던지면 된다.

> 불변(immutable	)은 어떠한 변경도 허용하지 않는다는 뜻으로 주로 변경을 허용하는 가변 객체와 구분하는 용도로 쓰인다. 대표적으로 String객체는 불변 객체이다
>  한편, 불변식은 프로그램이 실행되는 동안, 혹은 정해진 기간동안 반드시 만족해야 하는 조건을 말한다. 다시 말해 변경을 허용할 수는 있으나 주어진 조건 내에서만 허용한다는 뜻이다. 리스트의 크기는 반드시 0이상이어야 하니, 만약 한순간이라도 음수 값이 된다면 불변식이 깨진것이다. 또한 기간을 표현하는 Period클래스에서 start필드의 값은 반드시 end필드의 값보다 앞서야 하므로, 두 값이 역전되면 역시 불변식이 깨진 것이다
>  따라서 가변 객체에도 불변식은 존재할 수 있으면 넓게 보면 불변은 불변식의 극단적인 예라 할 수 있다.

**빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.**각 계층의 클래스에 관련 빌더를 멤버로 정의하자. 추상 클래스는 추상빌더를, 구체 클래스는 구체 빌더를 갖게 한다. 

```java
/*
    Item 2 
    생성자에 매개변수가 많다면 빌더를 고려하라

    빌더패턴 예시 - 추상 빌더, 구체 빌더
*/
import java.util.*;

public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의 하여 “this”를 반환하도록 해야 한다
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템50 참조
    }
}

// 뉴욕 피자
class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }
    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}

// 칼초네 피자
class Calzone extends Pizza {
    private final boolean sauceInside;
    
    public static class Builder extends Pizza.Builder<Builder> {
        private final boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }
    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}

// 클라이언트 사용 (실행)
// NyPizza pizza = new NyPizza.Builder(SMALL).addTopping(SAUSAGE).addTopping(ONION).build();
// Calzone calzone = new Calzone.Builder().addTopping(HAM).sauceInside().build();
```

Pizza.Builder 클래스는 재귀적 타입 한정을 이용하는 제네릭 타입이다
여기에 추상 메서드인 self를 더해 하위 클래스에서는 형변환하지 않고도 메서드 연쇄를 지원할 수 있다. self 타입이 없는 자바를 위한 이 우회 방법을 시뮬레이트한 셀프 타입 관용구라 한다.

각 하위 클래스의 빌더가 정의한 build메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다. NyPizza.Builder는 NyPizza를 반환하고, Calzone.Builder는 Calzone를 반환한다는 뜻이다.

하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌 그 하위 타입을 반환하는 기능을 공변반환 타이핑이라 한다. 이 기능을 이용하면 클라이언트가 형변환에 신경 쓰지 않고도 빌더를 사용할 수 있다.

생성자로는 누릴 수 없는 사소한 이점으로, 빌더를 이용하면 가변인수 매개변수를 여러개 사용할 수 있다. 각각을 적절한 메서드로 나눠 선언하면 된다. 아니면 메서드를 여러번 호출하도록 하고 각 호출때 넘겨진 매개변수들을 하나의 필드로 모을 수도 잇다. addTopping 메서드가 이렇게 구현한 예다.

빌더 패턴은 상당히 유연하다. 빌더 하나로 여러 객체를 순회하면서 만들수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다. 객체마다 부여되는 일련번호와 같은 특정 필드는 빌더가 알아서 채우도록 할 수도 있다.

빌터패턴은 객체를 만드려면 그에 앞서 빌더부터 만들어야 한다. 빌더 생성 비용이 크지는 않지만 민감한 상황에 문제가 될수 잇다. 또한 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다. 하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있다.

생성자나 정적 팩토리 방식으로 시작했다가 나중에 빌더 패턴으로 전환할 수도 있지만, 처음부터 하는 것이 좋을 수도 있다

> ::핵심 정리::
> 생성자나 정적 팩토리가 처리해야할 매개변수가 많다면 빌더 패턴을 선택하는 것이 낫다.
> 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 글허다. 빌더는 점층적생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.
