# Frontend Research

## React and TypeScript

**Verified:** React documents component-based UI and current React documentation at [react.dev](https://react.dev/); its versions page should be consulted at dependency selection. TypeScript provides static type checking but does not validate untrusted runtime data ([TypeScript handbook](https://www.typescriptlang.org/docs/handbook/intro.html)).

**Recommendation:** use function components and feature-oriented folders (catalog, search, title detail, playback, account, admin), with shared design primitives and explicit API DTO types. Parse/validate data at trust boundaries; generated OpenAPI client types can reduce contract drift but do not replace runtime validation. Keep UI state local, URL state navigable, and server state in the query cache.

**Risks:** `any`, unsafe casts, and type assertions can erase guarantees. React state is a render snapshot; avoid mutating state objects. React Strict Mode development behavior can expose unsafe effects; effects should have cleanup and not be the default data-fetching abstraction.

## Routing and server state

React Router is the route/navigation layer; select a documented mode (Declarative, Data, or Framework) intentionally and avoid mixing patterns. TanStack Query is for remote asynchronous state (query caching, invalidation, retries); it is not a global UI state store. Set query keys from all response-affecting inputs, use deliberate stale/cache times, and invalidate or update after mutations. Do not retry authorization failures or non-idempotent mutations blindly.

References: [React Router docs](https://reactrouter.com/home), [TanStack Query React docs](https://tanstack.com/query/latest/docs/framework/react/overview).

## HLS.js player

HLS.js feeds HLS into a normal HTML `<video>` element using Media Source Extensions; playback controls remain standard `HTMLMediaElement` APIs. Detect native HLS first where appropriate; otherwise use `Hls.isSupported()`, construct/load/attach, and destroy on source/component teardown. Handle fatal errors and distinguish network recovery from media recovery; surface terminal errors. Do not assume all HLS tags, codecs, DRM systems, or low-latency features behave identically across hls.js and native implementations.

References: [hls.js README](https://github.com/video-dev/hls.js), [API docs](https://hlsjs-dev.video-dev.org/api-docs/).

## Accessibility and performance

Use native video controls as the dependable baseline, provide keyboard-operable custom controls only when fully implemented, captions/subtitles via WebVTT where available, accessible labels, focus management, reduced motion, and non-color-only status. Validate with keyboard and screen reader testing against WCAG 2.2 AA target; this target is a recommendation, not a verified compliance claim. Lazy-load noncritical routes/images, use responsive poster images, avoid fetching manifests before user intent when product design allows, and measure Core Web Vitals on representative devices.

References: [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/), [MDN Web video text tracks](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track).

## Frontend security

Treat catalog metadata, descriptions, artwork URLs, subtitles, and playback URLs as untrusted. Render text escaped; avoid raw HTML. Apply CSP and secure headers at the edge, allowlist media origins, avoid tokens in query strings when possible, and do not put long-lived credentials in localStorage. Browser CORS is not authorization. The backend must authorize every protected API/playback request. Prevent open redirects and validate URL schemes before linking.
