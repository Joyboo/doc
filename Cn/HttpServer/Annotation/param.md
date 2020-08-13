---
title: easyswoole注解控制器 - 参数注解
meta:
  - name: description
    content: easyswoole注解控制器 - 参数注解
  - name: keywords
    content:  easyswoole注解控制器 - 参数注解
---
# 参数注解校验
Easyswoole控制器总共有三个参数注解标签，分别是
- @Param ```EasySwoole\HttpAnnotation\AnnotationTag\Param```
- @ApiAuth ```EasySwoole\HttpAnnotation\AnnotationTag\ApiAuth```
- @ApiGroupAuth ```EasySwoole\HttpAnnotation\AnnotationTag\ApiGroupAuth```

其中，```ApiAuth```与```ApiGroupAuth```均继承自```Param```，对于任意一个参数注解，都要求填写注解的```name```字段。```Param```对象实际上是对```Easyswoole/Validate```参数验证组件验证规则的封装，底层是调用该组件进行参数校验。当校验失败的时候，则会抛出一个```EasySwoole\HttpAnnotation\Exception\Annotation\ParamValidateError```异常，可以在控制器的```onExcepion```中进行处理。

## @Param
基础参数注解，作用域在控制器的```actionMethod```与```onRequest```均为有效。例如在以下代码中：
```
/**
* @Param(name="name",required="",lengthMax="25")
* @Param(name="age",integer="")
*/
function index()
{
    $data = $this->request()->getRequestParam();
    $this->response()->write("your name is {$data['name']} and age {$data['age']}");
}
```
那么则规定了```index```这个action需要```name```与```age```这两个参数，且校验规则分别为```required="",lengthMax="25"```与```integer=""```

#### 参数的接收
在控制器的```Request```对象中得到的参数值，为客户端提交的原始值，参数的注解校验或者预处理，并不会影响原始值。但是通过控制器自动传参或者是上下文注解标签得到的参数，则为经过预处理后的参数。

##### 自动传参
```
/**
* @Param(name="name",required="",lengthMax="25",from={GET,POST})
* @Param(name="age",type="int")
*/
function index($age,$name)
{
   $data = $this->request()->getRequestParam();
   $this->response()->write("your name is {$name} and age {$age}");
}
```
当某个action定义了参数，且有注解的时候，那么控制器会利用反射机制，根据函数定义的参数名，去取对应的参数。
##### 注解传参数
```
/**
* @Param(name="name",required="",lengthMax="25",from={GET,POST})
* @Param(name="age",type="int")
* @InjectParamsContext(key="data")
*/
function index()
{
    $data = ContextManager::getInstance()->get('data');
    $this->response()->write("your name is {$data['name']} and age {$data['age']}");
}
```
通过```@InjectParamsContext```标签，完整命名空间是```EasySwoole\HttpAnnotation\AnnotationTag\InjectParamsContext```，我们可以把通过验证的参数，设置到指定的协成上下文中，并通过上下文管理器```EasySwoole\Component\Context\ContextManager```得到对应的参数。其中，除了必填的```key```字段，还有如下几个字段：
- onlyParamTag

    忽略```@ApiAuth```与```@ApiGroupAuth```定义的参数
- filterNull
    
    忽略值为null的参数
- filterEmpty
    
    忽略值被empty()判定为true的参数，注意数字0或者是字符串0与空字符串等问题
    
#### 附加字段
```@Param```注解除了```name```字段为必填项，还有以下几个辅助字段。
##### from
例如在以下注解中：
```
* @Param(name="name",required="",lengthMax="25",from={GET,POST})
* @Param(name="age",integer="",from={POST})
```
则规定了```name```字段允许的取参顺序为：GET => POST，而```age```参数就仅仅允许为GET传参。目前from的允许值为：
```POST```，```GET```，```COOKIE```,```HEADER```,```FILE```,```DI```,```CONTEXT```,```RAW```。在无规定from字段时，默认以```$request->getRequestParam($paramName)```方法获得参数值。具体实现可以在```EasySwoole\HttpAnnotation\AnnotationController```的```__handleMethodAnnotation```方法中查看。

#### type
例如以下注解中：
```
* @Param(name="age",type="int")
```
通过函数自动传参，或者是```@InjectParamsContext```得到的参数时，会对```age```这个参数进行intval()处理。```type```字段可选值为：```string```，```int```,```double```,```real```,```float```，```bool```，```object```，```array```，具体可以在```EasySwoole\HttpAnnotation\AnnotationTag\Param```的```typeCast```方法中查看。

#### defaultValue
在客户端没有传递该参数的值时，可以用该字段进行默认值的定义。

#### preHandler
该字段是用于对某个参数值不为null时进行预处理。```preHandler```需要是一个callable，例如
```
* @Param(name="password",preHandler="md5")
```
则通过函数自动传参，或者是```@InjectParamsContext```得到的参数时，```password```会被自动执行md5()
## 控制器全局参数
全局注解标签是```@ApiGroupAuth```，完整的命名空间是
```
namespace App\HttpController;


use EasySwoole\HttpAnnotation\AnnotationController;
use EasySwoole\HttpAnnotation\AnnotationTag\ApiGroupAuth;


/**
 * Class Index
 * @ApiGroupAuth(name="token",required="")
 * @package App\HttpController
 */
class Index extends AnnotationController
{
   
}
```