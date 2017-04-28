---
layout: post
title: "connect redux复合组件测试 - jest+enzyme"
description: "jest connect redux复合组件测试; jest 连接组件测试"
category: tech
tags: [react, jest]
---
{% include JB/setup %} 

我们的部分组件是connect了redux里的数据的复合型组件，针对这种组件测试有些特殊。

譬如针对一个这样的复合组件：

    import { Component } from 'react';
    import { connect } from 'react-redux';
    import { bindActionCreators } from 'redux';
    import Cascader from 'antd';
    import * as branchSelectAction from './action.js';

    export class BranchCascader extends Component {
      static propTypes = {
        value: React.PropTypes.array.isRequired
        ,onChange: React.PropTypes.func.isRequired
      };

      constructor(props) {
        super(props);
      }

      onChange(value, selectedOptions){
        this.props.onChange(value, selectedOptions)
      }

      render() {
        let branchOptions = {...this.props.branchSelect}
        // dosomething with branchOptions
        return (
          <Cascader options={branchOptions} 
            value={this.props.chosedValue} 
            changeOnSelect 
            onChange={(value, selectedOptions)=>{this.onChange(value, selectedOptions)}} />
        );
    }

    export default connect(
      (state) => {
        return {
          branchSelect :state.branchSelect
        }
      },
      (dispatch) => {
        return {
          actions: bindActionCreators(branchSelectAction, dispatch)
        }
      }
    )(BranchCascader, branchSelectAction);


如果是想要不和装饰件打交道而测试 BranchCascader 组件本身，我们可以只`import BranchCascader`来测试，甚至强行赋值个`props.branchSelect` 和`props.action`。


如果想要测试connect后的复合组件，需要借助redux-mock-store，mock出 store，store可以注入到`Provider`中，所以我们可以这么测试：

    import React from 'react';
    import { render, mount, shallow } from 'enzyme';
    import toJson from 'enzyme-to-json';
    import configureMockStore from 'redux-mock-store';
    import { Provider } from 'react-redux';
    import thunkMiddleware from 'redux-thunk';
    import { initialState as branchSelect} from 'component_path/branch-select/reducer';
    import  ConnectBranchCascader from 'component_path/branch-select/BranchSelect';
    import { actBranch,productBranch } from './mockData/mockBranchData.js'
    import * as branchSelectAction from 'component_path/branch-select/action';
    const trueFn = branchSelectAction.getNodeInfo;
    branchSelectAction.getNodeInfo = jest.fn(trueFn)

    const mockStore = configureMockStore([thunkMiddleware]);
    const store = mockStore({
      branchSelect
    });

    describe('BranchSelect', () => {

      const basicProps = {
        value: []
      }

      function create(props){
        // PickerWrapper
        return (
          <ConnectBranchCascader 
            value={props.value}  
            onChange={props.onChange}
            isAuth = {props.isAuth}
            parentBranch = {props.parentBranch}
            handleBranchNoAuth = {props.handleBranchNoAuth} 
          />
        )
      }

      it('topBar Search type', () => {
        
        const props = Object.assign({},basicProps,{
          onChange: jest.fn()
        })
        const wrapper = mount(
            <Provider store={store}>
              {create(props)}
            </Provider>
          );
        expect(toJson(wrapper)).toMatchSnapshot();

        let actRootBranch = Object.assign({},branchSelect.branches[2])
        wrapper.find('Cascader').last().props().onChange([-3],[actRootBranch]);
        expect(wrapper.find('Connect(BranchCascader)').props().onChange).toBeCalledWith([-3],[actRootBranch])
        expect(branchSelectAction.getNodeInfo).toBeCalledWith(-3,[actRootBranch],undefined)
      })
    })


需要注意一下几点：

- 把需要注入的action和reducer引入，以及connet component
- mock store:

        const mockStore = configureMockStore([thunkMiddleware]);
        const store = mockStore({
            branchSelect
        });
        
- 将store注入到`<Provider>`上，connect component作为它的子组件

        <Provider store={store}>
            <ConnectBranchCascader>
        </Provider>
        
- 引入action时就对action下面会被调用的函数mock一次`jest.fn()`。譬如示例中的`branchSelectAction.getNodeInfo`，断言时直接断言`branchSelectAction.getNodeInfo`的调用情况`expect(branchSelectAction.getNodeInfo).toBeCalledWith(-3,[actRootBranch],undefined)`:，而不是`wrapper.find('BranchCascader').props().actions.getNodeInfo，,因为其被bindActionCreator包装过，返回的是一个新的未mock的函数，不能没mock追踪到，虽然最后mock的函数也被调用

# 参考

[http://www.wsbrunson.com/react/redux/test/2016/05/08/testing-redux-containers.html](http://www.wsbrunson.com/react/redux/test/2016/05/08/testing-redux-containers.html)

[https://github.com/reactjs/redux/issues/588](https://github.com/reactjs/redux/issues/588)
