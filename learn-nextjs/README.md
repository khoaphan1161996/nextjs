1. File-system Routing
   const router = useRouter() : hook

- pathname: Current route
- query: The query string parsed to an object
- basePath: The active basePath (if enabled)
- locale: The active locale (if enabled)
- isFallback: Whether the current page is in fallback mode

  1.1. Multiple parameters

- pages/categories/[categoryId](folder)/posts/[postId].tsx :

* URL: /categories/123/posts/456
  router.query: { categoryId: '123', postId: '456' }
* URL : /categories/frontend/posts/js
  router.query: { categoryId: 'frontend', postId: 'js' }

  1.2. Catch all routes

- pages/posts/[...slug].tsx

* URL : /posts/123
  router.query: {slug: ['123']}
* URL: /posts/easy-nextjs
  router.query: {slug: ['easy-nextjs']}
* URL: /posts/easy/frontend
  router.query: {slug: ['easy','frontend']}

nhưng nó ko match đường dẫn /posts, không gọi file slug

1.3. Optional catch all routes: giống thẳng ở trên nhưng match posts

- pages/posts/[[...slug]].tsx

2. Navigation between pages
   import Link from 'next/link'

2.1 Các thuộc tính của hàm router

- router.push(): Handles client-side transitions
- router.replace(): prevent adding a new URL entry into the history stack
- router.prefetch(): Prefetch pages for faster client-side transitions: down về file trước khi qua trang khác
- router.back(): Navigate back in history
- router.reload(): executes window.location.reload()

  2.2 CSR, SSR, SSG

- SSG Good, but build-time ONLY 😭
- CSR Good for private app
- SSR Latest data as always, performance concerns
- ISR SSG + able to generate HTML on demand

CSR: user gửi request, trả về dữ liệu, render html ở phía client
SSR: user gửi request, trả về html tương ứng với request
SSG: tạo tất cả file html khi build, user gửi request gì thì trả về (chậm khi build, cực nhanh khi load app)

2.3 getStaticProps : dùng khi muốn file html khi sử dụng có sẵn dữ liệu (vd: fetch api và truyền vào props cho file html trước khi sử dụng)
2.4 getStaticPaths : dùng với getStaticProps, dùng khi sử dụng dynamic route

2.5 SSG and Data Fetching on client side : dynamic ssr=false, shallow,

- Dùng dynamic ssr=false trong trường hợp muốn page có SSG tĩnh tạo ra html ko có dữ liệu và chỉ fetch data ở client side ( có những compo chỉ render ở client(có thể do hàm chỉ có ở client - window.reload))
- Trong trường hợp compo đó dùng router.push qua, và ta ko muốn khi gọi hàm sẽ gọi lại getStaticProps ở sv, thì ta truyền shallow = true ở tham số thứ 3

  2.6 server side props với cache ( coi docs )
