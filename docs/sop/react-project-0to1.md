---
title: react 从0到1搭建创建一个项目
date: 2024-01-21 21:23:07
tags: react
categories: 项目
description: 从0到1搭建创建一个react后台管理项目。
---

## 路由

```tsx
// main.tsx
ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>,
)
```

```tsx
import { useRoutes, Link } from 'react-router-dom'
import router from './router/index'

function App() {
  const outlet = useRoutes(router);
  return (
    <>
      <div>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </div>
      {outlet}
    </>
  )
}
```

```tsx
import Home from '@/views/Home'
import About from '@/views/About'
const router = [
    {
        path: '/',
        element: <Home />,
    },
    {
        path: '/about',
        element: <About />,
    }
            
]
export default router;
```

**路由懒加载：**

使用 react 的 `lazy` 模块

```tsx
import {lazy} from 'react';
const About = lazy(() => import('@/views/About'))
```

`Suspense` 组件用于指定加载中时要显示的组件：

```tsx
// app.tsx
<Suspense fallback={<div>Loading...</div>}>
  {outlet}
</Suspense>
```

