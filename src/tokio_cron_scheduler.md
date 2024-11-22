# tokio_cron_scheduler

## 概念
- tokio_cron_scheduler: 在异步 tokio 环境中使用类似 cron 的调度，支持任务立即执行，或者以一个固定的duration 重复执行，可以使用 PostgreSQL 或 Nats 保留任务数据。
- Nats：一个开源、轻量级、高性能的分布式消息中间件，实现了高可伸缩性和优雅的 Publish / Subscribe 模型，使用 Golang 语言开发。

## example
```rust
#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // 创建 scheduler
    let sched = JobScheduler::new().await?;
    // 添加一个 Job
    sched.add(Job::new("* * * * * *", |_uuid, _l| {
        println!("now: {}", Local::now().to_rfc3339())
    })?).await?;
    // 启动 scheduler
    sched.start().await?;
    // 等待一段时间后退出
    tokio::time::sleep(tokio::time::Duration::from_secs(10)).await;
    Ok(())
}
```

## JobScheduler
初始化 JobScheduler，有 2 种方法：
- pub async fn inited(&self) -> bool：返回 scheduler 是否完成了初始化
- pub async fn init(&mut self) -> Result<(), JobSchedulerError>：初始化各种 actor

## Job
### 创建 Job
```rust
// 默认的，创建 scheduler
pub async fn new() -> Result<Self, JobSchedulerError>

// Channel_size 参数用于设置在 Actor 之间通信的 channel 的大小。简而言之，数量会影响发送者被阻止之前可以缓冲的消息数量。当发送者被阻止时，处理就会滞后。
pub async fn new_with_channel_size(
    channel_size: usize,
) -> Result<Self, JobSchedulerError>

// 使用自定义元数据和通知运行程序、作业和通知代码提供程序创建新的 JobsSchedulerLocked
pub async fn new_with_storage_and_code(
    metadata_storage: Box<dyn MetaDataStorage + Send + Sync>,
    notification_storage: Box<dyn NotificationStore + Send + Sync>,
    job_code: Box<dyn JobCode + Send + Sync>,
    notification_code: Box<dyn NotificationCode + Send + Sync>,
    channel_size: usize,
) -> Result<Self, JobSchedulerError>
```
