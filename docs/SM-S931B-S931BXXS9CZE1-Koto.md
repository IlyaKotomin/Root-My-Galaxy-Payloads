# Exact S931BXXS9CZE1 profile (experimental until hardware validation)

The general S25 payload stopped at `pipe_reclaim_cache_gate` in the owner's standalone run (24 attempts, no bootstrap root or KernelSU loading).

## Factory evidence

Downloaded only the boot member of the exact Samsung FUS archive using ranged reads:

- FUS file: `SM-S931B_2_20260509165151_c72i9n4lz1_fac.zip.enc4`
- Region: EUX; firmware: `S931BXXS9CZE1/S931BOXM9CZE1/S931BXXS9CZE1/S931BXXS9CZE1`
- Boot SHA-256: `fb49cc54523ae869e92807fd2dcc234827baad9b65557785d662ea773c810fb7`
- Boot header v4; kernel size 38,849,024 bytes.
- Embedded kernel release matches the phone exactly: `6.6.98-android15-8-pe17667d-abogkiS931BXXS9CZE1-4k`.
- `vmlinux-to-elf` recovered 114,248 symbols; BTF was read from the raw Image at `0x18748d4`.

## Changes relative to pa3q-S938NKSUACZF1

| Constant | General S25 profile | Exact factory Image |
| --- | --- | --- |
| `KMALLOC_CACHES_OFF` | `0x017da710` | `0x017da850` |
| `SLIDE_NFULNL_LOGGER_NAME_OFF` | `0x0175e2a1` | `0x0175e3d5` |

The logger name was resolved through the `nfulnl_logger` object and points to the `nfnetlink_log` string. Twenty other audited global/function locations matched (including the computed ashmem fops field and boot-id data-pointer field). The full comparison is in `native/pa1q-S931BXXS9CZE1/offset-audit.json`.

BTF confirms the profile's relevant task/rt_mutex_waiter, configfs_buffer, page/slab, file_operations, pipe_buffer and workqueue offsets. The P0 fingerprint was regenerated from this exact Image with `tools/generate_p0_fingerprint.pl` at probe offset `0x1f0000` (32 slides, 256 qwords).

The reference KernelSU module's 222 undefined symbol names are all present in this kernel. It has an empty modversions section and is manually relocated; this is a symbol-availability check, **not a complete ABI proof or safety audit**.

## Build provenance

Payload source: BuSung-dev/Root-My-Galaxy-Payloads commit `6e3223e689688540060ddd97a0927e927bfed207`.
Only the target header/fingerprint are changed. The exploit implementation and helper remain unchanged. Header overrides are included in this repository under `native/pa1q-S931BXXS9CZE1`.

Copy that directory to the source repository's `src/targets`, then run:

```sh
make TARGET=pa1q-S931BXXS9CZE1 ANDROID_NDK_HOME=/path/to/ndk/28.2.13676358 release
```

The upstream Makefile's post-link size/padding recipe uses GNU `stat` and `truncate`. On macOS compilation succeeded, then the resulting 92,072-byte ELF was checked and padded to 104,128 bytes using Python instead. The asset SHA-256 is recorded in `targets.json`.

KotoRoot now requires the full kernel release to match, including manual selection, and limits each run to three attempts. No firmware or boot partition is flashed.

## Validation

Pending below; no hardware success should be inferred solely from the static audit.
