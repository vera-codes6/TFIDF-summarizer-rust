
![project-banner](https://github.com/shubham0204/tfidf-summarizer-rs/assets/41076823/f2855bcb-9573-4b70-9c38-7d4120511896)

# TF‑IDF Text Summarizer (Rust)

[![Crate](https://img.shields.io/crates/v/tfidf-text-summarizer.svg)](https://crates.io/crates/tfidf-text-summarizer)
[![Docs](https://docs.rs/tfidf-text-summarizer/badge.svg)](https://docs.rs/tfidf-text-summarizer)

> Extractive summarization using TF‑IDF to rank sentences. Parallel version included.

Do read the blog: Building a cross‑platform TF‑IDF text summarizer in Rust — https://medium.com/towards-data-science/building-a-cross-platform-tfidf-text-summarizer-in-rust-7b05938f4507

![tfidf_repo_image](https://github.com/shubham0204/tfidf-summarizer.rs/assets/41076823/09c06763-0098-49d0-b9fe-ca7e9995527e)

Contents

1. Usage
   - Rust
   - C/C++ (plus Debian package)
2. Contributing
3. Resources

## Usage

### Rust

This crate builds on stable Rust.

Add to Cargo.toml:

```toml
[dependencies]
tfidf-text-summarizer = "0.0.3"
```

API:
- `summarize(text: &str, reduction_factor: f32) -> String`
- `par_summarize(text: &str, reduction_factor: f32) -> String` (uses Rayon)

`reduction_factor` is the fraction of sentences to keep. If a document has 20 sentences and `reduction_factor = 0.4`, the summary contains the top 8 sentences.

Example:

```rust
use summarizer::{summarize, par_summarize};
use std::fs;

fn main() {
    let text = fs::read_to_string("wiki.txt").expect("read wiki.txt");
    let summary = summarize(&text, 0.4);
    println!("{}", summary);
}
```

Notes:
- Sentence splitting uses the Punkt tokenizer (`punkt`).
- Stop‑words are filtered before scoring.
- `par_summarize` parallelizes tokenization and scoring with Rayon.

### C/C++ (FFI)

This library exposes C‑ABI functions:

```c
const uint8_t* summarize(const uint8_t* text, uintptr_t length, float reduction_factor);
const uint8_t* par_summarize(const uint8_t* text, uintptr_t length, float reduction_factor);
```

Generate a header with cbindgen:

```
cargo install --force cbindgen
cbindgen --lang C --output examples/c/summarizer.h
```

Build a static/shared library (Linux example):

```
cargo build --release --target x86_64-unknown-linux-gnu
```

Compile and link with GCC (see `examples/c`):

```
gcc examples/c/main.c -Iexamples/c -Ltarget/x86_64-unknown-linux-gnu/release -lsummarizer -lpthread -lm -ldl -o main
```

Minimal C example (excerpt from `examples/c/main.c`):

```c
const char* summarized_text = (const char*) summarize((const uint8_t*)buffer, size, 0.5f);
printf("%s\n", summarized_text);
```

Debian package: a simple example is under `examples/debian/`. Copy the built library (`libsummarizer.a` or `libsummarizer.so`) and `summarizer.h` into `examples/debian/summarizer/`, then run:

```
cd examples/debian
bash build_package.sh
```

Adjust the `postinst` script to match the artifact you package (`.a` vs `.so`).

### Android (JNI)

JNI bindings are behind the optional `android` feature. Exported methods in `src/lib.rs`:
- `Java_com_projects_ml_summarizer_Summarizer_summarize`
- `Java_com_projects_ml_summarizer_Summarizer_parallelSummarize`

Build example (NDK/targets required):

```
cargo build --release --features android --target aarch64-linux-android
```

## Contributing

- Replace or complement TF‑IDF with graph‑based or contextual scoring.
- Benchmark on standard datasets (ROUGE‑1/2/L).
- Improve FFI ergonomics and memory‑management helpers.
- Polish the Python ctypes example under `examples/python`.
