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
然后用 php artisan tinker 进如 laravel 命令行
```php
factory(App\Product::class,50)->create(); //生成 50 条 存入数据库
factory(App\User::class,50)->make();      //生成 50 条 不存入数库
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