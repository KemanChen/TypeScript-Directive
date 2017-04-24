### Typescript改写angular后的directive服务的注入方法
### service服务的注入

* 首先，我们先说一下service服务的注入方法：

    * 初始化静态的$inject数组
    * 通过构造函数以参数的形式将所需服务绑定在类上面，类被调用的时候服务也将被初始化。
    * 在相应的方法上通过调用类属性的方式调用服务就可以了。
    
```javascript
static $inject = ["$log", "$http","$window"];

constructor(private $log: ng.ILogService, private $http: ng.IHttpService, private $window:ng.IWindowService){}  

``` 
    注意：构造函数上的参数即为数组上面的服务
###### `directive`有些不一样，指令在使用之前需要声明，而声明需要调实例方法。

    let cordovaApp: ng.IModule;
    cordovaApp.directive("hngTemplate",hngTemplate.instance());
基于这样的需求，指令最基本的需要有一个`constructor`，一个`instance`，一个`link`。那么如何在指令上面注入服务呢？跟service类似，需要在constructor函数上面写上所需的服务作为参数，实例方法里面绑定所需的服务和返回的对象。

    constructor(private $log: ng.ILogService, private $window: ng.IWindowService) { }

    //hngTemplate的实例化   

    static instance(): ng.IDirectiveFactory {
      let directive = ($log: ng.ILogService, $window:ng.IWindowService) => new hngTemplate($log, $window);
      directive.$inject = ["$log", "$window"];
      return directive;
    }

    link = ($scope: ng.IScope, elm: Element, attrs:hngTemplateInterface) => {});
###### 这里强调最重要的一点，关于`link`函数，`link`函数不能直接定义为函数，即不能写成`link()`，应该写成`link=()=>{}`。

###### 为什么？

###### 因为会在实例函数还没有执行之前执行(我的理解是：在编译链接阶段会执行)，而所有的服务都是在实例函数执行的时候才会被注入进来，所以这时候程序会报错。因此我们要写成第二种形式。
