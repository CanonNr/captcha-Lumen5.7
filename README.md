# Captcha for Lumen 5.7

本项目修改自[Captcha for Laravel 5](https://github.com/mewebstudio/captcha) 和 [lumen-captcha](https://github.com/Yangbx/captcha-lumen)

## Requirement
Lumen version >= v5.5


## Preview
![Preview](http://i.imgur.com/HYtr744.png)

## Install
* 此 Package 必须开启 Redis才能使用，因为验证码与绑定验证码的 uuid 都是保存在Redis的。

```
composer require canon/captcha-lumen
```


## How to use

在`bootstrap/app.php`中注册Captcha Service Provider：

```php
    $app->register(Canon\CaptchaLumen\CaptchaServiceProvider::class);
    class_alias('Canon\CaptchaLumen\Facades\Captcha','Captcha');
```


## Set

在`bootstrap/app.php`中可以设定各种自定义类型的验证码属性，更多详细设定请查看 [Captcha for Laravel 5](https://github.com/mewebstudio/captcha)
```php
/**
 * captcha set
 */
config(['captcha'=>
    [
        'useful_time' => 5, //验证码有效时间（分钟）
        'captcha_characters' => '2346789abcdefghjmnpqrtuxyzABCDEFGHJMNPQRTUXYZ',
        'sensitive' => false, //验证码是否判断大小写
        'login'   => [ //验证码样式
            'length'    => 4, //验证码字数
            'width'     => 120, //图片宽度
            'height'    => 36, //字体大小和图片高度
            'angle'     => 10, //字体倾斜度
            'lines'     => 2, //横线数
            'quality'   => 90, //品质
            'invert'    =>false, //反相
            'bgImage'   =>true, //背景图
            'bgColor'   =>'#ffffff',
            'fontColors'=>['#339900','#ff3300','#9966ff','#3333ff'],//字体颜色
        ],
    ]
]);
```
如果不配置设定档，默认就是default，验证码有效时限为5分钟。
## Example
因为 Lumen 都是无状态的 API，所以验证码图片都会绑上一个 UUID，先获得验证码的 UUID 跟图片的 URL，验证时再一併发送验证码与 UUID。
### Generate
获得验证码：
```
{Domain}/captchaInfo/{type?}
```
`type`就是在 config 中定义的 Type，如果不指定`type`，默认为`default`样式，Response：
```json
{
  "captchaUrl": "http://{Domain}/captcha/default/782fdc90-3406-f2a9-9573-444ea3dc4d5c",
  "captchaUuid": "782fdc90-3406-f2a9-9573-444ea3dc4d5c"
}
```
`captchaUrl`为验证码图片的 URL，`captchaUuid`为绑定验证码图片的uuid。

#### validate
在发送 Request 时将验证码与 UUID 一併送回 Server 端，在接收参数时做验证即可：
```php
# 原版验证
# 因为我修改了存储位置也没有使用该方法作为验证所以已失效,未来将会修改
public function checkCaptcha(Request $request, $type = 'default',$captchaUuid)
{
    $this->validate($request,[
        'captcha'=>'required|captcha:'.$captchaUuid
    ]);
    ...
}

# 我现在使用的
public function checkCaptchaCode($captchaId,$userCode)
{
    $captchaId = 'captcha_'.$captchaId;
    $redis = app('redis.connection');
    $captchaCode = $redis->get($captchaId);

    if(empty($captchaCode)){
        return $apidoc->loginCaptchaError();
    }

    $sensitive = config('captcha.sensitive');
    if (!$sensitive){
        $captchaCode = strtolower($captchaCode);
        $userCode = strtolower($userCode);
    }

    if($captchaCode == $userCode){
        return $apidoc->loginCaptchaSuccess();
    }

    return $apidoc->loginCaptchaError();
}
```
## Update log
*   v1.0 （2019-07）
    * 验证码图片的显示错误问题
    * 验证码存储位置由Cache=>Redis
    * 部分配置文件不生效
    * 原版验证修改（计划）

## Links
* [Intervention Image](https://github.com/Intervention/image)
* [L5 Captcha on Github](https://github.com/mewebstudio/captcha)
* [L5 Captcha on Packagist](https://packagist.org/packages/mews/captcha)
* [For L4 on Github](https://github.com/mewebstudio/captcha/tree/master-l4)
* [License](http://www.opensource.org/licenses/mit-license.php)
* [Laravel website](http://laravel.com)
* [Laravel Turkiye website](http://www.laravel.gen.tr)
* [MeWebStudio website](http://www.mewebstudio.com)
