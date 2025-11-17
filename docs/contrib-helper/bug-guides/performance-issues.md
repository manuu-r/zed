# Performance Issues

Guide for profiling and optimizing Zed.

## Profiling

### CPU Profiling (macOS)

```bash
# Build with optimizations
cargo build --release

# Run with Instruments
instruments -t "Time Profiler" target/release/zed
```

### CPU Profiling (Linux)

```bash
# Install perf
sudo apt install linux-tools-generic

# Profile
perf record -g target/release/zed
perf report
```

### Memory Profiling

```bash
# Use heaptrack (Linux)
heaptrack target/release/zed

# Analyze
heaptrack_gui heaptrack.zed.*.gz
```

## Common Issues

### Slow Rendering

**Causes:**
- Too many re-renders
- Expensive render functions
- Large element trees

**Solutions:**
- Use `when()` for conditional rendering
- Memoize expensive computations
- Batch updates with `transact()`

### Memory Leaks

**Causes:**
- Circular entity references
- Undetached subscriptions
- Undetached tasks

**Solutions:**
```rust
// Use WeakEntity to break cycles
struct MyType {
    other: WeakEntity<OtherType>,
}

// Detach tasks that should run independently
task.detach();

// Store subscriptions to clean up
struct MyComponent {
    _subscriptions: Vec<Subscription>,
}
```

## Optimization Tips

### 1. Batch Updates

```rust
// ❌ Multiple notifications
for item in items {
    self.process(item);
    cx.notify();
}

// ✅ Single notification
for item in items {
    self.process(item);
}
cx.notify();
```

### 2. Avoid Allocations

```rust
// ❌ Allocates each time
format!("Value: {}", value)

// ✅ Reuse buffer
let mut buffer = String::new();
write!(&mut buffer, "Value: {}", value)
```

### 3. Use Appropriate Data Structures

```rust
// For lookups: HashMap
// For ordered iteration: Vec
// For frequent inserts/removals: VecDeque or linked list
```

## Resources

- [Rust performance book](https://nnethercote.github.io/perf-book/)
- [GPUI performance tips](../../../crates/gpui/README.md)
