## Thread

* 线程的创建和调用

```csharp
Thread thread = new Thread(() => {});
thread.Start()
```

* 阻塞
```csharp
thread.Join();
```

* 中止
```csharp
try {
	thread.Abort();
}
catch (Exception e) {}
```

* 暂停/继续
