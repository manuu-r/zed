# Performance Optimization

Guide for profiling and optimizing Zed's performance.

## Profiling Tools

### macOS: Instruments

```bash
# Build with optimizations
cargo build --release --bin zed

# Profile with Instruments
instruments -t "Time Profiler" ./target/release/zed
```

**What to look for:**
- Hot paths (functions taking most time)
- Unexpected allocations
- Lock contention

### Linux: perf

```bash
# Record profile
perf record -g ./target/release/zed

# Analyze
perf report
```

### Cross-Platform: Cargo Flamegraph

```bash
cargo install flamegraph

# Generate flamegraph
cargo flamegraph --bin zed
```

## Common Optimizations

### 1. Reduce Allocations

```rust
// ❌ Allocates on every call
fn format_label(&self) -> String {
    format!("Label: {}", self.value)
}

// ✅ Return borrowed string or SharedString
fn label(&self) -> &str {
    &self.cached_label
}
```

### 2. Batch Notifications

```rust
// ❌ Multiple re-renders
for item in items {
    self.add(item);
    cx.notify();  // Re-renders each time
}

// ✅ Single re-render
self.transact(cx, |this, cx| {
    for item in items {
        this.add(item);
    }
    // Single notify
});
```

### 3. Lazy Computation

```rust
struct ExpensiveComputation {
    cached: Option<Result>,
    dirty: bool,
}

impl ExpensiveComputation {
    fn get(&mut self) -> &Result {
        if self.dirty || self.cached.is_none() {
            self.cached = Some(self.compute());
            self.dirty = false;
        }
        self.cached.as_ref().unwrap()
    }
}
```

### 4. Use Appropriate Data Structures

```rust
// For lookups: HashMap
let mut map: HashMap<String, Value> = HashMap::new();

// For ordered iteration: Vec
let mut list: Vec<Item> = Vec::new();

// For fast head/tail operations: VecDeque
let mut queue: VecDeque<Task> = VecDeque::new();

// For many small allocations: SmallVec
let mut small: SmallVec<[Item; 8]> = SmallVec::new();
```

### 5. Avoid Cloning Large Data

```rust
// ❌ Clones entire buffer
fn process(&self, buffer: Buffer) {
    // Uses buffer
}

// ✅ Borrow when possible
fn process(&self, buffer: &Buffer) {
    // Uses buffer
}

// ✅ Use Arc for shared ownership
fn process(&self, buffer: Arc<Buffer>) {
    // Can be cloned cheaply
}
```

## Benchmarking

### Criterion Benchmarks

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn benchmark_function(c: &mut Criterion) {
    c.bench_function("my_function", |b| {
        b.iter(|| {
            my_function(black_box(input))
        });
    });
}

criterion_group!(benches, benchmark_function);
criterion_main!(benches);
```

### Running Benchmarks

```bash
cargo bench
```

## Memory Optimization

### 1. Use WeakEntity to Break Cycles

```rust
struct Parent {
    child: Entity<Child>,
}

struct Child {
    parent: WeakEntity<Parent>,  // Prevents cycle
}
```

### 2. Drop Large Data Structures

```rust
impl MyComponent {
    fn cleanup(&mut self, cx: &mut Context<Self>) {
        // Explicitly drop large structures
        self.large_cache = None;
        cx.notify();
    }
}
```

### 3. Stream Large Files

```rust
// ❌ Loads entire file
let content = fs::read_to_string(path)?;

// ✅ Stream large files
let file = File::open(path)?;
let reader = BufReader::new(file);
for line in reader.lines() {
    process_line(line?);
}
```

## Rendering Performance

### 1. Minimize Render Complexity

```rust
impl Render for MyComponent {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        // ❌ Complex computation in render
        let items = self.compute_expensive_items();

        // ✅ Compute in update, cache result
        div().children(self.cached_items.clone())
    }
}
```

### 2. Use Conditional Rendering

```rust
div()
    .when(self.should_show, |this| {
        this.child(expensive_component())
    })
```

### 3. Virtualize Long Lists

```rust
// Only render visible items
list(items)
    .with_sizing_behavior(ListSizingBehavior::Infer)
    .track_scroll(scroll_handle)
```

## Async Performance

### 1. Use background_spawn for CPU Work

```rust
// ❌ Blocks foreground thread
cx.spawn(async move |cx| {
    let result = expensive_cpu_work();  // Blocks!
    // Update UI
})

// ✅ Run on background thread
cx.background_spawn(async move {
    expensive_cpu_work()
})
```

### 2. Batch Async Operations

```rust
// ❌ Sequential
for item in items {
    process(item).await;
}

// ✅ Parallel
let futures: Vec<_> = items.iter().map(|item| process(item)).collect();
futures::future::join_all(futures).await;
```

## Measurement

Always measure before and after optimization:

```rust
let start = Instant::now();
expensive_operation();
println!("Took: {:?}", start.elapsed());
```

## Resources

- [Rust Performance Book](https://nnethercote.github.io/perf-book/)
- [Criterion Benchmarking](https://github.com/bheisler/criterion.rs)
