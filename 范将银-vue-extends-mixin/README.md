#vue组件开发（风格、继承、混合、注入）
##风格
[<vue风格指南>](https://cn.vuejs.org/v2/style-guide/)

###组件名
组件名不要用index.vue, 独立封装打包的除外。
调试时，多个index文件提示信息定位会出错

单文件组件文件的大小写
---
~~~
components/
|- MyComponent.vue
components/
|- my_component.vue
~~~
基础组件
---
~~~
components/
|- BaseButton.vue
|- BaseTable.vue
|- BaseIcon.vue
components/
|- AppButton.vue
|- AppTable.vue
|- AppIcon.vue
components/
|- VButton.vue
|- VTable.vue
|- VIcon.vue
~~~
###Prop 定义

~~~
/**
* default = false, 单写defaultSort得到true;
* default = true, 单写defaultSort得到true,
* :defaultSort="true" 得到true
* 或default-sort
* 注：根据常用性定义default值
*/
<vue-c default-sort><vue-c/>

props: {
  status: {
    type: String,
    required: true,
    validator: function (value) {
      return [
        'syncing',
        'synced',
        'version-conflict',
        'error'
      ].indexOf(value) !== -1
    }
  }
}
~~~
##mixins(混合/扩展)
调用方式: mixins: [mixin1, mixin2]
是对父组件的扩充，包括methods、components、directive等。。。

触发钩子函数时，先调用mixins的函数，再调用父组件的函数。

虽然也能在创建mixin时添加data、template属性，但当父组件也拥有此属性时以父为准，从这一点也能看出制作者的用心（扩充）。

data、methods内函数、components和directives等键值对格式的对象均以父组件/实例为准

##extends(继承)
同样是对父组件的扩充，与mixins类似，但优先级均次于父组件


##provide / inject
~~~
provide：Object | () => Object
inject：Array<string> | { [key: string]: string | Symbol | Object }
~~~




