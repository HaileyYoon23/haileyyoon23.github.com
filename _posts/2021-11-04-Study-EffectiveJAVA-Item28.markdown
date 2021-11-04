---

layout: post  

title: "Effective JAVA, Item 28"  

subtitle: "배열보다는 리스트를 사용하라"  

categories: EffectiveJAVA  

tags: EffectiveJAVA Item28

comments: true  

header-img: 

---

## ITEM 28 배열보다는 리스트를 사용하라

### 배열은 공변, 제너릭은 불공변이다.

공변이란, 자기 자신과 자식 객체로 타입 변환을 허용해주는 것이다.
``` java
Object[] objects = new Long[2];
```

불공변이란, 두 타입이 전혀 관련없음을 의미한다.
``` java
List<String> strings = new ArrayList<>();
function(strings)   // Compile Error

public static void function(List<Object> objects) {
    ...
}
```

<br/>

제네릭은 불공변이다. 따라서, 배열과는 달리 런타임이 아닌 컴파일 단계에서 오류가 발생하므로, 런타임시에 타입 안정성을 가질 수 있다.  
``` java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException 을 던짐. 
```
위 코드는 배열의 공변 성질로 인해 컴파일은 되지만 런타임에 에러가 발생한다.
``` java
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다.
ol.add("타입이 달라 넣을 수 없다.);
```
위 코드는 제너릭의 불공변 성질로 인해 컴파일 단계에서 문제가 있음을 알아챌 수 있다.   
  
### 배열은 실체화(reify), 제너릭은 실체화 불가(non-reify) 이다
**실체화**란, 런타임에 자신의 타입 정보를 인지하고 있는 것이다.  배열은 런타임에도 자신이 넣기로 한 원소의 타입을 인지하고 확인한다.  
따라서 이전의 예시처럼 Long 배열에 String 을 넣으려고 했을 때, **ArrayStoreException** 에러가 발생했다.  

**실체화 불가**란, 런타임에 자신의 타입 정보를 인지하고 있지 않은 것이다.  
**제너릭은 타입 정보가 런타임엔 제거된다.** 컴파일 단계 후 런타임에선 타입 정보를 알 수 없다. 이는 제너릭과 제너릭 이전의 레거시 코드를 함께 사용할 수 있게 해준 매커니즘 이다.  

<br/>

제너릭은 배열과 함께 쓰일 수 없다. 만약 함께 쓰인다면 어떻게 될까?

<br/>

``` java
List<String>[] stringLists = new List<String>[1];   // (1)
List<Integer> intList = List.of(42);                // (2)
Object[] objects = stringLists;                     // (3)
objects[0] = intList;                               // (4)
String s = stringLists[0].get(0);                   // (5)
```
1. 제너릭 배열을 생성하는 것이 허용된다고 가정하자 (원래는 Compile Error)
1. 원소가 한개인 List&#60;Integer> 를 생성한다.
1. 1 에서 생성한 List&#60;String> 의 배열을 Object 배열에 할당한다. (배열은 공변이므로 문제가 없다)
1. 2 에서 생성한 List&#60;Integer> 의 인스턴스를 Object 배열의 첫 원소로 저장한다. (제너릭은 소거방식이므로 문제가 없다)
1. List&#60;String> 인스턴스만 담겠다고 선언한 stringLists 배열에 현재 List<Integer> 인스턴스가 저장되어있는 상태. 따라서, **원소를 꺼낼 때 컴파일러는 자동으로 String 으로 형변환을 하여 ClassCastException 이 발생한다.**  

실제 상황에선 1. 처럼 제너릭과 배열을 함께 쓸 수 있는 상황을 허용해주지 않기 때문에 컴파일 에러로 상황을 방지할 수 있다.   


### 비검사 경고를 완전히 제거하고 싶다면, 배열 대신 리스트를 써라
``` java
public class Chooser {
    private final Object[] choiceArray;
    
    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }
    
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```
지금까지의 이야기와 같다. 우선 이 클래스는 사용하면 안되는 로타입을 사용하고 있다.   
이 클래스를 사용한다면, ```choose``` 메서드를 사용할 때 마다 반환된 Object 를 원하는 타입으로 형변환해야 하며, 런타임에 형변환 오류가 날 가능성을 항상 가지고 있다.  

이를 제네릭으로 선언해주면서 해결한다고 해도, 비검사 경고는 남게 된다. 그 이후는 개발자가 안정성을 보장한다는 전제 하에 경고를 숨기는 방법이 지금까지의 결론이다.  

<details>
    <summary>
    <u>비검사 경고 결말</u>
    </summary>

우선 Object 가 아닌 제너릭 &#60;T> 를 사용하도록 코드를 변경한다.

``` java
public class Chooser<T> {
    private final T[] choiceArray;
    
    public Chooser(Collection<T> choices) {
        choiceArray = choices.toArray();    // Compile Error
    }
    
    // choose 메서드는 그대로
}

```

위 코드를 그대로 컴파일하게 되면 **Object[] 를 T[]로 형변환할 수 없다는 오류** 가 발생한다.   
이를 해결하기 위해선 아래와 같이 **T[] 로 형변환**을 해주면 된다.  

``` java
choiceArray = (T[]) choices.toArray();
```

그러면 형변환이 런타임에도 안전할 지 보장할 수 없다는 비검사 경고가 뜬다. 이는 개발자가 안정성을 검증하여 경고를 숨기는 수 밖에 없다.

</details>

하지만, 배열 대신 리스트를 사용하게 되면 위와 같은 결말을 맞이하지 않아도 된다.  

``` java
public class Chooser<T> {
    private final List<T> choiceList;
    
    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);  // Object[] 대신에 리스트 사용
    }
    
    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }

}
```

위 처럼 리스트를 사용하게 되면 오류나 비검사경고 없이 안전하게 컴파일 된다. 
<br/>

---

