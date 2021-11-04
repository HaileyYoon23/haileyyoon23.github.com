---

layout: post  

title: "Effective JAVA, Item 27"  

subtitle: "비검사 경고를 제거하라"  

categories: EffectiveJAVA  

tags: EffectiveJAVA Item27

comments: true  

header-img: 

---

## ITEM 27. 비검사 경고를 제거하라

> 비검사 경고 모두를 제거하면, 코드의 타입 안정성을 높일 수 있다. 


``` java
Set<String> set = new HashSet();
```

위의 코드는 HashSet에 타입을 명시하지 않아 추후에 문제가 생길 수 있는 여지가 있다.  
우선, 위의 코드 문제를 해결하는 방법은 아래와 같이 타입을 명시해주는 방법이다.
``` java
Set<String> set = new HashSet<String>();

// JAVA 7 부터는 다이아몬드연산자 <> 를 지원하여 아래로도 해결 가능
Set<String> Set = new HashSet<>();
```

이와 같이 추후 문제가 생길 여지가 있는 코드는 Compiler 에서 인지하고 알려줄 수 있다.

### 컴파일 시, 비검사 경고를 잘 확인하자
원래 비검사 경고는 컴파일 시엔 잘 확인할 수 없다.   
따라서 컴파일 시 **-Xlink:unchecked** 옵션을 추가할 경우, 비검사 경고 관련해서 자세히 확인이 가능하다


### 비검사 경고를 숨기려면 가능한 좁은 범위로 적용할 것

허나 경고를 제거할 수는 없지만, 해당 경고에 대해 안전하다고 확신할 수 있을 땐 어떻게 제거할 수 있을까?  
이럴 경우, **@SuppressWarning("unchecked")** 를 사용할 수 있다.   
 
**@SuppressWarning("unchecked")** 어노테이션은 선언된 곳에서 발생하는 비검사경고를 숨겨주는 기능을 한다. 안전하다고 판단된 부분의 경고를 숨김으로써 **진짜 경고** 들을 더 빠르고 쉽게 판별할 수 있기 때문이다.  

허나, 비검사 경고를 숨겨야 할땐 **가능한 좁은 범위에 적용** 하는 것이 좋다.   
보통은 변수 선언, 아주 짧은 메서드, 혹은 생성자에 붙여 사용한다.  
큰 범위로 적용하게 될 때엔 심각한 경고를 놓치게 될 수 있으므로, 절대 클래스 전체에 적용해서는 안된다.

``` java
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[] 로 같으므로
        // 올바른 형변환이다.
        @SuppressWarnings("unchecked") 
        T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length > size) 
        a[size] = null;
    return a;
}
```
<details>
<summary><u>실제 ArrayList 코드</u></summary>

``` java
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

</details>
그리고 **@SupressWarnings("unchecked")** 어노테이션을 사용할 땐, 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨두면 좋다.  


<br/>
---

