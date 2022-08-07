## Packagit Starter

starter 作为新项目的基础工程，与 laravel 8.0 分支同步更新.

starter 里面集成了若干常用的基础能力：

✔️ 微信APP/小程序/H5 的登录授权

✔️ 微信支付

✔️ 苹果登录授权

✔️ modules/Components/Common/Models/BaseModel.php

✔️ modules/Components/Common/Http/Controllers/BaseController.php

可以通过 `php artisan route:list --path=api` 查看暴露的API列表，相关API使用参考：

<https://artisansoft.feishu.cn/docs/doccnbinoOr0qUXOI00OMEMaqQg>

### Auth 授权模块

API token 目前集成的是 sanctum，可以根据需要增加 JWT, passport 等支持，修改 API token 逻辑需要调整的位置:

1、修改 config/auth.php 中的 guards.api.driver:

```
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver'   => 'sanctum',
        'provider' => 'users',
        'hash'     => false,
    ],
],
```

2、修改 modules/Auth/Models/User.php 中 getUserToken 方法

```
$appid = $this->appid ?? AppHelper::getAppId();
return $this->createToken($appid)->plainTextToken;
```

3、修改授权登录返回的 JSON 格式

修改文件 modules/Auth/Http/Resources/TokenResource.php

```
public function toArray($request)
{
    $data = ['token' => $this->resource->getUserToken()];

    if ($this->attachData) {
        $data = array_merge($data, $this->attachData);
    }

    return $data;
}
```


### BaseModel

新建 model 需继承 BaseModel，内含日期相关的 Carbon 的日期格式化，如果需要返回字符串日期时，需要指定 string 类型, 如 `(string)$user->created_at`.

### BaseController

新建的 Controller 需继承 BaseController, Controller 类里直接 $this->ok(), $this->error().

因为注入了 Macro, Controller 类外可以用 response()->ok(), response()->error()，使用方法一致。

常规JSON数据返回可直接用 response()->json，较复杂情况可以配合 Resource 的使用:

    response()->json(TokenResource::make($user));

只返回成功或者失败的错误信息，可以使用封装:

    $this->ok('您的密码已更新成功');
    // or
    $this->error('用户注册失败');

特殊约定使用参考:

    return app(BaseController::class)
        ->ok('Apple 授权成功，还差最后一步', 1403, [
            'provider'  => $oauthUser->provider,
            'search_by' => $oauthUser->getTable(),
            'search_id' => encrypt($oauthUser->id),
        ]);


### 基础用户 Model

进入 modules/Auth/Models 目录下查看.

1、IUserFlexible.php:

用户 Model 的 interface 定义，当需要自定义 User Model 时，需要 implement IUserFlexible

2、User.php

用户 Model，里面封装了基础功能

3、UserMeta.php

对 user_metas 表是对 users 表的扩展，user 不太频繁的属性字段定义可以放在 user_metas 表，频繁常用的小字段放在 users 表.

### 重要规范说明

starter 工程使用模块化，也就是根据功能来封装模块，也可以叫做插件化。

modules/Components/* 的作用，是模块间通用的组件封装，放在这个目录里。

modules/*，是各个模块，按需添加使用, 其中 Auth, Wechat 是常规模块，工程中经常会用到的。
