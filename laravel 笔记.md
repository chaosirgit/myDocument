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