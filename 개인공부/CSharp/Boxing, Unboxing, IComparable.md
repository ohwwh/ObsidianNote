

## Boxing, Unboxing
---

```c#
int n1 = 1;
object o2 = n1;
int n2 = (int)o2;
```

이런 경우, o2는 참조타입인 만큼 힙을 가르켜야 하지만, n1은 값 타입으로써 스택에 존재한다. 이런 모순을 이겨내기 위해, n1이 한 번 복제가 되어서 힙에 생성되고, o2는 그 복제본을 가리키게 된다. 이러한 과정을 Boxing이라고 한다.

또한 마지막 줄에서 다시 힙에 있는 객체를 스택에 존재해야 하는 값 타입에 넣으려고 한다면, 힙의 객체를 스택에 또 복사하게 된다. 이 과정은 Unboxing이라고 한다.


## IComparable
---
위와 같은 Boxing과 Unboxing은, 때로는 성능 이슈를 발생시킬 수 있다. IComparable이라는 인터페이스 내부의 CompareTo가 그 예시이다.

```csharp
interface Icaomparable
{
	int CompareTo(object other);
}
```

함수는 이런 식으로 구현되어 있다. 어떤 객체든 받을 수 있도록 인자를 object로 받는데, 여기서 문제가 발생한다. 만약 other의 자리에 값 타입이 들어온다면, 위에서 말한 Boxing이 일어나면서 값의 복사, 즉 메모리의 낭비가 발생한다는 것이다.

이러한 낭비를 막기 위해, 제너릭 인터페이스를 활용한다. 일반적인 object를 비교하는 CompareTo와 함께, 특정 타입에 대한 CompareTo도 구현하는 것이다.

```csharp
interface Icaomparable<T>
{
	int CompareTo(T other);
}
```

위와 같이 구현하는 경우 T의 자리에 값 타입이 와도 Boxing/Unboxing이 일어나지 않는다.