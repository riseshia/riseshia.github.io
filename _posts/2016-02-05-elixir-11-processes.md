---
layout: post
title: "Elixir - 11: Processes"
date: 2016-02-05 03:28:10 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 11: Processes

Elixir에서는 모든 코드가 프로세스 내부에서 동작합니다. 프로세스는 고립되어 있으며, 서로 동시에 동작하고, 메시지를 통해서 서로 통신합니다. 프로세스는 Elixir에서 동시성을 지원하는 기본일 뿐만 아니라, fault-tolerant하고 분산된 구조를 제공해줍니다.

Elixir의 프로세스는 OS 시스템의 프로세스와 다릅니다. Elixir의 프로세스는 (다른 언어에서의 스레드와는 다르게) 메모리/CPU 사용량이 무척 가벼우며, 이 때문에 수십, 수백개의 프로세스를 동시에 돌리는 것을 흔하게 볼 수 있습니다.

이번 장에서는 새 프로세스를 생성하는 법과 서로 다른 프로세스 간에 메시지를 주고 받는 법에 대해서 배워보겠습니다.

## `spawn`

새로운 프로세스는 일반적으로 `spawn/1` 함수를 통해서 생성됩니다.

```iex
iex> spawn fn -> 1 + 2 end
#PID<0.43.0>
```

`spawn/1`은 다른 프로세스에서 실행될 하나의 함수를 받습니다.

`spawn/1`는 PID(프로세스 식별자)를 반환합니다. 이미 이 시점에서 생성된 프로세스는 이미 죽어있을 겁니다. 생성된 프로세스는 주어진 함수를 실행한 뒤, 종료됩니다.

```iex
iex> pid = spawn fn -> 1 + 2 end
#PID<0.44.0>
iex> Process.alive?(pid)
false
```

`self/0`를 호출하여 현재 프로세스의 PID를 얻어올 수 있습니다.

```iex
iex> self()
#PID<0.41.0>
iex> Process.alive?(self())
true
```

프로세스는 메시지를 주고 받기 시작하면 좀 더 흥미로워집니ㄷ...는 가르치는 사람의 희망사항이겠지...!

## `send`와 `receive`

`send/2`를 사용해서 메시지를 보내고 `receive/1`를 통해 메시지를 받을 수 있습니다.

```iex
iex> send self(), {:hello, "world"}
{:hello, "world"}
iex> receive do
...>   {:hello, msg} -> msg
...>   {:world, msg} -> "won't match"
...> end
"world"
```

메시지가 프로세스로 전송되면, 메시지는 프로세스의 메일 상자에 저장됩니다. `receive/1`의 블럭은 현재 프로세스의 메일 상자를 확인하여 매칭하는 패턴을 가지고 있는 메시지를 찾습니다. `receive/1` 역시 함수 절과 `case/2`와 같은 가드를 지원하므로 코드를 간결하게 작성할 수 있습니다.

메일상자에 있는 메시지에 매칭하는 패전이 없다면, 현제 프로세스는 매칭하는 메시지가 올 때까지 기다리게 됩니다. 타임아웃을 설정할 수도 있습니다.

```iex
iex> receive do
...>   {:hello, msg}  -> msg
...> after
...>   1_000 -> "nothing after 1s"
...> end
"nothing after 1s"
```

타임아웃을 0으로 설정할 때에는 메시지가 이미 메일박스에 있을 거라고 기대합니다.

이를 모두 포함하는 예제를 보죠.

```iex
iex> parent = self()
#PID<0.41.0>
iex> spawn fn -> send(parent, {:hello, self()}) end
#PID<0.48.0>
iex> receive do
...>   {:hello, pid} -> "Got hello from #{inspect pid}"
...> end
"Got hello from #PID<0.48.0>"
```

쉘에서 코드를 작성하고 있다면 `flush/0`도 꽤 유용할 겁니다. 이 함수는 현재 메일 상자에 있는 모든 메시지를 출력하고 버립니다.

```iex
iex> send self(), :hello
:hello
iex> flush()
:hello
:ok
```

## Links

Elixir에서 프로세스를 만드는 가장 흔한 방법은 `spawn_link/1`입니다. 우선 예제를 보기 전에, 프로세스가 실패하면 어떻게 되는지를 살펴보죠.

```iex
iex> spawn fn -> raise "oops" end
#PID<0.58.0>

[error] Error in process <0.58.0> with exit value: ...
```

에러를 출력합니다만, 프로세스를 생성한 쪽의 프로세스는 여전히 동작합니다. 이는 프로세스들이 각각 독립되어 있기 때문입니다. 만약 한 프로세스가 실패했을 경우, 이 실패를 다른 프로세스에 전파하고 싶다면 프로세스들을 연결할 필요가 있으며, 이는 `spawn_link/1`를 통해서 처리할 수 있습니다.

```iex
iex> spawn_link fn -> raise "oops" end
#PID<0.41.0>

** (EXIT from #PID<0.41.0>) an exception was raised:
    ** (RuntimeError) oops
        :erlang.apply/2
```

쉘에서 문제가 발생하면, 쉘은 자동으로 이 실패를 감지하고 잘 포장하여 보여줍니다. 실제로 어떤 일이 발생하는지를 알아보기위해 소스 코드 형태로 `spawn_link/1`를 사용한 뒤에 이를 실행해보죠.

```iex
# spawn.exs
spawn_link fn -> raise "oops" end

receive do
  :hello -> "let's wait until the process fails"
end
```

이번에는 프로세스가 죽으면서 연결되어 있는 부모 프로세스를 같이 죽입니다. `spawn_link/1`을 사용하지 않고 `Process.link/1`를 사용해서 수동으로 연결할 수도 있습니다. 이처럼 프로세스들에게 제공되는 함수들을 확인하기 위해서 [`Process` 모듈](http://elixir-lang.org/docs/stable/elixir/Process.html)에 대한 설명을 읽어두기를 추천합니다.

프로세스와 링크는 fault-tolerant한 시스템을 만드는 데에 무척 중요한 역할을 합니다. Elixir 애플리케이션에서는 슈퍼바이저가 어떤 프로세스가 죽으면 이를 확인하고 다시 시작할 수 있게끔 연결해두는 일이 많습니다. 이는 프로세스들이 전부 고립되어 있으며, 기본적으로는 서로 아무것도 공유하지 않기에 가능한 것입니다.

다른 언어에서는 예외를 잡고/처리하기를 요구합니다만, Elixir에서는 그냥 프로세스가 실패하게 만들도록 합니다. 왜냐하면 슈퍼바이저가 그 프로세스를 다시 시작하기를 기대하기 때문이죠. "빠르게 실패하기"는 Elixir 소프트웨어를 작성하는 일반적인 철학입니다.

`spawn/1`과 `spawn_link/1`은 Elixir에 있어 가장 기본적인 프로세스를 생성하는 방식입니다. 이를 열심히 가르쳐드렸지만, 많은 경우에는 이들을 추상화한 함수를 사용할 겁니다. 그중 하나인 태스크를 살펴보죠.

## Tasks

태스크는 좀 더 나은 에러 리포트와 관리를 위해서 spawn 함수들을 감싸고 있습니다.

```iex
iex(1)> Task.start fn -> raise "oops" end
{:ok, #PID<0.55.0>}

15:22:33.046 [error] Task #PID<0.55.0> started from #PID<0.53.0> terminating
Function: #Function<20.90072148/0 in :erl_eval.expr/5>
    Args: []
** (exit) an exception was raised:
    ** (RuntimeError) oops
        (elixir) lib/task/supervised.ex:74: Task.Supervised.do_apply/2
        (stdlib) proc_lib.erl:239: :proc_lib.init_p_do_apply/3
```

`spawn/1`과 `spawn_link/1` 대신에 우리는 `Task.start/1`와 `Task.start_link/1`를 사용하여 단순히 PID를 돌려받는 대신에 `{:ok, pid}`를 돌려받을 수 있습니다. 이를 통해서 프로세스를 좀 더 편리하게 관리할 수 있습니다. 나아가서 태스크는 `Task.async/1`이나 `Task.await/1`와 같이 분산처리를 용이하게 만드는 여러가지 편의 함수를 제공합니다.

이러한 기능들에 대해서는 ***Mix and OTP guide***에서 알아볼 예정이니, 지금은 태스크를 사용하면 좀 더 나은 에러 보고가 가능하다는 점만 기억해두면 충분합니다.

## State

이제 상태에 대해서 이야기해볼 시간입니다. 예를 들어, 애플리케이션의 상태를 저장하거나, 파일을 파싱하여 메모리에 저장하는 등, 상태를 요구하는 애플리케이션을 작성하는 경우에는 어떻게 해야할까요?

프로세스는 이러한 질문에 대한 일반적인 대답이 될 수 있습니다. 무한히 반복되는 프로세스를 작성하여 여기에 상태를 저장하고, 메시지를 통해서 이 정보를 주고 받을 수 있습니다. `kv.exs`라는 파일에 키-값 저장소처럼 동작하는 프로세스를 생성하는 모듈을 작성해봅시다.

```elixir
defmodule KV do
  def start_link do
    Task.start_link(fn -> loop(%{}) end)
  end

  defp loop(map) do
    receive do
      {:get, key, caller} ->
        send caller, Map.get(map, key)
        loop(map)
      {:put, key, value} ->
        loop(Map.put(map, key, value))
    end
  end
end
```

여기에서는 `start_link` 함수가 빈 맵으로 `loop/1`를 실행하는 새로운 프로세스를 생성합니다. 그러면 `loop/1` 함수는 적절한 메시지가 올 때까지 기다립니다. `:get` 메시지의 경우, 호출한 사람에게 메시지를 돌려준 뒤에 `loop/1` 를 다시 호출하여 새로운 메시지를 기다립니다. 반면 `:put` 메시지는 넘겨 받은 `key`와 `value`를 추가한 새로운 맵과 함께 `loop/1`을 호출합니다.

그럼 `iex kv.exs`를 한번 실행해봅시다.

```iex
iex> {:ok, pid} = KV.start_link
#PID<0.62.0>
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
nil
```

처음에는 프로세스의 맵은 비어있기 때문에, `:get` 메시지를 보내고 메일 상자를 출력해보면 `nil`이 반환됩니다. 그럼 이제 `:put` 메시지를 보내봅시다.

```iex
iex> send pid, {:put, :hello, :world}
#PID<0.62.0>
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
:world
```

프로세스가 어떻게 상태를 유지하고, 이 정보를 메시지를 통해서 어떻게 가져오거나 변경할 수 있는지를 확인하세요. `pid`를 알고 있는 어떤 프로세스든 메시지를 통해서 상태를 변경할 수 있습니다.

물론 `pid`를 이름과 함께 등록하여, 이름을 아는 모두가 메시지를 보낼 수 있도록 만들 수 있습니다.

```iex
iex> Process.register(pid, :kv)
true
iex> send :kv, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
:world
```

프로세스를 사용한 상태 관리나, 이름 등록은 Elixir에서는 무척 일반적인 사용 방식입니다. 하지만 많은 경우 이를 위에서처럼 수작업으로 구현하지 않고 Elixir에 존재하는 수많은 구현체를 사용하게 됩니다. 예를 들어, Elixir는 상태와 관련된 간단한 처리를 추상화해 둔 [에이전트](http://elixir-lang.org/docs/stable/elixir/Agent.html)를 제공합니다.

```iex
iex> {:ok, pid} = Agent.start_link(fn -> %{} end)
{:ok, #PID<0.72.0>}
iex> Agent.update(pid, fn map -> Map.put(map, :hello, :world) end)
:ok
iex> Agent.get(pid, fn map -> Map.get(map, :hello) end)
:world
```

`:name` 옵션을 `Agent.start_link/2`에 넘기면 자동으로 등록됩니다. 에이전트에서 더 나아가서 Elixir는 일반적인 서버(보통 GenServer), 에이전트, 태스크 등, 프로세스를 기반으로 동작하는 것들을 만들기 위한 API를 제공합니다. 이러한 것들은 관리 트리와 마찬가지로 완전한 Elixir 애플리케이션은 처음부터 끝까지 만들어보는 ***Mix and OTP guide***에서 좀 더 자세히 다룰 예정입니다.

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir Processes](http://elixir-lang.org/getting-started/processes.html)
