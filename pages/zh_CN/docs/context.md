---
title: Context
layout: page
---

GORM 通过 `WithContext` 方法提供了 Context 支持

## 单会话模式

单会话模式通常被用于执行单次操作

```go
db.WithContext(ctx).Find(&users)
```

## 持续会话模式

持续会话模式通常被用于执行一系列操作，例如：

```go
tx := db.WithContext(ctx)
tx.First(&user, 1)
tx.Model(&user).Update("role", "admin")
```

## Chi 中间件示例

在处理 API 请求时持续会话模式会比较有用。例如，您可以在中间件中为 `*gorm.DB` 设置超时 Context，然后使用 `*gorm.DB` 处理所有请求

下面是一个 Chi 中间件的示例：

```go
func SetDBMiddleware(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    timeoutContext, _ := context.WithTimeout(context.Background(), time.Second)
    ctx := context.WithValue(r.Context(), "DB", db.WithContext(timeoutContext))
    next.ServeHTTP(w, r.WithContext(ctx))
  })
}

r := chi.NewRouter()
r.Use(SetDBMiddleware)

r.Get("/", func(w http.ResponseWriter, r *http.Request) {
  db, ok := ctx.Value("DB").(*gorm.DB)

  var users []User
  db.Find(&users)

  // 以下省略 32 个 DB 操作...
})

r.Get("/user", func(w http.ResponseWriter, r *http.Request) {
  db, ok := ctx.Value("DB").(*gorm.DB)

  var user User
  db.First(&user)

  // 以下省略 32 个 DB 操作...
})
```

**注意** 通过 `WithContext` 设置的 `Context` 是线程安全的，参考[会话](session.html)获取详情

## Logger

Logger 也可以接受 `Context` ，可用于日志追踪，请参考 [Logger](logger.html) 获取详情
