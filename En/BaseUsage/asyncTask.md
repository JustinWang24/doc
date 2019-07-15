<head>
     <title>EasySwoole asynchronous tasks|swoole asynchronism</title>
     <meta name="keywords" content="EasySwoole asynchronous tasks|swoole asynchronism|swoole asynchronism process"/>
     <meta name="description" content="Easyswoole delivers asynchronous tasks"/>
</head>
---<head>---

# Asynchronous Task

> Reference Demo: [Asynchronous Task Processing Demo] (https://github.com/easy-swoole/demo/tree/3.x-async)

> Asynchronous Task Manager Class: EasySwoole\EasySwoole\Swoole\Task\TaskManager

Anytime after the server is started, you may send asynchronous tasks to a task worker process pool to execute.
To simplify the delivery of asynchronous tasks, `EasySwoole` encapsulates the `TaskManager` class for delivering synchronous/asynchronous tasks easily. 
You may use either a closure function or a task delivery template class.

## Use PHP closure

A PHP closure is perfect for all kinds of simple task logic, you may send a task inside of any `Controller`, `Timer` or `Service`.

```php
<?php
    // Example: in a controller
    function index()
    {
        \EasySwoole\EasySwoole\Swoole\Task\TaskManager::async(function () {
            echo "execute asynchronous task...\n";
            return true;
        }, function () {
            echo "asynchronous task execution completed...\n";
        });
    }
    
    // Example: in a timer
    \EasySwoole\Component\Timer::getInstance()->loop(1000, function () {
        \EasySwoole\EasySwoole\Swoole\Task\TaskManager::async(function () {
            echo "execute asynchronous task...\n";
        });
    });
```

> Since PHP itself can't serialize closures, the delivery of closure is achieved by reflecting the closure function, getting the PHP code to serialize the PHP code directly, and then directly `eval`.
> Therefore, inside of your closure, there is no way to use **external object references** and **resource handler**. For those complex tasks, please use the task template method.

The following codes show a bad example of using closure:

```php
<?php
    $image = fopen('test.php', 'a'); 
    $a=1;
    
    TaskManager::async(function ($image,$a) {
        // WRONG: the external variable $image is not reachable
        var_dump($image);
        
        // WRONG: the external variable $a is not reachable
        var_dump($a);
        
        // WRONG: the reference to the external object $this is not reachable
        $this->testFunction();
        
        return true;
    },function () {});
```

## Delivery an async task with a Template Class

The task template class is reusable and fits for the task with complicated logic inside. It's quite easy to create your own task template class in `EasySwoole`, for example:
> Please refer to: Asynchronous Task Template Class: EasySwoole\EasySwoole\Swoole\Task\AbstractAsyncTask

```php
<?php
    // ...
    // Create your own task template class.
    class MyTaskTemplate extends \EasySwoole\EasySwoole\Swoole\Task\AbstractAsyncTask
    {
        /**
         * Content of the task
         * @param mixed $taskData - task data
         * @param int $taskId - the task number of the task to be executed
         * @param int $fromWorkerId  - the worker process number of the dispatch task
         * @param mixed $flags - Task's type: taskwait/task/taskCo/taskWaitMulti
         * @author : evalor <master@evalor.cn>
         */
        function run($taskData, $taskId, $fromWorkerId, $flags = null)
        {
            // note that the $taskId is not unique
            // the number of each worker process starts from 0
            // so the combination of $fromWorkerId and $taskId can be used as the unique ID
            // !!! completion of task requires return result
        }
    
        /**
         * Callback after task execution
         * @param mixed $result - the result of the task execution completion
         * @param int $task_id The task number of the task to be executed
         * @author : evalor <master@evalor.cn>
         */
        function finish($result, $task_id)
        {
            // processing after the task execution
        }
    }
```

Instead of PHP closures, now you may send the async task by using the task template class instance:

```php
<?php
    // example of delivery in the controller
    function index()
    {
        // instantiate the task template class and bring the data in it. You can get the data in the task class $taskData parameter.
        $taskClass = new MyTaskTemplate('taskData');
        
        \EasySwoole\EasySwoole\Swoole\Task\TaskManager::async($taskClass);
    }
    
    // example of posting in a timer
    \EasySwoole\Component\Timer::getInstance()->loop(1000, function () {
        \EasySwoole\EasySwoole\Swoole\Task\TaskManager::async(new MyTaskTemplate('foo'));
    });
```

### Using Quick Task Template
You can implement a task template by implementing the `\EasySwoole\EasySwoole\Swoole\Task\QuickTaskInterface` and adding the run method to run the task by directly posting the class name:
```php
<?php
namespace App\Task;
use EasySwoole\EasySwoole\Swoole\Task\QuickTaskInterface;

class QuickTaskTest implements QuickTaskInterface
{
    static function run(\swoole_server $server, int $taskId, int $fromWorkerId,$flags = null)
    {
        echo "fast task template";

        // TODO: Implement run() method.
    }
}
```
In your controller class, you may do something such as:
```php
$result = TaskManager::async(\App\Task\QuickTaskTest::class);
```

## Send Asynchronous Tasks to a custom process

Due to the limitation of the custom process mechanism, `Swoole`'s asynchronous task methods cannot be directly called. 
`EasySwoole` provides `TaskManager::processAsync()` function to facilitate asynchronous task delivery. Please see the following example.
>Note: In your custom process, you can't pass the `finish` callback to `Swoole`

```php
class MyProcess extends AbstractProcess
    {
        protected function run($arg)
        {
            // Send a closure
            TaskManager::processAsync(function () {
                echo "process async task run on closure!\n";
            });
    
            // Send a task template instance
            $taskClass = new MyTaskClass('task data');
            TaskManager::processAsync($taskClass);
        }

```

## Async Tasks Concurrent Execution

Sometimes it is necessary to execute multiple asynchronous tasks concurrently. 
A usual example is the data collection. After collecting data from multiple resources, you want them to be handled together. 
In this case, concurrent delivery of tasks can be performed. These async tasks will be sent and executed one by one in the system level, and when they're finished, a result set will be returned.

```php
// concurrent multi tasks execution
$tasks[] = function () { sleep(50000);return 'this is 1'; }; // task 1
$tasks[] = function () { sleep(2);return 'this is 2'; };     // task 2
$tasks[] = function () { sleep(50000);return 'this is 3'; }; // task 3

$results = \EasySwoole\EasySwoole\Swoole\Task\TaskManager::barrier($tasks, 3);

var_dump($results);
```

> Note: `Barrier` is very alike a blocked thread, the tasks will be distributed to different task processes (need to have enough task processes, otherwise blocking will happen) to run synchronously. The result set will be returned till all tasks end or timeout, the default The task timeout is 0.5 seconds. In the above example, only task 2 can execute normally and return the result.

## Class Methods Reference

```php
/**
 * deliver an asynchronous task
 * @param mixed $task - asynchronous task to be delivered
 * @param mixed $finishCallback - callback function after the task is executed
 * @param int $taskWorkerId - the number of tje specified delivery of task process (default delivery is random idle processes)
 * @return bool - successful delivery will return integer $task_id, or return false if the delivery failed 
 */
static function async($task,$finishCallback = null, $taskWorkerId = -1)
```

```php
/**
 * deliver a synchronized task
 * @param mixed $task - synchronized task to be delivered
 * @param float $timeout - task timeout
 * @param int $taskWorkerId - the number of tje specified delivery of task process (default delivery is random idle processes)
 * @return bool|string - successful delivery will return integer $task_id, or return false if the delivery failed 
 */
static function sync($task, $timeout = 0.5, $taskWorkerId = -1)
```

```php
/**
 * deliver multiple asynchronous tasks
 * @param array $taskList - tasks list to be sent
 * @param float $timeout - task execution timeout
 * @return array|bool - execution result set for each task
 */
static function barrier(array $taskList, $timeout = 0.5)
```
