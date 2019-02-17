#### audition纪要
***
* 1调用栈相关
    > 宏任务和微任务
    ```
        setTimeout(function(){
            console.log('定时器开始啦')
        });

        new Promise(function(resolve){
            console.log('马上执行for循环啦');
            for(var i = 0; i < 10000; i++){
                i == 99 && resolve();
            }
        }).then(function(){
            console.log('执行then函数啦')
        });

        console.log('代码执行结束');
        
    ```
    Promise是微任务，第一个函数的回调是立即执行的而then是要放在微任务列表里的