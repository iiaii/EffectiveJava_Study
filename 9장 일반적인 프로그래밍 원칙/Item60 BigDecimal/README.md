# #EffectiveJava/9장_일반적인프로그래밍원칙/60BigDecimal


## 60. 정확한 답이 필요하다면 float와 double은 피하라


float 와 double 은 과학과 공학 계산용으로 설계되어 넓은 범위의 수를 빠르게 정밀한 근사치로 계산한다. 따라서 금융 관련 계산과는 맞지 않다. 0.1 혹은 10의 음의 거듭제곱수 등을 표현할 수 없다.

```java
System.out.println(1.03-0.42);
// 0.610000000001
System.out.println(1.00 - 9*0.10);
// 0.09999999999998 
```

금융 계산에는 BigDecimal, int, long을 사용해야 한다.

```java
// BigDecimal을 사용
public static void main(String[] args) {
	final BigDecimal TEN_CENTS = new BigDecimal(“.10”);

	int itemsBought = 0;
	BigDecimal funds = new BigDecimal(“1.00”);
	for(BigDecimal price = TEN_CENTS;
		funds.compareTo(price) >= 0;
		price = price.add(TEN_CENTS)) {
		funds = funds.subtract(price);
		itemsBought++;
	}
	System.out.println(itemsBought +”개 구입”);
	System.out.println(“잔돈(달러): “+funds);
}
```
-> 기본 타입보다 쓰기 훨씬 불편하고 훨씬 느리다. 

```java
// 단위를 센트로하여 정수 타입 사용
public static void main(String[] args) {
	int itemsBought = 0;
	int funds = 100;
	for(int price=10 ; funds >= price ; price +=10) {
		funds -= price;
		itemsBought++;
	}
	System.out.println(itemsBought+”개 구입”);
	System.out.println(“잔돈(센트):”+funds);
}
```


**::핵심 정리::** 

> 정확한 답이 필요한 계산에 float, double 사용을 피하고 성능저하나 불편함을 감수한다면 BigDecimal, 소수점을 직접 추적 가능하다면 int 나 long을 사용하는 것이 좋다. (18자리를 넘어가면 BigDecimal 사용)



