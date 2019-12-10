对于后台的权限控制是非常重要的，每个人拥有不同的权限，每个人做的事情都不一样，这是非常重要的。用户只能看到他该看到的，只能修改他该修改的。这样才能做到安全的控制。

#### 1. 什么是权限控制

对于后台的权限控制，大家用到最多的就是基于角色的权限控制。简单来说就是把给用户一个角色，这个角色拥有什么权限，用户就拥有什么权限。用户和角色是多对一的，也就是一个用户只能有一种角色，一个角色可以被很多用户拥有。

这样就像用户和用户组一样，只要管理好角色的权限然后赋予用户都个角色就可以达到目的。

#### 2. 设计思想

在前端，我觉得对于用户来说应该是所见即所得，也就是能看到的就都是有权限的，不存在明明在那但是不能点的按钮。所以就需要对于路由和按钮都要做控制。

简单来说，根据用户的权限来过滤路由，最后只得到用户有权限的路由，在页面内，也要根据权限来隐藏部分内容（如用户只有读权限没有写权限的话，修改和添加按钮就不能展示）。

#### 3. 前端该怎么做

首先是路由的问题，这里我们把路由分为三个部分：基础路由（直接就有的路由，如登陆）、普遍路由（需要后添加，但是不用权限验证的路由，如首页和个人中心）、权限路由（需要权限验证的路由）。

一开始只有一个登陆的页面，登陆之后拿到用户拥有的权限之后就开始路由的过滤：

```javascript
function getRealRoute(userRoute) {
  let ins = routes;

  for(let j = 0; j < ins.length; j++) {
    let item = ins[j];

    // 有子节点，遍历子节点
    if(item.children && item.children.length > 0) {
      for(let i = 0; i < item.children.length; i++) {
        // 如果子节点没有权限的话，就删掉
        if(!Object.keys(userRoute).includes(item.children[i].name)) {
          ins[j].children.splice(i, 1);
          
          i--;
        }
        else
        {
          // 是否有写权限
          if(userRoute[item.children[i].name].write) {
            ins[j].children[i].meta.write = true;
          }
          else {
            ins[j].children[i].meta.write = false;
          }
        }
      }
      // 如果所有子节点都没权限的话，父节点删掉
      if(item.children && item.children.length < 1) {
        ins.splice(j, 1);
        j--;
      }
    }
    // 没有子节点，也没有权限的话，就删掉
    else if(!Object.keys(userRoute).includes(item.name)) {
      ins.splice(j, 1);
      j--;
    }
  }

  // ins 就是过滤之后，用户有权限的路由

  needToAdd = needToAdd.concat(ins);

  needToAdd[0].children = [].concat(base, ins);

  // 
  store.dispatch('permission/saveRoutes', needToAdd);

  // 使用 router 对象的方法添加路由
  router.addRoutes(needToAdd);

  return [].concat(base, ins);
}
```

这是简单的路由过滤，因为我们规定只能有一层子菜单，所以不需要用到深度遍历。这样我们就拿到了这个用户有权限的路由，然后在改变页面的时候使用 ```store``` 记录对于当前页面的具体权限（读、写）。

```javascript
router.beforeEach((to, from, next) => {
  if(to.name == 'login') {
    next();
  }
  else if(store.state.permission.getToken) {
    if(to.meta) {
      // 记录用户对于当前页面的读写权限
      store.dispatch('permission/setPageWrite', Boolean(to.meta.write));
    }

    if(!store.state.permission.account) {
      store.dispatch('permission/getInfo').then(() => {
        next({ ...to, replace: true })
      })
    }
    else {
      next();
    }
  }
  else {
    next({ path: '/login' })
  }
})
```

然后我们就可以对于当前的页面实现读写级别的控制。
```javascript
computed: {
  haswrite() {
    return this.$store.getters['permission/getWrite'];
  }
}
```

#### 4. 后端该怎么做

后端要做的其实就是两个事情：存储角色的权限、接口权限的验证。

存储角色的权限只是将结果前端修改之后的一串 json 存储下来。权限验证的话我这里通过使用 php ```lumen``` 框架的中间件来实现：

```php
private function checkAuth() {
    // 获取需要权限验证的接口，以及接口需要的权限
    $authList = $this->auth->getInterNeedCheck();

    // 不在验证列表里的就不验证
    if(!isset($authList[$this->nowAction])) {
      return true;
    }

    // 这个接口需要什么权限
    $needPower = $authList[$this->nowAction];

    // 获取用户拥有的权限
    session_start();
    $have = isset($_SESSION['admin']) ? $_SESSION['admin']->route : null;
    session_write_close();

    if(!$have || !is_array($have) || count($have) < 1) {
      return false;
    }

    // 判断权限是否满足
    foreach(array_keys($needPower) as $item) {
      $needSoon = $have[$item];
      $readOrWrite = $needPower[$item];

      if(empty($readOrWrite)) {
        return false;
      }

      // 只要有这个路由权限，就一定有读权限
      if($readOrWrite == 'read') {
        return true;
        break;
      }

      if(isset($needSoon[$readOrWrite]) && $needSoon[$readOrWrite]) {
        return true;
        break;
      }
    }

    return false;
  }
```

#### 5. 怎么管理

前面说了，对于每个角色维护一段 json。如图：
![img](//cdn.521plus.com/blog/auth-1.png)


以上是对于权限控制实现的总结，总的来说做的比较简陋，但是基本实现了对于 ```用户 - 角色 - 权限``` 的控制，也只是贴了部分代码。

后台页面UI组件使用 [Element](https://element.eleme.cn/#/zh-CN) 。挺好用哈哈
