---
layout: post  
title: "Mux源码分析"  
subtitle: "服务计算"  
author: "ChenJ"  
header-img: "img/post-bg-imgs/2-systems-analysis-design.jpg"  
header-mask: 0.4  
catalog: true
tags:  服务计算
---

* TOC
{:toc}
---

参考：[官方文档](https://github.com/gorilla/)

mux是通过路由处理请求的，接下来我们就看看它是怎么进行这一过程的：

### mux.go

- 下面是Router的结构体，注释可以获得很多信息，所有请求都会经由这个结构体代表的路由进行分发，接下来我们就看看它的具体内容：



```go
// This will send all incoming requests to the router.
type Router struct {
  // Configurable Handler to be used when no route matches.
  NotFoundHandler http.Handler

  // Configurable Handler to be used when the request method does not match the route.
  MethodNotAllowedHandler http.Handler

  // Parent route, if this is a subrouter.
  parent parentRoute
  // Routes to be matched, in order.
  routes []*Route
  // Routes by name for URL building.
  namedRoutes map[string]*Route
  // See Router.StrictSlash(). This defines the flag for new routes.
  strictSlash bool
  // See Router.SkipClean(). This defines the flag for new routes.
  skipClean bool
  // If true, do not clear the request context after handling the request.
  // This has no effect when go1.7+ is used, since the context is stored
  // on the request itself.
  KeepContext bool
  // see Router.UseEncodedPath(). This defines a flag for all routes.
  useEncodedPath bool
  // Slice of middlewares to be called after a match is found
  middlewares []middleware
}
```

NotFoundHandler表示没有入站规则满足请求时会触发的处理器，MethodNotAllowedHandle请求方法（也就是post、get那些）不满足时触发的触发器，parentRoute为上级路由器（请求由上至下层层传递），Route表示待匹配的路由列表，namedRoutes是一个url匹配对应路由的路由map，接下来是我们再看三个bool类型的标志值，注释提醒我们去找对应函数：

1. strictSlash



```go
// StrictSlash defines the trailing slash behavior for new routes. The initial
// value is false.
//
// When true, if the route path is "/path/", accessing "/path" will perform a redirect
// to the former and vice versa. In other words, your application will always
// see the path as specified in the route.
//
// When false, if the route path is "/path", accessing "/path/" will not match
// this route and vice versa.
func (r *Router) StrictSlash(value bool) *Router {
    r.strictSlash = value
    return r
}
```

​	表示路由路径末尾有斜杆时的行为，true时/path可以被重定向为/path/，false时/path/不会被重定向为/path。默认值为false。

2. skipClean



```go
// SkipClean defines the path cleaning behaviour for new routes. The initial
// value is false. Users should be careful about which routes are not cleaned
//
// When true, if the route path is "/path//to", it will remain with the double
// slash. This is helpful if you have a route like: /fetch/http://xkcd.com/534/
//
// When false, the path will be cleaned, so /fetch/http://xkcd.com/534/ will
// become /fetch/http/xkcd.com/534
func (r *Router) SkipClean(value bool) *Router {
    r.skipClean = value
    return r
}
```

​	表示path中存在冗余斜杆的行为，false表示不会去清除路径中的//，true会。默认值为false。

3. useEncodedPath



```go
// UseEncodedPath tells the router to match the encoded original path
// to the routes.
// For eg. "/path/foo%2Fbar/to" will match the path "/path/{var}/to".
//
// If not called, the router will match the unencoded path to the routes.
// For eg. "/path/foo%2Fbar/to" will match the path "/path/foo/bar/to"
func (r *Router) UseEncodedPath() *Router {
    r.useEncodedPath = true
    return r
}
```

​	表示是否会去匹配未编码的路径，true表示会，false表示不会

*  `HandleFunc` 函数 ：

```go
// HandleFunc registers a new route with a matcher for the URL path.
// See Route.Path() and Route.HandlerFunc().
func (r *Router) HandleFunc(path string, f func(http.ResponseWriter,
  *http.Request)) *Route {
  return r.NewRoute().Path(path).HandlerFunc(f)
}
```

先创建一个 `Route`（通过NewRoute()） ，并为它指定可匹配的路径 ，最后指定一个 `handler`作为请求处理函数。

* `ServeHTTP` 函数：

```go
// ServeHTTP dispatches the handler registered in the matched route.
//
// When there is a match, the route variables can be retrieved calling
// mux.Vars(request).
func (r *Router) ServeHTTP(w http.ResponseWriter, req *http.Request) {
  if !r.skipClean {
    path := req.URL.Path
    if r.useEncodedPath {
      path = req.URL.EscapedPath()
    }
    // Clean path to canonical form and redirect.
    if p := cleanPath(path); p != path {

      // Added 3 lines (Philip Schlump) - It was dropping the query string and #whatever from query.
      // This matches with fix in go 1.2 r.c. 4 for same problem.  Go Issue:
      // http://code.google.com/p/go/issues/detail?id=5252
      url := *req.URL
      url.Path = p
      p = url.String()

      w.Header().Set("Location", p)
      w.WriteHeader(http.StatusMovedPermanently)
      return
    }
  }
  var match RouteMatch
  var handler http.Handler
  if r.Match(req, &match) {
    handler = match.Handler
    req = setVars(req, match.Vars)
    req = setCurrentRoute(req, match.Route)
  }

  if handler == nil && match.MatchErr == ErrMethodMismatch {
    handler = methodNotAllowedHandler()
  }

  if handler == nil {
    handler = http.NotFoundHandler()
  }

  if !r.KeepContext {
    defer contextClear(req)
  }

  handler.ServeHTTP(w, req)
}
```

​	先对url进行处理，使用了上文的skipClean、useEncodedPath等，接着进行路径匹配，成功便将handle设为处理正常请求的match.Handler，否则根据不匹配原因设为methodNotAllowedHandler()或http.NotFoundHandler()，最后调用handle的ServeHTTP(w, req)。

* `Match` 函数，也就是上一个函数中出现检查匹配的函数：

```go
// Match attempts to match the given request against the router's registered routes.
//
// If the request matches a route of this router or one of its subrouters the Route,
// Handler, and Vars fields of the the match argument are filled and this function
// returns true.
//
// If the request does not match any of this router's or its subrouters' routes
// then this function returns false. If available, a reason for the match failure
// will be filled in the match argument's MatchErr field. If the match failure type
// (eg: not found) has a registered handler, the handler is assigned to the Handler
// field of the match argument.
func (r *Router) Match(req *http.Request, match *RouteMatch) bool {
  for _, route := range r.routes {
    if route.Match(req, match) {
      // Build middleware chain if no error was found
      if match.MatchErr == nil {
        for i := len(r.middlewares) - 1; i >= 0; i-- {
          match.Handler = r.middlewares[i].Middleware(match.Handler)
        }
      }
      return true
    }
  }
  // ...
}
```

​	获得传入路由的所有待匹配路由，然后进行Match，后文有具体内容。

### route.go

* 下面是Router的结构体：

```go
// Route stores information to match a request and build URLs.
type Route struct {
  // Parent where the route was registered (a Router).
  parent parentRoute
  // Request handler for the route.
  handler http.Handler
  // List of matchers.
  matchers []matcher
  // Manager for the variables from host and path.
  regexp *routeRegexpGroup
  // If true, when the path pattern is "/path/", accessing "/path" will
  // redirect to the former and vice versa.
  strictSlash bool
  // If true, when the path pattern is "/path//to", accessing "/path//to"
  // will not redirect
  skipClean bool
  // If true, "/path/foo%2Fbar/to" will match the path "/path/{var}/to"
  useEncodedPath bool
  // The scheme used when building URLs.
  buildScheme string
  // If true, this route never matches: it is only used to build URLs.
  buildOnly bool
  // The name used to build URLs.
  name string
  // Error resulted from building a route.
  err error

  buildVarsFunc BuildVarsFunc
}
```

​	parent表示父路由 ，`handler`为请求处理函数、 `matchers`为路由匹配表 、`regexp`为正则规则，后面和前面类似。

* `Path` 函数：

```go
// Path adds a matcher for the URL path.
// It accepts a template with zero or more URL variables enclosed by {}. The
// template must start with a "/".
// Variables can define an optional regexp pattern to be matched:
//
// - {name} matches anything until the next slash.
//
// - {name:pattern} matches the given regexp pattern.
//
// For example:
//
//     r := mux.NewRouter()
//     r.Path("/products/").Handler(ProductsHandler)
//     r.Path("/products/{key}").Handler(ProductsHandler)
//     r.Path("/articles/{category}/{id:[0-9]+}").
//       Handler(ArticleHandler)
//
// Variable names must be unique in a given route. They can be retrieved
// calling mux.Vars(request).
func (r *Route) Path(tpl string) *Route {
  r.err = r.addRegexpMatcher(tpl, regexpTypePath)
  return r
}
```

​	之前出现的，添加一个正则规则，用于匹配：

* Match函数

```go
// Match matches the route against the request.
func (r *Route) Match(req *http.Request, match *RouteMatch) bool {
  if r.buildOnly || r.err != nil {
    return false
  }

  var matchErr error

  // Match everything.
  for _, m := range r.matchers {
    if matched := m.Match(req, match); !matched {
      if _, ok := m.(methodMatcher); ok {
        matchErr = ErrMethodMismatch
        continue
      }
      matchErr = nil
      return false
    }
  }

  if matchErr != nil {
    match.MatchErr = matchErr
    return false
  }

  if match.MatchErr == ErrMethodMismatch {
    // We found a route which matches request method, clear MatchErr
    match.MatchErr = nil
    // Then override the mis-matched handler
    match.Handler = r.handler
  }

  // Yay, we have a match. Let's collect some info about it.
  if match.Route == nil {
    match.Route = r
  }
  if match.Handler == nil {
    match.Handler = r.handler
  }
  if match.Vars == nil {
    match.Vars = make(map[string]string)
  }

  // Set variables.
  if r.regexp != nil {
    r.regexp.setMatch(req, match, r)
  }
  return true
}
```

一路进行检查，不匹配直接返回false，通过在最后返回true，注意看10行中的for，表示请求需满足所有规则才可以通过。 `Path` 以及 `Host` 的验证是通过正则来匹配，`Methods` 和 `Schemes` 为其他匹配方式。



### 最后

到这里就结束了，大致可以知道主要流程了，通过路由分发请求，为每个路由指定相应的匹配路径和处理函数，然后进行匹配、调用处理函数即可。