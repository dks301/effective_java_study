## 아이템 53. 가변인수는 신중히 사용하라

## 가변인수란?

- 명시한 타입의 인수를 0개 이상 받을 수 있도록 하는 것이다.

```java
static int sum(int... numbers) {
	int sum = 0;
	for (int number : numers) {
		sum += number;
	}
	
	return sum;
}

sum(1, 2);
sum(1);
sum();
```

- 가변인수 메소드를 호출하면 먼저 인수의 갯수와 같은 배열을 만들고 인수들을 배열에 저장해 가변인수 메서드에 전달한다.
- 최소한 인수가 한개라도 필요하게 하려면 아래와 같이 validation 로직을 작성해주면 된다.

```java
static int sum(int... numbers) {
	if (numbers.lenght == 0) {
		throw new IllegalArgumentException("인수는 1개 이상이어야 합니다.");
	}
	int sum = 0;
	for (int number : numers) {
		sum += number;
	}
	
	return sum;
}

sum(1, 2);
sum(1);
sum(); // IllegalArgumentException
```

## 단점

- 위와 같이 validation을 하면 컴파일단계가 아니라 런타임시에 에러가 발생한다는 점입니다.
- 그러나 이러한 문제는 아래와 같이 해결할수 있다.

```java
static int sum(int firstNumber, int... numbers) {
	int sum = firstNumber;
	for (int number : numers) {
		sum += number;
	}
	
	return sum;
}

sum(1, 2);
sum(1);
sum(); // 컴파일 타임 체크
```

- 성능에 민감한 사항에서는 가변인수가 걸림돌이 될 수 있습니다.
- 왜냐하면 호출될 때마다 배열을 새로 하나 할당하고 초기화하기 때문입니다.
- 이를 해결하기 위해서는 메서드를 오버로딩하는 전략을 사용하면 된다.

```java
static int sum(int number1)
static int sum(int number1, int number2)
static int sum(int number1, int nubmer2, int number3)
static int sum(int number1, int nubmer2, int number3, int... numbers)
```

> 일정하지 않은 매개변수가 들어온다면 가변인수가 반드시 필요하지만, 역시나 단점은 존재한다. 최소한의 성능문제를 겪기 위해서는 메소드를 오버로딩해서 사용하는걸 추천한다.