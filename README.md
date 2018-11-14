# angular-v1.4.12
angular做项目记录
### 1.动态添加的dom绑定指令
动态添加的dom无法绑定angular的指令：ng-model, ng-if, ng-repeat等，要通过$compile服务编译，并传入$scope变量，例如：
```
// 需要引入$compile服务，$scope为当前作用域
var html = '<span ng-click='zjtest()'>click</span>';
$('.zjtest').html($compile(html)($scope));
```
如果直接使用$('.zjtest').html(html); zjtest()方法执行不到，因为在添加dom时，angular编译已经完成，需要手动编译。
### 2.特定场合需要$apply
在setTimeout中的操作需要使用$apply, 因为里面的操作不会触发更新dom，操作的值不会立即反映到dom上，需要额外点击之外的click或者其他时间触发一下，dom才会更新，这是可以使用
angular自带的$timeout服务，或者$scope.$apply(function(){ 操作写在这里});例如：
```
setTimeout(function(){
    $scope.$apply(function () {
        // 操作
    });
}, 500);
```
或者
```
// 需要先引入$timeout服务在控制器函数参数里面
$timeout(function(){
    // 操作
}, 500)
```
### 3.向子弹框传值，子弹框向该控制器传值
控制器给子弹框的控制器传值，可以使用$uibModal.open的resolve属性定义传入自控制器的属性，在自控制器的函数参数里面接收属性，给$uibModal.open添加scope: $scope,子控制器会继承当前控制器的$scope作用域，如果是在子控制器中改变该控制器中的值，在页面上的值如果要实时更新，需要关闭弹框的时候，例如在$scope.close函数里面，调用通过参数传递过来的函数，再去触发该函数下面的相同属性。例如：
```
$uibModal.open({
    templateUrl: 'test.html',
    windowTemplateUrl: 'window.html',
    backdrop: false,
    size: 'lg',
    scope: $scope,
    controller: 'bindUser',
    resolve: {
        refreshUserList: function () {
            return function (userList) {
                $scope.userList = userList;
            }
        },
        isDutyOrRole: function () {
            return $scope.isDutyOrRole
        }
    }
});
```
```
cBoard.controller('bindUser', function ($scope, $uibModalInstance, refreshUserList, isDutyOrRole) {
    ...
    if(isDutyOrRole == 'duty'){
        // isDutyOrRole是duty时的操作
    }else{
        // isDutyOrRole是role时的操作
    }
    ...
    $scope.close = function () {
        $uibModalInstance.close();
        refreshUserList($scope.userList);
    };
    ...
});
```
