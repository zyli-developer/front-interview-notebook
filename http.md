### Get和post请求的区别
  1. Get请求的参数包裹在url中，post请求通过request body 传递参数
  2. Get请求在浏览器回退中是无害的，而post请求会再次发起请求
  3. Get产生的url会被bookmark，而post不会
  4. Get请求浏览器会主动cache，而post请求需要手动设置。
  5. Get请求只能进行url编码，而post请求支持多种编码
  6. Get请求的参数会被完整的保留到历史记录中，而post请求不会
  7. Get请求url中参数的长度是限制，而post没有
  8. 对于参数类型Get请求只接受ASCII字符，而post没有限制。
  9. Get比post请求不安全，因为参数直接报漏在url上，所以不能带有敏感信息
