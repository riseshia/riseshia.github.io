---
layout: post
title: "Elixir - 03: Basic operators"
date: 2016-01-10 08:28:34 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 03: Basic operators

## Basic operators

### `++`, `--`

저번 화에서 이미 보셨을 리스트를 조작하는 연산자.

```iex
iex> [1,2,3] ++ [4,5,6]
[1,2,3,4,5,6]
iex> [1,2,3] -- [2]
[1,3]
```

### `<>`

문자열 연결할 때에 사용하는 연산자.

```iex
iex> "foo" <> "bar"
"foobar"
```

혹시나 싶어서 `+`를 호출해봤는데 안됩니다.

```iex
iex> "foo" + "bar"
** (ArithmeticError) bad argument in arithmetic expression
    :erlang.+("foo", "bar")
```

### `or`, `and`, `not`

Boolean 연산자. 첫번째 인수로 반드시 진리값(`true`, `false`)을 넘기길 요구합니다. 왜일까요?

```iex
iex> true and true
true
iex> false or is_atom(:example)
true
iex> 1 and true
** (ArgumentError) argument error
```

`or`, `and`는 short-circuit 연산을 합니다.

```iex
iex> false and raise("This error will never be raised")
false

iex> true or raise("This error will never be raised")
true
```

Erlang을 쓰지는 않지만, `and`, `or`는 Erlang의 `andalso`와 `orelse`와 각각 동일하다고 합니다. 예약어가 참 신박하네요.

### `||`, `&&`, `!`

Boolean 연산자와 비슷한 동작을 하는 이런 연산자도 있습니다. 어떤 타입의 데이터라도 사용할 수 있으며, `false`와 `nil`을 제외한 모든 값이 참으로 평가됩니다.

```iex
# or
iex> 1 || true
1
iex> false || 11
11

# and
iex> nil && 13
nil
iex> true && 17
17

# !
iex> !true
false
iex> !1
false
iex> !nil
true
```

고로 진리값이 아닌 경우에는  `||`, `&&`, `!`를 사용해야 합니다. 개인적으로는 왜 두가지를 따로 구분하는지 의문이 듭니다만...

### `==`, `!=`, `===`, `!==`, `<=`, `>=`, `<`, `>`

이런 비교 연산자들도 있습니다.

```iex
iex> 1 == 1
true
iex> 1 != 2
true
iex> 1 < 2
true
```

`==`와 `===`의 차이는 타입 검사까지 하느냐 안하느냐. js의 그것과 동일하네요.

특이한 부분은 다른 타입의 데이터끼리 **대소 비교**를 할 수 있다는 점입니다.

```iex
iex> 1 < :atom
true
```

이는 정렬 알고리즘이 다른 데이터 타입을 고민하지 않도록 만들기 위한, 극히 실용적인 이유에서 기인합니다. 타입간의 순서는 다음과 같습니다.

    number < atom < reference < functions < port < pid < tuple < maps < list < bitstring

## Operator table

Elixir의 전체 연산자 표. 위가 우선순위가 높고, 아래가 낮습니다.

Operator | Associativity
-------- | -------------
 `@` | Unary
 `.` | Left to right
 `+` `-` `!` `^` `not` `~~~` | Unary
 `*` `/` | Left to right
 `+` `-` | Left to right
 `++` `--` `..` `<>` | Right to left
 `in` | Left to right
 <code>&#124;></code> `<<<` `>>>` `~>>` `<<~` `~>` `<~` `<~>` <code>&lt;&#124;&gt;</code>  | Left to right
 `<` `>` `<=` `>=` | Left to right
 `==` `!=` `=~` `===` `!==` | Left to right
 `&&` `&&&` `and` | Left to right
 <code>&#124;&#124;</code> <code>&#124;&#124;&#124;</code> `or` | Left to right
 `=` | Right to left
 `=>` | Right to left
 <code>&#124;</code> | Right to left
 `::` | Right to left
 `when` | Right to left
 `<-`, `\\` | Left to right
 `&` | Unary

당연하다면 당연하지만 `&&`랑 `and`의 우선순위가 같은게 눈에 띄네요.

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir Basic operators](http://elixir-lang.org/getting-started/basic-operators.html)
