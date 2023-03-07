## 在vite中处理css
vite天生就支持对cSS文件的直接处理
1. vite在读取到main.js中引用到了Index.css
2. 直接去使用十s模块去读取index.css中文件内容
3. 直接创建一个style标签，将index.css中文件内容直接copy进style标签里
4. 按style标签插入到index.html的head中
5. 将该CsS文件中的内容直接替换为js即本(方便热更新或者css樉块化)
