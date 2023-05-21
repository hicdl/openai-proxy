# openai-proxy
Cloudflare Worker
创建Cloudflare Worker服务->HTTP路由器，快速编辑->配置如下
```
// 是否拒绝所有无 Origin 请求
const ALLOW_NO_ORIGIN = false;
// 允许的Origin列表
const ALLOWED_ORIGIN = [/^https?:\/\/.*\.example\.com$/, /^https?:\/\/test\.com$/];
const validateOrigin = (req) => {
  const origin = req.headers.get('Origin')
  if (origin) {
    for (let i = 0; i < ALLOWED_ORIGIN.length; i++) {
      if (ALLOWED_ORIGIN[i].exec(origin)) {
        return true
      }
    }
  }
  return ALLOW_NO_ORIGIN
}
export default {
  async fetch(request) {
    if (validateOrigin(request)) {
      const requestUrl = new URL(request.url)
      requestUrl.host = "api.openai.com"
      request = new Request(requestUrl, request)
      //如需前端发送API Key，注释掉下一行
      request.headers.set('Authorization', 'Bearer sk-your-token')
      return await fetch(request)
    } else {
      return new Response('[CloudFlare Workers] REQUEST NOT ALLOWED', {status: 403});
    }
  }
}
```
