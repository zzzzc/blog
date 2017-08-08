title: 使用Redux构造RESTful风格的Actions
date: 2016/12/1
---

# 使用Redux构造RESTful风格的Actions

用redux管理前端数据流总是要根据业务逻辑定义许多actions，通常这些actions的数据操作并不复杂，无非是更新一个字段或者向数组中插入/删除一个元素，构造一套统一标准的actions可以减少工作量和复杂度。RESTful风格的actions简洁明了，并可以实现大部分资源修改需求。

## 目标功能
传入store数据结构，自动生成相应的actions。此处以一个todolist为例，config如下：

```js
[
  {
    key:'formData', //store的名字，也是action的名字
    value:{ //value会作为默认值
    	input:'',
    	star:false
    }
  },
  {
    key: 'todos',
    value: []
  },
]
```
在store中创建了formData储存用户输入，进行表单验证；todos储存已添加的代办项。

如果store是Object，例如：

1. 更改input的值，调用`actions.formData('UPDATE',{input:'买电影票'})`;
2. 清空input的值，调用`actions.formData('UPDATE',{input:undefined})`
3. ......

如果store是Array，例如：

1. 当向todos添加待办时，调用`actions.todos('ADD',{id:1, star:false, text:'买电影票'})`;
2. 当向todos修改待办时，调用`actions.todos('UPDATE',{id:1},{star:true})`;
3. 当向todos删除待办时，调用`actions.todos('DELETE',{id:1})`;
4. ......

## 具体实现
首先，在actionTypes.js中定义资源操作动作：

```js
'use strict';

export const SET = 'SET';
export const UPDATE = 'UPDATE';
export const PATCH = 'PATCH';

export const ADD = 'ADD';
export const DELETE = 'DELETE';
```

实现目标功能：

```js

'use strict';

import {bindActionCreators} from 'redux';
import {SET,UPDATE,PATCH,ADD,DELETE} from './actionTypes.js';

// 获取action type
function getType (key,action){
  return `${key}.${action}`;
}

// 从数组中找到条目
function arrayFind(array,find){
  let index = _.findIndex(array,find);
  return [index,array[index]];
}

// 获取Reducers
function getReducers(states){
  let reducers = {};
  states.forEach(data => {
    let {key,value} = data;
    if (_.isArray(value)){
      // 这里为数组定义了5总操作方法
      let set = getType(key,SET); //设置store的值
      let update = getType(key,UPDATE); //修改某一项的值
      let patch = getType(key,PATCH); //接收一个修改方法，更新某一项的值
      let add = getType(key,ADD); //添加一项
      let del = getType(key,DELETE); //删除某项
      reducers[key] = function(state = value,action){
        let {type,find,data} = action;
        let [index,item] = find ? arrayFind(state,action.find) : [-1];
        switch (type){
          case set:
            return data;
          case add:
            return [...state,data];
          case update:
            if (index !== -1){
              state[index] = {...item,...data};
              return [...state];
            }
          case del:
            if (index !== -1){
              state.splice(index,1);
              return [...state];
            }
          case patch:
            if (index !== -1){
              state[index] = data(item);
              return [...state];
            }
          default :
            return state;
        }
      }
    }else{
      // 为Object定义了3总操作方法
      let set = getType(key,SET); //设置Object的值
      let update = getType(key,UPDATE); //更新Object的值
      let patch = getType(key,PATCH); //接收一个修改方法，更新Object的值
      reducers[key] = function(state = value,action){
        let {type,data} = action;
        switch (type){
          case set:
            return data;
          case update:
            return {...state , ...data};
          case patch:
            return {...data(state)};
          default:
            return state;
        }
      }
    }
  });
  return reducers;
}

// 获取Actions
function getActions(states){
  let actions = {};
  states.forEach(data => {
    let {key,value} = data;
    if (_.isArray(value)){
      actions[key] = function(act,...arg) {
        let type = getType(key,act);
        let [find,data] = arg.lenght === 1 ? [undefined,arg[0]] : arg;
        return {type,find,data};
      }
    }else{
      actions[key] = function(act,data) {
        let type = getType(key,act);
        return {type,data};
      }
    }
  });
  return actions;
}

// 输出方法，接受storesConfig输出actions和reducers;
export default function(storesConfig){
  let actions = getActions(states);
  let reducers = getReducers(states);
  return {actions, reducers};
}

```

## 需要改进
以上代码可以实现大部分数据流操作，但一些不满足需求的actions（如批量操作）还是需要自己编写。


