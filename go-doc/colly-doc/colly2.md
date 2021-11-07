```
func (c *Collector) scrape(u, method string, depth int, requestData io.Reader, ctx *Context, hdr http.Header, checkRevisit bool) error {
   parsedURL, err := url.Parse(u)
   if err != nil {
      return err
   }
   // requestCheck通过检查collector中的限制条件，判断将要发出的请求是否合法
   if err := c.requestCheck(u, parsedURL, method, requestData, depth, checkRevisit); err != nil {
      return err
   }

   if hdr == nil {
      hdr = http.Header{}
   }
   if _, ok := hdr["User-Agent"]; !ok {
      hdr.Set("User-Agent", c.UserAgent)
   }
   rc, ok := requestData.(io.ReadCloser)
   if !ok && requestData != nil {
      rc = ioutil.NopCloser(requestData)
   }
   // The Go HTTP API ignores "Host" in the headers, preferring the client
   // to use the Host field on Request.
   host := parsedURL.Host
   if hostHeader := hdr.Get("Host"); hostHeader != "" {
      host = hostHeader
   }
   req := &http.Request{
      Method:     method,
      URL:        parsedURL,
      Proto:      "HTTP/1.1",
      ProtoMajor: 1,
      ProtoMinor: 1,
      Header:     hdr,
      Body:       rc,
      Host:       host,
   }
   // note: once 1.13 is minimum supported Go version,
   // replace this with http.NewRequestWithContext
   req = req.WithContext(c.Context)
   setRequestBody(req, requestData)
   u = parsedURL.String()
   c.wg.Add(1)
   if c.Async {
      go c.fetch(u, method, depth, requestData, ctx, hdr, req)
      return nil
   }
   return c.fetch(u, method, depth, requestData, ctx, hdr, req)
}
```



```
func (c *Collector) fetch(u, method string, depth int, requestData io.Reader, ctx *Context, hdr http.Header, req *http.Request) error {
   defer c.wg.Done()
   if ctx == nil {
      ctx = NewContext()
   }
   request := &Request{
      URL:       req.URL,
      Headers:   &req.Header,
      Ctx:       ctx,
      Depth:     depth,
      Method:    method,
      Body:      requestData,
      collector: c,
      ID:        atomic.AddUint32(&c.requestCount, 1),
   }

   c.handleOnRequest(request)

   if request.abort {
      return nil
   }

   if method == "POST" && req.Header.Get("Content-Type") == "" {
      req.Header.Add("Content-Type", "application/x-www-form-urlencoded")
   }

   if req.Header.Get("Accept") == "" {
      req.Header.Set("Accept", "*/*")
   }

   var hTrace *HTTPTrace
   if c.TraceHTTP {
      hTrace = &HTTPTrace{}
      req = hTrace.WithTrace(req)
   }
   origURL := req.URL
   checkHeadersFunc := func(req *http.Request, statusCode int, headers http.Header) bool {
      if req.URL != origURL {
         request.URL = req.URL
         request.Headers = &req.Header
      }
      c.handleOnResponseHeaders(&Response{Ctx: ctx, Request: request, StatusCode: statusCode, Headers: &headers})
      return !request.abort
   }
   response, err := c.backend.Cache(req, c.MaxBodySize, checkHeadersFunc, c.CacheDir)
   if proxyURL, ok := req.Context().Value(ProxyURLKey).(string); ok {
      request.ProxyURL = proxyURL
   }
   if err := c.handleOnError(response, err, request, ctx); err != nil {
      return err
   }
   atomic.AddUint32(&c.responseCount, 1)
   response.Ctx = ctx
   response.Request = request
   response.Trace = hTrace

   err = response.fixCharset(c.DetectCharset, request.ResponseCharacterEncoding)
   if err != nil {
      return err
   }

   c.handleOnResponse(response)

   err = c.handleOnHTML(response)
   if err != nil {
      c.handleOnError(response, err, request, ctx)
   }

   err = c.handleOnXML(response)
   if err != nil {
      c.handleOnError(response, err, request, ctx)
   }

   c.handleOnScraped(response)

   return err
}
```



```go
// OnRequest registers a function. Function will be executed on every
// request made by the Collector
func (c *Collector) OnRequest(f RequestCallback) {
   c.lock.Lock()
   if c.requestCallbacks == nil {
      c.requestCallbacks = make([]RequestCallback, 0, 4)
   }
   c.requestCallbacks = append(c.requestCallbacks, f)
   c.lock.Unlock()
}

// OnResponseHeaders registers a function. Function will be executed on every response
// when headers and status are already received, but body is not yet read.
//
// Like in OnRequest, you can call Request.Abort to abort the transfer. This might be
// useful if, for example, you're following all hyperlinks, but want to avoid
// downloading files.
//
// Be aware that using this will prevent HTTP/1.1 connection reuse, as
// the only way to abort a download is to immediately close the connection.
// HTTP/2 doesn't suffer from this problem, as it's possible to close
// specific stream inside the connection.
func (c *Collector) OnResponseHeaders(f ResponseHeadersCallback) {
	c.lock.Lock()
	c.responseHeadersCallbacks = append(c.responseHeadersCallbacks, f)
	c.lock.Unlock()
}

// OnResponse registers a function. Function will be executed on every response
func (c *Collector) OnResponse(f ResponseCallback) {
	c.lock.Lock()
	if c.responseCallbacks == nil {
		c.responseCallbacks = make([]ResponseCallback, 0, 4)
	}
	c.responseCallbacks = append(c.responseCallbacks, f)
	c.lock.Unlock()
}

// OnHTML registers a function. Function will be executed on every HTML
// element matched by the GoQuery Selector parameter.
// GoQuery Selector is a selector used by https://github.com/PuerkitoBio/goquery
func (c *Collector) OnHTML(goquerySelector string, f HTMLCallback) {
	c.lock.Lock()
	if c.htmlCallbacks == nil {
		c.htmlCallbacks = make([]*htmlCallbackContainer, 0, 4)
	}
	c.htmlCallbacks = append(c.htmlCallbacks, &htmlCallbackContainer{
		Selector: goquerySelector,
		Function: f,
	})
	c.lock.Unlock()
}

// OnXML registers a function. Function will be executed on every XML
// element matched by the xpath Query parameter.
// xpath Query is used by https://github.com/antchfx/xmlquery
func (c *Collector) OnXML(xpathQuery string, f XMLCallback) {
	c.lock.Lock()
	if c.xmlCallbacks == nil {
		c.xmlCallbacks = make([]*xmlCallbackContainer, 0, 4)
	}
	c.xmlCallbacks = append(c.xmlCallbacks, &xmlCallbackContainer{
		Query:    xpathQuery,
		Function: f,
	})
	c.lock.Unlock()
}

// OnHTMLDetach deregister a function. Function will not be execute after detached
func (c *Collector) OnHTMLDetach(goquerySelector string) {
	c.lock.Lock()
	deleteIdx := -1
	for i, cc := range c.htmlCallbacks {
		if cc.Selector == goquerySelector {
			deleteIdx = i
			break
		}
	}
	if deleteIdx != -1 {
		c.htmlCallbacks = append(c.htmlCallbacks[:deleteIdx], c.htmlCallbacks[deleteIdx+1:]...)
	}
	c.lock.Unlock()
}

// OnXMLDetach deregister a function. Function will not be execute after detached
func (c *Collector) OnXMLDetach(xpathQuery string) {
	c.lock.Lock()
	deleteIdx := -1
	for i, cc := range c.xmlCallbacks {
		if cc.Query == xpathQuery {
			deleteIdx = i
			break
		}
	}
	if deleteIdx != -1 {
		c.xmlCallbacks = append(c.xmlCallbacks[:deleteIdx], c.xmlCallbacks[deleteIdx+1:]...)
	}
	c.lock.Unlock()
}

// OnError registers a function. Function will be executed if an error
// occurs during the HTTP request.
func (c *Collector) OnError(f ErrorCallback) {
	c.lock.Lock()
	if c.errorCallbacks == nil {
		c.errorCallbacks = make([]ErrorCallback, 0, 4)
	}
	c.errorCallbacks = append(c.errorCallbacks, f)
	c.lock.Unlock()
}

// OnScraped registers a function. Function will be executed after
// OnHTML, as a final part of the scraping.
func (c *Collector) OnScraped(f ScrapedCallback) {
	c.lock.Lock()
	if c.scrapedCallbacks == nil {
		c.scrapedCallbacks = make([]ScrapedCallback, 0, 4)
	}
	c.scrapedCallbacks = append(c.scrapedCallbacks, f)
	c.lock.Unlock()
}
```



```go
func (c *Collector) handleOnRequest(r *Request) {
   if c.debugger != nil {
      c.debugger.Event(createEvent("request", r.ID, c.ID, map[string]string{
         "url": r.URL.String(),
      }))
   }
   for _, f := range c.requestCallbacks {
      f(r)
   }
}

func (c *Collector) handleOnResponse(r *Response) {
   if c.debugger != nil {
      c.debugger.Event(createEvent("response", r.Request.ID, c.ID, map[string]string{
         "url":    r.Request.URL.String(),
         "status": http.StatusText(r.StatusCode),
      }))
   }
   for _, f := range c.responseCallbacks {
      f(r)
   }
}

func (c *Collector) handleOnResponseHeaders(r *Response) {
   if c.debugger != nil {
      c.debugger.Event(createEvent("responseHeaders", r.Request.ID, c.ID, map[string]string{
         "url":    r.Request.URL.String(),
         "status": http.StatusText(r.StatusCode),
      }))
   }
   for _, f := range c.responseHeadersCallbacks {
      f(r)
   }
}

func (c *Collector) handleOnHTML(resp *Response) error {
   if len(c.htmlCallbacks) == 0 || !strings.Contains(strings.ToLower(resp.Headers.Get("Content-Type")), "html") {
      return nil
   }
   doc, err := goquery.NewDocumentFromReader(bytes.NewBuffer(resp.Body))
   if err != nil {
      return err
   }
   if href, found := doc.Find("base[href]").Attr("href"); found {
      baseURL, err := resp.Request.URL.Parse(href)
      if err == nil {
         resp.Request.baseURL = baseURL
      }
   }
   for _, cc := range c.htmlCallbacks {
      i := 0
      doc.Find(cc.Selector).Each(func(_ int, s *goquery.Selection) {
         for _, n := range s.Nodes {
            e := NewHTMLElementFromSelectionNode(resp, s, n, i)
            i++
            if c.debugger != nil {
               c.debugger.Event(createEvent("html", resp.Request.ID, c.ID, map[string]string{
                  "selector": cc.Selector,
                  "url":      resp.Request.URL.String(),
               }))
            }
            cc.Function(e)
         }
      })
   }
   return nil
}

func (c *Collector) handleOnXML(resp *Response) error {
   if len(c.xmlCallbacks) == 0 {
      return nil
   }
   contentType := strings.ToLower(resp.Headers.Get("Content-Type"))
   isXMLFile := strings.HasSuffix(strings.ToLower(resp.Request.URL.Path), ".xml") || strings.HasSuffix(strings.ToLower(resp.Request.URL.Path), ".xml.gz")
   if !strings.Contains(contentType, "html") && (!strings.Contains(contentType, "xml") && !isXMLFile) {
      return nil
   }

   if strings.Contains(contentType, "html") {
      doc, err := htmlquery.Parse(bytes.NewBuffer(resp.Body))
      if err != nil {
         return err
      }
      if e := htmlquery.FindOne(doc, "//base"); e != nil {
         for _, a := range e.Attr {
            if a.Key == "href" {
               baseURL, err := resp.Request.URL.Parse(a.Val)
               if err == nil {
                  resp.Request.baseURL = baseURL
               }
               break
            }
         }
      }

      for _, cc := range c.xmlCallbacks {
         for _, n := range htmlquery.Find(doc, cc.Query) {
            e := NewXMLElementFromHTMLNode(resp, n)
            if c.debugger != nil {
               c.debugger.Event(createEvent("xml", resp.Request.ID, c.ID, map[string]string{
                  "selector": cc.Query,
                  "url":      resp.Request.URL.String(),
               }))
            }
            cc.Function(e)
         }
      }
   } else if strings.Contains(contentType, "xml") || isXMLFile {
      doc, err := xmlquery.Parse(bytes.NewBuffer(resp.Body))
      if err != nil {
         return err
      }

      for _, cc := range c.xmlCallbacks {
         xmlquery.FindEach(doc, cc.Query, func(i int, n *xmlquery.Node) {
            e := NewXMLElementFromXMLNode(resp, n)
            if c.debugger != nil {
               c.debugger.Event(createEvent("xml", resp.Request.ID, c.ID, map[string]string{
                  "selector": cc.Query,
                  "url":      resp.Request.URL.String(),
               }))
            }
            cc.Function(e)
         })
      }
   }
   return nil
}

func (c *Collector) handleOnError(response *Response, err error, request *Request, ctx *Context) error {
   if err == nil && (c.ParseHTTPErrorResponse || response.StatusCode < 203) {
      return nil
   }
   if err == nil && response.StatusCode >= 203 {
      err = errors.New(http.StatusText(response.StatusCode))
   }
   if response == nil {
      response = &Response{
         Request: request,
         Ctx:     ctx,
      }
   }
   if c.debugger != nil {
      c.debugger.Event(createEvent("error", request.ID, c.ID, map[string]string{
         "url":    request.URL.String(),
         "status": http.StatusText(response.StatusCode),
      }))
   }
   if response.Request == nil {
      response.Request = request
   }
   if response.Ctx == nil {
      response.Ctx = request.Ctx
   }
   for _, f := range c.errorCallbacks {
      f(response, err)
   }
   return err
}

func (c *Collector) handleOnScraped(r *Response) {
   if c.debugger != nil {
      c.debugger.Event(createEvent("scraped", r.Request.ID, c.ID, map[string]string{
         "url": r.Request.URL.String(),
      }))
   }
   for _, f := range c.scrapedCallbacks {
      f(r)
   }
}
```