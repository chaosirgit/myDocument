# Laravel 学习笔记

## 前言
记录 Laravel 开发中的问题，及笔记。  

## Validator 类的用法
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