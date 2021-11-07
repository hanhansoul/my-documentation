```go
// Collector provides the scraper instance for a scraping job
type Collector struct {
   // UserAgent is the User-Agent string used by HTTP requests
   
   UserAgent string
   // MaxDepth limits the recursion depth of visited URLs.
   // Set it to 0 for infinite recursion (default).
   MaxDepth int
   // AllowedDomains is a domain whitelist.
   // Leave it blank to allow any domains to be visited
   AllowedDomains []string
   // DisallowedDomains is a domain blacklist.
   DisallowedDomains []string
   // DisallowedURLFilters is a list of regular expressions which restricts
   // visiting URLs. If any of the rules matches to a URL the
   // request will be stopped. DisallowedURLFilters will
   // be evaluated before URLFilters
   // Leave it blank to allow any URLs to be visited
   DisallowedURLFilters []*regexp.Regexp
   // URLFilters is a list of regular expressions which restricts
   // visiting URLs. If any of the rules matches to a URL the
   // request won't be stopped. DisallowedURLFilters will
   // be evaluated before URLFilters

   // Leave it blank to allow any URLs to be visited
   URLFilters []*regexp.Regexp

   // AllowURLRevisit allows multiple downloads of the same URL
   AllowURLRevisit bool
   // MaxBodySize is the limit of the retrieved response body in bytes.
   // 0 means unlimited.
   // The default value for MaxBodySize is 10MB (10 * 1024 * 1024 bytes).
   MaxBodySize int
   // CacheDir specifies a location where GET requests are cached as files.
   // When it's not defined, caching is disabled.
   CacheDir string
   // IgnoreRobotsTxt allows the Collector to ignore any restrictions set by
   // the target host's robots.txt file.  See http://www.robotstxt.org/ for more
   // information.
   IgnoreRobotsTxt bool
   // Async turns on asynchronous network communication. Use Collector.Wait() to
   // be sure all requests have been finished.
   Async bool
   // ParseHTTPErrorResponse allows parsing HTTP responses with non 2xx status codes.
   // By default, Colly parses only successful HTTP responses. Set ParseHTTPErrorResponse
   // to true to enable it.
   ParseHTTPErrorResponse bool
   // ID is the unique identifier of a collector
   ID uint32
   // DetectCharset can enable character encoding detection for non-utf8 response bodies
   // without explicit charset declaration. This feature uses https://github.com/saintfish/chardet
   DetectCharset bool
   // RedirectHandler allows control on how a redirect will be managed
   // use c.SetRedirectHandler to set this value
   redirectHandler func(req *http.Request, via []*http.Request) error
   // CheckHead performs a HEAD request before every GET to pre-validate the response
   CheckHead bool
   // TraceHTTP enables capturing and reporting request performance for crawler tuning.
   // When set to true, the Response.Trace will be filled in with an HTTPTrace object.
   TraceHTTP bool
   // Context is the context that will be used for HTTP requests. You can set this
   // to support clean cancellation of scraping.
   Context context.Context

   store                    storage.Storage
   debugger                 debug.Debugger
   robotsMap                map[string]*robotstxt.RobotsData
   htmlCallbacks            []*htmlCallbackContainer
   xmlCallbacks             []*xmlCallbackContainer
   requestCallbacks         []RequestCallback
   responseCallbacks        []ResponseCallback
   responseHeadersCallbacks []ResponseHeadersCallback
   errorCallbacks           []ErrorCallback
   scrapedCallbacks         []ScrapedCallback
   requestCount             uint32
   responseCount            uint32
   backend                  *httpBackend
   wg                       *sync.WaitGroup
   lock                     *sync.RWMutex
}
```





```go
// RequestCallback is a type alias for OnRequest callback functions
type RequestCallback func(*Request)

// ResponseHeadersCallback is a type alias for OnResponseHeaders callback functions
type ResponseHeadersCallback func(*Response)

// ResponseCallback is a type alias for OnResponse callback functions
type ResponseCallback func(*Response)

// HTMLCallback is a type alias for OnHTML callback functions
type HTMLCallback func(*HTMLElement)

// XMLCallback is a type alias for OnXML callback functions
type XMLCallback func(*XMLElement)

// ErrorCallback is a type alias for OnError callback functions
type ErrorCallback func(*Response, error)

// ScrapedCallback is a type alias for OnScraped callback functions
type ScrapedCallback func(*Response)
```



```
var envMap = map[string]func(*Collector, string){
   "ALLOWED_DOMAINS": func(c *Collector, val string) {
      c.AllowedDomains = strings.Split(val, ",")
   },
   "CACHE_DIR": func(c *Collector, val string) {
      c.CacheDir = val
   },
   "DETECT_CHARSET": func(c *Collector, val string) {
      c.DetectCharset = isYesString(val)
   },
   "DISABLE_COOKIES": func(c *Collector, _ string) {
      c.backend.Client.Jar = nil
   },
   "DISALLOWED_DOMAINS": func(c *Collector, val string) {
      c.DisallowedDomains = strings.Split(val, ",")
   },
   "IGNORE_ROBOTSTXT": func(c *Collector, val string) {
      c.IgnoreRobotsTxt = isYesString(val)
   },
   "FOLLOW_REDIRECTS": func(c *Collector, val string) {
      if !isYesString(val) {
         c.redirectHandler = func(req *http.Request, via []*http.Request) error {
            return http.ErrUseLastResponse
         }
      }
   },
   "MAX_BODY_SIZE": func(c *Collector, val string) {
      size, err := strconv.Atoi(val)
      if err == nil {
         c.MaxBodySize = size
      }
   },
   "MAX_DEPTH": func(c *Collector, val string) {
      maxDepth, err := strconv.Atoi(val)
      if err == nil {
         c.MaxDepth = maxDepth
      }
   },
   "PARSE_HTTP_ERROR_RESPONSE": func(c *Collector, val string) {
      c.ParseHTTPErrorResponse = isYesString(val)
   },
   "TRACE_HTTP": func(c *Collector, val string) {
      c.TraceHTTP = isYesString(val)
   },
   "USER_AGENT": func(c *Collector, val string) {
      c.UserAgent = val
   },
}
```





```go
// NewCollector creates a new Collector instance with default configuration
func NewCollector(options ...CollectorOption) *Collector {
   c := &Collector{}
   c.Init()

   for _, f := range options {
      f(c)
   }

   c.parseSettingsFromEnv()

   return c
}
```



```go
func (c *Collector) scrape(u, method string, depth int, requestData io.Reader, ctx *Context, hdr http.Header, checkRevisit bool) error {
   parsedURL, err := url.Parse(u)
   if err != nil {
      return err
   }
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

func (c *Collector) requestCheck(u string, parsedURL *url.URL, method string, requestData io.Reader, depth int, checkRevisit bool) error {
	if u == "" {
		return ErrMissingURL
	}
	if c.MaxDepth > 0 && c.MaxDepth < depth {
		return ErrMaxDepth
	}
	if len(c.DisallowedURLFilters) > 0 {
		if isMatchingFilter(c.DisallowedURLFilters, []byte(u)) {
			return ErrForbiddenURL
		}
	}
	if len(c.URLFilters) > 0 {
		if !isMatchingFilter(c.URLFilters, []byte(u)) {
			return ErrNoURLFiltersMatch
		}
	}
	if !c.isDomainAllowed(parsedURL.Hostname()) {
		return ErrForbiddenDomain
	}
	if method != "HEAD" && !c.IgnoreRobotsTxt {
		if err := c.checkRobots(parsedURL); err != nil {
			return err
		}
	}
	if checkRevisit && !c.AllowURLRevisit {
		h := fnv.New64a()
		h.Write([]byte(u))

		var uHash uint64
		if method == "GET" {
			uHash = h.Sum64()
		} else if requestData != nil {
			h.Write(streamToByte(requestData))
			uHash = h.Sum64()
		} else {
			return nil
		}

		visited, err := c.store.IsVisited(uHash)
		if err != nil {
			return err
		}
		if visited {
			return ErrAlreadyVisited
		}
		return c.store.Visited(uHash)
	}
	return nil
}
```



---

```go
func Parse(rawURL string) (*URL, error)
```



---



```
func (c *Collector) Visit(URL string) error
```

```
func (c *Collector) scrape(u, method string, depth int, requestData io.Reader, ctx *Context, hdr http.Header, checkRevisit bool) error
```

```
func (c *Collector) fetch(u, method string, depth int, requestData io.Reader, ctx *Context, hdr http.Header, req *http.Request) error
```

