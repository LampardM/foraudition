#### audition纪要
***
* 调用栈相关
    > 宏任务和微任务
    * macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
    * micro-task(微任务)：Promise，process.nextTick
    ```
        setTimeout(function(){
            console.log('1')
        });

        new Promise(function(resolve){
            console.log('2');
            for(var i = 0; i < 10000; i++){
                i == 99 && resolve();
            }
        }).then(function(){
            console.log('3')
        });

        console.log('4');

    ```
    Promise是微任务，第一个函数的回调是立即执行的而then的回调是要放在微任务列表里的，process.nextTick的回调是微任务，需要放在微任务列表。setTtimeout是宏任务，回调注册到宏任务列表，也就是微任务回调2先执行，3放入微任务列表，执行4然后微任务列表里3开始执行，最后执行宏任务回调1。
    注意：一个宏任务里包含多个微任务，一定是需要把此宏任务里的微任务全部执行完毕才能进入下一个宏任务的执行。
***