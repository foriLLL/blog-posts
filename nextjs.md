服务端渲染可以解决纯webapp首屏渲染慢，无法seo等问题，并且还可以在渲染的时候携带初始数据，减少数据请求

图片等静态文件只能放在 static 目录下，不能通过 require 来引入，也就是没办法通过 webpack 来进行模块化管理，如果各个组件有自身依赖的图片，也只能一股脑放 static 里，也很难实现版本管理控制浏览器缓存。样式同样也没办法通过 webpack 进行模块化管理，只能通过 style 标签嵌入或直接内联。

pages下存放路由，类似写html

用Link包裹超链接a可实现client端路由（利用Javascript实现）

Furthermore, in a production build of Next.js, whenever Link components appear in the browser’s viewport, Next.js automatically prefetches the code for the linked page in the background. By the time you click the link, the code for the destination page will already be loaded in the background, and the page transition will be near-instant!
 (in production).

 If you need to add attributes like, for example, className, add it to the a tag, not to the Link tag

Head组件代替head,每个页面都可以编辑title  
每个页面如果需要定制，都需要单独写Head？

“CSS-in-JS” library — it lets you write CSS within a React component, and the CSS styles will be scoped (other components won’t be affected).
如 styled-jsx styled-components or emotion.

postcss  https://segmentfault.com/a/1190000003909268$$

The two forms of pre-rendering: Static Generation and Server-side Rendering.

## 预渲染
默认情况下，Next.js 预渲染每个页面。这意味着 Next.js 为每个页面生成 HTML，而不是让它全部由客户端 JavaScript 完成。预渲染可以使项目拥有更好的性能和搜索引擎优化。
### 两种形式的预渲染
* 静态渲染（Static Generation）
* 服务器端渲染（Serverside Rendering, SSR）

Next.js 有两种预呈现形式:  **静态生成** 和 **服务器端渲染** 。二者不同之处在于它**什么时机生成**了每一页的HTML。  

**静态生成** 是在编译环节预生成 HTML 然后在每个请求上**重复使用**的预渲染方法。

**服务器端渲染** 是在每个请求时生成 HTML 的预渲染方法。

> 在开发模式下(当您运行 `npm run dev` 或`yarn dev` 时) ，每个页面都会**在每个请求上预渲染**ー即使对于使用 Static Generation 的页面也是如此。


### 什么时候应该使用静态生成？
建议您尽可能使用 静态生成 （带有或不带数据），因为你的所有 page（页面）都可以只构建一次并托管到 CDN 上，这比让服务器根据每个页面请求来渲染页面快得多。  
“我可以在用户请求之前预先渲染此页面吗？” 如果答案是肯定的，则应选择“静态生成”。  
“静态生成”与 客户端渲染 一起使用
>  重点： 
> * getStaticProps
> * getStaticPaths
> * getServerSideProps




You might have noticed that each markdown file has a metadata section at the top containing title and date. This is called YAML Front Matter, which can be parsed using a library called gray-matter.

To use Server-side Rendering, you need to export getServerSideProps instead of getStaticProps from your page.

不使用预渲染
The team behind Next.js has created a React hook for data fetching called SWR. We highly recommend it if you’re fetching data on the client side. It handles caching, revalidation, focus tracking, refetching on interval, and more. 


When should you use Client-side rendering?    

动态路由：  
getStaticProps, getStaticPaths 可以获取外部数据源

**装饰器就是对类进行处理的函数**
