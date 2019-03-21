### 一、项目说明

![enter image description here](http://images.gitbook.cn/25f4d520-572c-11e8-87ee-2304b79e0e0a)

这是环境目录配置：

1. 首选创建一个项目文件，然后 npm init 一下，配置好说明信息。
2. 再次创建 index.html 和 index.js 作为主入口文件，然后创建 src 文件为需要的开发主要文件。
3. 再次创建 webpack.config.js 为本地开发的环境配置，webpack.production.config.js 为生产环境配置。
4. 在 src 下创建一个 container 作为页面文件的仓库，contents 为组件仓库，img、style 为样式资源仓库。
5. 创建 fetch 请求。
6. 创建 redux 相关参考上图。

### 二、路由配置

在 router 文件夹下新建一个 routerMap.jsx：

```
    import React from 'react'
    import { Router, Redirect,Route, IndexRoute,browserHistory,hashHistory } from 'react-router'

    import App from '../containers/App'
    import Home from '../containers/Home'
    import Phone from '../containers/phone/phone'
    import Ditu from '../containers/Ditu/Ditu'
    import Anli from '../containers/Anli/Anli'
    import HomeIndex from '../containers/homeIndex/HomeIndex'
    import List from '../containers/List'
    import Detail from '../containers/Detail'
    import About from '../containers/about/About'
    import News from '../containers/news/News'
    import Team from '../containers/Team/Team'
    import NotFound from '../containers/NotFound'

    class RouteMap extends React.Component {
        render() {
            return (
                 <Router history={hashHistory}>
                    <Redirect from="/" to="/HomeIndex" />
                    <Route path='/' component={App}>
                        <Route path='/HomeIndex' component={Home}>
                            <IndexRoute component={HomeIndex}/>
                            <Route path='/Home/About' component={About}/>
                            <Route path='/Home/News' component={News}/>
                            <Route path='/Home/Team' component={Team}/>
                        </Route>
                        <Route path='/list' component={List}/>
                        <Route path='/Phone' component={Phone}>
                        </Route>
                        <Route path='/Ditu' component={Ditu}/>
                        <Route path='/Anli' component={Anli}/>
                        <Route path='/detail/:id' component={Detail}/>
                        <Route path="*" component={NotFound}/>
                    </Route>
                </Router>
            )
        }
    }

export default RouteMap

```

首先需要引入 react ，然后还需要 Router、Redirect、Route、IndexRoute、browserHistory、hashHistory 这些。

`import ** form **` 就是引入我们的开发页面。这里使用 hashHistory 而不使用 browserHistory，后者刷新页面你会发现什么东西都没有了，需要后端的配合。前者是带有后缀的哈希记录。

Redirect 是页面的重定向，比如 www.aaaa.com/ 会定向到 www.aaaa.com/HomeIndex，to 自定义。

项目图：![enter image description here](http://images.gitbook.cn/d91eda20-5753-11e8-87ee-2304b79e0e0a)

```
    <Route path='/' component={App}></Router>  //包裹所有的路由文件
    <Route path='/HomeIndex' component={Home}> //HomeIndex作为父路由（里面的都是嵌套的子路由）
        <IndexRoute component={HomeIndex}/>   //这是首页（目前展示的页面）
        <Route path='/Home/About' component={About}/>  （页面中间位置的关于我们）
        <Route path='/Home/News' component={News}/>（页面中间位置的家装新闻）
        <Route path='/Home/Team' component={Team}/>（页面中间位置的案例展示）
    </Route>
    这样配置因为跳转这些子路由时候需要首页的位置高亮显示（活跃状态）。
    <Route path='/list' component={List}/>  //列表
    <Route path='/Phone' component={Phone}></Route>  //电话 
    <Route path='/Ditu' component={Ditu}/>    //地图
    <Route path='/Anli' component={Anli}/>    //案例
    <Route path='/detail/:id' component={Detail}/>  //列表详情
    <Route path="*" component={NotFound}/>     //404
    这些页面是和Home平级的。

```

### 三、列表页面携带参数进入详情页面

containers 下面的 List 文件夹新建一个 index.jsx：

```
    import React from 'react'
    import { hashHistory,browserHistory } from 'react-router'

    class List extends React.Component {
        render() {
            const arr = [1, 2, 3] **/*定义数据（一般这里是从服务端获取，这里只做模拟）*/**
            return (
                <ul>
                    {arr.map((item, index) => {
                        return <li key={index} style={{paddingTop:'2rem'}} onClick={this.clickHandler.bind(this, item)}>列表{item}，点我进详情</li>   /*这里是一个列表*/
                    })}
                </ul>
            )
        }
        clickHandler(value) {   **/*定义的点击进入详情时事件，携带value参数*/**
            hashHistory.push('/detail/' + value)
        }
    }

    export default List
containers下面的Detail文件夹新建一个index.jsx

    import React from 'react'

    class Detail extends React.Component {
        render() {
            return (
                <p>Detail，url参数：{this.props.params.id}</p>
            )
        }
    }
    //this.props.params.id  来接受参数 ，这个id正是配置路由的时候的path='/detail/:id'定义的
    export default Detail

```

### 四、Fetch 请求配置

src 下面的 fetch 文件，新建一个 get.js：

```
    import 'whatwg-fetch'
    import 'es6-promise'

    export function get(url) {
        // var result = fetch('http://www.mockhttp.cn'+url, { //打包apk时候使用
        var result = fetch(''+url, {
            credentails: 'include',
            mode: "cors",
            headers: {
                'Accept': 'application/json, text/plain, */*',
                'Content-Type': 'application/x-www-form-urlencoded'
            }
        });
        return result;
    }

```

src 下面的 fetch 文件，新建一个 post.js：

```
import 'whatwg-fetch'
import 'es6-promise'

//将json对象拼接成 key=val&key=val 的字符串形式

    //转化为对象格式 参数
    function obj2params(obj) {
        var result = '';
        var item;
        for(item in obj){
            result += '&' + item + '=' +encodeURIComponent(obj[item]);
        }

        if(result) {
            result = result.slice(1);
        }
        return result;
    }

    //发送 post 请求
    export function post(url, paramsObj) {
        var result = fetch(url, {
            method: 'POST',
            credentials: 'include',
            headers: {
                'Accept': 'application/json, text/plain, */*',
                'Content-Type': 'application/x-www/form-urlencoded'
            },
            body: obj2params(paramsObj)
        });

        return result;
    }

```

然后再新建一个 config.js 文件，用来管理所有的 api 请求：

```
    import { get } from '../get'
    import { post } from '../post'
    //首次进入注册
    export function postRegist(benjihao) {
        const result = get('http://qqqq.aaaaa.com:7000/data/?t=cfg&uptime='+Date.parse(new Date())+'&imei='+benjihao)
        return result
    }

    //定时查询报警
    export function    searchWarn(param,uptimes){
        const result = get('http://1111.aaaaa.com:7000/data/?t=warn&imei='+param+'&ss='+Date.parse(new Date())+'&uptime='+uptimes)
        return result
    }

    //Gpio
    export function    sendGpio(param){
        const result = get('http://aaaa.wwwwww.com:7000/data/?t=cmd&ss='+Date.parse(new Date())+'&'+param)
        return result
    }

```

使用，比如在 homeIndex 中使用请求：

```
    //需要哪些接口就引入进来，注意后面是相对路径
    import { postRegist, searchWarn, sendGpio } from '../../fetch/index/index.js'
    //定义一个函数来触发请求
    sengpiodata() {
        var that = this;
        var param = {};//自定义传递的参数，没有则不传
        const result = sendGpio(param)  //获取上方引入的接口
        //发起请求
        result.then(res => {
            return res.json()
        }).then(json => {
            const data = json;  //获取返回的数据
            Toast.success('Send success !', 1.5);
        }).catch(ex => {
            // 发生错误
            // Toast.fail('获取数据出错',1)
        })
    }

```

### 五、Redux 状态管理

使用 Redux 的场景（个人理解） ：

1. 组件通信（个人认为这才是使用 redux 的最大威力）
2. 初始数据渲染（比如一些个人信息，全局使用，不需要改变的数据，比如页面语言控制）
3. 非持久化的数据状态处理
4. 持久化的数据（修改时候本地存储，其他地方调取本地数据，避免多次请求，不做本地存储刷新页面 state 会被重置，因为它是一个内存。）

src 下创建一个 store–>confiStore.js，store 是保存数据的地方，整个应用只能有一个 store

```
    import { createStore } from 'redux'
    import rooterReducer from '../reducers/index.js'

    export default function configStore(initialState) {
        const store = createStore(rooterReducer, initialState, 
            //触发redux-devtools
            window.devToolsExtension ? window.devToolsExtension() : undefined
        )
        return store
    }

```

store 对象包含所有的数据，如果想要得到某个时间点的数据，就要对 store 生成快照，这种时点的集合就叫做 state，一个 state 对应一个 view。

src 下创建一个 reducers–>index.js：

```
     import { combineReducers } from 'redux'

    import userinfo from './userinfo'

    const rootReducer = combineReducers({
        userinfo
    })

    export default rootReducer

```

src 下创建一个 reducers-->userinfor.js：

```
    import * as actionTypes from '../contents/reduxUser'

    const initialState = {}

    export default function userinfo(state = initialState, action) {
        switch (action.type) {
            //登录
            case actionTypes.USERINFO_LOGIN:
                return action.data

            //修改城市
            case actionTypes.UPDATE_CITYNAME:
                return action.data

            default:
                return state
        }
    }

```

src 下 contents 一个 reduxUser.js.js：

```
    export const USERINFO_LOGIN = 'USERINFO_LOGIN'
    export const UPDATE_CITYNAME = 'UPDATE_CITYNAME'
src下创建一个actions-->userinfor.js:

    import * as actionTypes from '../contents/reduxUser'
    //设置默认值
    const defaultData = {
                userid: 'abc1111',
                city: 'beijing'
            }
    export function login(datas) {
        let datass = datas == undefined ? defaultData : datas;
        return {
            type:actionTypes.USERINFO_LOGIN,
            data:datass
        }
    }

    export function updateCityName(data) {
        return {
            type: actionTypes.UPDATE_CITYNAME,
            data
        }
    }

```

使用 phone.jsx：

```
    import React from 'react'
    import { connect } from 'react-redux'
    import { bindActionCreators } from 'redux'
    import * as userinfoActions from '../../actions/userinfo'

    class Phone extends React.Component {
        render() {
            return (
                <div className="content">
                    <p>{this.props.userinfo.userid}</p>
                    <p>{this.props.userinfo.city}</p>
                    <p onClick={this.changeUserInfo.bind(this)}>修改城市</p>
                </div>
            )
        }

        componentDidMount() {
            // 模拟登陆获取初始的action数据  this.props.userinfo.city
            this.props.userinfoActions.login()

        }

        changeUserInfo() {
            const actions = this.props.userinfoActions
            actions.login({
                userid: '123',
                city: 'nanjing'
            })
        }
    }
    //声明当前组件有关系的state的属性
    function mapStateToProps(state) {
        return {
            userinfo: state.userinfo
        }
    }
    //mapDispatchToProps就是把action存到了一个名为userInfoActions的对象中了。告诉action它想要做什么事情，action就找到对应的函数去执行。如果说需要修改redux数据则需要通过action来完成（changeUserInfo函数就是修改）
    function mapDispatchToProps(dispatch) {
        return {
            userinfoActions: bindActionCreators(userinfoActions, dispatch)
        }
    }

    export default connect(
        mapStateToProps,
        mapDispatchToProps
    )(Phone)

```

工作流程：通过 store 提供的 dispatch 函数将 action（上面的 componentDidMount 初始，和修改就是触发了 action）发送到 reducer，reducer 通过一个 switch 函数处理数据（combineReducers 处理合并，内部完成），reducer 将数据提供给 store，更新 state，组件自动刷新，我的理解这是一个闭环的操作。

### 六、Webpack 代理设置

本地开发时候有时候会跨域，在 Webpack 中可以这样配置：

```
    devServer: {
            historyApiFallback: true,
              hot: true,
            inline: true,
            stats: { colors: true },
            proxy: {
                '/': {
                  target: 'http://www.mockhttp.cn',
                  pathRewrite: {'^/column' : '/column'},
                  changeOrigin: true
                }
             }
        }

```

### 七、蚂蚁金服 Ant 框架使用

> install antd --save下载
>
> 使用：import { 组件名称 } from 'antd'（比如import { Switch } from 'antd'）
>
> 还要引用其样式：import 'antd/dist/antd.css'

但是这样直接引用样式会有一些不必要的组件也被引入进来，需要配置 webpack 进行按需加载。

安装babel-plugin-import 实现按需加载：

```
npm install babel-plugin-import --save-dev

```

修改 webpack 配置：

```
    { 

                    test: /\.(js|jsx)$/, 

                    exclude: /node_modules/, 

                    loader: "babel-loader", 

                    query:

                      {

                        presets:['es2015','react'],

                        plugins: [

                            ["import", {libraryName: "antd", style: "css"}] //按需加载

                        ]

                      },

                },

```

使用组件：

```
<Switch defaultChecked={false} />

```

### 八、调用手机摄像头功能、音视频等功能

这个使用的前提是我们需要 hbuilder 的打包，api 是 h5+。

```
    //调取手机摄像头
            var i=1,gentry=null,w=null;
            var hl=null,le=null,de=null,ie=null;
            var unv=true;
            // H5 plus事件处理
            function plusReady(){
                // 获取摄像头目录对象
                plus.io.resolveLocalFileSystemURL( "_doc/", function ( entry ) {
                    entry.getDirectory( "camera", {create:true}, function ( dir ) {
                        gentry = dir;
                        updateHistory();
                    }, function ( e ) {

                    } );
                }, function ( e ) {

                } );
            }
            if(window.plus){
                plusReady();
            }else{
                document.addEventListener("plusready",plusReady,false);
            }
            // 拍照
            function getImage() {
                var cmr = plus.camera.getCamera();
                cmr.captureImage( function ( p ) {
                    plus.io.resolveLocalFileSystemURL( p, function ( entry ) {
                        createItem( entry );
                    }, function ( e ) {

                    } );
                }, function ( e ) {

                }, {filename:"_doc/camera/",index:1} );
            }
            //点击拍照
            var camera = this.refs.city;
            camera.onclick = function(){getImage()}

```

H5+ 提供的原生 API 很多，能够适应大部分的开发需求，这里不作深入介绍了。这个目的就是说明，我们使用 React 开发完 Web APP，反手 HB 打包一下，就成了一款 APP。

### 九、父子组件通信

父组件通过 props 向子组件传递需要的信息：

```
    import React,{ Component } from 'react'
    //父
    class Father extends Component {
        render (){
            let data = '父组件的数据'
            return <div>
                <Children fatherDataToChild={ data } />
            </div>
        }
    }
    //子
    class Children extends Component {
        constructor(props) {
            super(props);
            this.state = {
              getdata: this.props.fatherDataToChild
            }
          }
        render (){
            const that = this
            return <div>
                <h1>{ that.state.getdata}</h1>
            </div>
        }
    }

```

子组件像父组件传递：父组件向子组件通信时 props 可以传任何类型，包括函数的特性，然后使用回调把值传给父组件。



```
    import React,{ Component } from 'react'

    class Father extends Component {
        constructor (props){
            super(props)
            this.state = {
                data: ''
            }
        }
        fatherHandleClick(data){
            this.setState({
                data:data
            })
        }
        render (){

            return <div>
                <Children fatherHandleClick={ this.fatherHandleClick.bind(this) } />
                <h1>{this.state.data}</h1>
            </div>
        }
    }
    class Children extends Component {
        constructor(prpos,context) {
        super(prpos,context);
        // this.shouldComponentUpdate = PureRenderMixin.shouldComponentUpdate.bind(this);
        this.state = {
            index: 0
        }
    }
        render (){
            return <div onClick={ ()=>{
                this.props.fatherHandleClick('传给父组件')
            } }>
                <h1>点我传值</h1>
            </div>
        }
    }
```

git地址

> https://github.com/wineSu/myReact