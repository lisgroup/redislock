# redislock

[English](README.md) | 简体中文

## 介绍
redislock 是基于 Redis 实现的分布式锁。项目是在 github.com/jefferyjob/go-redislock 的基础上进行修改的，感谢原作者的贡献。

## 快速开始

### 安装
```bash
go get -u github.com/lisgroup/redislock
```

### 使用Demo
```go
package main

import (
    "context"
    "fmt"
    "github.com/go-redis/redis/v8"
    "github.com/lisgroup/redislock"
)

func main() {
    // 创建 Redis 客户端
    redisClient := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
        Password: "",
        DB:       0,
    })

    // 创建一个上下文，用于取消锁操作
    ctx := context.Background()

    // 创建 RedisLock 对象
    lock := redislock.New(ctx, redisClient, "test_key")

    // 获取锁
    err := lock.Lock()
    if err != nil {
        fmt.Println("锁获取失败：", err)
        return
    }
    defer lock.UnLock() // 解锁

    // 在锁定期间执行任务
    // ...

    fmt.Println("任务执行完成")
}
```

## 高级用法

### 自定义Token
您可以通过使用 `WithToken` 选项在自定义锁的token功能：
```go
lock := redislock.New(ctx, redisClient, 
	redislock.WithToken("your token")
)
```

### 自定义锁超时时间
您可以通过使用 `WithTimeout` 选项在自定义锁的超时时间功能：
```go
lock := redislock.New(ctx, redisClient, 
	redislock.WithTimeout(time.Duration(10) * time.Second)
)
```

### 自动续期
您可以通过使用 `WithAutoRenew` 选项在获取锁时启用自动续约功能：
```go
lock := redislock.New(ctx, redisClient,
	redislock.WithAutoRenew()
)
```

当使用自动续约时，锁会在获取后自动定期续约，以防止锁过期。要手动续约锁，可以调用 `Renew` 方法。

### 自旋锁
自旋锁是一种尝试在锁可用之前反复获取锁的方式，可以使用 `SpinLock` 方法来实现自旋锁：
```go
err := lock.SpinLock(time.Duration(5) * time.Second) // 尝试获取锁，最多等待5秒
if err != nil {
    fmt.Println("自旋锁超时：", err)
    return
}
defer lock.UnLock() // 解锁

// 在锁定期间执行任务
// ...
```

### 锁释放
如果您希望手动释放锁而不等待锁超时，您可以使用 `UnLock` 方法：
```go
err := lock.Lock()
if err != nil {
    fmt.Println("锁获取失败：", err)
    return
}

// 执行任务

err = lock.UnLock() // 手动释放锁
if err != nil {
    fmt.Println("锁释放失败：", err)
    return
}
```

## 注意事项
- 请确保您的 Redis 服务器设置正确，并且能够正常连接和运行。
- 在使用自动续约功能时，确保在任务执行期间没有出现长时间的阻塞，以免导致续约失败。
- 考虑使用适当的超时设置，以避免由于网络问题等原因导致死锁。
- 尽量保证使用相同的 key 来获取和释放锁，以确保正确性。
- 在使用锁的过程中，建议对关键代码块进行精心设计和测试，以避免出现竞态条件和死锁问题。

## 许可证
本库采用 MIT 进行授权。有关详细信息，请参阅 LICENSE 文件。

