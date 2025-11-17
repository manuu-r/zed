# GPUI Async Tasks

**Last Updated:** 2025-11-16

---

## Task System

GPUI provides `Task<T>` for managing asynchronous operations.

### Creating Tasks

#### Foreground Tasks

```rust
let task = cx.spawn(async move |this, cx| {
    let data = fetch_data().await?;

    this.update(cx, |this, cx| {
        this.data = Some(data);
        cx.notify();
    })?;

    Ok(())
});
```

#### Background Tasks

```rust
let task = cx.background_spawn(async move {
    expensive_computation()
});
```

### Task Lifecycle

```
Create Task
  │
  ├─→ Store in field → Cancelled when field dropped
  ├─→ Await → Runs to completion
  └─→ Detach → Runs independently
```

### Detaching Tasks

```rust
// Fire and forget
cx.spawn(async move |cx| {
    background_work().await;
}).detach();

// With error logging
cx.spawn(async move |cx| {
    fallible_work().await
}).detach_and_log_err(cx);
```

## Async Contexts

### AsyncApp

```rust
cx.spawn(async move |cx: AsyncApp| {
    let data = fetch().await;

    cx.update(|cx| {
        // Use data
    }).ok();
})
```

### AsyncWindowContext

```rust
cx.spawn_in(window, async move |this, cx: AsyncWindowContext| {
    let result = fetch().await;

    this.update_in(cx, |this, window, cx| {
        this.process(result, window, cx);
    })?;

    Ok(())
})
```

## Common Patterns

### Loading State

```rust
pub struct DataView {
    data: Option<Data>,
    loading: bool,
    _task: Option<Task<()>>,
}

impl DataView {
    fn load(&mut self, cx: &mut Context<Self>) {
        self.loading = true;
        cx.notify();

        self._task = Some(cx.spawn(async move |this, cx| {
            let data = fetch_data().await.ok()?;

            this.update(cx, |this, cx| {
                this.data = Some(data);
                this.loading = false;
                cx.notify();
            }).ok();

            Some(())
        }));
    }
}
```

### Cancellation

```rust
pub struct Worker {
    task: Option<Task<()>>,
}

impl Worker {
    fn start(&mut self, cx: &mut Context<Self>) {
        self.task = Some(cx.spawn(async move |this, cx| {
            loop {
                // Work will be cancelled when task is dropped
                do_work().await;
            }
        }));
    }

    fn stop(&mut self) {
        self.task = None; // Drops task, cancels work
    }
}
```

### Sequential Tasks

```rust
cx.spawn(async move |this, cx| {
    let result1 = step1().await?;
    let result2 = step2(result1).await?;
    let result3 = step3(result2).await?;

    this.update(cx, |this, cx| {
        this.apply_results(result3, cx);
    })?;

    Ok(())
}).detach();
```

### Parallel Tasks

```rust
cx.spawn(async move |this, cx| {
    let (result1, result2, result3) = futures::join!(
        fetch_data1(),
        fetch_data2(),
        fetch_data3(),
    );

    this.update(cx, |this, cx| {
        this.apply_all(result1, result2, result3, cx);
    }).ok();
}).detach();
```

## Error Handling

```rust
// Propagate errors
cx.spawn(async move |this, cx| {
    let data = fetch().await?;
    this.update(cx, |this, cx| {
        this.data = Some(data);
    })?;
    Ok(())
}).detach_and_log_err(cx);

// Handle errors explicitly
cx.spawn(async move |this, cx| {
    match fetch().await {
        Ok(data) => {
            this.update(cx, |this, cx| {
                this.data = Some(data);
            }).ok();
        }
        Err(e) => {
            this.update(cx, |this, cx| {
                this.error = Some(e);
            }).ok();
        }
    }
}).detach();
```

## Best Practices

1. **Always detach or store tasks**
   - Dropped tasks are cancelled
   - Detach fire-and-forget tasks
   - Store cancellable tasks in fields

2. **Use weak references in async**
   - Prevents keeping entities alive
   - Handle errors when updating

3. **Background for CPU work**
   - Use `background_spawn` for intensive computation
   - Use `spawn` for I/O and entity access

4. **Handle async context errors**
   - `cx.update()` returns `Result`
   - Entity might be dropped
   - Use `.ok()` or `?` appropriately

## Further Reading

- [Core Concepts](./core-concepts.md)
- [Entity System](./entity-system.md)
