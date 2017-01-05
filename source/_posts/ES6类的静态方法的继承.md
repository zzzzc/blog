title: 使用React context踩坑记录之：ES6类的静态方法/属性的继承
date: 2017/01/05
---

如果在React组件中使用context，需要定义组件类的contextTypes。如果在contextTypes中没有定义需要使用的context属性类型，那么访问该属性的值会得到undefined;

当用到context的组件被另一个组件继承，父类代码如下：

```js
export default class SaleHouseItem extends React.Component {
  static propTypes = {
    name: React.PropTypes.string,
  };

  static contextTypes = {
    houseItemConfig:React.PropTypes.object,
    actions:React.PropTypes.object,
    ht:React.PropTypes.number,
    onHouseItemClick:React.PropTypes.func
  };
 ...
}
```

子类代码如下：

```
export default class RentHouseItem extends SaleHouseItem{
  _getHouseName(props,config){
    return [
      config.showEstateName ? props.estateName : null,
      `${props.bedroomSum}室${props.livingRoomSum}厅${props.wcSum}卫`,
      props.spaceArea ? `${props.spaceArea}m²` : null,
    ]
      .filter(Boolean)
      .join(' · ');
  }
  ...
}
```

以上代码在IE10及以下会出现不可描述的效果。debugger之后发现是SaleHouseItem.contextTypes没有被继承到RentHouseItem里。

## ES6类的静态方法的继承

ES6 标准里父类的静态方法/属性能不能被子类继承？查了一些资料后发现是可以继承的（准确的说ES6标准里没有规定类的静态属性，但父类的静态方法是可以被子类继承的）；
查看Babel编译之后的代码，继承的实现如下：

```js
function _inherits(subClass, superClass) {
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
  }
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: {
      value: subClass,
      enumerable: false,
      writable: true,
      configurable: true
    }
  });
  if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}
```
注意以上代码的最后一行，作用就是继承父类的静态属性。**父类的静态属性，实际是继承在子类的`__proto__`上的。由于IE10及以下不支持`__proto__`,Babel编译后的代码继承类的静态属性需要插件实现。**（其实Babel的文档里有警告，原谅我没有完整的撸完Babel的文档T T）




