---
layout: post
title: "Elixir - 18: Sigils"
date: 2016-02-11 12:58:58 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 18: Sigils

Elixir는 큰 따옴표를 사용하는 문자열과 작은 따옴표를 사용하는 문자열 목록을 제공합니다. 하지만 이는 언어에 있어서 문자열 표현을 처리하는 표면적인 구조일 뿐입니다. 예를 들어 아톰은 `:atom`이라는 표현식으로 생성됩니다.

Elixir의 목표중 하나는 확장성입니다. 이는 개발자는 어떤 도메인으로도 언어를 확장할 수 있어야 한다는 의미 입니다. 컴퓨터 과학은 점점 영역을 넓히고 있으며, 각 언어들은 자기 자신의 핵심 부분만으로는 많은 부분에 대한 문제를 모두 처리할 수 없습니다. 그보다는 언어를 확장 가능하게 설계하여, 개발자, 회사, 커뮤니티에서 자신들의 필요한 도메인으로 확장할 수 있게끔 만드는 것이 최선입니다.

이 장에서는 텍스트 표현을 처리하기 위한 방법을 제공하는 시길(Sigil)에 대해서 설명합니다. 시길은 물결 기호(`~`)로 시작하며, 구분 문자를 사용합니다. 때때로 마지막 구분 문자 뒤에 몇몇 수식자를 가질 수도 있습니다.

## 정규 표현

Elixir에서 가장 일반적인 시길은 `~r`이며, 이는 [정규표현](https://ko.wikipedia.org/wiki/정규_표현식)을 만들기 위해서 사용됩니다.

```iex
# A regular expression that matches strings which contain "foo" or "bar":
iex> regex = ~r/foo|bar/
~r/foo|bar/
iex> "foo" =~ regex
true
iex> "bat" =~ regex
false
```

Elixir는 [PCRE](http://www.pcre.org/) 라이브러리를 사용하여 Perl과 호환되는 정규 표현직을 지원합니다. 정규 표현식은 플래그 옵셕 역시 지원하며, 예를 들어, 다음과 같이 `i` 수식자는 정규 표현을 대소문자를 구분하지 않고 매칭하게 만들어 줍니다.

```iex
iex> "HELLO" =~ ~r/hello/
false
iex> "HELLO" =~ ~r/hello/i
true
```

더 많은 플래그나 지원하는 연산자를 확인하려면 [`Regex` 모듈](http://elixir-lang.org/docs/stable/elixir/Regex.html)을 읽어보세요.

지금까지의 예제에서는 `/`를 정규 표현식의 구분자로 사용하고 있었습니다, 그런데 시길은 이외에도 8가지의 구분자를 지원합니다.

```
~r/hello/
~r|hello|
~r"hello"
~r'hello'
~r(hello)
~r[hello]
~r{hello}
~r<hello>
```

이러한 다양한 구분자를 지원하는 이유는 다른 종류의 시길에 각각 맞는 구분자를 사용하기 위해서 입니다. 예를 들어서 정규 표현식에서는 소괄호를 사용하면 표현식을 읽을 때에 많은 혼란을 가져올 수 있습니다. 그러나 다음 절에서 살펴보게 될 시길에서는 소괄호가 유용하게 사용됩니다.

## 문자열, 문자 리스트, 단어 시길

Elixir는 정규 표현식이외에도 3개의 다른 시길을 지원합니다.

### 문자열

`~s` 시길은 큰 따옴표와 마찬가지로 문자열을 생성하기 위해서 사용됩니다. `~s` 시길은 문자열이 작은 따옴표, 큰 따옴표를 모두 포함하고 있는 경우에 유용합니다.

```iex
iex> ~s(this is a string with "double" quotes, not 'single' ones)
"this is a string with \"double\" quotes, not 'single' ones"
```

### 문자 리스트

`~c` 시길은 문자 리스트를 생성할 때 사용합니다.

```iex
iex> ~c(this is a char list containing 'single quotes')
'this is a char list containing \'single quotes\''
```

### 단어 리스트

`~w` 시길은 단어 리스트를 생성할 때에 유용합니다(**단어들**은 단순히 문자열입니다). `~w` 시길에는 문자열이 공백으로 연결됩니다.

```iex
iex> ~w(foo bar bat)
["foo", "bar", "bat"]
```

`~w` 시길은 `c`, `s` 그리고 `a` 수식자를 받으며(각각 문자 목록, 문자열과 아톰을 의미합니다), 이들은 결과를 어떤 데이터 타입으로 반환할지를 정의합니다.

```iex
iex> ~w(foo bar bat)a
[:foo, :bar, :bat]
```

## 시길에서의 보간과 이스케이프 처리

Elixir는 소문자 시길외에도 이스케이프와 보간 처리를 위한 대문자 시길을 지원하고 있습니다. 예를 들어 `~s`와 `~S`는 모두 문자열을 반환합니다만, 전자는 이스케이프와 보간을 지원하며, 후자는 그렇지 않습니다.

```iex
iex> ~s(String with escape codes \x26 #{"inter" <> "polation"})
"String with escape codes & interpolation"
iex> ~S(String without escape codes \x26 without #{interpolation})
"String without escape codes \\x26 without \#{interpolation}"
```

다음 이스케이프 코드는 문자열과 문자 리스트에서 사용할 수 있습니다.

* `\"` – double quote
* `\'` – single quote
* `\\` – single backslash
* `\a` – bell/alert
* `\b` – backspace
* `\d` - delete
* `\e` - escape
* `\f` - form feed
* `\n` – newline
* `\r` – carriage return
* `\s` – space
* `\t` – tab
* `\v` – vertical tab
* `\0` - null byte
* `\xDD` - character with hexadecimal representation DD (e.g., `\x13`)
* `\x{D...}` - character with hexadecimal representation with one or more hexadecimal digits (e.g., `\x{abc13}`)

시길은 Heredocs도 지원하며, 3개의 큰 따옴표, 또는 작은 따옴표를 사용할 수 있습니다.

```iex
iex> ~s"""
...> this is
...> a heredoc string
...> """
```

Heredoc 시길은 문서를 작성할 때에 사용되는 경우가 많으며, 예를 들어, 문서에서 이스케이프 문자를 사용하여 작성하게 되면, 두번 이스케이프해야하는 상황이 종종 있으므로 몇몇 문자를 에러를 만들기 쉽습니다.

```elixir
@doc """
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\\\"foo\\\"")
    "'foo'"

"""
def convert(...)
```

`~S`를 사용해서 이러한 문제를 회피할 수도 있습니다.

```elixir
@doc ~S"""
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\"foo\"")
    "'foo'"

"""
def convert(...)
```

## 커스텀 시길

이 장의 처음에 이야기 했던 대로, Elixir의 시길은 확장 가능합니다. 사실 `~r/foo/i`라는 시길은 `sigil_r`이라는 함수를 바이너리와 문자 리스트를 매개 변수로 호출한 것과 동등합니다.

```iex
iex> sigil_r(<<"foo">>, 'i')
~r"foo"i
```

그러므로 `~r` 시길의 문서를 `sigil_r` 함수를 통해서도 접근할 수 있습니다.

```iex
iex> h sigil_r
...
```

`sigil_{identifier}` 패턴을 따라서 함수를 구현하는 것으로 자신만의 시길을 만들 수 있습니다. 예를 들어서, 정수를 반환하는 (`n` 옵션으로 음의 정수로 만들어주기도 하는) `~i`라는 시길을 만들어보죠.

```iex
iex> defmodule MySigils do
...>   def sigil_i(string, []), do: String.to_integer(string)
...>   def sigil_i(string, [?n]), do: -String.to_integer(string)
...> end
iex> import MySigils
iex> ~i(13)
13
iex> ~i(42)n
-42
```

시길은 매크로외 함께 컴파일 시간에 작업을 할 때에도 유용합니다. 예를 들어, Elixir의 정규 표현식은 컴파일하는 동안에 좀 더 효율적인 표현으로 변환되어 소스 코드에 삽입되므로, 런타임에는 이러한 부분을 생략할 수 있습니다. 이 주제에 대해서 더 관심이 있다면, `Kernel` 모듈에서 어떻게 시길이 구현되어 있는지, 매크로가 어떻게 동작하는지에 대해서 알아보기를 추천합니다.

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir Sigils](http://elixir-lang.org/getting-started/sigils.html)
