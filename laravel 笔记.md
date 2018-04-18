# Laravel 笔记

## 前言
记录 Laravel 开发中的问题，及笔记。  

## 用法
### Validator 类的用法
用法如下:
```php
    /**
     * 修改用户信息
     * @route   api/user/modify
     * @method  post
     * @param   Request $request
     * @return \Illuminate\Http\JsonResponse
     */
     
use Illuminate\Support\Facades\Validator;

    public function modifyUserInfo(Request $request)
    {
        //自定义验证错误信息
        $messages  = [
            'password.regex'       => '密码必须为6-25位',
            'avatar.url'           => '头像必须为有效的url地址',
            'birthday.date_format' => '生日必须为 YYYY-mm-dd 格式',
        ];
        
        //验证
        $validator = Validator::make($request->all(), [
            'password' => 'regex:/^.{6,25}$/', //正则验证 如有多条不能用| 必须是数组 ['required','regex:/^[a-zA-Z0-9]$/']
            'avatar'   => 'url',
            'birthday' => 'date_format:Y-m-d',
        ], $messages);

        //以上验证通过后 继续验证
        $validator->after(function ($validator) use ($request) { //use ($request) 后,在这个闭包里可以用 $request
                    $user = User::getUserByphone($request->mobile);
                    if (!$user) {
                        $validator->errors()->add('isUser', '此用户不存在');//添加错误信息
                    }
                });

        //如果验证不通过
        if ($validator->fails()) {
            return $this->error($validator->errors()->first());
        }

        $user           = User::find($user_id);
        $user->nickname = $request->nickname;
        $user->password = User::generatePassword($request->password);
        $user->avatar   = $request->avatar;
        $user->birthday = $request->birthday;

        try {
            $user->save();
            return $this->success("操作成功");
        } catch (\Exception $ex) {                  //\Exception 捕获所有异常
            return $this->error($ex->getMessage()); // getMessage() 异常信息
        }

    }
```

### Faker 生成测试数据

打开 `app/database/factories/ModelFactory.php` 用法如下：  

```php
$factory->define(App\User::class, function (Faker\Generator $faker) {
    $faker = Faker\Factory::create('zh_CN'); //中文包
    return [
        'openid'       => str_random(10),
        'nickname'     => $faker->name,  //中文姓名
        'mobile'       => $faker->phoneNumber,
        'avatar'       => $faker->imageUrl(),     //图片URL地址
        'integral'     => $faker->randomNumber(3), //随机3位整型(0-999)
        'balance'      => $faker->randomFloat(2, 0, 10000), //随机浮点数,2位小数点,最小0，最大10000
        'birthday'     => $faker->date(), //日期
        'created_time' => $faker->unixTime(), //unix时间戳
        'password'     => App\User::generatePassword('haha123'), //可用模型方法生成数据
    ];
});

$factory->define(App\Product::class, function (Faker\Generator $faker) {
    $faker  = Faker\Factory::create('zh_CN');
    //用模型生成要关联随机的数组
    $seller = App\Seller::where('status', 1)->get()->toArray();
    foreach ($seller as $value) {
        $row[] = $value['id'];
    }
    return [
        'product_no'   => $faker->randomNumber(8),
        'name'         => '【测试商品】' . $faker->streetName,
        'cate_id'      => random_int(1, 6), //1-6随机int
        'seller_id'    => $faker->randomElement($row),//随机数组
        'mod_id'       => random_int(1, 10),
        'price'        => $faker->randomFloat(2, 1, 500),
        'weight'       => random_int(1, 100),
        'img'          => $faker->imageUrl('480','480'),//宽480 高480 的图片URL
        'thumb'        => $faker->imageUrl('480','480'),
        'content'      => '【测试商品内容】' . $faker->text(),
        'is_point'     => random_int(0, 1),
        'is_new'       => random_int(0, 1),
        'is_recommend' => random_int(0, 1),
        'is_sale'      => random_int(0, 1),
        'create_time'  => $faker->unixTime(),
    ];
});

$factory->define(App\Seller::class, function (Faker\Generator $faker) {
    $faker = Faker\Factory::create('zh_CN');
    $user  = App\User::all()->toArray();
    foreach ($user as $value) {
        $row[] = $value['id'];
    }
    return [
        'user_id'     => $faker->randomElement($row),
        'seller_name' => $faker->colorName,  //很有意境的中文色彩名
        'corporate'   => $faker->imageUrl(),
        'business'    => $faker->imageUrl(),
        'create_time' => $faker->unixTime(),
    ];
});

```
然后用 php artisan tinker 进入 laravel 命令行
```php
factory(App\Product::class,50)->create(); //生成 50 条 存入数据库
factory(App\User::class,50)->make();      //生成 50 条 不存入数库
```

### 模型文件
用法如下：
```php
class Seller extends Model
{
    protected $table = 'seller';    //定义表名
    protected $hidden = ['corporate', 'business', 'province', 'city', 'county', 'address'];    //all()方法不会被返回的字段
    protected $appends = ['nickname', 'mobile']; //额外添加的返回信息 配合getColumnAttribute()方法得到。注意命名，如 nick_name 就是getNickNameAttribute()
    protected $dates = ['create_time']; //需要被转换成日期的属性 Carbon 类
    public $timestamps = false;     //保存时不自动生成 created_at 与 updated_at 字段
    

    public function getNicknameAttribute()
    {
        return $this->hasOne('App\User', 'id', 'user_id')->value('nickname'); // hasOne 一对一关系 id 是 to 表 user_id 是 本表
    }

    public function getCreateTimeAttribute()
    {
        $value = $this->attributes['create_time'];
        return $value ? date('Y-m-d H:i:s', $value) : '';
    }

    public function getMobileAttribute()
    {
        return $this->hasOne('App\User', 'id', 'user_id')->value('mobile');
    }

}
```

### 中间件用法

注册中间件，打开 `app/Http/Kernel.php` 用法如下：
```php
class Kernel extends HttpKernel
{
    /**
     * 全局中间件
     */
    protected $middleware = [
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        //\App\Http\Middleware\VerifyCsrfToken::class,
        \Lib\ClusterSession\Middleware\StartSession::class,
    ];

    /**
     * 路由中间件
     */
    protected $routeMiddleware = [
        'access_control' => \App\Http\Middleware\AccessControl::class,
        'api' => \App\Http\Middleware\Api::class,
        'auth' => \App\Http\Middleware\Authenticate::class, //别名
        'admin_auth' => \App\Http\Middleware\AdminAuthenticate::class,
        'server_auth' => \App\Http\Middleware\ServerAuthenticate::class,
        'CORS' => \App\Http\Middleware\CORS::class
    ];
}
```

创建 `app/Http/Middleware/Authenticate.php` 文件，如下：

```php
class Authenticate
{
    public function handle($request, Closure $next)
    {
        $value = SessionManager::get('user_id', null);

        if(empty($value)){
            return response()->json(['error' => '999', 'message' => '请先登录']);
        }

        return $next($request); //如果通过，去下一个请求
    }


}
```

在 `routes.php` 中使用，如下：

```php
// prefix 路由分组,middleware 中间件
Route::group(['prefix' => 'api', 'middleware' => ['access_control', 'api', 'CORS']], function () {
    Route::post('/user/info', ['uses' => 'Api\UserController@info', 'middleware' => ['auth']]);//用户中心
    Route::get('/user/address', ['uses' => 'Api\UserController@UserAddress', 'middleware' => ['auth']]);//个人收货地址列表
    Route::get('/user/address/info', ['uses' => 'Api\UserController@UserAddressInfo', 'middleware' => ['auth']]);//获取收货地址详情
    Route::post('/user/address/post', ['uses' => 'Api\UserController@UserAddressPost', 'middleware' => ['auth']]);//获取收货地址详情
    Route::post('/user/address/delete', ['uses' => 'Api\UserController@UserAddressDelete', 'middleware' => ['auth']]);//获取收货地址详情
    Route::post('/user/modify', ['uses' => 'Api\UserController@modifyUserInfo', 'middleware' => ['auth']]);//获取收货地址详情
        Route::post('/message/send', 'Api\SmsController@send');//发送短信
        Route::post('/upload/configure', 'Api\DefaultController@upload');//发送短信
    
        Route::post('/user/register', 'Api\UserController@register');//用户注册接口
        Route::post('/user/login', 'Api\UserController@login');//用户注册接口
        Route::post('/user/signout', 'Api\UserController@signout');//退出接口
}

```

### 事务用法
数据库引擎必须为 InnoDB 用法如下：
```php
public function test(Request $request){    
    $car = new Car();
    $car->user_id        = $request->get('user_id');
    $car->product_id     = $request->get('product_id');
    $car->product_sku_id = $request->get('product_sku_id');
    $car->product_num    = $request->get('product_num');
        
        DB::beginTransaction();  //开始
        try
        {
            $car->save();
            $success = 1;
        }catch (\Exception $ex){
            DB::rollback();     //回滚
            $success = 0;
            
        }
        DB::commit();           //提交
return $success ? $this->success("操作成功") : $this->error("操作失败");
}
```


### 全局作用域 global Scope 用法
建立 App/Scopes/SiteScope.php  
```php
<?php
namespace App\Scopes;
use App\Site;
use Illuminate\Database\Eloquent\ScopeInterface;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;
class SiteScope implements ScopeInterface{
    public function apply(Builder $builder, Model $model) {
        $site = Site::getSiteId();
        return $builder->where('site_id', $site);
    }
    public function remove(Builder $builder, Model $model)      //必须有remove
    {
        $column = $model->getQualifiedDeletedAtColumn();

        $query = $builder->getQuery();

        foreach ((array) $query->wheres as $key => $where)
        {
            // If the where clause is a soft delete date constraint, we will remove it from
            // the query and reset the keys on the wheres. This allows this developer to
            // include deleted model in a relationship result set that is lazy loaded.
            if ($this->isSoftDeleteConstraint($where, $column))
            {
                unset($query->wheres[$key]);

                $query->wheres = array_values($query->wheres);
            }
        }
    }
}
```
在所使用模型里重写 boot 方法
```php
    protected static function boot(){
        parent::boot();
        static::addGlobalScope(new SiteScope());
    }
```
## 问题

### paginate 分页
看文档以为最后的数据是 data ，其实是 items 方法，如下：

```php
 public function showApi(Request $request)
    {

        $limit = $request->get('limit');
      //$page  = $request->get('page');
        $results = Seller::paginate($limit); //无须接收 $page ,laravel 自动接收
      //$results = Seller::forPage($page,$limit)->get();  或者用这种 
        return response()->json(['code' => 0, 'data' => $results->items(), 'count' => $results->total()]);
    }
```

### 临时显示隐藏属性
Laravel 5.1 中没有makeVisible 和 makeHidden 方法来临时显示或隐藏属性，打开 `/vendor/laravel/framework/src/Illuminate/Database/Eloquent/Model.php` 文件。添加如下两个方法：

```php
    public function makeVisible($attributes = null)
    {
        $attributes = is_array($attributes) ? $attributes : func_get_args(); //func_get_args() 把函数接收到的参数转为数组

        $arr = array_diff($this->hidden,$attributes);  //array_diff($arr1,$arr2) 计算数组差集 这里用作删除元素，如： 
        //$arr1 = ['0' =>'a','1'=> 'b','2'=>'c']; 
        //$arr2 = ['0' => 'b'];
        //return array_diff($arr1,$arr2);
        // ['0'=>'a','2'=>'c'];

        $this->hidden = $arr;

        return $this;  //由于最后return $this; 此方法在末尾调用有效
    }

    public function makeHidden($attributes = null)
    {
        $attributes = is_array($attributes) ? $attributes : func_get_args();

        $visible = array_diff($this->visible,$attributes);
        $appends = array_diff($this->appends,$attributes);
        $hidden = array_merge($this->hidden,$attributes);

        $this->visible = $visible;
        $this->appends = $appends;
        $this->hidden = $hidden;

        return $this;
    }
```

只能用于 `find` 方法，where 构造查询报错，我也很绝望啊。示例：
```php
public function test(Request $request)
{
    $id = $request->get('id');
    
    $pro = Product::find($id)->makeHidden('seller_id');
    return response()->json($pro);
}
```

### 树状分类
```php

public static function tree($data,$pid=0,$level=0){
        $results = array();
        foreach ($data as $value){
        // 递归点 如果当前记录的父 id 等于传入的父 id ，说明这个记录是传入父 id 的子级
            if($value['p_id'] == $pid){
                $value['level'] = $level;
                //递归调用，获得子级下的子级
                $value['children'] = self::tree($data,$value['id'],$level + 1);
                //push把以上结果赋给返回数据
                $results[] = $value;
            }
            //递归出口：遍历完成。
        }
        //返回结果集
        return $results;
    }
    
```

### 自增字段值
相当于 `update table set column = column + :value where user_id = :user_id` ，用法如下：
```php

        $car = Car::where('user_id',$user_id)->where('product_id',$product_id)->where('product_sku_id',$product_sku_id)->first();
        if(empty($car)) {
            $car = new Car();
            $car->user_id        = $user_id;
            $car->product_id     = $product_id;
            $car->product_sku_id = $product_sku_id;
            $car->product_num    = $product_num;
        }else{
            $car->increment('product_num',$product_num); //自增字段值
        }

```

### 模型括号查询
相当于 `select * from queue_second_sold where status=0 and (buy_user_id = 341 or (sell_user_id = 341 and buy_user_id != 0)) order by id desc`,用法如下:

```php
    /**
     * 交易大厅-定向交易
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function soldList(Request $request){
        $type = $request->get('type',0);    //交易大厅
        $limit = $request->get('limit',10);
        $user = User::getUserInfo();
        if($type == 1){                     //定向交易
            $results = QueueSecondSold::where('status',0)
            ->where(function($query) use ($user){
                $query->where('sell_user_id',$user->id)
                ->where('buy_user_id','!=',0);
            })->orWhere('buy_user_id',$user->id)
            ->orderBy('id','desc')->paginate($limit);

        }else{
            $results = QueueSecondSold::where('status',0)->where('buy_user_id',0)->orderBy('id','desc')->paginate($limit);
        }

        return $this->success(['result'=>$results->items(),'total'=>$results->total(),'page'=>$results->currentPage(),'pages'=>$results->lastPage(),'user_id'=>$user->id]);
    }

```

### 模型连接查询
相当于 `select count(*) from user as a left join chunk as b on a.id=b.user_id where a.parent_id = 341`  

```php
$chunk_count = User::leftJoin('chunk','user.id','=','chunk.user_id')->where('user.parent_id',$user->parent_id)->count();
```

### 列值查询
```php
$search_user = User::where('mobile','like','%'.$mobile.'%')->get()->pluck('id');
```

## ElasticSearch 问题及用法

### 出现 index.max_result_window 报错的解决办法  

```shell
curl -XPUT "http://localhost:9200/my_index/_settings" -d '{ "index" : { "max_result_window" : 100000000 } }'
```

### 搜索封装
```php
public static function SearchAccountLogEs($must = array(),$must_not = array(),$should = array(),$aggs = array(),$size = 10,$page = 1,$sort = array(),$debug = false){
        $index = Config::get('elasticsearch.index');
        $type = 'accountlog';
        $must = $must ?? array();
        $must_not = $must_not ?? array();
        $should = $should ?? array();
        $aggs = $aggs ?? array();
        $sort = $sort ?? array();
        $from = ($page - 1) * $size;
        $params = [
            'index'=>$index,
            'type' => $type,
            'body'=>[
                'query'=>[
                    'bool'=>[
                        'must'=>$must,
                        'must_not'=>$must_not,
//                        'should' => $should
                        ]
                    ]
//                'aggs'=>$aggs,
                ],
            'size' => $size,
            'from' => $from
        ];
        
        $client = ClientBuilder::create()
            ->setHosts( Config::get( 'elasticsearch.hosts' ) )
            ->setRetries( 2 )
            ->build();
        if(!empty($aggs)){
            $params['body']['aggs'] = $aggs;
        }
        if(!empty($should)){
            $params['body']['query']["bool"]["should"] = $should;
            $params['body']['query']["bool"]["minimum_should_match"] = 1;
        }
        if(!empty($sort)){
            $params['body']['sort'] = array($sort);
        }
        if($debug == true)
            return $params;
        $response = $client->search($params);
        $results = array(
          'total' => $response['hits']['total'],
//            key($aggs) => $response['aggregations'][key($aggs)]['value']
        );
        if (!empty($aggs)){
            $results[key($aggs)] = $response['aggregations'][key($aggs)]['value'];
        }
        $results['data'] = [];
        foreach ($response['hits']['hits'] as $key => $value){
            $results['data'][] = $value['_source'];
        }

        return $results;
    }
```

### 更新封装
```php
public static function updateAccountLogEs($log)
    {
        $other = User::find($log->user_id);
        //更新ES索引
        $client = ClientBuilder::create()
            ->setHosts(Config::get('elasticsearch.hosts'))
            ->setRetries(2)
            ->build();
        $index = Config::get('elasticsearch.index');
        $type = 'accountlog';

        $params = [
            'index' => $index,
            'type'  => $type,
            'id'    => $log->id
        ];

        try {
            $client->delete($params);
        }catch (\Exception $ex){}

        $params = [
            'index' => $index,
            'type' => $type,
            'id' => $log->id,
            'body' => [
                'id' => $log->id,
                'user_id' => $log->user_id,
                'user_money' => $log->user_money,
                'user_money1' => $log->user_money1,
                'user_money3' => $log->user_money3,
                'frozen_money' => $log->frozen_money,
                'letter_of_credit' => $log->letter_of_credit,
                'shop_letter_credit' => $log->shop_letter_credit,
                'rank_points' => $log->rank_points,
                'pay_points' => $log->pay_points,
                'created_time' => $log->time,
                'info' => $log->info,
                'province_id' => $other->province_id,
                'city_id' => $other->city_id,
                'county_id' => $other->county_id,
                'industry_id' => $other->industry_id,
                'parent_id' => $other->parent_id,
                'type' => $log->type
            ]
        ];

        $client->index($params);
    }

```

## Laravel-Excel 插件用法

### 安装

1.`composer` 安装
```
composer require "maatwebsite/excel:~2.1.0"
```

2.安装完成后，修改 `config/app.php` 在 `providers` 数组内追加如下内容
```php
Maatwebsite\Excel\ExcelServiceProvider::class,
```

3.同时在 `aliases` 数组内追加如下内容:
```php
'Excel' => Maatwebsite\Excel\Facades\Excel::class,
```

4.生成配置文件 `config/excel.php` :
```php
php artisan vendor:publish --provider="Maatwebsite\Excel\ExcelServiceProvider"
```

### 用法

#### 解析 Excel 文件

```php
ini_set ('memory_limit', '1024M');
$data = Excel::load('excel.xlsx',function ($reader){
        },'UTF-8')->toArray();
```

#### 将数据导成 Excel 文件
```php
// 导出 Excel 并能直接在浏览器下载
# $export_file_name = 要生成的文件名
Excel::create($export_file_name, function ($excel) {
    $excel->sheet('Sheetname', function ($sheet) {
        $sheet->appendRow(['name', 'age']);
        $sheet->appendRow(['LiLei', '22']);
        $sheet->appendRow(['HanMeimei', '22']);
    });
})->download('xls');

// 导出 Excel 并存储到指定目录
Excel::create($export_file_name, function ($excel) {
    $excel->sheet('Sheetname', function ($sheet) {
        $sheet->appendRow(['name', 'age']);
        $sheet->appendRow(['LiLei', '22']);
        $sheet->appendRow(['HanMeimei', '22']);
    });
})->store('xls', $path);
```

## 七牛云 SDK 结合 layui 用法

### 安装
```php
composer require qiniu/php-sdk
```

### 使用

#### PHP 返回上传 token  
```php
<?php

namespace App\Http\Controllers\Admin;

use App\Setting;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use Qiniu\Auth;

class DefaultController extends Controller
{
    public function upload(){
        $accessKey = Setting::getValueByKey('qn_accessKey');
        $secretKey = Setting::getValueByKey('qn_secretKey');
        $bucket = Setting::getValueByKey('qn_bucket_static');
        $baseUrl = Setting::getValueByKey('qn_static_url');
        
        //构建鉴权对象
        $auth = new Auth($accessKey,$secretKey);
        
        //生成上传 token
        $token = $auth->uploadToken($bucket);

        return $token;
    }
}
```

####  layui 携带 token 请求七牛云上传接口

##### layui HTML 代码
```html
<div class="layui-form-item">
            <label class="layui-form-label" for="fileInput">缩略图</label>
            <div class="layui-input-block">
                <button type="button" class="layui-btn" id="thum">上传图片</button>
                <div class="layui-upload-list">
                    <div  id="thum_img"></div>
                </div>
            </div>
        </div>


        <div class="layui-form-item">
            <label class="layui-form-label" for="fileInput">详细图</label>
            <div class="layui-input-block">
                <button type="button" class="layui-btn" id="imgs">
                    上传
                </button>
                <div class="layui-upload-list">
                    <div id="imgs_list"></div>
                </div>
            </div>
        </div>
```

##### layui JS 代码
```javascript
     function uploadInsts(elem,multiple,number,imgDiv){
            if(multiple == null){
                multiple = false;
            }
            if(number == null){
                number = 0;
            }
            layui.use(['upload'],function () {
                var $ = layui.jquery
                    ,upload = layui.upload;
                upload.render({

                    elem: elem                      //上传按钮ID
                    ,url: 'https://up.qbox.me/'     //七牛云上传接口
                    ,data:{'token':'{{$token ?? null}}'}    //携带 token
                    ,multiple:multiple              //是否多个图片
                    ,number:number                  //最多上传图片数量
                    // ,before: function(obj){
                    //     //预读本地文件示例，不支持ie8
                    //     obj.preview(function(index, file, result){
                    //         $('#demo1').attr('src', result); //图片链接（base64）
                    //     });
                    // }
                    ,done: function(res){
                        //res 上传成功后七牛云返回数据 
                        $.ajax({
                            url:"{{url('admin/upload_save')}}"      //请求 php 接口
                            ,data:{'key':res.key,'multiple':multiple,'num':number} //res.key图片名
                            ,type:'post'
                            ,success:function(result){
                                if(result.error > 0){           //失败
                                    layer.msg(result.msg);
                                }else{                          //成功 拿到拼接后的图片url展示
                                    $(imgDiv).append("<img class='layui-upload-img' style='width:150px;height:150px;margin-right:3px;' src='"+result.msg+"'>");
                                }
                            }
                        });
                    }
                    ,error: function(){     //上传失败
                        layer.msg('请刷新重试');
                        return false;
                    }
                });
            });
    }
    
                uploadInsts('#thum',false,0,'#thum_img');//单图片
                uploadInsts('#imgs',true,5,'#imgs_list');//多图片
```

##### PHP 接收七牛云 key 拼接图片地址
```php
<?php

public function uploadSave(Request $request){
        $key = $request->get('key',null);
        $multiple = $request->get('multiple',false);
        $num = $request->get('number',1);
        if(empty($key)){
            return $this->error('上传图片错误请重试');
        }
        $type = 'image';
        $user_id = User::getAdminId();
        $uploads = new Uploads();
        $uploads->type = $type;
        $uploads->user_id = $user_id;
        $uploads->key = $key;
        $uploads->time = time();
        try{
            $uploads->save();
            $result = Setting::getValueByKey('qn_static_url').'/'.$key;
            return $this->success($result);
        }catch (\Exception $exception){
            return $this->error($exception->getMessage());
        }
    }
    
?>
```

##### layui + PHP + 七牛云 实现单个多个图片上传单选多选图片设计
1. 点击上传图片按钮公共弹出层函数
```javascript
function layer_show(title,url,w,h) {
            var width = w || null;
            var height = h || null;
            var areaValue;
            if (width != null) {
                areaValue = width + 'px';
                if (height != null) {
                    areaValue = [width + 'px', height + 'px'];
                }
            }else{
                areaValue = ['100%','100%'];
            }
                layui.use('layer', function () { //独立版的layer无需执行这一句
                    var $ = layui.jquery, layer = layui.layer; //独立版的layer无需执行这一句
                    layer.open({
                        type: 2 //此处以iframe举例
                        , title: title
                        , area: areaValue
                        , shade: 0
                        , maxmin: true
                        , content: url
                        , offset: '10px'

                    });
                });
        }
```

2. PHP 接收 url 中的参数，并显示弹出层视图，要有回调函数
```php
<?php
public function uploadShow(Request $request){
        $limit = $request->get('limit',9);
        $type = $request->get('type','radio');          //图片选择是单选还是多选
        $callback = $request->get('callback',null);     //回调函数名

        $user_id = User::getAdminId();
        $results = Uploads::where('user_id',$user_id)->orderBy('id','desc')->paginate($limit);
        $token = Uploads::getQiniuUpToken();            //七牛云上传 token
        return view('admin.upload.show')->with(['results'=>$results->items(),'type'=>$type,'callback'=>$callback,'token'=>$token]);
    }
    ?>
```

3. 上传图片弹出层视图
```blade
<form class="layui-form" action="">
        <div class="layui-form-item">
            <div class="layui-input-block">
        <div class="layui-upload">
            <button type="button" class="layui-btn" id="upload_img"><i class="layui-icon"></i>上传图片</button>
        </div>
            </div>
        </div>

        <div class="layui-form-item" pane="">
            <label class="layui-form-label">选择图片</label>
            <div class="layui-input-block">
            <!--type 为单选框还是多选框-->
                @foreach($results as $result)
                <input class="layui-inline" type="{{$type}}" name="like[]" lay-skin="primary" title="<img style='width:150px;height:150px;' src='{{$result->img_url}}' class='imgs'>" value="{{$result->img_url}}">
                @endforeach
            </div>
        </div>

        <div class="layui-form-item">
            <div class="layui-input-block">
                <button class="layui-btn" lay-submit="" lay-filter="demo1">确定</button>
                <button type="reset" class="layui-btn layui-btn-primary">重置</button>
            </div>
        </div>
    </form>
```

4. 弹出层-上传图片按钮
```javascript
//elem          jQuery ID 选择器  
//multiple      单图 or 多图上传 
//number        多图上传最大可选图片数量
 function uploadInsts(elem,multiple,number){
            if(multiple == null){
                multiple = false;
            }
            if(number == null){
                number = 0;
            }
            layui.use(['upload'],function () {
                var $ = layui.jquery
                    ,upload = layui.upload;
                upload.render({

                    elem: elem
                    ,url: 'https://up.qbox.me/'         //七牛云上传接口
                    ,data:{'token':'{{$token ?? null}}'}    //携带上传 token
                    ,multiple:multiple
                    ,number:number
                    // ,before: function(obj){
                    //     //预读本地文件示例，不支持ie8
                    //     obj.preview(function(index, file, result){
                    //         $('#demo1').attr('src', result); //图片链接（base64）
                    //     });
                    // }
                    ,done: function(res){
                        //上传成功后七牛云返回 key 提交到后端接口保存图片地址及其他信息
                        $.ajax({
                            url:"{{url('admin/upload_save')}}"      
                            ,data:{'key':res.key,'multiple':multiple,'num':number}
                            ,type:'post'
                            ,success:function(result){
                                if(result.error > 0){
                                    layer.msg(result.msg);
                                }else{                      //后端保存成功后重载此页面
                                    location.reload();      
                                }
                            }
                        });
                        //上传成功
                    }
                    ,error: function(){
                        layer.msg('请刷新重试');
                        return false;
                    }
                });
            });
        }
```

5. 弹出层-确定按钮
```javascript
//点击确定后给执行父级页面传入的回调函数名,并关闭此页面
        function backParent() {
            layui.use('form', function () {
                var $ = layui.jquery
                    , form = layui.form;

                //注意：parent 是 JS 自带的全局对象，可用于操作父页面
                var index = parent.layer.getFrameIndex(window.name); //获取窗口索引

                //监听提交
                form.on('submit(demo1)', function (data) {
                    //给父页面传值
                    var s = 0;
                    var res = new Array();
                    for (var i in data.field) {
                        res.push(data.field['like[' + s + ']']);
                        s++;
                    }
                    //把构建好的数据传入回调函数
                    parent.{{$callback}}(res);      //调用父级页面传入的回调函数名，这里的回调函数名是从 php 返回的.
                    parent.layer.close(index);      //关闭此页面

                    return false;
                });
            });
        }
```

6. 父级页面-定义回调函数
```javascript
        function imgsList(res){
            layui.use('element',function(){
                var $ = layui.jquery;
                for(var i=0;i<res.length;i++){
                    if(res[i] != undefined){
                        var html = '';
                        html += '<img src="'+res[i]+'" style="width:150px;height:150px;">';
                        html += '<button class="layui-btn img_delete" type="button" >删除</button>';

                        $('#imgs_list').append(html);
                    }

                }
            });
        }
```

7. 事件实例
```blade
<button type="button" class="layui-btn" onclick="layer_show('上传图片','{{url('admin/upload_select')}}?type=checkbox&callback=imgsList',800,600)" >
```


## laravel overtrue/wechat 用法:

### 网页授权中间件

#### 创建 `app/Http/Middleware/WechatAuth.php` 中间件
```php
    public function handle($request, Closure $next, $account = 'default', $scopes = null)
        {
            // $account 与 $scopes 写反的情况
            if (is_array($scopes) || (\is_string($account) && str_is('snsapi_*', $account))) {
                list($account, $scopes) = [$scopes, $account];
                $account || $account = 'default';
            }
    
            $isNewSession = false;
            $sessionKey = \sprintf('user_id', $account);
            $config = config(\sprintf('wechat.official_account.%s', $account), []);
            $officialAccount = app(\sprintf('wechat.official_account.%s', $account));
            $scopes = $scopes ?: array_get($config, 'oauth.scopes', ['snsapi_base']);
    
            if (is_string($scopes)) {
                $scopes = array_map('trim', explode(',', $scopes));
            }
    
    
            //$session = session([$sessionKey=>'24260']); //测试环境直接赋值session 正式环境注释
            
               /**
                * 1. 是否有 session
                * 2. 没有 session 判断是否有 code
                * 3. 没有 code 获取code-回调到本页
                * 4. 微信返回 code 参数请求本页
                * 5. 有 code 取得 openid 执行数据库操作
                * 6. openid 在数据库里赋值 session 跳到下一步请求
                * 7. openid 不再数据库里 判断能否取到 nickname ,取不到使用 snsapi_userinfo 授权登陆，否则存入相关信息到数据库并赋值 session
                */
            
            if (!session($sessionKey)) {                          //1.没有session
                if ($request->has('code')) {                 //2.有code
    
                    $wechat_user = $officialAccount->oauth->user();         
                    $data['openid'] = $wechat_user->getId() ?? '';          //取得openid
                    $has = UserWechat::getByOpenid($data['openid']);
                    $has_user = User::where('tync_openid',$data['openid'])->first();
                    if(!empty($has) && !empty($has_user)){              //openid 在库里跳转到下一步
                        $user_id = User::where('tync_openid',$data['openid'])->first()->id ?? 0;
                        session([$sessionKey => $user_id ?? '']);
                        return $next($request);
                    }else{                                              //不再库里
                        $data['nickname'] = $wechat_user->getNickname() ?? '';
                        if(empty($data['nickname'])){                   //取不到nickname 使用授权登陆
                            session([$sessionKey => '']);
                            return $officialAccount->oauth->scopes(['snsapi_userinfo'])->redirect($request->fullUrl());
                        }else{                                          //取到nickname 存库
                            $data['headimgurl'] = $wechat_user->getAvatar() ?? '';
                            UserWechat::createUser($data);                  //存入相关信息
                            User::createWeChat($data);
                            $user_id = User::where('tync_openid',$data['openid'])->first()->id ?? 0;
                            session([$sessionKey => $user_id ?? '']);
                        }
                    }
    
                    $isNewSession = true;
    
    //                Event::fire(new WeChatUserAuthorized(session($sessionKey), $isNewSession, $account));
    
                    return redirect()->to($this->getTargetUrl($request));
                }
                /**
                 * 3.没有code 获取code-回调到本页
                 */
    
                session()->forget($sessionKey);
    
                return $officialAccount->oauth->scopes($scopes)->redirect($request->fullUrl()); //获取code-回调到本页
            }
    
    //        Event::fire(new WeChatUserAuthorized(session($sessionKey), $isNewSession, $account));
            
            return $next($request);
        }
```

#### 使用中间价即可
```php
Route::group(['prefix' => 'api', 'middleware' => ['wechat.oauth']],function(){
    Route::get('login','WeChatController@login');
});
```

### 微信公众号支付

#### 生成支付 jssdk
```php
public function payment(Request $request)
    {
        $type = Input::get("type");//支付类型
        $order_no = Input::get("order_no");//订单号
        if (!in_array($type,array("wechat_pay","amount","ali_pay","alipay_app"))){
            return $this->error("支付类型错误");
        }
        $is_wechat = Input::get("is_wechat","0");//是否是微信浏览器
        if (empty($order_no)) return $this->error("参数错误");
        $order = FarmOrder::where('order_no',$order_no)->where("user_id",User::getUserId())->where("pay_status",0)->first();
        if (empty($order)) return $this->error("订单未找到");
        $user_id = User::getUserId();

            //微信支付
            if($type == "wechat_pay"){
            try {
                $order->pay_id = 2;
                $user          = User::find($user_id);
                $app           = app('wechat.payment.default');
                //统一下单(预支付)
                $result        = $app->order->unify([
                    'body'         => '田园农场-支付订单',
                    'out_trade_no' => $order_no,
                    'total_fee'    => $order->total_price * 100,
//                'spbill_create_ip' => '123.12.12.123', // 可选，如不传该参数，SDK 将会自动获取相应 IP 地址
                    'notify_url'   => 'http://www.tianyuannongchang.cn/wechat/notify', // 支付结果通知网址，如果不设置则会使用配置里的默认地址
                    'trade_type'   => 'JSAPI',
                    'openid'       => $user->tync_openid,
                ]);
                //根据预支付数据 生成支付 jssdk
                $payment = Factory::payment(config('wechat.payment.default'));
                $jssdk = $payment->jssdk;
                $json = $jssdk->bridgeConfig($result['prepay_id'],false); // 返回 json 字符串，如果想返回数组，传第二个参数 false
                return $this->success($json);

            }catch (\Exception $exception){
                return $this->error($exception->getMessage());
            }
        }else{
            return $this->error("支付类型错误");
        }
    }
```

#### JS 根据返回 jssdk 唤起微信支付

```javascript

 $("#payment").click(function(){
                $.ajax({
                  url:_SERVER + "api/payment",          //上边的接口
                  type:'post',
                  dataType:'json',
                  data:{
                    type:type,
                    order_no:order_no
                  },
                  success:function(data){
                      if(data.error == 0){
                          if(type == "wechat_pay"){
                                  return  WeixinJSBridge.invoke(
                                      'getBrandWCPayRequest',
                                      data.msg,
                                      function(res){
                                          if(res.err_msg == "get_brand_wcpay_request:ok" ) {
                                              layer.open({
                                                  content: '恭喜您，订单支付成功'
                                                  ,btn: [ '确定']
                                                  ,yes: function(){
                                                      window.location.href="my-order.html";
                                                  }
                                              });
                                          }else{
                                              layer.open({content: '请重试' ,skin: 'msg' ,time: 1 });
                                              return false;
                                          }
                                      }
                                  );
                              
                          }
                      }else{
                          layer_msg(data.message);
                      }
                  }
                })
            })
```


#### 微信回调

```php
 public function notifyWechat(Request $request)
    {

        $app = app('wechat.payment.default');
        $response = $app->handlePaidNotify(function($message, $fail){
            // 使用通知里的 "微信支付订单号" 或者 "商户订单号" 去自己的数据库找到订单
            $order = FarmOrder::getOrderByNo($message['out_trade_no']);

            if (empty($order) || $order->pay_status == 1) { // 如果订单不存在 或者 订单已经支付过了
                return true; // 告诉微信，我已经处理完了，订单没找到，别再通知我了
            }


            if ($message['return_code'] === 'SUCCESS') { // return_code 表示通信状态，不代表支付状态
                // 用户是否支付成功
                if ($message['result_code'] === 'SUCCESS') {
                    $order->pay_time = time(); // 更新支付时间为当前时间
                    $order->pay_status = 1;

                    // 用户支付失败
                } elseif ($message['result_code'] === 'FAIL') {
                    $order->pay_status = 0;
                }
            } else {
                return $fail('通信失败，请稍后再通知我');
            }

            $order->save(); // 保存订单

            return true; // 返回处理完成
        });

        return $response;

    }
```

