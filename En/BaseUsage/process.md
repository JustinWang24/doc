<head>
     <title>EasySwoole custom process|Swoole custom process|swoole process|swoole multiprocess|php multiprocess</title>
     <meta name="keywords" content="EasySwoole custom process|swoole custom process|swoole process|swoole multiprocess|php  custom process"/>
     <meta name="description" content="This paper mainly describes how to add swoole's custom process to realize PHP multi-process task processing."/>
</head>
---<head>---

# Process

## purpose
Processing time-consuming tasks, such as:
* processing dead-loop queue consumption
* cleaning up token data in redundant redis, and so on.

## Example

### Define a process class
```php
<?php

    use EasySwoole\Component\Process\AbstractProcess;
    
    class MyProcess extends AbstractProcess
    {
    
        protected function run($arg)
        {
            // When the process starts, a callback is executed
            var_dump($this->getProcessName()." run");
            var_dump($arg);
        }
        
        protected function onPipeReadable(\Swoole\Process $process)
        {
            /**
             * This callback is optional
             * When a primary process sends a message to a child-process, a callback will be triggered. When triggered, be sure to use it
             */
            
            // Read messages
            $process->read();
        }
        
        protected function onShutDown()
        {
            /**
             * This callback is optional
             * When the process exits, the callback is executed
             */
        }
        
        
        protected function onException(\Throwable $throwable, ...$args)
        {
            /**
             * This callback is optional
             * When the process exits, the callback is executed
             */
        }
    }
```

### Register your process

You shall register your process in the `EasySwoole` global `mainServerCreate` event:

```php
<?php
    // ...
    use App\MyProcess;
    use EasySwoole\Component\Process\Config;
    
    $processConfig = new Config();
    $processConfig->setProcessName('myOwnProcess');
    
    /**
     * Parameters passed to the process
     */
    $processConfig->setArg([
        'arg1'=>time()
    ]);
    
    ServerManager::getInstance()
        ->getSwooleServer()
        ->addProcess(
            (new MyProcess($processConfig))->getProcess()
        );
    
    // ...
```

> Please be aware of that a process model can be registered multiple times, which means, to create multiple processes of the same type
