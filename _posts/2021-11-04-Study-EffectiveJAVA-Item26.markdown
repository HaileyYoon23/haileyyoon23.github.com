---

layout: post  

title: "Effective JAVA, Item 26"  

subtitle: "로타입(Raw Type)은 사용하지 말라"  

categories: EffectiveJAVA  

tags: EffectiveJAVA Item26

comments: true  

header-img: 

---

# 5장 제너릭 Generic
> 형 변환 오류를 막는 제너릭을 잘 활용할 수 있는 방법.

<br/>

| 한글 용어  | 영문 용어 | example |
|:-------------:|:-------------:|:-------------:|
|매개변수화 타입|parameterized type|List&#60;String>|
|실제 타입 매개변수|actual type parameter|String|
|제네릭 타입|generic type|List&#60;E>|
|정규 타입 매개변수|formal type parameter|E|
|비한정적 와일드카드 타입|unbounded wildcard type|List&#60;?>|
|로 타입|raw type|List|
|한정적 타입 매개변수|bounded type parameter|&#60;E extends Number>|
|재귀적 타입 한정|recursive type bound|&#60;T extends Comparable&#60;T>>|
|한정적 와일드카드 타입|bounded wildcard type|List&#60;? extends Number>|
|제네릭 메서드|generic method|static &#60;E> List&#60;E> as List(E[] a)|
|타입 토큰|type token|String.class|


## ITEM 26 로(Raw) 타입은 사용하지 말라

### 제네릭 타입 이란
> 선언에 타입 매개변수가 사용된 클래스, 인터페이스.



<br/>

```java
public interface List<E> extends Collection<E> {
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    
    ...
}
```
List&#60;String> 로 선언 시 내부에서는 코드의 **E** 부분이 String 으로 타입이 통용되며, 타입이 클래스 외부에서 사용자에 의해 지정되도록 한다

<br/>


### 로타입은 제너릭 마이그레이션 지원을 위해 존재한다

``` java
private final List list;
```

로(Raw) 타입이란, 제너릭 타입에서 타입 매개변수를 전혀 사용하지 않은 상태의 타입을 말한다.  

자바가 제네릭을 받아들이기까지 거의 10년이 걸렸기 때문에, 많은 코드들이 제네릭 없이 생성이 되었다.   
로타입을 지원하는 것은, 제너릭이 없는 코드와의 호환성 때문.  


### 로타입은 코드를 불안정하게 한다
  런타임보다 컴파일 시에 에러를 잡을 수 있는 코드가 더 안전한 코드라고 할 수 있다. 로타입을 사용하면 런타임에 예외가 일어날 수 있다.  
  따라서 로타입은 코드를 불안정하게 한다고 할 수  있다.  
``` java
// Stamp 인스턴스만 취급한다.
private final Collection stamps = ...;

...

// [warning] unchecked call ...
stamps.add(new Coin(...)); 

// [Error] ClassCastException
for (Iterator i = stamps.iterator(); i.hasNext(); ) {
    Stamp stamp = (Stamp) i.next();
    stamp.cancel();
}
```

위 코드에서 ```Collection``` 로타입으로 선언된 ```stamps``` 는 ```Stamp``` 인스턴스만 취급하기 위해 정의되었지만, 실수로 ```Coin``` type 을 넣게 되었을 때 문제없이 실행되지만 컴파일러가 수상하다고 여겨 경고를 주고 끝난다.  
허나 이후에 꺼내어 형변환 등의 처리를 하게 되었을 때, 런타임 에러가 발생하게 된다.  

<br/>

``` java
private final Collection<Stamp> stamps = ...;
```
위처럼 제네릭 설정을 해놓으면 실수로 ```Coin``` 을 넣게 되었을 때 컴파일 에러가 발생하여 오류를 사전에 막을 수 있게 된다. 

<br/>

``` java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
    
    // ClassCastException 발생
    String s = strings.get(0);
}

private static void unsafeAdd(List list, Object o) {
    list.add(o);
}
```

위의 경우도 정상적으로 컴파일이 실행되지만, ```strings.get(0)``` 을 사용하여 원소를 꺼내 강제 형변환이 실행될 때 런타임 에러가 발생하게 된다.  
이를 위해 ```unsafeAdd``` 에서 ```List``` 대신에 ```List<Object>``` 을 사용하게 되면 아래와 같이 컴파일 에러가 발생하여 문제를 사전에 막을 수 있다.
```
java: incompatible types: 
java.util.List<java.lang.String> cannot be converted to java.util.List<java.lang.Object>
    unsafeAdd(strings, Integer.valueOf(42);
         ^
```

**(주의) List&#60;Object> 의 하위타입으로 List&#60;String> 이 될 수 없다.**

: List&#60;Object> 는 어떤 객체든 넣을 수 있는 List 타입이고, List&#60;String> 은 String 객체만 넣을 수 있는 List 로 둘은 별개의 개념이다.


### 비한정적 와일드카드 타입을 사용해라
> 제네릭 타입을 쓰고 싶지만, 실제 타입이 무엇인지 신경쓰고 싶지 않을 땐 <?> 를 사용하자.

비한정적 와일드카드 타입(Unbounded Wildcard Type) 이란, 제네릭 타입에서 &#60;?> 로 표기되어 있는 것을 말하며, 아직 알려지지 않은 타입을 가르킨다.  

비한정적 와일드카드 타입은 **매개변수 타입에 의존하지 않는 제너릭 클래스의 매서드를 사용할 때**를 위해 쓰인다.   
``` java
public static void printList(List<?> list) {
    Iterator<?> iterator = list.iterator();
    while(iterator.hasNext()) {
        System.out.println(iterator.next());
    }
}

public static void main(String[] args) {
    List<String> str = new ArrayList<>();
    str.add("h");
    str.add("i");
    printList(str);
}
```

위는 와일드카드 적절한 사용에 대한 예시이다. `printList` 에선 `List<?> list` 의 타입이 중요한 요소가 아니다. 따라서 어떤 타입이 와일드카드에 해당한다 하더라도 오류가 발생하지 않는다.  
또한, `List<?>` 은 `List<String>` , `List<Integer>` 등. 모든 타입을 자신의 하위타입으로 취급하기 때문에 어떤 타입의 List 라도 타입을 보존한 채 출력할 수 있다.   
이는 `List<String>` 이 `List<Object>` 의 하위타입이 아닌 것과는 다르다.   

다만 주의해야할 사항은, Object 에는 Object 의 하위 타입을 넣을 순 있지만, List<?> 에는 **null 을 제외하고 어떤 타입도 넣을 수 없다.** 이는 와일드카드의 **불변성** 에 관련이 있다.  

아래는 그에 대한 예시이다. 

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2) {
    int result = 0;
    for (Object o1 : s1) 
    
}
```
비한정적 와일드카드 &#60;?> 는 **어떤 타입이라도 허용하는 범용적인 매개변수이다.** 이를 위해서 와일드카드는 null 을 제외한 어떤 원소의 insert 도 허용하지 않는다.(대신 remove 는 허용한다.)  
&#60;?> 을 사용하게 되면 Collection&#60;?> 에는 (null 외엔) 어떤 원소도 넣을 수 없어 불변성을 유지할 수 있다. 
``` java
Collection<?> collection = new ArrayList<>();
collection.add(null);   // 가능
collection.add(1234);   // Compile Error
```



### 로타입을 써도 좋은 예

* class 리터럴에는 로타입으로 써야한다  
: List.class, String[].class, int.class ....

* instanceof 연산자엔 비한정적 와일드카드 타입 외의 제너릭이 적용 불가능

``` java
if (o instanceof Set) {
    Set<?> s = (Set<?>) o;
    ...
}
```
`instanceof` 의 결과가 `true` 라는 것은, 참조변수가 검사한 타입으로 형변환이 가능하다는 뜻이다. 따라서 `instanceof` 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다.  
그리고 로타입과 비한정적 와일드카드타입이 완전히 동일하게 동작하므로 로타입(Raw type) 을 쓰는 편이 더 깔끔하다. 

<br/>

---

