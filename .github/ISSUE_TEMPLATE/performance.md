---
name: "⚡ Performance Issue"
about: "Report video streaming latency, query slowdowns, or transcoding bottlenecks"
title: "[PERF]: "
labels: ["performance", "needs-triage"]
assignees: ""
---

## ⚡ Performance Bottleneck Summary
<!-- Briefly explain where the performance degradation or slowdown was observed. -->

## ⏱️ Observed Metrics
- **Current Metric:** <!-- e.g. 5-second video buffer delay, 1200ms API response -->
- **Expected / Target Metric:** <!-- e.g. <1.5s video startup, <200ms p95 API response -->

## 🔄 Reproduction Steps
1. Navigate to '...'
2. Trigger action '...'
3. Observe slowdown / high resource usage

## 🖥️ System & Resource Details
- **Browser / Client:** <!-- e.g. Chrome 124, Firefox 125 -->
- **Device / OS:** <!-- e.g. Windows 11, macOS M2 -->
- **Network Speed:** <!-- e.g. 100 Mbps, 4G, throttled 3G -->
- **Component:** [ ] Frontend HLS.js Player / [ ] Backend FastAPI / [ ] Celery FFmpeg Worker / [ ] PostgreSQL Query / [ ] Redis Cache

## 📊 Profiling Data / Logs
```text
<!-- Paste Network waterfall, Lighthouse score, FFmpeg logs, or SQL query EXPLAIN ANALYZE here -->
```

## 💡 Potential Optimization Ideas
<!-- Ideas or references on how to resolve the bottleneck -->
