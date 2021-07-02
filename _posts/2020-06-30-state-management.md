---
layout:     post

title:      "react hooks 重构状态管理"

subtitle:   "mobx"

date:       2020-06-30

author:     "Jacob"

tags:

- react-native
- react-hooks

---



>  使用 react hooks 重构状态管理部分

- inject

  class 中一般使用 `@inject` 这种装饰器写法，重构使用 react hooks 则可以使用 hook 注入，首先创建 hook

  ```typescript
  import { createContext, useContext } from "react"
  import { RootStore } from "./root-store"
  
  // 创建 context 
  const RootStoreContext = createContext<RootStore>({} as RootStore)
  
  // 创建 Provider 
  export const RootStoreProvider = RootStoreContext.Provider
  
  // hook 
  export const useStores = () => useContext(RootStoreContext)
  
  ```

  使用时

  ```typescript
  // 引入 hook
  import { useStores } from "../../models/root-store"
  
  // 解构 rootStore 中可能存在多个 store
  const { form } = useStores()
  // form.xx
  ```

- observer

  class 方式依赖 mobx-react ，使用 `@observer` ，函数写法依赖 [mobx-react-lite](https://mobx-react.js.org/)[^1]，写法如下

  ```typescript
  export const WelcomeScreen: React.FunctionComponent<WelcomeScreenProps> = observer(props => {})
  ```

- provide

  不管哪种写法，都需要把 store 推给根组件，也就是设置 Provider，之前的写法

  ```typescript
  import { Provider } from 'mobx-react';
  
   <Provider store={store}>
  
  ```

  重构后的写法，不依赖 mobx-react 提供的 Provider ，使用 react 提供的 Context ，注意第一步在创建 Context 的时候，已经创建了 `RootStoreProvider` ，可以直接使用

  ```typescript
  import { RootStore, RootStoreProvider, setupRootStore } from "./models/root-store"
  enableScreens()
  
  const App: React.FunctionComponent<{}> = () => {
    const [rootStore, setRootStore] = useState<RootStore | undefined>(undefined)
    useEffect(() => {
      ;(async () => {
        setupRootStore().then(setRootStore)
      })()
    }, [])
  
    if (!rootStore) {
      return null
    }
    return (
      <RootStoreProvider value={rootStore}>
        <SafeAreaProvider initialSafeAreaInsets={initialWindowSafeAreaInsets}>
          <RootNavigator
            ref={navigationRef}
            initialState={initialNavigationState}
            onStateChange={onNavigationStateChange}
          />
        </SafeAreaProvider>
      </RootStoreProvider>
    )
  }
  
  export default App
  
  ```

上面👆 是状态管理相关的整体重构方案，只体现了 rootStore ，接下来看具体到每个业务 store 以及 业务store 之间的聚合

- 每个 store 中添加方法

  ```typescript
  export const defaults = {}
  export const createFormStoreModel = () => types.optional(FormStoreModel, defaults as any)
  ```

- 添加到 rootStore 中

  ```typescript
  export const RootStoreModel = types.model("RootStore", {
    form: createFormStoreModel(),
  })
  ```



[^1]: Lightweight React bindings for MobX based on React 16.8 and Hooks

