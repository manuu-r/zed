# Debugging Crashes and Panics

Guide for tracking down and fixing crashes in Zed.

## Common Crash Causes

### 1. Unwrap on None/Err

**Symptom:** `thread 'main' panicked at 'called \`Option::unwrap()\` on a \`None\` value'`

**Solution:**
```rust
// ❌ Causes panic
let value = option.unwrap();

// ✅ Proper error handling
let value = option.ok_or_else(|| anyhow!("Missing value"))?;
```

### 2. Index Out of Bounds

**Symptom:** `index out of bounds: the len is 5 but the index is 10`

**Solution:**
```rust
// ❌ Can panic
let item = vec[index];

// ✅ Safe access
let item = vec.get(index).ok_or_else(|| anyhow!("Invalid index"))?;
```

### 3. Entity Borrow Conflicts

**Symptom:** `already borrowed: BorrowMutError`

**Solution:**
```rust
// ❌ Nested borrows
entity.update(cx, |e, cx| {
    let other = e.field.read(cx);  // Borrow
    e.method();  // Error if method tries to borrow again
});

// ✅ Clone before update
entity.update(cx, |e, cx| {
    let value = e.field.read(cx).value.clone();
    e.method_with_value(value);
});
```

## Debugging Steps

### 1. Get Stack Trace

```bash
RUST_BACKTRACE=1 cargo run
# or
RUST_BACKTRACE=full cargo run
```

### 2. Add Debug Logging

```rust
eprintln!("Before operation: {:?}", state);
perform_operation();
eprintln!("After operation: {:?}", state);
```

### 3. Use Debugger

```bash
rust-lldb target/debug/zed
# Set breakpoint
(lldb) breakpoint set --name my_function
(lldb) run
```

## Resources

- [Rust panic handling](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html)
