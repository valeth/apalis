# apalis-cron

A simple yet extensible library for cron-like job scheduling for rust. Since apalis-cron is build on top of apalis which supports tower middlerware, you should be able to easily add middleware such as tracing, retries, load shed, concurrency etc.

## Example

```rust
use apalis::prelude::*;
use apalis::layers::{Extension, DefaultRetryPolicy, RetryLayer};
use apalis::cron::{CronWorker, Schedule};
use tower::ServiceBuilder;
use std::str::FromStr;
use serde::{Serialize,Deserialize};

#[derive(Serialize, Deserialize, Default, Debug, Clone)]
struct Reminder;

impl Job for Reminder {
    const NAME: &'static str = "reminder::DailyReminder";
}
async fn send_reminder(job: Reminder, ctx: JobContext) {
    // Do reminder stuff
}

#[tokio::main]
async fn main() {
    let schedule = Schedule::from_str("@daily").unwrap();

    let service = ServiceBuilder::new()
        .layer(RetryLayer::new(DefaultRetryPolicy))
        .service(job_fn(send_reminder));

    let worker = CronWorker::new(schedule, service);

    Monitor::new()
        .register(worker)
        .run()
        .await
        .unwrap();
}
```
