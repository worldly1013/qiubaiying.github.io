---
layout:     post
title:      elementUI实现顶部与侧边联动导航栏
subtitle:   
date:       2020-03-19
author:     worldly
header-img: img/post-bg-centerpark-4.jpeg
catalog: true
tags:
    - CSS
---

### 前言
单纯的一级、二级菜单很好做，*elementUI* 中也有现成的组件 *<el-menu>*。但对于大型后台管理项目，因模块众多，通常为顶部导航栏、左侧导航栏加右侧内容区的框架布局方式。那如何实现两个导航栏联动了？？

#### 一、框架布局
1、项目整体框架大致由顶部导航栏、左侧导航栏，右侧内容区三大模块组成，通过点击导航栏加载路由，从而展示不同的页面

![](http://local.fenzhitech.com:81/res/c578ce2e787052cf6dacc36bcc7b45f1.png)

2、页面框架布局代码
```
<template>
  <div class="view-page">
    <v-header></v-header>
    <div class="view-content">
      <left-slide></left-slide>
      <main-content>
        <router-view></router-view>
      </main-content>
    </div>
  </div>
</template>
<script type="text/javascript">
  import {mainContent, leftSlide, vHeader} from 'components'

  export default{
    name: 'page',
    components: {
      vHeader,
      mainContent,
      leftSlide
    }
  }
</script>
```

#### 二、两导航栏如何联动

##### 1、路由设计
当点击顶部导航栏时,左侧导航栏肯定会随之变化。一般顶部导航都属于大的模块项，左侧导航分别属于顶部导航的下属子导航。因此我们可以把路由设计为父子关系。左侧的导航路由数据设计为顶部导航的子数据

![](http://local.fenzhitech.com:81/res/50732353c394be06c29c11156a060256.png)

自己随便画了张草图，大概就是上面这个意思。当点击顶部导航 *A* 选项时左侧导航 *A1*、*A2*、*A3*、*A4* 数据出现。这些数据可以设计为 *A* 导航数据的子数据

```
const ARoute = [
  {
      path: '/ACenter',
      type: 'top',
      name: 'ACenter',
      component: home,
      menuShow: true,
      children: [
        {
            path: '/A1',
            name: 'A1',
            meta: {
                auth: true,
                title: 'A1'
            },
            component: A1
        },
        {
          path: '/A2',
          name: 'A2',
          meta: {
              auth: true,
              title: 'A2'
          },
          component: A2,
          redirect: '/A2/A21',
          children: [
            {
              path: '/A2/A21',
              name: 'A21',
              meta: {
                  auth: true,
                  title: 'A21'
              },
              component: A21
            },
            {
              path: '/A2/A22',
              name: 'A22',
              meta: {
                  auth: true,
                  title: 'A22'
              },
              component: A22
            }
          ]
        },
        {
            path: '/A3',
            name: 'A3',
            meta: {
                auth: true,
                title: 'A3'
            },
            component: A3
        },
        {
            path: '/A4',
            name: 'A4',
            meta: {
                auth: true,
                title: 'A4'
            },
            component: A4
        },
      ]
  },
]

export default ARoute
```

配置好 *A* 模块的路由后，我们依次把其余两个模块也配好，然后注册到 *VueRouter* 上。

```
import ARoute from './A'
import BRoute from './B'
import CRoute from './C'

let routes = [
    {
        path: '/404',
        name: 'notPage',
        component: noPageComponent
    },
    {
        path: '*',
        redirect: '/404'
    },
    {
        path: '/user/login',
        name: 'login',
        component: loginComponent
    },
]

routes = routes.concat(ARoute).concat(BRoute).concat(CRoute)

const router = new VueRouter({
    routes,
    mode: 'hash',
    scrollBehavior(to, from, savedPosition) {
        if (savedPosition) {
            return savedPosition
        } else {
            return {
                x: 0,
                y: 0
            }
        }
    }
})
```

##### 2、顶部导航组件

当我们把路由设计好后，没个导航组件在UI层面上就是单一的了。我们直接用 *elementUI* 的 *<el-menu>* 写就好了

```
<template>
  <div class="tabbar-container">
    <el-row>
      <el-col :span="24">
        <el-menu :default-active="defaultActivePath" class="el-menu-demo" mode="horizontal" @select="handleSelect" :router="true">
          <el-menu-item :index="item.path" v-for="(item,index) in menuList" v-if="privileges.indexOf(item.code) > -1">{{item.name}}</el-menu-item>
        </el-menu>
      </el-col>
    </el-row>
  </div>
</template>

<script type="text/javascript">

import { eventBus } from 'common/tools'
import menuListData from '../menuData/index'

export default {
  data(){
    return {
      privileges:localStorage.getItem("privileges"),
      defaultActivePath: '',
      menuList:menuListData,
      defaultRoute:''
    }
  },
  created() {

  },
  mounted() {
    this.initData()
  },
  methods: {
    initData(){
      const privileges = this.privileges , menuList = this.menuList

      for (var i = 0; i < menuList.length; i++) {
        if( privileges.indexOf(menuList[i]['code']) > -1 ){
          this.defaultActivePath = menuList[i]['path']
          break
        }
      }
    }
  },

}
</script>
```

数据层设计如下，考虑到后台管理项目常会有权限涉及，增加了权限字段
```
// name 名称
// code 权限编码
// path 路由路径
export default [
  {'name':'A',code:'lb001',path: '/A'},
  {'name':'B',code:'lc001',path: '/B'},
  {'name':'C',code:'ld001',path: '/C'},
]
```

##### 3、左侧导航组件
左侧导航组件跟顶部一样

```
<template>
  <div class="left-side">
    <div class="left-side-inner">
      <el-menu
        class="menu-box"
        theme="dark"
        router
        :default-active="$route.path"
        :default-openeds="defaultOpenPath"
        >
        <div v-for="(item, index) in nav_menu_data" :key="index" v-if="privileges.indexOf(item.code) != -1">
          <el-submenu :index="item.path[0]">
            <template slot="title" >
              <img :src="item.icon" class="icon-image">
              <span v-text="item.title" class="text"></span>
            </template>
            <el-menu-item
              class="menu-list"
              v-for="(sub_item, sub_index) in item.child"
              :class=" sub_item.path.indexOf($route.path) > -1 ? 'is-active':''"
              :index="sub_item.path[0]"
              :key="sub_index"
              v-if="privileges.indexOf(sub_item.code) != -1"
              >
              <i class="icon fa" :class="sub_item.icon"></i>
              <span v-text="sub_item.title" class="text"></span>
            </el-menu-item>
          </el-submenu>
        </div>
      </el-menu>
    </div>
  </div>
</template>

<script type="text/javascript">
import { AData, BData , CData } from './menuData/index'

export default{
    name: 'slide',
    data(){
      return {
        privileges:localStorage.getItem("privileges"),
        nav_menu_data:memberData,
        defaultOpenPath:[]
      }
    },
    created(){
      this.initLeftMenu()
    },
    methods:{
      initLeftMenu(){
        eventBus.$on('on-home-left-tab-change',(data) => {

          var refreshMenu = () => {
            const name = data['name']
            let allMenuData = JSON.parse(JSON.stringify(this.getMatchData(name)))
            this.defaultOpenPath = []

            let res = []
            allMenuData.forEach(el => {
              // 从上到下依次判断一级权限点、二级权限点
              const oneFlag = this.privileges.indexOf(el.code)
              if( oneFlag == -1) return
              if( oneFlag > -1){
                filterMenuData.push(el)
                if(el.child.length == 0){
                  return
                }

                el.child.forEach(el1 => {
                  if(this.privileges.indexOf(el1.code) > -1){
                    res.push(el1)
                  }
                })
              }
            })

            let defaultRoute = res[0]['path'][0]
            this.nav_menu_data = allMenuData
            this.defaultOpenPath.push(defaultRoute)
          }

          refreshMenu()
        })
      },
      getMatchData(name){
        let res = []

        switch (name) {
          case 'A':
            res = AData
            break;
          case 'B':
            res = BData
            break;
          case 'C':
            res = CData
            break;
          default:
        }

        return res
      },
    }
  }
</script>
```

因左侧导航栏经常会有二级菜单，因此设计了二级菜单数据
```
// code   权限编码
// title  导航名称
// path   路由路径
// icon   导航图标
// child  子菜单
const IOTData = [
  {
    code:'ld001',
    title: "A1",
    path: ['/A1'],
    icon: require('../../../assets/images/icon/icon_leftbar_entrance.png'),
    child: []
  },
  {
    code:'ld002',
    title: "B1",
    path: ['/IOT/accResource'],
    icon: require('../../../assets/images/icon/icon_leftbar_entrance.png'),
    child: [
      {
        title: "B11",
        path: ['/B11'],
        code:'ld101',
      },
      {
        title: "B12",
        path: ['/B12'],
        code:'ld104'
      },
      {
        title: "B13",
        path: ['/B13'],
        code:'ld105'
      },
    ]
  },
]

export default IOTData
```

##### 4、事件交互，传递数据
路由和组件都设计好了，现在就差最后一步了，如何用数据交互将两个组件联系起来了？？这个问题其实就是组件间的通信问题，之前文章中也总结过。在此为了方便，就不用 *vuex* 了，直接用 *eventBus* 来实现，当组件初始化时，我们订阅好顶部导航栏触发事件 *on-home-top-tab-change* 和左侧导航栏的触发事件 *on-home-top-tab-change*，当点击时触发实现数据交流。具体就不多说了

```
initTopMenu(){
  // path 路由路径
  eventBus.$on('on-home-top-tab-change',(path) => {

    var refreshTopMenu = () => {
      this.defaultActivePath = path
      let data = this.menuList.filter(el => el.path == path)[0]

      const name = data['name']
      let nav_menu_data = this.getMatchData(name)
      this.defaultRoute = nav_menu_data[0]['path'][0]

      eventBus.$emit('on-home-left-tab-change',data)
    }

    refreshTopMenu()
  })
},
initLeftMenu(){
  // 导航栏数据
  eventBus.$on('on-home-left-tab-change',(data) => {

    var refreshMenu = () => {
      const name = data['name']
      let allMenuData = JSON.parse(JSON.stringify(this.getMatchData(name)))
      this.defaultOpenPath = []

      let defaultRoute = allMenuData[0]['path'][0]
      this.nav_menu_data = allMenuData
      this.defaultOpenPath.push(defaultRoute)
    }

    refreshMenu()
  })
},

```

基本功能已经完成了。当点击顶部导航栏时，就触发了 *on-home-top-tab-change* 事件，然后将数据传递给 *on-home-left-tab-change* ，从而实现左侧导航栏的刷新


#### 三、加入权限控制
后台权限控制是常见需求。如果要考虑权限，我们过滤掉没有权限的数据即可。我们丰富一下两个事件的功能，增加数据过滤功能
```
getUserPrivileges(){
  return new Promise((resolve,reject) => {
    this.$fetch.api_user.getUserPrivileges().then((res) =>{
      this.privileges = res
      resolve()
    })
  })
},
initTopMenu(){
  eventBus.$on('on-home-top-tab-change',(path) => {

      var refreshTopMenu = () => {
        // 过滤掉没有权限的顶层菜单
        this.menuList = menuListData.filter(el => {
          return this.privileges.indexOf(el.code) > -1
        })

        this.defaultActivePath = path
        let data = this.menuList.filter(el => el.path == path)[0]

        const name = data['name']
        let nav_menu_data = this.getMatchData(name)
        this.defaultOpenPath = []

        let res = []
        nav_menu_data.forEach(el => {
          // 过滤掉没有权限的一二级节点数据
          const oneFlag = this.privileges.indexOf(el.code)
          if( oneFlag == -1) return
          if( oneFlag > -1){
            if(el.child.length == 0) return
            el.child.forEach(el1 => {
              if(this.privileges.indexOf(el1.code) > -1) res.push(el1)
            })
          }
        })

        let defaultRoute = res[0]['path'][0]
        this.defaultRoute = defaultRoute
        eventBus.$emit('on-home-left-tab-change',data)
      }

      this.getUserPrivileges().then(() => {
        refreshTopMenu()
      })
  })
},
initLeftMenu(){
  eventBus.$on('on-home-left-tab-change',(data) => {
      var refreshMenu = () => {
        const name = data['name']
        let allMenuData = JSON.parse(JSON.stringify(this.getMatchData(name)))
        let filterMenuData = []
        this.defaultOpenPath = []

        let res = []
        allMenuData.forEach(el => {
          // 从上到下依次判断一级权限点、二级权限点
          const oneFlag = this.privileges.indexOf(el.code)
          if( oneFlag == -1) return
          if( oneFlag > -1){
            filterMenuData.push(el)
            if(el.child.length == 0){
              return
            }

            let _child = []
            el.child.forEach(el1 => {
              if(this.privileges.indexOf(el1.code) > -1){
                _child.push(el1)
                res.push(el1)
              }
            })

            el['child'] = _child
          }
        })

        let defaultRoute = res[0]['path'][0]
        this.nav_menu_data = filterMenuData
        this.defaultOpenPath.push(defaultRoute)
      }

      this.getUserPrivileges().then(() => {
        refreshMenu()
      })
  })
},
