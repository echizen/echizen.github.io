---
layout: post
title: "使用jest+enzyme进行react项目测试 - 测试手法篇"
description: "如何基于jest+enzyme进行react项目测试。react项目测试方案。组件UI测试：snapshot、dom交互测试、action测试、reducer测试。snapshot的使用场景。如何利用事件测试交互行为。redux项目测试。enzyme中render、mount、shallow的区别和使用场景"
category: tech
tags: [测试, react]
---
{% include JB/setup %}


基于jest和enzyme的特性，我们对一个react项目，可以进行以下几类测试。

## react项目测试方案

(由于自己实践在私有项目上不方便贴代码，以下所有代码将来源于对开源库的测试和其他demo代码)

### 组件UI测试：snapshot

snapshot可以测试到组件的渲染结果是否符合预期，预期就是指你上一次录入保存的结果，`toMatchSnapshot`方法会去帮你对比这次将要生成的结构与上次的区别

感谢jest改方法的强大，这是最便捷的测试，每个组件都该覆盖到。

snapshot的测试案例形如调用这个组件，传入依赖的props。譬如对antd的`ToolTip`组件：

        import { Tooltip } from 'antd';
        import { render } from 'enzyme';
        import toJson from 'enzyme-to-json';

        describe('FileUploadInput render', () => {

          it('basic use', () => {
            const wrapper = render(
              <Tooltip title="prompt text">
                <span>Tooltip will show when mouse enter.</span>
              </Tooltip>
            );
            expect(toJson(wrapper)).toMatchSnapshot();
          })
    
          it('use arrowPointAtCenter', () => {
            const wrapper = render(
              <div>
                <Tooltip placement="topLeft" title="Prompt Text">
                  <Button>Align edge / 边缘对齐</Button>
                </Tooltip>
                <Tooltip placement="topLeft" title="Prompt Text" arrowPointAtCenter>
                  <Button>Arrow points to center / 箭头指向中心</Button>
                </Tooltip>
              </div>
            );
            expect(toJson(wrapper)).toMatchSnapshot();
          })

          it('use placement',() => {
            const wrapper = render(<div>
                  <div style={{ marginLeft: 60 }}>
                    <Tooltip placement="topLeft" title={text}>
                      <a href="#">TL</a>
                    </Tooltip>
                    <Tooltip placement="top" title={text}>
                      <a href="#">Top</a>
                    </Tooltip>
                    <Tooltip placement="topRight" title={text}>
                      <a href="#">TR</a>
                    </Tooltip>
                  </div>
                  <div style={{ width: 60, float: 'left' }}>
                    <Tooltip placement="leftTop" title={text}>
                      <a href="#">LT</a>
                    </Tooltip>
                    <Tooltip placement="left" title={text}>
                      <a href="#">Left</a>
                    </Tooltip>
                    <Tooltip placement="leftBottom" title={text}>
                      <a href="#">LB</a>
                    </Tooltip>
                  </div>
                  <div style={{ width: 60, marginLeft: 270 }}>
                    <Tooltip placement="rightTop" title={text}>
                      <a href="#">RT</a>
                    </Tooltip>
                    <Tooltip placement="right" title={text}>
                      <a href="#">Right</a>
                    </Tooltip>
                    <Tooltip placement="rightBottom" title={text}>
                      <a href="#">RB</a>
                    </Tooltip>
                  </div>
                  <div style={{ marginLeft: 60, clear: 'both' }}>
                    <Tooltip placement="bottomLeft" title={text}>
                      <a href="#">BL</a>
                    </Tooltip>
                    <Tooltip placement="bottom" title={text}>
                      <a href="#">Bottom</a>
                    </Tooltip>
                    <Tooltip placement="bottomRight" title={text}>
                      <a href="#">BR</a>
                    </Tooltip>
                  </div>
                </div>)
            expect(toJson(wrapper)).toMatchSnapshot();
          })
        })
        
需要注意的是一个足够健壮的测试应该覆盖到所有的渲染请况，也就是如果你的组件根据props传入的参数渲染结果可能不同，这样的情况必须写多个测试案例都覆盖到。

譬如还有`ToolTip`组件，还有`placement`参数表示箭头方向，`arrowPointAtCenter`参数表示将tooltip的箭头位于指示框中间位置。所以测试案例应该加上它们。

        it('use arrowPointAtCenter', () => {
          const wrapper = render(
            <div>
              <Tooltip placement="topLeft" title="Prompt Text">
                <Button>Align edge / 边缘对齐</Button>
              </Tooltip>
              <Tooltip placement="topLeft" title="Prompt Text" arrowPointAtCenter>
                <Button>Arrow points to center / 箭头指向中心</Button>
              </Tooltip>
            </div>
          );
          expect(toJson(wrapper)).toMatchSnapshot();
        })

        it('use placement',() => {
          const wrapper = render(<div>
                <div style={{ marginLeft: 60 }}>
                  <Tooltip placement="topLeft" title={text}>
                    <a href="#">TL</a>
                  </Tooltip>
                  <Tooltip placement="top" title={text}>
                    <a href="#">Top</a>
                  </Tooltip>
                  <Tooltip placement="topRight" title={text}>
                    <a href="#">TR</a>
                  </Tooltip>
                </div>
                <div style={{ width: 60, float: 'left' }}>
                  <Tooltip placement="leftTop" title={text}>
                    <a href="#">LT</a>
                  </Tooltip>
                  <Tooltip placement="left" title={text}>
                    <a href="#">Left</a>
                  </Tooltip>
                  <Tooltip placement="leftBottom" title={text}>
                    <a href="#">LB</a>
                  </Tooltip>
                </div>
                <div style={{ width: 60, marginLeft: 270 }}>
                  <Tooltip placement="rightTop" title={text}>
                    <a href="#">RT</a>
                  </Tooltip>
                  <Tooltip placement="right" title={text}>
                    <a href="#">Right</a>
                  </Tooltip>
                  <Tooltip placement="rightBottom" title={text}>
                    <a href="#">RB</a>
                  </Tooltip>
                </div>
                <div style={{ marginLeft: 60, clear: 'both' }}>
                  <Tooltip placement="bottomLeft" title={text}>
                    <a href="#">BL</a>
                  </Tooltip>
                  <Tooltip placement="bottom" title={text}>
                    <a href="#">Bottom</a>
                  </Tooltip>
                  <Tooltip placement="bottomRight" title={text}>
                    <a href="#">BR</a>
                  </Tooltip>
                </div>
              </div>)
          expect(toJson(wrapper)).toMatchSnapshot();
        })

### dom交互测试: jest+enzyme

enzyme有3种渲染方式：`render`、`mount`、`shallow`，先了解下区别。

#### `render`、`mount`、`shallow`的区别

render采用的是第三方库`Cheerio`的渲染，渲染结果是普通的html结构，对于snapshot使用`render`比较合适。

shallow和mount对组件的渲染结果不是html的dom树，而是react树，如果你chrome装了react devtool插件，他的渲染结果就是react devtool tab下查看的组件结构，而`render`函数的结果是element tab下查看的结果。

这些只是渲染结果上的差别，更大的差别是shallow和mount的结果是个被封装的`ReactWrapper`，可以进行多种操作，譬如`find()、parents()、children()`等选择器进行元素查找；`state()、props()`进行数据查找，`setState()、setprops()`操作数据；`simulate()`模拟事件触发。

shallow只渲染当前组件，只能能对当前组件做断言；mount会渲染当前组件以及所有子组件，对所有子组件也可以做上述操作。一般交互测试都会关心到子组件，我使用的都是`mount`。但是mount耗时更长，内存啥的也都占用的更多，如果没必要操作和断言子组件，可以使用shallow。

#### 交互测试

主要利用`simulate()`接口模拟事件，实际上simulate是通过触发事件绑定函数，来模拟事件的触发。触发事件后，去判断props上特定函数是否被调用，传参是否正确；组件状态是否发生预料之中的修改；某个dom节点是否存在是否符合期望。

譬如antd的table组件的pagination分页功能的测试：

      import React from 'react';
      import { render, mount } from 'enzyme';
      import { renderToJson } from 'enzyme-to-json';
      import { Table } from 'antd';

      describe('Table.pagination', () => {
        const columns = [{
          title: 'Name',
          dataIndex: 'name',
        }];

        const data = [
          { key: 0, name: 'Jack' },
          { key: 1, name: 'Lucy' },
          { key: 2, name: 'Tom' },
          { key: 3, name: 'Jerry' },
        ];

        const pagination = { pageSize: 2 };

        function createTable(props) {
          return (
            <Table
              columns={columns}
              dataSource={data}
              pagination={pagination}
              {...props}
            />
          );
        }

        function renderedNames(wrapper) {
          return wrapper.find('TableRow').map(row => row.props().record.name);
        }

        it('paginate data', () => {
          const wrapper = mount(createTable());

          expect(renderedNames(wrapper)).toEqual(['Jack', 'Lucy']);
          wrapper.find('Pager').last().simulate('click');
          expect(renderedNames(wrapper)).toEqual(['Tom', 'Jerry']);
        });

通过触发最后一页的click事件达到页面改变，去判断table中渲染的数据是否符合预期。

        it('repaginates when pageSize change', () => {
          const wrapper = mount(createTable());

          wrapper.setProps({ pagination: { pageSize: 1 } });
          expect(renderedNames(wrapper)).toEqual(['Jack']);
        });

这个是直接用`setProps`操作了`pagination`参数，再去判断table中渲染的数据是否符合预期。

        it('fires change event', () => {
          const handleChange = jest.fn();
          const noop = () => {};
          const wrapper = mount(createTable({
            pagination: { ...pagination, onChange: noop, onShowSizeChange: noop },
            onChange: handleChange,
          }));

          wrapper.find('Pager').last().simulate('click');

          expect(handleChange).toBeCalledWith(
            {
              current: 2,
              onChange: noop,
              onShowSizeChange: noop,
              pageSize: 2,
            },
            {},
            {}
          );
        });

这个触发click事件，断言`handleChange`是否以预期参数被调用。

        it('should display pagination as prop pagination changed', () => {
          const wrapper = mount(createTable());
          expect(wrapper.find('.ant-pagination')).toHaveLength(1);
          expect(wrapper.find('.ant-pagination-item')).toHaveLength(2);
          wrapper.setProps({ pagination: false });
          expect(wrapper.find('.ant-pagination')).toHaveLength(0);
          wrapper.setProps({ pagination });
          expect(wrapper.find('.ant-pagination')).toHaveLength(1);
          expect(wrapper.find('.ant-pagination-item')).toHaveLength(2);
          wrapper.find('.ant-pagination-item-2').simulate('click');
          expect(renderedNames(wrapper)).toEqual(['Tom', 'Jerry']);
          wrapper.setProps({ pagination: false });
          expect(wrapper.find('.ant-pagination')).toHaveLength(0);
          wrapper.setProps({ pagination: true });
          expect(wrapper.find('.ant-pagination')).toHaveLength(1);
          expect(wrapper.find('.ant-pagination-item')).toHaveLength(1); // pageSize will be 10
          expect(renderedNames(wrapper)).toEqual(['Jack', 'Lucy', 'Tom', 'Jerry']);
        });
      });

这个先判断渲染结果中的子节点'.ant-pagination'、'.ant-pagination-item'数量是否符合预期，以达到渲染分页情况是否正确的判断，然后再通过`setProps`操作`pagination`参数，判断子节点是否符合预期。

从以上案例也可看出，断言时既可以通过获取props()、state()中的数据是否符合预期，也可以通过dom selector查询找到特定节点再通过`text()`接口拿到数据，判断是否符合预期

### 功能函数测试

功能函数除了普通的工具处理类函数，直接引入函数，传入特定参数调用，判断函数返回值是否符合预期。要注意的也是全面性，保证函数的每个逻辑判断都能在所有测试案例跑完后被覆盖到。

这时候纯函数和函数式编程的优势确实体现出来了，很方便测试。不扯了，我还有很多没有return 返回值的函数需要去改。不适合测试的代码就要去改改再测，还有那种函数功能不清的，一个函数干了一堆事情，这种也得拆拆。

接入了redux的项目，比较特殊的是action、reducer函数的测试。

## action

其实也是调用函数判断返回值是否符合预期。

如下action:

		export function addTodo(text) {
		  return {
		    type: 'ADD_TODO',
		    text
		  }
		}

测试代码：

		import * as actions from '../../actions/TodoActions'
		import * as types from '../../constants/ActionTypes'

		describe('actions', () => {
		  it('should create an action to add a todo', () => {
		    const text = 'Finish docs'
		    const expectedAction = {
		      type: types.ADD_TODO,
		      text
		    }
		    expect(actions.addTodo(text)).toEqual(expectedAction)
		  })
		})
	
### 异步action

jest对异步提供了很方便的3种测试方式：[http://facebook.github.io/jest/docs/asynchronous.html#content](http://facebook.github.io/jest/docs/asynchronous.html#content)

但是异步action还是要借助第三方库`configureMockStore`，将redux-thunk这种异步中间件传入进去处理，获得封装后的store.dispatch来派发action。

		import fetch from 'isomorphic-fetch';

		function fetchTodosRequest() {
		  return {
		    type: FETCH_TODOS_REQUEST
		  }
		}

		function fetchTodosSuccess(body) {
		  return {
		    type: FETCH_TODOS_SUCCESS,
		    body
		  }
		}

		function fetchTodosFailure(ex) {
		  return {
		    type: FETCH_TODOS_FAILURE,
		    ex
		  }
		}

		export function fetchTodos() {
		  return dispatch => {
		    dispatch(fetchTodosRequest())
		    return fetch('http://example.com/todos')
		      .then(res => res.json())
		      .then(json => dispatch(fetchTodosSuccess(json.body)))
		      .catch(ex => dispatch(fetchTodosFailure(ex)))
		  }
		}

测试代码：

		import configureMockStore from 'redux-mock-store'
		import thunk from 'redux-thunk'
		import * as actions from '../../actions/TodoActions'
		import * as types from '../../constants/ActionTypes'
		import nock from 'nock'

		const middlewares = [ thunk ]
		const mockStore = configureMockStore(middlewares)

		describe('async actions', () => {
		  afterEach(() => {
		    nock.cleanAll()
		  })

		  it('creates FETCH_TODOS_SUCCESS when fetching todos has been done', () => {
		    nock('http://example.com/')
		      .get('/todos')
		      .reply(200, { body: { todos: ['do something'] }})

		    const expectedActions = [
		      { type: types.FETCH_TODOS_REQUEST },
		      { type: types.FETCH_TODOS_SUCCESS, body: { todos: ['do something']  } }
		    ]
		    const store = mockStore({ todos: [] })

		    return store.dispatch(actions.fetchTodos())
		      .then(() => { // 异步 actions 的返回
		        expect(store.getActions()).toEqual(expectedActions)
		      })
		  })
		})

注意：

1. 异步的测试案例一定要 `return` 异步promise，return不能丢，这样才能让jest明白这是个异步过程，才会去等待异步执行结果，否则会立即触发expect断言导致测试失败。

2. 对于请求的mock可能需要第三方库，这是redux官网的示例，但是fetch请求并不能用`nock`库来mock，`nock`官方说了他只改写了node里的http模块，[https://github.com/node-nock/nock](https://github.com/node-nock/nock)。并不能mock fetch接口，所以这里使用有错误

## 善用mock

很多对于测试没有价值的东西可以mock掉，这并不会影响测试的准确性。

### mock函数

对于一个react组件，测试这个组件时并不需要关心注入他的action函数的行为，你所要做的是特定场景下，这个action函数被正确调用，置于调用结果是action函数的单元测试该验证的事。

所以你可以定义`const handler = jest.fn()`，后续可以通过`expect(handler).toBeCalled()`检查mock的函数是否如期被调用，`expect(handler).toBeCalledWith('arg')`检查mock的函数是否调用时传入的参数是`arg`等。

### mock文件

诸如css、image等于逻辑测试无关的资源文件可以在`package.json中`的配置里被统一mock掉:



    "jest": {
      "moduleNameMapper": {
        "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/spec/__mocks__/fileMock.js",
        "\\.(css|scss)$": "<rootDir>/spec/__mocks__/styleMock.js",
        "^component_path$": "<rootDir>/src/components",
        "^root_path$": "<rootDir>/src"
      }
    }
    
    这样`import`时就不会真的去引入这些对于测试无用的文件了。

### mock请求

我们并不想在测试时真的去发一个请求，一方面是会耗时很长，另一方面可能会用测试数据污染线上数据库。

所以我们需要mock掉真正的http请求，模拟返回值。不用担心不准确，你只用保证请求时的参数符合期望就好，mock的返回值按预期编写就好，置于这些请求是否真的能返回这些结果，是接口测试改干的活。

手动一个个mock请求实在效率低，所以我们需要一个组件帮我们直接拦截掉所有请求，我们的项目代码不需改变，再传入要mock的请求返回值，组件帮我们封装成请求的返回值返回给我们的测试代码拿到。

我引用的第三方mock请求的组件是`fetch-mock`;

先封装个统一的mock方法：

    const fetchMock = require('fetch-mock');
    import { HOST } from '../src/util/api.js';

    export function mockRequest(path,res){
      let reg = new RegExp(`${HOST}${path}.*`);
      return fetchMock.get( reg, {
        body: {
          status:{
            code: "0",
            detail: "成功",
            msg: "success"
          },
          result: res
        },
        status: 200
      })
    }
    
调用：

    import { mockRequest } from '../mockRequest.js';
    import { multiData } from './mockModuleData.js';

    describe('DataEdit render', () => {
      afterEach(fetchMock.restore)
      it('renders DataEdit correctly', () => {
        mockRequest('/pageModule/getData', multiData);
        return wrapper.node.fetchData(moduleInfo.moduleId).then(()=>{
          expect(toJson(wrapper)).toMatchSnapshot();
        })
      });
    })

## 测试注意事项

可能第一次接触测试，开始时有很多错误的思维，贴出来。

1.拆分单元，关注输入输出，忽略中间过程。dom测试时只用确保正确调用了action函数，传参正确，而不用关注函数调用结果，置于action处理结果，reducer中对state的改变这些都留给action和reducer自己的单元测试区测。不要想着测试整个大功能的流程，不要有闭环的思想，单元测试需要保证的当前单元正常，对于每个单元模块输入输出都正确，理论串联后一起使用闭环时也会正确。

2.多种情况的测试覆盖，如果不能保证测试的全面性，每种情况都覆盖到，那么这个测试就是个不敢依靠的不全面的测试。当然在实际项目中，可能因为时间、资源等问题，无法保证每种情况都测试到，而只测试主要的内容，这时候要做到心里有数，反正我是对于每个测试都写注释的，交代清楚测试覆盖了哪些，还有哪些没有覆盖，需要其他手段保持稳定性。

3.关注该关注的，无关紧要的mock掉。css、图片这种mock掉，http请求mock掉

4.原本不利于测试的代码还是需要修改的，并不能为了原代码稳定不变，在测试时不敢动原代码。譬如函数不纯，没有返回值等。