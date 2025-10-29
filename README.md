# IsArrayBuffer

## Motivation

Given an input value, and passing it to `new Uint8Array(value, byteOffset, length)`, the behavior
is different depending on whether the input value is an `ArrayBuffer` or not.

- If the input value is an `ArrayBuffer` (or `SharedArrayBuffer`), the `byteOffset` and `length` are
  used to create a view on the buffer.
- If the input value is not an `ArrayBuffer`, it is treated as an iterable, and the `byteOffset` and
  `length` arguments are ignored.

However, checking if a value is an `ArrayBuffer` is not straightforward.

```js
// ❌ Not reliable across realms
function isArrayBuffer (value) {
  return value instanceof ArrayBuffer
}

// ❌ Symbol.toStringTag can be faked
function isArrayBuffer(value) {
  return Object.prototype.toString.call(value) === '[object ArrayBuffer]';
}

// ❌ byteLength can be faked, and `Uint8Array` ignores arbitrary objects with a `byteLength` property
function isArrayBuffer(v) {
  return typeof v === 'object' && v !== null && v.byteLength !== undefined;
}

// ✅ Reliable check, but verbose
function isArrayBuffer(value) {
  try {
    Object.getOwnPropertyDescriptor(ArrayBuffer.prototype, byteLength).get.call(value);
    return true;
  } catch {
    return false;
  }
}
```

## Proposed solution

Add a built-in function `ArrayBuffer.isArrayBuffer(value)` that returns `true` if `value` is an `ArrayBuffer`,
and `false` otherwise. And similarly, `SharedArrayBuffer.isSharedBuffer(value)`.

```js
ArrayBuffer.prototype.isArrayBuffer = function isArrayBuffer(value) {
  return [[ArrayBufferData]] in value && !isSharedBuffer(value);
}

SharedArrayBuffer.prototype.isSharedBuffer = function isSharedBuffer(value) {
  return [[ArrayBufferData]] in value && [[ArrayBufferData]] is shared;
}
```

## Usages

[Code Search](https://github.com/search?type=code&q=isarraybuffer+%28lang%3AJavaScript+OR+lang%3ATypeScript+%29)

