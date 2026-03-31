---
layout: page
---

<script setup>
if (typeof window !== 'undefined') {
  const lang = navigator.language || navigator.languages?.[0] || ''
  const target = lang.startsWith('zh') ? '/zh/' : '/en/'
  if (!window.location.pathname.replace(/\/$/, '').endsWith(target.replace(/\/$/, ''))) {
    window.location.replace(target)
  }
}
</script>

# Learn Harness Engineering

Redirecting...

<template>
<p>Choose your language / 选择语言：</p>
<ul>
  <li><a href="/zh/">中文</a></li>
  <li><a href="/en/">English</a></li>
</ul>
</template>
