---
title: 첫 게시물 테스트
published: 2025-06-27
description: 'This is a test'
image: ''
tags: [랑, init]
category: 'init'
draft: false 
lang: 'kr'
---

:::important
test important
:::

# intro

Hi hello, I'm starting my tech blog.

## I don't know

What should I write?

# Let's go

## hot weather

hihi

### Text & Line Markers

[Text & Line Markers](https://expressive-code.com/key-features/text-markers/)

#### Marking full lines & line ranges

```js {1, 4, 7-8}
// Line 1 - targeted by line number
// Line 2
// Line 3
// Line 4 - targeted by line number
// Line 5
// Line 6
// Line 7 - targeted by range "7-8"
// Line 8 - targeted by range "7-8"
```

```jsx {"1":5} del={"2":7-8} ins={"3":10-12}
// labeled-line-markers.jsx
<button
  role="button"
  {...props}
  value={value}
  className={buttonClassName}
  disabled={disabled}
  active={active}
>
  {children &&
    !active &&
    (typeof children === 'string' ? <span>{children}</span> : children)}
</button>
```
