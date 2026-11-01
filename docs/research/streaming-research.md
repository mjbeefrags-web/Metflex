# Streaming and Media Research

## HLS

HLS is a playlist-and-segment protocol delivered over ordinary HTTP/CDN. RFC 8216 (2017) specifies protocol version 7 and is informational; Apple publishes the newer HLS 2nd Edition specification and device authoring requirements. Follow the current Apple authoring profile for Apple compatibility and validate streams with Apple tooling where available. HLS playlists enumerate resources: every referenced manifest, segment, key, and subtitle must be served with correct MIME/CORS/cache behavior and valid authorization.

References: [RFC 8216](https://www.rfc-editor.org/rfc/rfc8216), [Apple HLS hub](https://developer.apple.com/streaming/), [Apple authoring specification](https://developer.apple.com/documentation/http-live-streaming/hls-authoring-specification-for-apple-devices/).

## Transcoding and ABR

FFmpeg's HLS muxer emits playlists and segments; its docs advise closed GOP and GOP size aligned to segment duration. Align keyframes across renditions for reliable switching. Create a small measured ladder rather than adopting generic bitrates blindly; preserve source frame rate where justified, cap resolutions to source/device need, encode a compatible baseline rendition, and validate measured bandwidth/codec declarations. Benchmark CPU/GPU throughput, storage, job duration, and quality before setting worker concurrency.

Reference: [FFmpeg formats, HLS muxer](https://ffmpeg.org/ffmpeg-formats.html).

## Workflow

1. Ingest only from a source with documented authority and rights scope.
2. Validate container/codecs/size/duration; probe in a resource-isolated process.
3. Persist immutable source identity/hash and licensing evidence.
4. Transcode to staged object prefixes; generate thumbnails, playlists, subtitles/metadata as applicable.
5. Validate output playlist, codecs, segment continuity, durations, and sample playback.
6. Promote atomically by updating a database asset/version pointer; retain rollback/versioning policy.
7. Deliver via CDN; monitor origin errors, cache hit ratio, startup time, rebuffering, and egress.

## Storage, CDN, and signed delivery

Keep origin buckets private and expose through CDN. Use immutable versioned segment names and long cache TTLs; manifests may need shorter TTL or tokenized access. Authorize at manifest entry and ensure child segment requests remain protected (signed cookies, tokenized path/query, or CDN-native mechanism). A signed URL is a bearer credential: scope it narrowly, expire it, avoid logging it, and understand URL leakage via browser history/referrer. Per-object presigned URLs for every HLS segment can produce poor ergonomics and token management; signed cookies/tokenized path can fit a whole asset path but depend on CDN/provider. No provider is selected, so exact behavior, costs, limits, and configuration remain undecided.

## Subtitles

WebVTT is broadly supported by HTML `<track>` for captions/subtitles; include language, label, kind, and default policy, and verify cross-browser HLS subtitle rendition behavior. Keep subtitle files authorized and validate encodings/size. Preserve caption accessibility and do not burn captions into video by default.

References: [W3C WebVTT](https://www.w3.org/TR/webvtt1/), [MDN `<track>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track).

## DRM and lawful distribution

HLS supports encryption mechanisms; encrypted transport alone is not robust commercial DRM. FairPlay Streaming and corresponding platform DRM require Apple/other vendor program participation, licensing, key management, device compatibility, and contractual rights. Do not implement DRM circumvention. Distribution authority, territorial scope, term, attribution, takedown, and public-domain status must be established per title and jurisdiction. No legal conclusion is made here.
