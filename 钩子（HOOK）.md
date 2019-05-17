## 行为类代码

```
<?php
/**
 * Created by IntelliJ IDEA.
 * User: wojiukankan
 * Date: 2019/5/6
 * Time: 14:36
 */

namespace app\index\hook;

use think\Request;

class Test {

    public function run() {
        echo 'hellow word';
    }

    public function appInit() {
        echo 'appInit';
    }

    public function appEnd() {
        echo 'appEnd';
    }

    public function test(Request $request) {
        echo 'hellow test';
        dump($request->param());
    }
}
```

## 钩子注册

- 在要使用的场景前`Hook::add('test','app\\index\\hook\\Test');`，test为钩子名称，
- 在模块的目录下面定义 tags.php 
    ```
    <?php
    // 应用行为扩展定义文件
    return [
        // 应用初始化
        'app_init'     => [],
        // 应用开始
        'app_begin'    => [],
        // 模块初始化
        'module_init'  => [],
        // 操作开始执行
        'action_begin' => [],
        // 视图内容过滤
        'view_filter'  => [],
        // 日志写入
        'log_write'    => [],
        // 应用结束
        'app_end'      => [],
        // 自定义
        'test'         => ['app\\index\\hook\\Test']
    ];
    ```

## 钩子使用

- `Hook::listen('test',$param);`
- 会自动调用行为类中的相应方法，优先级钩子名>>run方法