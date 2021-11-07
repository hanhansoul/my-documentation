# strings包

### func [HasPrefix](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#371)

```
func HasPrefix(s, prefix string) bool
```

判断s是否有前缀字符串prefix。

### func [HasSuffix](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#376)

```
func HasSuffix(s, suffix string) bool
```

判断s是否有后缀字符串suffix。

### func [Contains](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#112)

```
func Contains(s, substr string) bool
```

判断字符串s是否包含子串substr。

### func [Count](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#65)

```
func Count(s, sep string) int
```

返回字符串s中有几个不重复的sep子串。

### func [Index](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#127)

```
func Index(s, sep string) int
```

子串sep在字符串s中第一次出现的位置，不存在则返回-1。

### func [LastIndex](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#164)

```
func LastIndex(s, sep string) int
```

子串sep在字符串s中最后一次出现的位置，不存在则返回-1。

### func [ToLower](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#437)

```
func ToLower(s string) string
```

返回将所有字母都转为对应的小写版本的拷贝。

### func [ToUpper](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#434)

```
func ToUpper(s string) string
```

返回将所有字母都转为对应的大写版本的拷贝。

### func [Repeat](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#424)

```
func Repeat(s string, count int) string
```

返回count个s串联的字符串。

### func [Replace](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#638)

```
func Replace(s, old, new string, n int) string
```

返回将s中前n个不重叠old子串都替换为new的新字符串，如果n<0会替换所有old子串。

### func [Trim](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#586)

```
func Trim(s string, cutset string) string
```

返回将s前后端所有cutset包含的utf-8码值都去掉的字符串。

### func [TrimSpace](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#613)

```
func TrimSpace(s string) string
```

返回将s前后端所有空白（unicode.IsSpace指定）都去掉的字符串。

### func [Split](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#294)

```
func Split(s, sep string) []string
```

用去掉s中出现的sep的方式进行分割，会分割到结尾，并返回生成的所有片段组成的切片（每一个sep都会进行一次切割，即使两个sep相邻，也会进行两次切割）。如果sep为空字符，Split会将s切分成每一个unicode码值一个字符串。

### func [Join](https://github.com/golang/go/blob/master/src/strings/strings.go?name=release#349)

```
func Join(a []string, sep string) string
```

将一系列字符串连接为一个字符串，之间用sep来分隔。