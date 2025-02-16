# swift-nextjs

所用技术栈：`Next.js`, `Tailwind CSS`, `TypeScript`, `DaisyUI`

## Link

使用 *anchor* 标签这种导航方式会重新加载重复网页文件的，而不是替换需要的内容：

```tsx
export default function Home() {
  return (
    <main>
      <h1>Hello World</h1>
      <a href='/users'>Users</a>
    </main>
  )
}
```

因此使用next中的`Link`组件：

```tsx
<Link href='/users'>Users</Link>
```

## CSR & SSR

- 用户端可交互，暴露密钥给用户，标准React App的工作方式 

- 服务端资源占用小，搜索引擎bot可以查看页面并建立索引， 在服务器上保留API等敏感数据，失去了交互性

因此现实中默认使用服务端组件（Next中就是），必须使才使用客户端组件。

 比如在一个商城应用中，应该把导航、侧边栏、页码页脚等组件放在服务端，对于需要交互的商品组件，可以只把需要点击的微小组件（如“AddToCart”）发送到客户端，

## Data Fetching
### fetch

1. Client:
- `useState()`,`useEffect()`
- `ReactQuery`

Large bundles, No SEO(Search Engine Optimization), Less secure, **Extra roundtrip to server**

2. Server:

Typescript Magic: 
```tsx
interface User {
  id: number;
  name: string;
}

const UsersPage = async () => {
  const res = await fetch('https://jsonplaceholder.typicode.com/users');
  const users: User[] = await res.json();

  return (
    <>
      <h1>Users</h1>  
      <ul>
        {users.map(users => <li key={users.id}>{users.name}</li>)}
      </ul>
    </>
  )
}
```

### Caching

(-> slower)

Memory  -> File System -> Networks

For this reason, `Next.js` has a built in data cache( in `fetch` ):

```tsx
 const res = await fetch(
    'https://jsonplaceholder.typicode.com/users',
    {cache: 'no-store'});
    // or 
	{next: {revalidate: 10}} // 10s
```

## Rendering

![Screenshot 2024-02-08 at 19 12 32](https://github.com/bjut-swift/swift-ts/assets/101612750/e5e2f823-95dc-4fc7-b8ba-7f3a871b3a23)


对于
```tsx
<p>{new Date().toLocaleTimeString()}</p>
```

```bash
npm run build
```

<img width="462" alt="Screenshot 2024-02-08 at 19 17 47" src="https://github.com/bjut-swift/swift-ts/assets/101612750/133f3b9b-5d58-415f-804c-3b1e0d2725b9">


使用`cache: 'no-store'`后：

<img width="475" alt="Screenshot 2024-02-08 at 19 20 33" src="https://github.com/bjut-swift/swift-ts/assets/101612750/19f2a5c1-669c-4977-8573-4a2d73ae45cc">


## Tailwind CSS
### CSS Module

scoped to a single component / page, preventing clashing and **overwriting**

新建`ProductCard.module.css`，使用：
```tsx
import styles from './ProductCard.module.css'

<div className={styles.card}>      
```

Nextjs 使用 `postcss` 来生成唯一类名。 

### Tailwind

```tsx
<div className='p-5 my-5 bg-sky-400 text-white text-xl hover:bg-sky-600'>
```

好神奇，删除这个组件就是删除了，不需要再去找对应的css文件。

### daisyUI

类似boostrap。

1. Installation

```zsh
npm i -D daisyui@latest
```

2. Then add daisyUI to the `tailwind.config.ts` files:

```ts
module.exports = {
  //...
  plugins: [require("daisyui")],
}
```

#### button

```tsx
button className='btn btn-primary' onClick={() => console.log('clicked')}>Add to cart</button>
```
#### theme

1. `tailwind.config.ts` :

```ts
	plugins: [require("daisyui")],
	daisyui: {
    themes: ["winter"],
  },
```

2. `layout.tsx` :

```tsx
<html lang="en" data-theme="winter">
```

## Routing

- `page.tsx`：可公开访问的页面文件
- `layout.tsx`：定义页面通用布局
- `loading.tsx`：显示加载的UI ^bbaedd
- `route.tsx`：创建API
- `not-found.tsx`：显示常规错误
- `error.tsx`：自定义错误页面

  ### Dynamic Route

动态路由就是带参数的路由：

1. 在`[id]`文件夹的`page.tsx`中：
```tsx
interface Props {
    parmas: { id: number }
}

// 这里是直接解构出参数变量
const UserDetailPage = ({ parmas: {id} }: Props) => {
  return (
    <div>UserDetailPage {id}</div>
  )
}
```

2. 更多参数的路由

<img width="244" alt="Screenshot 2024-02-10 at 16 56 05" src="https://github.com/bjut-swift/swift-ts/assets/101612750/581e37b7-79ee-4cea-809d-7dc2ae24ec78">

```tsx
interface Props {
    params: { id: number; photoId: number }
}

const PhotoPage = ({params: {id, photoId}}: Props) => {
  return (
    <div>PhotoPage {id} {photoId}</div>
  )
}
```

 ### Catch-all Segments
 
文件名：`[[...slug]]` 使所有路径segments的捕获变为可选项，即还可以匹配所在主路径本身：

```tsx
interface Props {
  params: {slug: string[]}
}

const ProductPage = ({ params: { slug }}: Props) => {
  return (
    <div>ProductPage {slug}</div>
  )
}
```

### Accessing Query String Parameter

1. `page.tsx`中将路由字符传入组件中：

```tsx
interface Props {
  searchParams: { sortOrder: string };
}

const UsersPage = async ({ searchParams: { sortOrder }}: Props) => {
  return (
    <>
      <h1>Users</h1>   
      <UserTable sortOrder={ sortOrder }/>
    </>
  )
}
```

2. `UserTable.tsx`中：

```tsx
import { sort } from 'fast-sort';

interface Props {
    sortOrder: string;
}

const UserTable = async ({ sortOrder }: Props) => {
    const res = await fetch(
        'https://jsonplaceholder.typicode.com/users',
        {cache: 'no-store'});
    const users: User[] = await res.json();

    const sortedUsers = sort(users).asc(
        sortOrder == 'email'
            ? user => user.email 
            : user => user.name
    );

  return (...
                <Link href="/users?sortOrder=name">Name</Link>
            </th>
            <th>
                <Link href="/users?sortOrder=email">Email</Link>
           ...
          {sortedUsers.map(users => <tr key={users.id}>
            <td>{users.name}</td>
            <td>{users.email}</td></tr>)}...
}
```

## Layout

使用layout来创建在多个页面中共享的UI

- 在新建的admin文件夹中`layout.tsx`，可以定义这个文件夹中`page.tsx`的布局：

```tsx
import React, { ReactNode } from 'react'

interface Props {
    children: ReactNode;
}

const AdminLayout = ({ children }: Props) => {
  return (
    <div className='flex'>
        <aside className='bg-slate-200 p-5 mr-5'>Admin Sidebar</aside>
        <div>{children}</div>
    </div>
  )
}

export default AdminLayout
```

### define global NavBar

1. app文件夹中`NavBar.tsx`：

```tsx
import Link from 'next/link'
import React from 'react'

const NavBar = () => {
  return (
    <div className='flex bg-slate-200 p-5'>
        <Link href="/" className='mr-5'>Next.js</Link>
        <Link href="/users">Users</Link>
    </div>
  )
}

export default NavBar
```

2. `layout.tsx`：
```tsx
return (
    <html lang="en" data-theme="winter">
        <body className={inter.className}>
          <NavBar /> 
          <main className='p-5 '>
            {children}   
          </main>
        </body>
    </html>
```
### overwrite base layer style

in `global.css`：

```css
@layer base {
  h1 {
    @apply font-bold text-2xl mb-3
  }
}
```

## Navigation

### Link

- 只下载目标页面内容
- 预获取viewport内链接的页面
- 将页面缓存在客户端中

### Porgrammatic Navigation

注意这里，默认的import路径可能不是这个：

```tsx
'use client'
import { useRouter } from 'next/navigation'
import React from 'react'

const NewUserPage = () => {
  const router = useRouter()

  return (
    <button className='btn' onClick={() => router.push('/users')}>Create</button>
  )
}
```

## Show Loading UIs

通过流式传输（streaming），客户端初始接受的html文件后续生命周期中会接受loading后的内容，不影响SEO：

```tsx
<Suspense fallback={<p>Loading...</p>}>
	<UserTable sortOrder={ sortOrder }/>
</Suspense>
```

多页面loading：

1. 在全局`layout.css`中：

```tsx
<Suspense fallback={<p>Loading...</p>}>
	{children}   
</Suspense>
```

2. 通过[loading files](Nextjs.md#^bbaedd)(`loading.tsx`)：

```tsx
const Loading = () => {
  return (
    <span className="loading loading-spinner loading-md"></span>
  )
}
```
例子中是DaisyUI的组件。

## Errors
### Not Found Errors

在想要实现的文件目录下编辑`not-found.tsx`，如：

```tsx
import { notFound } from 'next/navigation'

const UserDetailPage = ({ params: {id} }: Props) => {
  if (id > 10) notFound()
```

### Unexpected Errors

生产环境下，报错的页面可以自定义`error.tsx`：

```tsx
'use client'
import React from 'react'

interface Props {
    error: Error;
    reset: () => void;
}

const ErrorPage = ({ error, reset }: Props) => {
  console.log('error', error)
  return (
    <>
        <div>An Unexpected Error has Occured</div>
        <button className='btn' onClick={() => reset()}>Retry</button>
    </>
  )
}
```

`global-error.tsx`可以捕获全局layout中的错误。

## Building APIs

Route Handler in `app/api/users/route.tsx`：

```tsx
import { NextRequest, NextResponse } from "next/server";

export function GET(request: NextRequest) {
    return NextResponse.json([
        { id: 1, name: 'Mosh'},
        { id: 2, name: 'John'},
    ]);
}
```

- Get a single object

```tsx
interface Props {
    params: { id: number }
}

export function GET(
request: NextRequest, 
{ params }: Props) {}

// or

export function GET(
    request: NextRequest, 
    { params }: { params: {id: number}}) {

}
```

in `api/users/[id]`：

```tsx
import { NextRequest, NextResponse } from "next/server";

export function GET(
    request: NextRequest, 
    { params }: { params: {id: number}}) {
        if ( params.id > 10)
            return NextResponse.json({ error: 'User Not Found!'}, { status: 404 })

        return NextResponse.json({ id: 1, name: 'mosh'})
}
```

- Creating Object

```tsx
import { NextRequest, NextResponse } from "next/server";

export function GET(request: NextRequest) {
    return NextResponse.json([
        { id: 1, name: 'Mosh'},
        { id: 2, name: 'John'},
    ]);
}

export async function POST(request: NextRequest) {
    const body = await request.json();
    // if invalid, return 400
    if (!body.name)
        return NextResponse.json({ error: 'Name is required'}, { status: 400 })
    return NextResponse.json({ id: 1, name: body.name }, { status: 201});
}
```

<img width="645" alt="Screenshot 2024-02-11 at 08 15 29" src="https://github.com/bjut-swift/swift-ts/assets/101612750/d678fdc5-86f4-4540-ac57-c696f756352c">



- Update an Object

```tsx
// PUT for replacing object, PATCH for updating 1 or more properties
export async function PUT(
    request: NextRequest, 
    {params}: {params: {id: number}}) {
    // Validate the request body
    const body = await request.json();
    if (!body.name)
        return NextResponse.json({ error: 'Name is required'}, { status: 400 });

    if (params.id > 10)
        return NextResponse.json({ error: 'User Not Found!'}, { status: 404 });

    return NextResponse.json({ id: 1, name: body.name });
}
```

-  Delete an Object

```tsx
export function DELETE(
    request: NextRequest, 
    {params}: {params: {id: number}}
) {
    if (params.id > 10)
        return NextResponse.json({ error: 'User Not Found!'}, { status: 404 });

    return NextResponse.json({}) ;
}
```

### Validating Request with Zod

对于复杂的 object，if-else 显然不再方便使用，最好使用 validation library，如[zod](https://zod.dev/)

`api/users/schema.ts`：

```tsx
import { z } from 'zod'

const schema = z.object({
    name: z.string().min(3),
    // email: z.string().email(),
    // age: z.number()
})

export default schema
```

`api/users/[id]/route.tsx`：

```tsx
export async function POST(request: NextRequest) {
    const body = await request.json();
    const validation = schema.safeParse(body);
    // if invalid, return 400
    if (!validation.success)
        return NextResponse.json(validation.error.errors, { status: 400 })
    return NextResponse.json({ id: 1, name: body.name }, { status: 201});
}
```
# 推荐学习项目

- [https://github.com/leerob/nextzy](https://github.com/leerob/nextzy)
