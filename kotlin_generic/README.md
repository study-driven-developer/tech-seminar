# 코틀린의 Generic

## 제네릭이란?

> 내가 알지 못하는 타입의 데이터를 조작하기 위해서 필요한 기능이다.
>

## 타입 파라미터

타입 파라미터는 클래스 이름 바로 뒤에 `<T>` 이런식으로 오는 걸 타입 파리미터라고 한다.

```kotlin
class TreeNode<T> (data: T) : DataHolder<T>(data) {}
```

타입 파라미터의 이름은 아무거나 가능하지만 관습적으로 `T, U, V` 등의 짧은 대문자를 사용한다.

### 바운드와 제약

기본적으로 타입 인자로 들어갈 수 있는 타입의 제한은 없다.

그러므로 타입 파라미터들은 `Any?` 와 동의어라고 생각하면 된다.

- 자바로 치면 ``Object`` 가 된다.

그런데 때로는 제네릭 타입의 제한을 걸고 싶을 때가 있다.

`TreeNode` 클래스 기준으로 수를 저장하는 트리를 구성하고 싶다면 상위 바운드 (upper bound) 로 제한을 걸고 싶을 것이다

```kotlin
class TreeNode<T: Number> (val data: T)

fun main() {
    val treeNode = TreeNode<Double>(5.0)
    val treeNode1 = TreeNode<Int>(5)
}
```

- Double 과 Int 는 Number 의 하위 타입이므로 만들 수 있다.
- 자바에도 이렇게 타입의 제한을 거는 방법이 있다. ``<T extends Number>`` 이렇게 쓰면 됨.

타입 파라미터 구문을 사용하면 상위 바운드를 하나만 지정할 수 있지만 타입 파라미터에 여러 제약을 가할 수도 있다. 이 경우 좀 더 복잡한 타입 제약 구문을 이용할 수 있는데 예시는 다음과 같다.

```kotlin
interface Named {
    val name: String
}

interface Identified {
    val id: Int    
}

class Registry<T> where T : Named, T : Identified {
}
```

- 자바에서는도 여러 타입 제한을 거는게 가능하다. `<T extends A1 & A2 & A>` 이런식으로.
- 걍 자바에서 되는건 다 된다고 생각해도 됨.

### 타입 소거와 구체화

자바에서는 제네릭 호환성 문제로 제네틱 타입은 타입 소거가 된다.

- 요 이슈에 대해서 찾아보려고 했는데 나오진 않음.

즉 `List<String>` 과 `List<Number>` 는 기본적으로 `List` 라는 동일한 타입으로 인식된다.

코틀린은 이후에 만들어졌으므로 이런 문제를 겪지 않을 수 있지만 JVM 호환성 문제로 같은 이슈를 가지고 왔다.

즉 제네릭으로 선언한 타입 비교는 의미가 없다.

- `MyObject is T` 라는 타입 비교문은 에러가 난다. 왜냐하면 T 가 타입이 제거되서 어떤 타입인지 알지 못하니까.

자바에서는 이런 타입 소거의 문제를 해결할려면 리플렉션을 써야한다.

- 리플렉션을 통해서 런타임중에 클래스에대한 정보를 얻는게 가능하다.
- 대신에 성능상의 이슈가 있다. JVM 의 최적화가 먹히지 않는다고 한다. Direct Access 보다 104배 정도 느리다고 한다.

코틀린에서는 `reified` 를 통한 구체화를 통해 타입 파라미터를 런타임 시점까지 유지하는게 가능하다.

- 대신에 `inline` 함수에만 적용이 가능함.
- 코틀린에서는 인라인 함수를 제공 해주는데 이는 함수 호출 비용을 줄이기 위함임. 함수 본문의 내용을 호출 시점으로 인라인 시키는 것. 이를 통해서 컴파일러가 실제 타입을 추론할 수 있다.

## 변성 (Variance)

Effective Java 의 Item 31 를 보면 `한정적 와일드카드를 사용해 API 유연성을 제공하라`  라는 말이 있다.

- 이게 변성을 잘 이용하란 뜻임.

변성은 타입 파라미터가 달라질 떄 제네릭 타입의 하위 타입의 관계가 어떻게 달라지는 지를 설명하는 것이다.

코틀린 기준으로 배열과 가변 컬렉션의 경우에는 변성이 작용하지 않는다. (이를 Invariance 라고함.)

이 뜻은 `Array<String>` 은 `Array<Any>` 의 하위 타입이 아니라는 뜻이다.

하지만 불변 타입의 컬렉션의 경우에 `List<String>` 은 `List<Any>` 의 하위 타입이다.

변성이 없으면 타입 안정성을 제공해줄 순 있지만 유연함을 제공해줄 순 없다.

```kotlin
public class Stack<E> {
	public Stack(); 
	public void push(E e);
	public E pop(); 
	public boolean isEmpty(); 
}

public void pushAll(Iterable<E> src) {
	for (E e : src) {
  	    push(e); 
	}
}
```

- 변성이 없다면 ``Stack<Number>`` 로 선언하면 ``Iterable<Number>`` 만 요소로 받는게 가능하다.
- ``Stack<Number>`` 에 ``Iterable<Integer>`` 를 넣을 순 없음. 넣어도 문제 없는데.

근데 또 변성을 제공해주면 타입 안정성이 깨지는 경우도 있다.

```kotlin
val stringNode = TreeNode<String>("Hello")
val anyNode: TreeNode<Any> = stringNode
anyNode.addChild(123)
val s = stringNode.childeren.first() // ???
```

즉 변성을 잘 제공해줘야 한단 뜻이다. **모든 제네릭 타입의 경우에는 사용가능한 케이스가 세 가지 존재한다.**

1) T 타입의 값을 반환하는 연산만 제공하고, 입력으로 T 타입을 받는 연산은 제공하지 않는 **생산자 (Publisher)**.

2) T 타입의 값을 입력으로 받는 연산만 제공하고, T 타입의 값을 반환하지는 않는 제네릭 **소비자 (Consumer)**.

3) 위 두 가지 경우에 해당하지 않는 경우들

- 1 번과 2번은 변성을 제공해야 하며 3번은 변성을 제공하면 안된다.
- 코틀린의 ``List`` 같은 경우는 불변 컬렉션인데, 이는 생산자 역할을 해줄 수 있으므로 변성을 제공해준다. 여기서의 변성을 공변성 (covariant) 라고 함.

주의할 건 공변성과 불변성을 헷갈려하는 경우가 있는데 그렇지 않다는 것.

```kotlin
interface NonGrowingList<T> {
	val size: Int
	fun get(index: Int): Int
	fun remove(index: Int): Int 
}
```

- 이 리스트는 삭제 할 수 있는데 공변성을 제공해준다.

그리고 다른 인터페이스를 보자. 이 인터페이스를 구현한 요소는 불변성을 가지지만 공변성을 가지지 않는다.

```kotlin
interface Set<T> {
    fun contains(element: T): Boolean
}
```

- ``Set<Any>`` 와 ``Set<String>`` 을 비교해보면 ``Set<String>`` 은 ``Set<Any>`` 를 대체할 수 없다.
- 즉 타입 파라미터를 기준으로 제네릭 타입의 하위 타입 관계를 유지해주지 않는다.

하지만 이 Set 은 반대가 가능하다. 타입 파라미터를 기준으로 제네릭 타입의 상위 타입 관계를 유지해준다.

- 이를 반공변성 (contravariant) 라고 한다.

예로 ``Set<Number>`` 와 ``Set<Int>`` 를 비교해보자.

Int 는 Number 의 하위 타입이다.

``Set<Number>`` 는 Int 뿐 아니라 Number 도 처리가 가능하지만 ``Set<Int>`` 는 Int 만 처리가 가능하다.

즉 ``Set<Number>`` 는 ``Set<Int>`` 를 대체하는게 가능하지만 그 역은 안되므로 `Set<Number>` 는 ``Set<Int>`` 의 하위타입이라고 생각해볼 수 있다.

## 코틀린에서 변성을 제공해주는 방법.

### 선언 지점 변성 (**Declaration-site variance)**

기본으로 타입 파라미터를 쓰면 무공변 (invariant) 로 취급된다.

```kotlin
interface List<T> {
    val size: Int

    fun get(index: Int): T
}
```

- 클래스 앞에 타입 파라미터를 쓰는 경우를 선언 지점 변성이라고 한다.
- 이렇게 쓰는 경우에 List 는 무공변으로 취급됨.

```kotlin
interface List<out T> {
    val size: Int

    fun get(index: Int): T
}
```

- 타입 파라미터 앞에 out 을 쓰는 경우를 covariant 를 적용하는 경우임.
- 이렇게 하면 ``List<Number>`` 에 ``List<Int>`` 를 넣는게 가능해짐.
- 자바에서는 이런식으로 변성을 주는게 불가능함. 필요한 경우에 ``List<? extends T>`` 이런식으로 변성로 일일히 변성을 줘야함. (= 노가다)
- 공변을 사용할 땐 생산자 역할만 해줘야 함. ``fun set(index: Int, value: T`` 와 같은 메소드를 추가한다면 (= 소비자용 메소드) 컴파일 에러가 남.
- 반대로 반공변성 (contravariant) 를 주고 싶다면 `in` 을 쓰면 됨. ``out`` 대신에.

### 프로젝션을 사용한 변성.

- 이건 제네릭 타입을 사용하는 곳에서 (= 제네릭 함수 등) 변성을 지정하는 방식이다. 자바와 크게 다르지 않다.
