---
layout: post
title: "Elixir - 17: Comprehensions"
date: 2016-02-11 12:58:37 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 17: Comprehensions

Elixir에서는 Enumarable을 사용해서 결과를 필터링하고 사상으로 새로운 목록을 만드는 것은 흔한 일입니다. Comprehension은 그러한 구조를 위한 문법적인 설탕입니다. 그러한 일반적인 작업들을 `for` 라는 특별한 구조를 통해서 한번에 관리할 수 있습니다.

예를 들어서, 목록에 있는 정수들을 제곱수로 바꿀 수 있습니다.

```iex
iex> for n <- [1, 2, 3, 4], do: n * n
[1, 4, 9, 16]
```

Comprehension은 제너레이터, 필터, 컬렉터블로 구성됩니다.

## 제너레이터와 필터

위의 표현은 `n <- [1, 2, 3, 4]`는 **제너레이터**라고 부릅니다. 제너레이터는 Comprehension에서 사용할 값을 생성하는 역할을 합니다. 어떤 열거 가능한 객체를 제너레이터 표현의 우측에 넘길 수 있습니다.

```iex
iex> for n <- 1..4, do: n * n
[1, 4, 9, 16]
```

제너레이터 역시 패턴 매칭을 좌측에서 지원합니다. 패턴 매칭되지 않는 것들은 모두 **무시**됩니다. 범위 객체 대신에 아톰으로 `:good`이나 `:bad`를 가지는 키워드 목록을 넘기고, `:good`의 값만의 제곱수를 구하고 싶다고 해봅시다.

```iex
iex> values = [good: 1, good: 2, bad: 3, good: 4]
iex> for {:good, n} <- values, do: n * n
[1, 4, 16]
```

필터를 패턴 매칭 대신에 특정 요소를 필터링하기 위해서 사용할 수 있습니다. 예를 들어서 모든 3의 배수를 골라내고, 남아있는 값들을 제곱할 수도 있습니다.

```iex
iex> multiple_of_3? = fn(n) -> rem(n, 3) == 0 end
iex> for n <- 0..5, multiple_of_3?.(n), do: n * n
[0, 9]
```

Comprehension는 필터 표현식에서 `false`나 `nil`를 반환하는 값들을 배제합니다. 나머지 값은 모두 유지합니다.

Comprehension는 `Enum`과 `Stream` 모듈에서 비교 함수를 사용하는 것에 비해서 좀 더 일반적이고 정밀한 표현을 제공해줍니다. 폴더의 목록을 받아서 각 폴더들의 크기를 계산해주는 예제를 봅시다.

```elixir
for dir  <- dirs,
    file <- File.ls!(dir),
    path = Path.join(dir, file),
    File.regular?(path) do
  File.stat!(path).size
end
```

다수의 제너레이터를 사용하여 카르테시안 곱을 계산할 수도 있습니다.

```iex
iex> for i <- [:a, :b, :c], j <- [1, 2], do:  {i, j}
[a: 1, a: 2, b: 1, b: 2, c: 1, c: 2]
```

여러 개의 제너레이터와 필터를 사용하여 피타고라스의 정리를 만족하는 값을 찾아보죠. 피타고라스의 삼각형은 `a*a + b*b = c*c`를 만족하는 양의 정수들의 집합입니다. `triple.exs`에 Comprehension을 사용하여 코드를 작성해봅시다.

```elixir
defmodule Triple do
  def pythagorean(n) when n > 0 do
    for a <- 1..n,
        b <- 1..n,
        c <- 1..n,
        a + b + c <= n,
        a*a + b*b == c*c,
        do: {a, b, c}
  end
end
```

그럼 터미널에서 실행해보죠.

```bash
iex triple.exs
```

```iex
iex> Triple.pythagorean(5)
[]
iex> Triple.pythagorean(12)
[{3, 4, 5}, {4, 3, 5}]
iex> Triple.pythagorean(48)
[{3, 4, 5}, {4, 3, 5}, {5, 12, 13}, {6, 8, 10}, {8, 6, 10}, {8, 15, 17},
 {9, 12, 15}, {12, 5, 13}, {12, 9, 15}, {12, 16, 20}, {15, 8, 17}, {16, 12, 20}]
```

이 코드는 범위 객체를 사용한 검색이기 때문에 비용이 꽤나 비쌉니다. 그리고 `{b, a, c}` 튜플이 `{a, b, c}`와 동일함에도 불구하고 이 함수에서는 중복된 트리플을 호출하고 있습니다. 이제 이 Comprehension을 좀 더 최적화하고 예를 들어 다음과 같은 제너레이터를 사용해서 변수들을 참조하여 중복된 결과들을 제거해봅시다.

```elixir
defmodule Triple do
  def pythagorean(n) when n > 0 do
    for a <- 1..n-2,
        b <- a+1..n-1,
        c <- b+1..n,
        a + b + c <= n,
        a*a + b*b == c*c,
        do: {a, b, c}
  end
end
```

마지막으로 Comprehension에서의 변수 할당은 제너레이터, 필터, 블록 내부에서만 사용될 뿐, 외부에 영향을 주지 않는다는 점을 기억하세요.

## 비트스트링 제너레이터

비트스트링 제너레이터 역시 지원되며, 비트스트링 스트림을 이해하는데에 있어서 무척 유용합니다. 다음 예제에서는 바이너리로부터 RGB값을 받아와서 픽셀들의 리스트를 만들고, 이 값들을 각각으로 분리하는 튜플로 변환합니다.

```iex
iex> pixels = <<213, 45, 132, 64, 76, 32, 76, 0, 0, 234, 32, 15>>
iex> for <<r::8, g::8, b::8 <- pixels>>, do: {r, g, b}
[{213, 45, 132}, {64, 76, 32}, {76, 0, 0}, {234, 32, 15}]
```

비트스트링 제너레이터는 일반적인 열거가능한 제너레이터와 함께 사용할 수 있으며, 필터 역시 문제없이 사용할 수 있습니다.

## `:into` 옵션

위의 모든 예제에서는 Comprehension는 그 결과로 리스트를 반환합니다. 그런데, Comprehension의 결과는 `:into`를 사용해서 또다른 구조에 삽입될 수 있습니다.

예를 들어서 비트스트링 제너레이터 `:into` 옵션과 함께 사용하여 문자열에 있는 공백을 손쉽게 제거할 수 있습니다.

```iex
iex> for <<c <- " hello world ">>, c != ?\s, into: "", do: <<c>>
"helloworld"
```

집합이나 맵과 다른 사전 역시 `:into` 옵션을 통해서 주어질 수 있습니다. 사실, `:into`는 `Collectable` 프로토콜을 지원하는 모든 데이터 구조를 받을 수 있습니다.

`:into`의 일반적인 사용법은 키를 손대지 않고 맵에 있는 값을 변환하는 것입니다.

```iex
iex> for {key, val} <- %{"a" => 1, "b" => 2}, into: %{}, do: {key, val * val}
%{"a" => 1, "b" => 4}
```

스트림을 사용하는 다른 예제를 한번 봅시다. `IO` 모듈(`Enumerable`과 `Collectable`을 구현하고 있으므로)은 스트림을 제공하기 때문에 어떤 문자열을 입력하더라도 터미널이 대문자로 변환된 결과를 돌려주는 Comprehension을 만들 수 있습니다.

```iex
iex> stream = IO.stream(:stdio, :line)
iex> for line <- stream, into: stream do
...>   String.upcase(line) <> "\n"
...> end
```

이제 터미널에 아무 문자열이나 입력해보고, Comprehension을 지나온 값이 어떻게 돌아오는지 살펴보세요. 그리고 `Ctrl+C`를 두번 눌러서 끕시다. :)

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir Comprehensions](http://elixir-lang.org/getting-started/comprehensions.html)
