## 目录结构
```
www  WEB部署目录（或者子目录）
├─application           应用目录
│  └─api                api目录
│    └─controller      
│       ├─V1            接口版本
│       │  ├─Test.php   接口类
│       │  └─...
│       └─...
└─route
    ├─route.php         路由规则  
    └─...
```
## 代码实例(tp5.1)
```
//路由规则
Route::rules([
    'api/:version/test/[:id]' => ['api/:version.Test/getData', [], ['id' => '\d+']],
], 'GET');
Route::rules([
    'api/:version/test/:id/:data' => ['api/:version.Test/updateData', [], ['id' => '\d+', 'data' => '\S+']],
], 'POST');
Route::rules([
    'api/:version/test/:id/:data' => ['api/:version.Test/addData', [], ['id' => '\d+', 'data' => '\S+']],
], 'PUT');
Route::rules([
    'api/:version/test/:id' => ['api/:version.Test/deleteData', [], ['id' => '\d+']],
], 'DELETE');
```

```
<?php
/**
 * 接口类
 * Created by PhpStorm.
 * User: lzhd
 * Date: 2019/5/27
 * Time: 14:10
 */

namespace app\api\controller\V1;

class Test {

    private $testData = [//测试数据
        '1' => 'A',
        '2' => 'B',
        '3' => 'C',
        '4' => 'D',
    ];

    /**
     * 获取id对应的数据
     * @author: 853130802<853130802@qq.com>
     * @http: post
     * @date: 2019/5/27 15:42
     * @param string $id
     * @return \think\response\Json
     */
    public function getData($id = '') {
        try {
            $data = $this->testData[$id];
        } catch (\Exception $e) {
            return json(['code' => 400, 'msg' => $e->getMessage()]);
        }
        return json(['code' => 200, 'msg' => 'SUCCESS', 'data' => $data]);
    }

    /**
     * 修改id对应的数据
     * @author: 853130802<853130802@qq.com>
     * @http: post
     * @date: 2019/5/27 15:42
     * @param $id
     * @param $data
     * @return \think\response\Json
     */
    public function updateData($id, $data) {
        try {
            $this->testData[$id] = $this->testData[$id] ? $data : $this->testData[$id];
        } catch (\Exception $e) {
            return json(['code' => 400, 'msg' => $e->getMessage()]);
        }
        return json(['code' => 200, 'msg' => 'SUCCESS', $this->testData]);
    }

    /**
     * 添加id=>data数据
     * @author: 853130802<853130802@qq.com>
     * @http: post
     * @date: 2019/5/27 15:42
     * @param $id
     * @param $data
     * @return \think\response\Json
     */
    public function addData($id, $data) {
        try {
            isset($this->testData[$id]) ? exception('索引已存在') : $this->testData[$id] = $data;
        } catch (\Exception $e) {
            return json(['code' => 400, 'msg' => $e->getMessage()]);
        }
        return json(['code' => 200, 'msg' => 'SUCCESS', 'data' => $this->testData]);
    }

    /**
     * 删除id对应的数据
     * @author: 853130802<853130802@qq.com>
     * @http: post
     * @date: 2019/5/27 15:43
     * @param $id
     * @return \think\response\Json
     */
    public function deleteData($id) {

        try {
            unset($this->testData[$id]);
        } catch (\Exception $e) {
            return json(['code' => 400, 'msg' => $e->getMessage()]);
        }
        return json(['code' => 200, 'msg' => 'SUCCESS', 'data' => $this->testData]);
    }
}
```
> 补充：tp可以直接定义资源型路由，具体可以查看文档。