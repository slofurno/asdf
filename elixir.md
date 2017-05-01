## Testing (easily mockable)
- ExUnit
  - can run tests async, so it's fast

- functional code can be easier to test
  - keep our logic in functions with no mutable state

- create mock then use config + enviroment (prod, test, etc) to inject

- can explicitly define interfaces with behaviours

    ```elixir
    callback player_history(integer) :: {:ok, [%DotaMatch{}]} | {:error, String.t}
    callback match_details(integer) :: {:ok, %MatchDetails{}} | {:error, String.t}
    ```

## Learning curve (is it easy to pick up as a beginner?)
- good documentation/ intro guides
- familiar syntax (rubish)

## Easily dockerizable (what is running in the container?)
- use distillary to build releases
- edib automates building a release in docker + making final docker image
- erlang runtime + all app bytecode + alpine linux = 25mb

## Concurrency Performance (how are requests handled concurrently?)
- erlang processes (like goroutines)

## Error model/handling (what is the standard approach?)
- return + match on tuples

    ```elixir
    case some_function(x) do
      {:ok, result} -> IO.inspect(result)
      {:error, error} -> IO.inspect(error)
    end
    ```

- with clause to catch errors in a chain of calls

    ```elixir 
    with {:ok, matches} <- HistoryFetcher.player_history(67760037),
      {:ok, id} <- MatchUtils.first_match_id(matches),
        {:ok, %{players: players}} = HistoryFetcher.match_details(id) do
        send_resp(conn, 200, Poison.encode!(players))
      else
        {:error, reason} -> send_resp(conn, 404, reason)
      end
    ```

- or, just make assertion + cause exception
  - let it crash
  - crashing a process won't bring down runtime

## JSON transformations-fluency (what easy/difficult is to ingest/transform JSON?)
- by default decodes into a map
- can decode directly into structs
- need to specify typespec of nested struct relationships while decoding

    ```elixir
    Poison.decode!(body, as: %{"result" => %MatchHistoryResponse{matches: [%DotaMatch{players: [%DotaMatchPlayer{}]}]}})
    ```
- in this example:
	- result is type #MatchHistoryResponse 
  - matches is a list of #DotaMatch
  - DotaMatch.player is type #DotaMatchPlayer

## Restful Design / Framework (what are the common frameworks/libraries available for writing RESTful APIs?)
- Plug
- a pipeline of functions which act on our conn state
	- checking for auth tokens, etc
- url parameters in scope, functions to get headers, etc. from conn state

    ```elixir
    get "/api/something/:id" do
      send_resp(conn, 200, "id = #{id}")
    end
    ```

## Dependency Mgmt. (what’s the module ecosystem like? Ballpark # on third-party modules that we would need)
- lock file for versioning
- hex package manager
- can also bring in deps from git repos
- can specify version/tag

## Profiling (how can we get visibility into resource-usage so that our code is more performant?)
- erlang comes with a lot of profiling tools
  - time spent in each function
  - memory usage per process
- square enix built a web frontend for profiling
  -	https://github.com/shinyscorpion/wobserver

## Security (framework-level)

## Documentation generation (Built-in or third-party?)
- inline heredocs for modules + functions
- ex_Doc builds html docs from annotations + typespecs

## Stability (Is the language stable? What version? Answers to questions on SO?)
- maintainers are active on SO
- http://theerlangelist.com/
- people blog about how they use elixir in production
- and opensource the tools they build
