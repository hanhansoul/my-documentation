# request.go

Request结构体

```go
// Request is the representation of a HTTP request made by a Collector
type Request struct {
    // HTTP请求的URL地址
	URL *url.URL
	// Headers contains the Request's HTTP headers
    // HTTP请求头
	Headers *http.Header
	// Ctx is a context between a Request and a Response
    // 请求和响应的上下文对象
	Ctx *Context
	// Depth is the number of the parents of the request
    // 请求的深度
	Depth int
	// Method is the HTTP method of the request
    // 请求方法
	Method string
	// Body is the request body which is used on POST/PUT requests
    // 请求体
	Body io.Reader
	// ResponseCharacterencoding is the character encoding of the response body.
	// Leave it blank to allow automatic character encoding of the response body.
	// It is empty by default and it can be set in OnRequest callback.
	ResponseCharacterEncoding string
	// ID is the Unique identifier of the request
	ID        uint32
    // 
	collector *Collector
    // OnRequest中调用Abort()方法将abort置为true，终止当前请求
	abort     bool
	baseURL   *url.URL
	// ProxyURL is the proxy address that handles the request
	ProxyURL string
}
```



AbsoluteURL方法：返回URL的绝对路径

```go
// AbsoluteURL returns with the resolved absolute URL of an URL chunk.
// AbsoluteURL returns empty string if the URL chunk is a fragment or
// could not be parsed
func (r *Request) AbsoluteURL(u string) string {
   if strings.HasPrefix(u, "#") {
      return ""
   }
   var base *url.URL
   if r.baseURL != nil {
      base = r.baseURL
   } else {
      base = r.URL
   }
   absURL, err := base.Parse(u)
   if err != nil {
      return ""
   }
   absURL.Fragment = ""
   if absURL.Scheme == "//" {
      absURL.Scheme = r.URL.Scheme
   }
   return absURL.String()
}
```



HTTP请求相关方法

最终依赖于调用collector.scrape方法

```go
// Visit continues Collector's collecting job by creating a
// request and preserves the Context of the previous request.
// Visit also calls the previously provided callbacks
// 发起GET请求
func (r *Request) Visit(URL string) error {
   return r.collector.scrape(r.AbsoluteURL(URL), "GET", r.Depth+1, nil, r.Ctx, nil, true)
}

// HasVisited checks if the provided URL has been visited
// 检查URL是否已经被访问过了
func (r *Request) HasVisited(URL string) (bool, error) {
	return r.collector.HasVisited(URL)
}

// Post continues a collector job by creating a POST request and preserves the Context
// of the previous request.
// Post also calls the previously provided callbacks
// 发起POST请求，同Visit方法
func (r *Request) Post(URL string, requestData map[string]string) error {
	return r.collector.scrape(r.AbsoluteURL(URL), "POST", r.Depth+1, createFormReader(requestData), r.Ctx, nil, true)
}

// PostRaw starts a collector job by creating a POST request with raw binary data.
// PostRaw preserves the Context of the previous request
// and calls the previously provided callbacks
func (r *Request) PostRaw(URL string, requestData []byte) error {
	return r.collector.scrape(r.AbsoluteURL(URL), "POST", r.Depth+1, bytes.NewReader(requestData), r.Ctx, nil, true)
}

// PostMultipart starts a collector job by creating a Multipart POST request
// with raw binary data.  PostMultipart also calls the previously provided.
// callbacks
func (r *Request) PostMultipart(URL string, requestData map[string][]byte) error {
	boundary := randomBoundary()
	hdr := http.Header{}
	hdr.Set("Content-Type", "multipart/form-data; boundary="+boundary)
	hdr.Set("User-Agent", r.collector.UserAgent)
	return r.collector.scrape(r.AbsoluteURL(URL), "POST", r.Depth+1, createMultipartReader(boundary, requestData), r.Ctx, hdr, true)
}

// Retry submits HTTP request again with the same parameters
func (r *Request) Retry() error {
	r.Headers.Del("Cookie")
	return r.collector.scrape(r.URL.String(), r.Method, r.Depth, r.Body, r.Ctx, *r.Headers, false)
}

// Do submits the request
// 根据给定Request发起请求
func (r *Request) Do() error {
	return r.collector.scrape(r.URL.String(), r.Method, r.Depth, r.Body, r.Ctx, *r.Headers, !r.collector.AllowURLRevisit)
}
```



```go
// Marshal serializes the Request
func (r *Request) Marshal() ([]byte, error) {
   ctx := make(map[string]interface{})
   if r.Ctx != nil {
      r.Ctx.ForEach(func(k string, v interface{}) interface{} {
         ctx[k] = v
         return nil
      })
   }
   var err error
   var body []byte
   if r.Body != nil {
      body, err = ioutil.ReadAll(r.Body)
      if err != nil {
         return nil, err
      }
   }
   sr := &serializableRequest{
      URL:    r.URL.String(),
      Method: r.Method,
      Depth:  r.Depth,
      Body:   body,
      ID:     r.ID,
      Ctx:    ctx,
   }
   if r.Headers != nil {
      sr.Headers = *r.Headers
   }
   return json.Marshal(sr)
}
```