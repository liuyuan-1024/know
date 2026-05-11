- **case-sensitive 大文件 streaming**：当前仍走 `slice_text(search_range)` 物化 haystack 喂给 `str::find`。要做到完全流式，需要 KMP/Boyer-Moore 跨 chunk 状态机——工程复杂度高于收益（case-insensitive 是 OOM 主因，case-sensitive 至少是 1× 而非 3×）。延后到独立 phase。
- **regex 大文件 streaming**：`regex` crate 要求连续 `&str`，跨 chunk 流式 regex 需要 `regex_automata` 直接驱动 DFA——同样属于独立大工程。
- **Locale-aware Unicode case folding**：Rust `char::to_lowercase` 是 Unicode 默认折叠，未处理土耳其 dotless I / 希腊终末 sigma 等 locale 特例。需引 `caseless` crate；属配置策略层，与本 Phase 解耦。

### 暂未做的部分（保留给后续）

- **`lib.rs` 暴露面砍到 15-20 个核心类型**：当前仍暴露 ~120 个类型，主要是 `projection::*` 的 14 个内部类型 + tracking/metadata/versioned 的细分类型。改成 `pub mod projection { ... }` 命名空间分层会破坏所有现有调用方（tests / examples），属于"API 冻结"级别的独立工程，单独 phase 处理更合理。
- **`zom_ffi` 子 crate**：FFI 边界二次封装。当前的 `#[repr(transparent)]` 已经让 ID 类型可直接跨 FFI；公共类型（`Buffer`、`Snapshot` 等）的 ABI 封装等到真正接入 C/Swift/JS 宿主时再做。