---
layout:     post

title:      "封装 NavigationService 管理应用的路由动作"

subtitle:   "基于 react-navigation"

date:       2019-11-28 

author:     "Jacob"

tags:

- react-native

---



> 基于 react-navigation 4.x， 以下简称 rn4

使用 react-navigation 管理 APP 路由的话，我们最常用的就是把 navigation 注入当前页面，在跳转过程中使用 `this.props.navigation.navigate() ` ，但是随着应用愈发庞大，这种方式就显得很散乱，而且不易维护，所以，最好在项目架构确定之前就把路由相关的功能处理好。

这篇 blog 主要想记录一下在开发过程中常用的基础路由动作

> getCurrentRouteName 获取当前路由名称

获取当前路由名称这个方法经常在对特殊页面做特殊处理的时候，或者是在应用中引入统计等相关功能的时候使用，代码比较简单：

```typescript
export function getCurrentRouteName(): string {
  if (!navigationContainer) {
    return '';
  }
  const {state} = navigationContainer.props.navigation as any;
  const route = state.routes[state.index];

  return route ? route.routeName : '';
}
```

基于 rn4 的页面，每个页面都被包含在 `navigationContainer` 这个容器中，类似传统的用法，将 navigation 以 props 的形式注入到每个页面，那么这个容器从哪来，还要依赖下面的方法：

```typescript
export function setNavigator(container: NavigationContainerComponent) {
  navigationContainer = container;
}
 
<AppStack
  ref={navigatorRef => {
    setTopLevelNavigator(navigatorRef);
  }}
  navigation={navigation}
/>
```

在调用 `getCurrentRouteName` 方法时，当前活跃的路由栈会通过 `setTopLevelNavigator` 方法设置成最高等级，其实也就是完成了对 `navigationContainer` 的赋值，此时，`navigationContainer.props.navigation` 其实就等于在当前活跃的页面调起 `this.props.navigation` ，不同的是，这种调用方式不局限于页面，所以可以很灵活的在 store 中或是某些 service 中调用。

> navigate/goBack

跳转和返回这类常规动作也可以使用 `navigationContainer` 调用：

```typescript
export function navigate(
  routeName: string,
  params?: NavigationParams,
  key: string = '',
) {
  if (navigationContainer && lastNavigateTime + 500 < Date.now()) {
    navigationContainer.dispatch(
      NavigationActions.navigate({
        routeName,
        params,
        key,
      }),
    );
    lastNavigateTime = Date.now();
  }
}
```

就酱。