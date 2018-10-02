# 杂项 {ignore=true}

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [辅助工具](#辅助工具)
	* [性能工具](#性能工具)
	* [reslect](#reslect)
* [优化](#优化)
	* [减少不比较的创建](#减少不比较的创建)
		* [错误实例](#错误实例)
		* [最佳实践](#最佳实践)
	* [优先使用Key](#优先使用key)
	* [优雅的删除属性](#优雅的删除属性)
	* [高阶组件](#高阶组件)
		* [代理方式](#代理方式)
			* [操作props](#操作props)
			* [访问ref](#访问ref)
			* [抽取状态](#抽取状态)
			* [包装组件](#包装组件)
		* [继承方式](#继承方式)
			* [操作props](#操作props-1)
			* [操作生命周期函数](#操作生命周期函数)
		* [高阶组件的显示名](#高阶组件的显示名)
* [生命周期](#生命周期)
	* [组件更新](#组件更新)

<!-- /code_chunk_output -->

## 辅助工具

### 性能工具

React Perf

### reslect

## 优化

### 减少不比较的创建

React中使用的是浅比较

#### 错误实例

```js
<Foo style={{color:'red'}}/>
```

#### 最佳实践

```javascript
const fooStyle={color:'red'};
<Foo style={fooStyle}>
```

### 优先使用Key

### 优雅的删除属性

通过ES6语法过滤对象属性
避免`user`传给`<Foo/>`

```js
const {user,...otherProps}=this.props;
<Foo {...otherProps}/>
```

### 高阶组件

高阶组件(Higher Order Component,HOC)
高价组件返回的是**组件**,而不是函数,
redux 中 `connect` 不是一个高阶组件,`connect` 返回的是**函数**
`connect` **返回的函数** 才是高阶组件

#### 代理方式

##### 操作props

添加props 高阶组件

```js
const addNewPropsHOC=(WrappedComponent,newProps)=>{
    return class WrappingComponent extends React.Component{
        render(){
            return <WrappedComponent {...this.props} {...newProps} />
        }
    };
}
const FooComponent = addNewPropsHOC(DemoComponent,{foo:'foo'});
const BarComponent = addNewPropsHOC(DemoComponent,{bar:'bar'});
```

##### 访问ref

```js
const refsHOC=(WrappedComponent)=>{
    return class HOCComponent extends React.Component{
        constructor(){
            super(...arguments);
            this.linkRef = this.linkRef.bind(this);
        }

        linkRef(wrappedInstance){
            this._root = wrappedInstance;
        }

        render(){
            const props={...this.props,ref:this.linkRef};
            return <WrappedComponent {...props} />;
        }
    };
};
```

##### 抽取状态

```js
const doNothing=()=>({});
function connect(mapStateToProps=doNothing,mapDispatchToProps=doNothing){
    return function(WrappedComponent){
        class HOCComponent extends React.Component{
            constructor(){
                super(...arguments);
                this.onChange=this.onChange.bind(this);
            }
            componentDidMount(){
                this.context.store.subscribe(this.onChange);
            }
            conponentWillUnmont(){
                this.context.store.unsubscribe(this.onChange);
            }
            onChange(){
                this.setState({});
            }
        }

        HOCComponent.contextTypes={
            store:React.PropTypes.object
        }

        return HOCComponent;
    }
}
```

##### 包装组件

```js

const styleHOC=(WrappedComponent,style)=>{
    return class HOCComponent extends React.Component{

        render(){
            return (
                <div style={style}>
                    <WrappedComponent {...this.props} />
                </div>
            )
        }
    }
};

const style={color:'red'};
const NewComponent=styleHOC(DemoComponent,style);

```

#### 继承方式

##### 操作props

```js
const addNewPropsHOC=(WrappedComponent)=>{
    return class NewComponent extends WrappedComponent{
        render(){
            const{user,...otherProps}=this.props;
            this.props = otherProps;
            return super.render();
        }
    };
}

```

`this.props` 直接修改不是一个明智的做法,可能会产生不可预知的结果

通过`React.cloneElement`重新绘制组件

```js
const modifyPropsHOC = (WrappedComponent)=>{
    return class NewComponent extends React.Component{
        render(){
            const elements = super.render();
            const newStyle={
                color:(elements && elements.type === 'div')?'red':'green'
            }
            const newProps={...this.props,style:newStyle};
            return React.cloneElement(elements,newProps,elements.props.children);
        }
    }
}
```

代理方式和继承方式最大的区别是**使用被包裹组件的方式**.

代理方式下`WrappedComponent`经历了一个完成的生命周期,但在继承方式下 `super.render` 只是一个生命周期的一个函数而已;在代理方式下产生的新组件和参数组件是两个不同的组件,一次渲染,两个组件都要经历各自的生命周期,在继承方式下两者合二为一,只有一个生命周期

##### 操作生命周期函数

只在登录的时候显示组件

```js
const onlyForLoggedinHOC=(WrappendComponent)=>{
    retrun class NewComponent extends WrappendComponent{
        render(){
            if  (this.props.loggedIn){
                return super.render();
            }else{
                return null;
            }
        }
    }
}
```

缓存组件

```js
const cacheHOC=(WrappedComponent) => {
    return class NewComponent extends WrappedComponent {
        shouldComponentUpdate(nextProps,nextState){
            return !nextProps.useCache;
        }
    }
}
```

#### 高阶组件的显示名

重新定义`connect`显示名

```js
    function getDisplayName(WrappedComponent){
        return WrappedComponent.displayName ||
        WrappedComponent.name ||
        'Component';
    }

    HOCComponent.displayName=`Connect(${getDisplayName(WrappedComponent)})`
```

## 生命周期

### 组件更新

1. shouldCompoentUpdate
2. componentWillReceiveProps
3. componentWillUpdate
4. render
5. componentDidUpdate

参见
><<深入浅出React和Redux>>