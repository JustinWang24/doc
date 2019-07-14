<head>
     <title>EasySwoole controller|swoole controller|swoole Api service</title>
     <meta content="text/html; charset=utf-8" http-equiv="Content-Type">
     <meta name="keywords" content="EasySwoole Console commands|EasySwoole User Guide|EasySwoole Api"/>
     <meta name="description" content="EasySwoole Console commands|EasySwoole User Guide|EasySwoole Api"/>
</head>
---<head>---

# Custom commands
`EasySwoole` has provided 5 console commands out of the box:
  
```bash
php easyswoole help     # Command Help
php easyswoole install  # Installation (called in ./vendor/easyswoole/easyswoole/bin/easyswoole file)
php easyswoole start    # Start
php easyswoole stop     # Stop (Need to guard the process)
php easyswoole reload   # Hot restart(Need to guard the process)
```

> More details in [Service management](../Introduction/server.md)

## Writing commands
In addition to the commands provided with `EasySwoole`, you may also build your own custom commands.
By implementing `EasySwoole\EasySwoole\Command\CommandInterface` interface, you may have your own command:

````php
public function commandName():string;
public function exec(array $args):?string ;
public function help(array $args):?string ;
````

For example, create a new file named App/Command/Test.php:
```php
<?php
namespace App\Command;

use EasySwoole\EasySwoole\Command\CommandInterface;
use EasySwoole\EasySwoole\Command\Utility;

class Test implements CommandInterface
{
    public function commandName(): string
    {
        return 'test';
    }

    public function exec(array $args): ?string
    {
        // Print parameters, print test values
        var_dump($args);
        echo 'test'.PHP_EOL;
        return null;
    }

    public function help(array $args): ?string
    {
        // Print the EasySwoole logo
        $logo = Utility::easySwooleLog();
        return $logo.<<<HELP_START
        This is test.
        
HELP_START;
    }
}
```

## Register your own command
You may create a new file: `/bootstrap.php`:
````php
<?php
    // ...

    \EasySwoole\EasySwoole\Command\CommandContainer::getInstance()->set(new \App\Command\Test());
    
    // ...
````

> Bootstrap is a new event in 3.2.5 that allows you to execute your own logic before the framework is initialized

## Run your own command
```bash

php easyswoole test
array(0) {
}
test

php easyswoole test 123 456
array(2) {
  [0]=>
  string(3) "123"
  [1]=>
  string(3) "456"
}
test

php easyswoole help test
  ______                          _____                              _
 |  ____|                        / ____|                            | |
 | |__      __ _   ___   _   _  | (___   __      __   ___     ___   | |   ___
 |  __|    / _` | / __| | | | |  \___ \  \ \ /\ / /  / _ \   / _ \  | |  / _ \
 | |____  | (_| | \__ \ | |_| |  ____) |  \ V  V /  | (_) | | (_) | | | |  __/
 |______|  \__,_| |___/  \__, | |_____/    \_/\_/    \___/   \___/  |_|  \___|
                          __/ |
                         |___/
        This is test
        
```