---
layout: post
title: "组件生命周期测试 - jest+enzyme"
description: "react组件生命周期测试; componentDidMount异步请求测试；componentWillMount异步代码测试"
category: tech
tags: [react,jest]
---
{% include JB/setup %} 

react的生命周期有：

1. componentWillMount
2. componentDidMount
3. componentWillReceiveProps
4. shouldComponentUpdate
5. componentWillUpdate
6. componentDidUpdate
7. componentWillUnmount

其中1、2、7都会在组件渲染及销毁时自动执行，通过断言`expect(wrapper.state().XXX)`即可测试。当然也可以通过enzyme的`mount()`和`unmount`接口手动调用。

5和6可以通过`update()`强制调用，然后依旧通过断言`expect(wrapper.state().XXX)`和`expect(wrapper.props().XXX`测试。

不过更正常的做法是通过调用`setState()`、`setProps()`来引起变化，触发3、4、5、6，再通过断言`expect(wrapper.state().XXX)`和`expect(wrapper.props().XXX`测试。


## 生命周期中含有异步过程如何测试

这才是个难点。我们经常会在componentDidMount、componentWillMount里进行些异步操作，譬如请求数据。

目前没有很官网的做法，只有比较投机取巧的方式。

首先我们需要mock掉请求，而不是真的去请求。然后创建个可控的异步内容做钩子。

譬如我们有以下这个组件要测试，`componentDidMount`里包含了异步请求：

    export default class View extends React.Component {
        render() {
          <div>{this.state.name}</div>
        }

        componentDidMount() {
          return fetch(`http://www.example.com/page/getData`)
            .then(response => response.json())
            .then(res => this.setState(res))
            .catch(err => console.error('SERVER REQUEST FAILED: ' + err))

        }
    }


我们可以这样测试：

    function flushPromises() {
        return new Promise(resolve => setImmediate(resolve));
    }

    it('...', () => {
        const response = {
            body: {
              name: 'example'
            }
            , status: 200
          }
        fetchMock.get(`http://www.example.com/page/getData`,response);
        const wrapper = mount(<View/>);
        return flushPromises().then(() => {
            expect(wrapper.state().name).toBe(response.body.name)
        });
    });
    
因为我们mock了异步请求，所以并不会真的依据网络情况要等一段时间才会有返回，`fetchMock`转化了请求过程为异步过程，返回的是promise，因为调用时间是`mount()`后的`componentDidMount`，比我们调用`flushPromises`早，`flushPromises`也是返回了个promise，实现的和`fetchMock`一致，所以我们在`flushPromises.then()`的状态下，请求的promise已经是resolve状态，处理函数已经完成，我们就可以顺利断言了。