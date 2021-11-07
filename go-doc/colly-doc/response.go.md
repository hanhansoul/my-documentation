# response.go

```go
// Response is the representation of a HTTP response made by a Collector
type Response struct {
   // StatusCode is the status code of the Response
   // 响应状态码
   StatusCode int
   // Body is the content of the Response
   Body []byte
   // Ctx is a context between a Request and a Response
   Ctx *Context
   // Request is the Request object of the response
   Request *Request
   // Headers contains the Response's HTTP headers
   Headers *http.Header
   // Trace contains the HTTPTrace for the request. Will only be set by the
   // collector if Collector.TraceHTTP is set to true.
   Trace *HTTPTrace
}
```



```go
// Save writes response body to disk
func (r *Response) Save(fileName string) error {
   return ioutil.WriteFile(fileName, r.Body, 0644)
}

// FileName returns the sanitized file name parsed from "Content-Disposition"
// header or from URL
func (r *Response) FileName() string {
	_, params, err := mime.ParseMediaType(r.Headers.Get("Content-Disposition"))
	if fName, ok := params["filename"]; ok && err == nil {
		return SanitizeFileName(fName)
	}
	if r.Request.URL.RawQuery != "" {
		return SanitizeFileName(fmt.Sprintf("%s_%s", r.Request.URL.Path, r.Request.URL.RawQuery))
	}
	return SanitizeFileName(strings.TrimPrefix(r.Request.URL.Path, "/"))
}

```



```go
func (r *Response) fixCharset(detectCharset bool, defaultEncoding string) error {
   if len(r.Body) == 0 {
      return nil
   }
   if defaultEncoding != "" {
      tmpBody, err := encodeBytes(r.Body, "text/plain; charset="+defaultEncoding)
      if err != nil {
         return err
      }
      r.Body = tmpBody
      return nil
   }
   contentType := strings.ToLower(r.Headers.Get("Content-Type"))

   if strings.Contains(contentType, "image/") ||
      strings.Contains(contentType, "video/") ||
      strings.Contains(contentType, "audio/") ||
      strings.Contains(contentType, "font/") {
      // These MIME types should not have textual data.

      return nil
   }

   if !strings.Contains(contentType, "charset") {
      if !detectCharset {
         return nil
      }
      d := chardet.NewTextDetector()
      r, err := d.DetectBest(r.Body)
      if err != nil {
         return err
      }
      contentType = "text/plain; charset=" + r.Charset
   }
   if strings.Contains(contentType, "utf-8") || strings.Contains(contentType, "utf8") {
      return nil
   }
   tmpBody, err := encodeBytes(r.Body, contentType)
   if err != nil {
      return err
   }
   r.Body = tmpBody
   return nil
}

func encodeBytes(b []byte, contentType string) ([]byte, error) {
   r, err := charset.NewReader(bytes.NewReader(b), contentType)
   if err != nil {
      return nil, err
   }
   return ioutil.ReadAll(r)
}
```