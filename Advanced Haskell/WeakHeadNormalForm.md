# 지연 평가

Haskell은 **지연 평가(lazy evaluation)**라는 평가 전략을 취하는 언어입니다. 대부분의 언어들이 적극 평가(eagar evaluation) 전략을 취하기 때문에 지연 평가라는 개념이 대부분의 사람들에게는 상당히 생소하게 느껴질거에요. 지연 평가, 적극 평가가 무슨 의미인지 정확하게 이해하기 위해서는 우선 **축약(reduction)**이라는 개념을 알아야합니다.

## 축약(reduction)

축약은 쉽게 말해서 어떤 계산식을 더 단순한 형태로 변환하는 것을 말합니다. 예를 들어 아래의 람다 식이 있다고 해봅시다.

```Haskell
\x y -> x + y
```

이 때 이 함수에 2와 3이라는 인자를 적용하는 다음 식에 대해,

```Haskell
(\x y -> x + y) 2 3
```

x와 y를 각각 2와 3으로 치환하면 다음과 같은 식이 나오겠죠.

```Haskell
2 + 3
```

이렇게 람다식에 어떤 인자를 넘기는 표현식이 있을 때, 그 인자 값을 람다 내부에 적용하여 다른 형태의 표현식으로 바꾸는 것을 축약(reduction)이라고 부릅니다. 그리고 표현식에 대해 축약을 반복하다보면 언젠가는 축약이 불가능한 식이 나타나죠. 예를 들어 위의 식의 경우 2 + 3을 다시 축약하면 5가 될 것이고, 이 5는 더 이상 축약할 수 없는 표현식입니다. 이런 표현식을 ```정규형(normal from)```이라고 부릅니다.

## 축약 순서

어떤 식을 축약한다고 할 때 그 순서는 여러 가지가 존재할 수 있습니다. 아래의 식을 예로 들어보죠.

```Haskell
(\x y -> x + y) ((\x -> 2*x) 3) ((\x -> 3*x) 5)
```

이 식을 맨 오른쪽, 맨 안쪽에 있는 것부터 순서대로 축약해나간다고 해봅시다. 그럼 축약은 아래와 같은 순서로 일어나게 될 겁니다.

```Haskell
(\x y -> x + y) ((\x -> 2*x) 3) ((\x -> 3*x) 5)
=> ((\x -> 3*x) 5) 축약
(\x y -> x + y) ((\x -> 2*x) 3) (3*5)
=> 3*5 축약
(\x y -> x + y) ((\x -> 2*x) 3) 15
=> ((\x -> 2*x) 3) 축약
(\x y -> x + y) (2*3) 15
=> 2*3 축약
(\x y -> x + y) 6 15
=> (\x y -> x + y) 6 15 축약
6+15
=>6+15 축약
21
```

반면에 맨 왼쪽, 맨 바깥쪽에 있는 것부터 순서대로 축약해나간다고 해봅시다. 그럼 축약은 아래와 같은 순서로 일어나게 될겁니다.

```Haskell
(\x y -> x + y) ((\x -> 2*x) 3) ((\x -> 3*x) 5)
=> (\x y -> x + y) ((\x -> 2*x) 3) ((\x -> 3*x) 5) 축약
((\x -> 2*x) 3) + ((\x -> 3*x) 5)
=> ((\x -> 2*x) 3) 축약
(2*3) + ((\x -> 3*x) 5)
=> ((\x -> 3*x) 5) 축약
(2*3) + (3*5)
=> 2*3 축약
6 + (3*5)
=> 3*5 축약
6 + 15
=> 6+15 축약
21
```

최종적인 결과는 같지만 축약하는 순서에 따라 연산 과정이 많이 달라질 수 있다는 것을 알 수 있습니다. 이 축약하는 순서의 차이가 바로 평가전략의 차이가 됩니다. 이 예제에서 전자가 적극 평가에 해당하는 축약 순서이고, 후자가 바로 지연 평가에 해당하는 축약 순서입니다.

 적극 평가는 맨 안쪽에 있는 표현식부터 계산하기 때문에 어떤 함수의 호출 시점에서 그 함수에 인자로 넘어가는 값들은 모두 이미 계산이 된 상태(정규형)이라는 특징이 있습니다. 반면 지연 평가는 맨 바깥쪽에 있는 표현식부터 계산하기 때문에 그 함수에 인자로 넘어가는 값들이 반드시 계산이 되어 있지 않아도 되죠.

# Weak Head Normal Form

Haskell에서는 식을 평가할 때 식이 **Weak Head Normal Form(WHNF)**의 형태가 될 때까지만 평가합니다. WHNF는 다음과 같은 형태를 가진 표현식을 의미합니다.

 * 식의 맨 바깥쪽이 값 생성자인 식.
 * 식의 맨 바깥쪽이 람다 표현식인 식.

따라서 다음 식들은 WHNF입니다.

```Haskell
(\x -> x + 2) -- 식의 맨 바깥이 람다 표현식이므로 WHNF.
Just (2+3) -- 식의 맨 바깥쪽이 값 생성자(Just)이므로 WHNF.
'L' : ("am" ++ "bda") -- 식의 맨 바깥쪽이 값 생성자(:)이므로 WHNF.
```

반면에 다음 식들은 WHNF가 아닙니다.

```Haskell
(\x -> x + 2) 3 -- 함수에 어떤 인자값을 적용하는 식이므로 WHNF가 아닙니다.
1 + 2 -- 역시 함수(+)에 어떤 인자값을 적용하는 식이므로 WHNF가 아닙니다.
"Lam" ++ "bda" -- 역시 함수(++)에 어떤 인자값을 적용하는 식이므로 WHNF가 아닙니다.
```

Haskell은 식들이 WHNF의 형태가 될 때까지만 평가한 다음, 필요가 없다면 보조 표현식 부분은 아예 평가하지 않습니다. 예를 들어 다음과 같은 표현식이 있다고 합시다.

```Haskell
isJust (Just (1+3))
```

`isJust` 함수는 인자로 들어온 `Maybe` 값이 `Just something`이면 `True`, 아니면 `False`를 반환합니다. 여기서 이 식 자체는 WHNF가 아니죠. 그러니 `isJust` 함수를 평가합니다. 이 때, `Just (1+3)` 값은 WHNF이고, `isJust` 함수는 `Just` 내부에 어떤 값이 들어가있는지는 신경쓰지 않는 함수입니다. 따라서 `1+3` 식은 아예 계산되지 않고 `True` 값을 반환하게 됩니다.

 WHNF는 위의 예시처럼 어떤 값에 대해 데이터 구조에 대한 평가와 값에 대한 평가를 분리하는 효과를 갖고 있습니다. 식의 맨 바깥쪽에 값 생성자가 있다면 그 내부의 표현식(보조 표현식)들은 일단 평가하지 않고 값을 넘겨주기 때문에, 내부 표현식의 값이 필요할 때만 그 값을 계산하고 그렇지 않을 때에는 계산을 하지 않게 되므로 데이터 구조에 대한 평가(값 생성자가 뭔지)와 값 자체에 대한 평가(보조 표현식의 최종 계산값)를 분리해서 할 수 있게 되는 거죠.

 이런식으로 Haskell이 식을 평가할 때 WHNF까지만 평가하는 전략을 취하기 때문에 무한대 크기의 리스트 등을 다룰 수 있게 되는 거지요. 이런 내용을 이해하고 있다면 Haskell의 코드가 내부적으로 어떻게 동작할 지 좀 더 정확하게 이해할 수 있습니다.