---
layout: post
title: lua non-preemptive multithreading 
tags: [lua文章]
categories: [topic]
---
* * *

## Lua Coroutine

Lua Coroutine is a kind of collaborative multithreading, equivalent to a
thread. A pair yield-resume switches control from one thread to another.
However, unlike “real” multithreading, coroutines are non preemptive. A
coroutine only suspends its execution by explicitly calling a `yield`
function.

### Coroutine Grammar

Method | Description  
---|---  
coroutine.create (f) | Creates a new coroutine, with f must be a function,
Returns this new coroutine, an object with type “thread”.  
coroutine.resume (co [, val1, ···]) | Starts or continues the execution of
coroutine  
coroutine.yield (…) | Suspends the execution of the calling coroutine. Any
arguments to yield are passed as extra results to resume.  
coroutine.running () | Returns the running coroutine plus a boolean, true when
the running coroutine is the main one.  
coroutine.status (co) | Returns the status of coroutine co, as a string:
“running”, “suspended”,”normal” “dead”  
coroutine.wrap (f) | Creates a new coroutine, Returns a function that resumes
the coroutine each time it is called.  
coroutine.isyieldable () | Returns true when the running coroutine can yield.  
  
## Multithreading Example

Let us assume a typical multithreading situation: We want to download several
remote files through HTTP. Of course, to download several remote files, we
must know how to download one remote file. In this example, we will use the
`LuaSocket` library. To download a file, we must:

  1. open a connection to its site
  2. send a request to the file
  3. receive the file (in blocks)
  4. close the connection.

In Lua, we can write this task as follows.

    
    
    local socket = require "socket"
    
    local host = "www.w3.org"
    local file = "/TR/REC-html32.html"
    
    local c = assert(socket.connect(host, 80))
    c:send("GET " .. file .. " HTTP/1.0rnrn")
    c:close()
    

To rewrite the program with coroutines, let us first rewrite the previous
download code as a function:

    
    
    function download (host, file)
        local c = assert(socket.connect(host, 80))
        local count = 0    -- counts number of bytes read
        c:send("GET " .. file .. " HTTP/1.0rnrn")
        while true do
            local s, status = receive(c)
            count = count + string.len(s)
            if status == "closed" then break end
        end
        c:close()
        print(file, count)
    end
    
    function receive (connection)
        return connection:receive(2^10)
    end
    

For the concurrent implementation, this function must receive data without
blocking. Instead, if there is not enough data available, it yields. The new
code is like this:

    
    
    function receive (connection)
        connection:timeout(0)   -- do not block
        local s, status = connection:receive(2^10)
        if status == "timeout" then
            coroutine.yield(connection)
        end
        return s, status
    end
    

The call to timeout(0) makes any operation over the connection a non-blocking
operation. When the operation status is “timeout”, it means that the operation
returned without completion. In this case, the thread yields. The non-false
argument passed to yield signals to the dispatcher that the thread is still
performing its task. (Later we will see another version where the dispatcher
needs the timed-out connection.) Notice that, even in case of a timeout, the
connection returns what it read until the timeout, so receive always returns s
to its caller. The next function ensures that each download runs in an
individual thread:

    
    
    threads = {}    -- list of all live threads
    function get (host, file)
        -- create coroutine
        local co = coroutine.create(function ()
            download(host, file)
        end)
        -- insert it in the list
        table.insert(threads, co)
    end
    

The table threads keeps a list of all live threads, for the dispatcher. The
dispatcher is simple. It is mainly a loop that goes through all threads,
calling one by one. It must also remove from the list the threads that finish
their tasks. It stops the loop when there are no more threads to run:

    
    
    function dispatcher ()
        while true do
            local n = table.getn(threads)
            if n == 0 then break end   -- no more threads to run
            for i=1,n do
              local status, res = coroutine.resume(threads[i])
              if not res then    -- thread finished its task?
                table.remove(threads, i)
                break
              end
            end
        end
    end
    

Finally, the main program creates the threads it needs and calls the
dispatcher. For instance, to download four documents from the W3C site, the
main program could be like this:

    
    
        host = "www.w3.org"
        
        get(host, "/TR/html401/html40.txt")
        get(host,"/TR/2002/REC-xhtml1-20020801/xhtml1.pdf")
        get(host,"/TR/REC-html32.html")
        get(host,
            "/TR/2000/REC-DOM-Level-2-Core-20001113/DOM2-Core.txt")
        
        dispatcher()   -- main loop
    

My machine takes six seconds to download those four files using coroutines.
With the sequential implementation, it takes more than twice that time (15
seconds). Despite the speedup, this last implementation is far from optimal.
Everything goes fine while at least one thread has something to read. However,
when no thread has data to read, the dispatcher does a busy wait, going from
thread to thread only to check that they still have no data. As a result, this
coroutine implementation uses almost 30 times more CPU than the sequential
solution.

To avoid this behavior, we can use the select function from LuaSocket. It
allows a program to block while waiting for a status change in a group of
sockets. The changes in our implementation are small. We only have to change
the dispatcher. The new version is like this:

    
    
    function dispatcher ()
        while true do
            local n = table.getn(threads)
            if n == 0 then break end   -- no more threads to run
            local connections = {}
            for i=1,n do
              local status, res = coroutine.resume(threads[i])
              if not res then    -- thread finished its task?
                table.remove(threads, i)
                break
              else    -- timeout
                table.insert(connections, res)
              end
            end
            if table.getn(connections) == n then
              socket.select(connections)
            end
        end
    end
    

Along the inner loop, this new dispatcher collects the timed-out connections
in table connections. Remember that receive passes such connections to yield;
thus resume returns them. When all connections time out, the dispatcher calls
select to wait for any of those connections to change status. This final
implementation runs as fast as the first implementation with coroutines.
Moreover, as it does no busy waits, it uses just a little more CPU than the
sequential implementation.

* * *