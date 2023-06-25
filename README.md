0. Khái niệm - Nextjs là React-Framework

- Pre-rendering: là quá trình chuyển đổi từ react-component sang file html trước khi html được trình chiếu:

* SSR: react-component => file html, file html chỉ dc tạo khi browser gửi request
* SSG: react-component => file html, file html dc tạo sẵn khi dc request sẽ trình chiếu lên

- getstaticprops : hàm dùng để fetch data, giá trị return của hàm sẽ truyền data qua props vào react-component
- getstaticpath : hàm dùng để truyền đường dẫn path (cho nextjs biết tất cả path có trong app), giá trị return của hàm sẽ truyền path qua props vào react-component

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

3. Proxy request
   3.1 http-server:
   yarn add http-proxy
   yarn add --dev @types/http-proxy

3.2 setup
trong file pages/api/[...path].ts

import type { NextApiRequest, NextApiResponse } from 'next'
import httpProxy from 'http-proxy'

// gửi thẳng body lên server (dùng khi post, put, ...)
export const config = {
api: {
bodyParser: false
}
}

const proxy = httpProxy.createProxyServer()

export default function handler(
req: NextApiRequest,
res: NextApiResponse
) {
// don't send cookies to API server
req.headers.cookie = ''

proxy.web(req,res , {
target: 'https://js-post-api.herokuapp.com',
changeOrigin: true,
selfHandleResponse: false // gửi thẳng lên server ko xử lý ở proxy next
})
}

3.3 login, logout
cài thêm yarn add cookies, yarn add --dev @types/cookies

login.ts
export default function handler(
req: NextApiRequest,
res: NextApiResponse
) {
if(req.method !== 'POST') return res.status(404).json({message: 'method not supported'})

req.headers.cookie = ''

return new Promise(resolve => {
req.headers.cookie = ''

    const handleLoginResponse = (proxyRes, req, res) => {
        let body = ''
        proxyRes.on('data', chunk => {
            body += chunk
        })

         proxyRes.end('data', () => {
            try {
                const {accessToken, expiredAt} = JSON.parse(body)
                const cookies = new Cookie(req, res, {secure: process.env.NODE_ENV !== 'production'})

                cookies.set('access_token', accessToken, {
                    httpOnly: true,
                    sameSite: 'lax',
                    expires: new Date(expiredAt)
                })

                res.status(200).json({message: 'login successfully'})

            } catch(err) {
                res.status(500).json({message: 'sth went wrong'})
            }
        })
    }

})
}

file khác.ts
thêm : const cookies = new Cookie()
if(cookies.get('access_token')) {
req.headers.Authorization = 'Bearer ' + cookies.get('access_token')
}

logout.js
const cookies = new Cookie()
cookies.set('access_token')
res.status(200).json({message: 'Logout successfully'})

3.4 useSWR: xài giống useQuery, chuyên dùng cho khi tương tác phía server( fetch api và lưu )
yarn add swr
const {data, error, mutate, isValidating} = useSWR(`/students/${studentId}`, {
revalidateOnFocus: false ,( khi true sẽ gọi lại api khi qua tab khác và quay lại )
dedupingInterval: 5000 ,( sẽ gọi lai api sau thời gian 5s sau khi gọi api lần đầu (stale))
revalidateOnMount : true (mặc định true, sẽ gọi lần đầu khi vào app)
})

mutate({name : 'khoa', true}) : call api post , lấy dữ liệu ở phía FE bỏ vào trước, sau khi call xong (vì xét true) nên tiến hành bỏ dữ liệu vào useSWR
