---
title: laravel-permission权限管理的使用
date: 2021-06-05 13:07:42
tags: [2021,laravel,laravel-permission]
categories: [代码]
top_img: /img/laravel-permission.png
cover: /img/laravel-permission.png
---
# 安装
## 安装composer包
`composer require spatie/laravel-permission`
## 配置
* 生成迁移文件
    `php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="migrations"`
* 运行迁移命令 
    `php artisan migrate`
* 生成配置文件
    `php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="config"`


* 用户模型需使`Spatie\Permission\Traits\HasRoles`,如果使用了其他guard,还需设置`guard_name`属性

* 如果要扩展Role和Permission模型,需要继承扩展包的
`Spatie\Permission\Models\Role`和`Spatie\Permission\Models\Permission`,并在 项目根目录/config/permission.php 文件中,修改对应的模型

```php

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Spatie\Permission\Traits\HasRoles;

class Admin extends Authenticatable
{
    use HasFactory, Notifiable,HasRoles;

    protected $table = 'admins';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name',
        'phone',
        'password',
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
    
    //设置guard为jwt
    protected $guard_name = 'jwt';
}

```

# 使用方法

```php
use App\Models\Admin;
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;

//创建权限
Permission::create('admin.index');
Permission::findOrCreate('admin.update');
//创建角色
$role = Role::create(['name' => '测试管理员']);
//给角色添加权限
$role->givePermissionTo('admin.index','admin.update');
//给角色同步权限(用数组中的权限替换用户的权限)
$role->syncPermissions(['admin.index','admin.store']);
//给角色删除权限
$role->revokePermissionTo('admin.index');

/*
 * 用户直接管理权限
 * 该扩展支持不通过角色,直接给用户分配权限
 */
$admin = new Admin();
//给用户添加权限
$admin->givePermissionTo('admin.index','admin.update');
$admin->givePermissionTo(['admin.index','admin.update']);
//给用户同步权限
$admin->syncPermissions(['admin.index','admin.store']);
//给用户删除权限
$admin->revokePermissionTo('edit');


//给用户添加角色
$admin->assignRole('测试管理员');
$admin->assignRole('测试管理员','测试管理员二号');
$admin->assignRole(['测试管理员','测试管理员二号']);
//给用户移除一个角色
$admin->removeRole('测试管理员二号');
//给用户同步角色
$admin->syncRoles(['测试管理员']);


//角色是否有权限
$role->hasPermissionTo('admin.index');
//用户是否有角色
$admin->hasRole('测试管理员');
//用户是否有以下任意一个角色
$admin->hasAnyRole(['测试管理员','测试管理员二号']);
//用户是否有以下所有角色
$admin->hasAllRoles(['测试管理员','测试管理员二号']);
//用户是否有权限
$admin->hasPermissionTo('admin.index');
$admin->hasPermissionTo(1);//直接传权限的id也可以
//用户是否有以下任意一个权限
$admin->hasAnyPermission(['admin.index','admin.update']);
//用户是否有以下所有权限
$admin->hasAllPermissions(['admin.index','admin.update']);

//获取用户的所有权限
$permissions = $user->getAllPermissions();
//获取用户被直接分配的权限
$permissions = $user->getDirectPermissions();
//获取用户通过角色分配的权限
$permissions = $user->getPermissionsViaRoles();

// 获取用户所有角色名称
$roles = $user->getRoleNames(); // 返回一个collection

```

ok,先这么多


>我这边直接给项目的原中间件修改了一下,当然也可以用他提供的中间件`RoleMiddleware`,`PermissionMiddleware`,`RoleOrPermissionMiddleware`

