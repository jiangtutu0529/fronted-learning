##### axios
```
const axiosInstance = axios.create({
  // todo 转换成配置
  baseURL: __ENV__.isProduction
    ? 'https://kfsi.woa.com/web/module/handle'
    : 'https://testkfsi.woa.com/web/module/handle',
})
axiosInstance.interceptors.request.use((config) => {
  const k_token = sessionStorage.getItem('k_token') as string
  const timestamp = Date.now().toString()
  ;(config as AxiosConfig).headers['k-token'] = k_token
  ;(config as AxiosConfig).headers['k-sign'] = getKSign(k_token, timestamp)
  ;(config as AxiosConfig).headers['k-timestamp'] = timestamp
  return config
})
```