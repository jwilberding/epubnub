README
======
Author: Tristan Sloughter tristan.sloughter@gmail.com
website: http://blog.erlware.org

Version: 0.0.4

Quick Start
-----------

.Dependencies

Install agner (http://erlagner.org/):

```bash
$ curl https://raw.github.com/agner/agner/master/scripts/oneliner | sh
```

```bash
$ ./deps
```

```bash
        $ ./sinan shell
        Erlang R14B01 (erts-5.8.2) [source] [smp:2:2] [rq:2] [async-threads:0] [hipe] [kernel-poll:false]

        Eshell V5.8.2  (abort with ^G)
        1> starting: depends
        starting: build
        Building /Users/tristan/Devel/epubnub/src/epubnub.erl
        Building /Users/tristan/Devel/epubnub/src/epubnub_sup.erl
        Building /Users/tristan/Devel/epubnub/src/epubnub_app.erl
        starting: shell
        Eshell V5.8.2  (abort with ^G)
        1> epubnub_app:start_deps().
        ok
        2> epubnub:publish("hello_world", <<"hello">>).
        [1,<<"D">>]
        3> epubnub:history("hello_world", 10).
        [<<"hello">>,<<"hello">>,<<"hello">>,<<"hello">>,
         <<"hello">>,<<"hello">>,<<"hello">>,<<"hello">>,<<"hello">>,
         <<"hello">>]
        3> epubnub:time().
        3588286186558
        4> epubnub:subscribe("chat", fun(X) -> io:format("~p~n", [X]) end).
        ....
        BREAK: (a)bort (c)ontinue (p)roc info (i)nfo (l)oaded
               (v)ersion (k)ill (D)b-tables (d)istribution
        a
```

Examples
--------

src/epn_example_client has a simple example of a gen_server subscribing to a channel. The init function takes an EPN record,
created with epubsub:new() in start_link and requests to subscribe to the "hello_world" channel and sends its PID with the
self() function so new messages are sent to this process.

```erlang

init([EPN]) ->
    {ok, PID} = epubnub_sup:subscribe(EPN, "hello_world", self()),
    {ok, #state{pid=PID}}.

```

Since currently the subscribe loop sends the message with the bang (!) the gen_server will handle the new message in handle_info.
The example simply prints out the contents of the message:

```erlang

handle_info({message, Message}, State) ->
    io:format("~p~n", [Message]),
    {noreply, State}.

```

To unscribe we call the module stop function that sends an async stop message to the process. The handle_cast function handles this
message and return stop telling the gen_server to go to terminate. In terminate we take the PID of the subscribed process to the
epubnub unsubscribe function which sends it a terminate message and it exits.

```erlang

stop() ->
    gen_server:cast(?SERVER, stop).

handle_cast(stop, State) ->
    {stop, normal, State}.

terminate(_Reason, #state{pid=PID}) ->
    epubnub:unsubscribe(PID),
    ok.

```
