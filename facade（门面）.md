# facade代理示例

## 目录结构

```
├─extend                第三方类库
│  ├─facade             facade类库
│  │  ├─Redis.php       redis代理类库
│  │  └─ ...            更多类库
│  ├─redis              redis类库
│  │  ├─Redis.php       redis类
│  │  └─ ...            更多类
```

## redis类代码

> `extend/redis/redis`

```
<?php
/**
 * Created by PhpStorm.
 * User: wojiukankan
 * Date: 2019/4/1
 * Time: 17:28
 */

namespace redis;
class Redis {

    private $handler;
    private $options = [
        'host' => '127.0.0.1',
        'port' => 6379,
        'timeout' => 0,
    ];

    /**
     * Redis constructor.
     * @param array $options
     */
    public function __construct(array $options = array()) {
        $this->options = array_merge($this->options, $options);
        $this->handler = new \Redis();
        $this->handler->connect($this->options['host'], $this->options['port'], $this->options['timeout']);
    }

    /**
     * 队列入队
     * @param string $key
     * @param $value
     * @return bool|int
     */
    public function lPush(string $key, $value) {
        return $this->handler->lPush($key, $value);
    }

    /**
     * 队列出队
     * @param string $key
     * @return string|array
     */
    public function rPop(string $key) {
        return $this->handler->rPop($key);
    }

    /**
     * 队列长度
     * @param string $key
     * @return int
     */
    public function Llen(string $key){
        return $this->handler->Llen($key);
    }
}
```

## facade代理类代码

> `extend/facade/redis`

```
<?php
/**
 * Created by PhpStorm.
 * User: wojiukankan
 * Date: 2019/4/1
 * Time: 17:30
 */

namespace facade;

use think\Facade;

/**
 * @see \redis\Redis
 * @mixin \redis\Redis
 * @method bool|int lPush(string $key, $value) static 队列入队
 * @method array|string rPop(string $key) static 队列出队
 * @method int Llen(string $key) static 队列长度
 */
class Redis extends Facade {

    protected static function getFacadeClass() {
        return 'redis\Redis';
    }
}
```

## 调用时：

```
<?php

namespace app\index\controller;

use facade\Redis;

class Index {

    public function index() {

        $message = Redis::rPop('wechat');
        dump($message);
    }
}
```
> 引入代理类，不需要实例化，以静态方式直接调用redis类的方法。