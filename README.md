- Nextjs là React-Framework
- Pre-rendering: là quá trình chuyển đổi từ react-component sang file html trước khi html được trình chiếu:

* SSR: react-component => file html, file html chỉ dc tạo khi browser gửi request
* SSG: react-component => file html, file html dc tạo sẵn khi dc request sẽ trình chiếu lên

- page
- layout
- Link
- seo
- title header

- getstaticprops : hàm dùng để fetch data, giá trị return của hàm sẽ truyền data qua props vào react-component
- getstaticpath : hàm dùng để truyền đường dẫn path (cho nextjs biết tất cả path có trong app), giá trị return của hàm sẽ truyền path qua props vào react-component
