---
title: 使用jwt在laravel中实现api的登录授权
date: 2021-05-26 10:55:50
tags: [2021,代码,api,jwt]
categories: [代码]
top_img: /img/jwt.jpeg
cover: /img/jwt.jpeg
---
# 简单了解jwt
jwt全称json web token，是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（(RFC 7519)。特别适用于分布式站点的单点登录（SSO）场景。

## jwt认证流程
1. 客户端表单提交用户认证信息发送到服务端
2. 服务端验证用户信息，生成jwt（包括header，payload，signature），jwt的组成后面再具体研究
3. 服务端返回token给客户端
4. 客户端存储token，并在后续每次请求服务端时在Http请求的header中的Authorization中上送token，（也可不放在header中，但建议在header中上送，保证安全）
5. 服务端收到请求，验证jwt，然后处理业务逻辑返回结果

## jwt认证与session认证的优势
1. 用户认证后会存储用户认证记录在session中，而session通常保存在内存中，这就导致随用户增多，内存使用也会逐步增加。jwt则不存在内存问题。
2. session认证的方式，认证记录存储在服务器的内存中，这就导致分布式应用中，其他服务器上无法获取认证记录。

# 在laravel中具体使用

## 安装composer包
`composer require tymon/jwt-auth`
## 配置
### 生成配置文件 config/jwt.php
`php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"`
### 生成加密秘钥(会在env中添加一个JWT_SECRET配置项)
`php artisan jwt:secret`
### 更换laravel的auth配置
编辑 项目根目录/config/auth.php

```php
//这边项目为纯api,否则可不修改默认的guard
'defaults' => [
    'guard' => 'jwt',
    'passwords' => 'users',
],
...
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'token',
        'provider' => 'users',
        'hash' => false,
    ],
    //这边新增了一个guard,也可直接修改原有的web及api,修改driver为jwt即可
    'jwt' => [
        'driver' => 'jwt',
        'provider' => 'users',
        'hash' => false,
    ],
],
...
//这边修改了默认的登录用户模型,如果使用原始的User,则不需改动
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\Admin::class,
    ],
],
```
### 修改用户模型
>继承JWTSubject类并实现getJWTIdentifier及getJWTCustomClaims方法

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Spatie\Permission\Traits\HasRoles;
use Tymon\JWTAuth\Contracts\JWTSubject;

//此处需继承JWTSubject类,并实现getJWTIdentifier及getJWTCustomClaims两个方法,如下
class Admin extends Authenticatable implements JWTSubject
{
    use HasFactory, Notifiable,HasRoles,SoftDeletes;

    protected $table = 'admins';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name',
        'phone',
        'email',
        'password',
        'is_super'
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];

    // Rest omitted for brevity

    /**
     * Get the identifier that will be stored in the subject claim of the JWT.
     *
     * @return mixed
     */
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    /**
     * Return a key value array, containing any custom claims to be added to the JWT.
     *
     * @return array
     */
    public function getJWTCustomClaims(): array
    {
        return [];
    }

}
```

### 注册facade
>不注册也行,直接是用auth()函数也是可以的

编辑 项目根目录/config/app.php
```php
...
'aliases' => [
    ...
    'JWTAuth' => Tymon\JWTAuth\Facades\JWTAuth::class,
    'JWTFactory' => Tymon\JWTAuth\Facades\JWTFactory::class,
],
```
## 逻辑实现
### 注册中间件
>这边项目需要,基于官方的中间件重写了一个,如无特殊需要可不注册,直接使用官方的jwt.auth中间件

项目根目录/app/Http/Kernel.php
```php
protected $routeMiddleware = [
    ...
    'api.auth' => \App\Http\Middleware\ApiAuth::class,
];
```

### 添加路由

```php
Route::namespace('Auth')->post('register', 'AuthController@register')->name('erp.register');
Route::namespace('Auth')->post('login', 'AuthController@login')->name('erp.login');
Route::namespace('Auth')->middleware('api.auth')->post('logout', 'AuthController@logout')->name('erp.logout');
```

    
### 中间件代码

```php
<?php

namespace App\Http\Middleware;


use App\Exceptions\Erp\PermissionForbiddenException;
use App\Exceptions\Erp\RouteNotFoundException;
use App\Exceptions\Erp\UnauthorizedException;
use Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException;
use Tymon\JWTAuth\Exceptions\TokenBlacklistedException;
use Tymon\JWTAuth\Exceptions\TokenExpiredException;
use Tymon\JWTAuth\Exceptions\TokenInvalidException;
use Tymon\JWTAuth\Http\Middleware\BaseMiddleware;

class ApiAuth extends BaseMiddleware//继承jwt的BaseMiddleware,以使用他的方法
{

    /**
     * @throws PermissionForbiddenException
     * @throws \Tymon\JWTAuth\Exceptions\JWTException
     * @throws RouteNotFoundException
     */
    public function handle($request, \Closure $next)
    {
        try {
            /**
             * 检查此次请求中是否带有 token，
             * 如果没有则抛出UnauthorizedHttpException异常。
             */
            $this->checkForToken($request);
            /**
             * 检测用户的登录状态
             * 如果校验失败则抛出TokenInvalidException异常
             * 如果过期则抛出TokenBlacklistedException异常
             * 如果过期后刷新token后又过期了,则后面无法再刷新token,抛出TokenExpiredException异常
             */
            if ($this->auth->parseToken()->authenticate()) {
                //todo 身份验证通过,具体实现其他业务逻辑
            }
            throw new UnauthorizedException('未登录');
        } catch (UnauthorizedHttpException $exception) {
            throw new UnauthorizedException('未获取到token');
        } catch (TokenExpiredException | TokenBlacklistedException $exception) {
            throw new UnauthorizedException('token令牌已失效,请重新登录');
        } catch (TokenInvalidException $exception) {
            throw new UnauthorizedException('无效的token');
        }
    }

}

```

### 控制器代码
```php
<?php

namespace App\Http\Controllers\Api\Auth;

use App\Exceptions\Erp\InvalidArgumentException;
use App\Exceptions\Erp\RepeatException;
use App\Exceptions\Erp\ServiceException;
use App\Http\Controllers\Controller;
use App\Models\Admin;
use Illuminate\Foundation\Validation\ValidatesRequests;
use Illuminate\Http\Request;
use Tymon\JWTAuth\Exceptions\JWTException;
use Tymon\JWTAuth\Facades\JWTAuth;
use Validator;

class AuthController extends Controller
{
    use ValidatesRequests;

    /**
     * @param Request $request
     * @return array
     * @throws \Illuminate\Validation\ValidationException
     * @throws RepeatException
     */
    public function register(Request $request): array
    {
        $data = $request->all();
        $validator = Validator::make($data, [
            'phone' => 'required',
            'name' => 'required|max:64',
            'password' => 'required|max:64',
        ], [
            'phone.required' => '手机号必填',
            'phone.digits' => '手机号格式错误',
            'name.required' => '用户名必填',
            'name.max' => '用户名长度超限',
            'password.required' => '密码必填',
            'password.max' => '密码长度超限',
        ]);
        if ($validator->fails()) {
            throw new InvalidArgumentException($validator->errors()->messages());
        }

        try{
            $admin = Admin::query()->create([
                'phone' => $data['phone'],
                'name' => $data['name'],
                'password' => bcrypt($data['password'])
            ]);
        }catch (\Exception $exception){
            throw new RepeatException('手机号已注册,请直接登录');
        }
        $token = JWTAuth::fromUser($admin);

        return [
            'token' => $token
        ];
    }

    /**
     * @param Request $request
     * @return array
     * @throws ServiceException
     * @throws \Illuminate\Validation\ValidationException
     */
    public function login(Request $request): array
    {
        $data = $request->all();
        $validator = Validator::make($data, [
            'name' => 'required',
            'password' => 'required|max:64',
        ], [
            'name.required' => '用户名必填',
            'password.required' => '密码必填',
            'password.max' => '密码长度超限',
        ]);
        if ($validator->fails()) {
            throw new InvalidArgumentException($validator->errors()->messages());
        }

        $credentials = $request->only('name', 'password');

        try {
            if (! $token = JWTAuth::attempt($credentials)) {
                throw new InvalidArgumentException('用户名或者密码错误');
            }
        } catch (JWTException $e) {
            throw new ServiceException('token 无法生成');
        }

        return [
            'token' => $token
        ];

    }

    public function logout()
    {
        JWTAuth::setToken(JWTAuth::getToken())->invalidate();
        return [];
    }
}
``` 

#完事,测试一下
给整个postman组添加统一的authorization里的token变量
![-w571](/images/16220968473522.jpg)


![-w993](/images/16220967381821.jpg)
![-w1000](/images/16220967715985.jpg)
![-w1005](/images/16220968118863.jpg)

退出后再退出测试一下中间键
![-w1021](/images/16220969071886.jpg)

完事,撤
![](/images/16220969707137.jpg)
