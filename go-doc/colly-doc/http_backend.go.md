# http_backend.go

```go
type httpBackend struct {
   LimitRules []*LimitRule
   Client     *http.Client
   lock       *sync.RWMutex
}
```



```
// LimitRule provides connection restrictions for domains.
// Both DomainRegexp and DomainGlob can be used to specify
// the included domains patterns, but at least one is required.
// There can be two kind of limitations:
//  - Parallelism: Set limit for the number of concurrent requests to matching domains
//  - Delay: Wait specified amount of time between requests (parallelism is 1 in this case)
type LimitRule struct {
   // DomainRegexp is a regular expression to match against domains
   DomainRegexp string
   // DomainGlob is a glob pattern to match against domains
   DomainGlob string
   // Delay is the duration to wait before creating a new request to the matching domains
   Delay time.Duration
   // RandomDelay is the extra randomized duration to wait added to Delay before creating a new request
   RandomDelay time.Duration
   // Parallelism is the number of the maximum allowed concurrent requests of the matching domains
   Parallelism    int
   waitChan       chan bool
   compiledRegexp *regexp.Regexp
   compiledGlob   glob.Glob
}
```