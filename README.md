# Config  khi sử dụng:

## Installation

npm i directus-migration-tools

## B1. Vào file ./config/config.ts để sửa lại các thuộc tính
```
    collection (string) : route truy cập chính vào controller
    route (Array<string> | string) : path truy cập vào các route trong controller
    path_render?: string : đường dẫn tới file template cần render -> response trả về mã html ; khi nhận giá trị rỗng -> response sẽ trả về json
```
    
