## 全局数据流

@参数：

```tsx
  isCollapsed: false,
  currentRobot: {
    id: '',
    name: '',
    robotId: '',
    tenantId: ''
  },
  routePath: { isWelcome: false, path: '' }, // 当前访问的路径

  setPath: () => undefined, // 设定路径
  setRobot: () => undefined, // 设定当前机器人
  toggleCollapse: () => undefined // 设定是否缩放了
```

@全局数据流传递方式：

React.createContext + useContext(context)



@切换机器人，刷新页面时，缓存当前robotId：

```tsx
// 缓存当前机器人的配置，使得刷新页面时候能够记住之前选择的机器人
export const cache = {
  get robot() {
    const rb = sessionStorage && sessionStorage.getItem('CURRENT_ROBOT')

    return rb
      ? JSON.parse(rb)
      : {
          id: '',
          name: '',
          robotId: '',
          tenantId: ''
        }
  },
  set robot(rb: RobotIf) {
    sessionStorage &&
      sessionStorage.setItem('CURRENT_ROBOT', JSON.stringify(rb))
  }
}
```

`sessionStorage`存储方式

