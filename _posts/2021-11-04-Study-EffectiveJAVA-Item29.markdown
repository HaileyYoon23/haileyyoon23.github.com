---

layout: post  

title: "Effective JAVA, Item 29"  

subtitle: "이왕이면 제너릭 타입으로 만들어라"  

categories: Study  

tags: EffectiveJAVA Item29

comments: true  

header-img: 

---

## ITEM 29 이왕이면 제너릭 타입으로 만들어라

``` java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null;  // 다 쓴 참조 해제
        return result;
    }
    
    public boolean isEmpty() {
        return size == 0;
    }
    
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```
위의 코드는 Item 7 에서 다룬 Stack 코드인데, 이 코드를 제너릭 형태로 변경해보려 한다.  

### 실체화 불가(non-reify) 타입으로 배열을 만들 수 없다

Stack 코드를 제너릭으로 변경 시, 다음과 같이 된다.

``` java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY]; // Compile Error
    }
    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E result = elements[--size];
        elements[size] = null;  // 다 쓴 참조 해제
        return result;
    }
    ... // isEmpty 와 ensureCapcacity 메서드는 동일.
}
```
<br/>

``` java
elements = new E[DEFAULT_INITIAL_CAPACITY];
```

에서 오류가 발생한다.   
실체화 타입으로는 배열을 만들 수 없기 때문에 발생한 오류이다. 

<br/>

#### Solution 1) 실체화 불가 타입의 배열 생성금지 제약을 우회 
위의 오류를 제약사항을 우회하는 방법으로 해결할 수 있다. 
``` java
element = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
```
이렇게 변경하면 오류는 해결하여 컴파일이 가능해진다.  허나, 비검사 경고는 존재한다.  
만약 안전성을 사용자가 직접 증명했다면, **@SuppressWarnings** 어노테이션으로 경고를 숨겨 정리할 수 있다.

이 방법은 배열의 타입을 ```E[]``` 로 선언하여 가독성이 좋다.  
하지만 이 방법은 컴파일타임의 타입과 런타임 후의 타입이 달라 힙 오염을 발생시킨다 (Item 32)
<br/>

#### Solution 2) Object[] 사용

``` java
elements = new Object[DEFAULT_INITIAL_CAPACITY];
```
문제의 코드를 위와 같이 변경하여 오류를 해결할 수 있는데, 이 경우엔 새로운 오류가 발생한다.

``` java
E result = elements[--size];

```
Object[] 를 제네릭 E 로 형변환 할 수 없기 때문에 형변환 컴파일 오류가 발생한다.

<br/>

```java
E result = (E) elements[size--];
```
따라서 위 처럼 배열이 반환한 원소를 ```E``` 로 형변환하면 오류는 해결되고 비검사 경고가 뜬다.   
이 또한, 안전성을 사용자가 직접 증명했다면 **@SuppressWarnings** 어노테이션으로 경고를 숨겨 정리할 수 있다.

이 방법은 원소를 읽을 때 마다 형변환을 해주어 가독성이 좋지 않다. 하지만 힙 오염을 발생시키지 않는다.

<br/>

**제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 좋은 것도 아니다.** 자바가 리스트를 기본 타입으로 제공하지 않으므로, ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 한다. 또한 HashMap 같은 제네릭 타입은 성능을 높을 목적으로 배열을 사용하기도 한다.  

대부분 **제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않으며**, Stack 예처럼, ```Stack<Object>``` ```Stack<int[]>```, ```Stack<List<String>>``` 등 기본 타입을 제외한 어떤 참조 타입으로도 Stack 을 만들 수 있다.

추가로, 한정적 타입 매개변수를 사용해 매개변수에 제약을 둘 수도 있다.
``` java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```
**&#60;E extends Delayed>** 는 java.util.concurrent.Delayed 의 하위 타입만 받는다는 뜻이며, DelayQueue 의 원소에서 형변환없이 바로 Delayed 메서드를 사용할 수 있다. 
또한, ```ClassCastException``` 오류도 걱정할 필요가 없다.

#### tips
```
클라이언트에서 직접 형변환해야 하는 타입보다 제너릭 타입이 더 안전하고 쓰기 편하다. 그러므로 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하여라. 

또한 기존 타입 중 제네릭이었어야 하는게 있다면 제네릭으로 변경하자. 이것이 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해주는 길이다.
```

<br/>

<br>
Fin.
<br>

---

