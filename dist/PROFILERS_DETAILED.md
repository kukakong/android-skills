# Android Skills - 性能分析器模块详解

> **模块路径**: `profilers/`  
> **包含技能**: `perfetto-sql`, `perfetto-trace-analysis`  
> **更新日期**: 2026-06-01

---

## 目录

1. [模块概述](#模块概述)
2. [Perfetto SQL 技能](#perfetto-sql-技能)
3. [Perfetto Trace Analysis 技能](#perfetto-trace-analysis-技能)
4. [核心概念与数据模型](#核心概念与数据模型)
5. [常用查询模式](#常用查询模式)
6. [性能分析领域指南](#性能分析领域指南)
7. [实战案例](#实战案例)
8. [最佳实践与约束](#最佳实践与约束)

---

## 模块概述

性能分析器模块是 Android Skills 中用于深度性能分析的核心组件，基于 **Perfetto** 开源项目构建。该模块提供了两个互补的技能：

| 技能名称 | 主要职责 | 使用场景 |
|----------|----------|----------|
| **perfetto-sql** | SQL查询生成与执行 | 将自然语言转换为有效的Perfetto SQL查询 |
| **perfetto-trace-analysis** | 跟踪文件分析 | 查找延迟、内存、卡顿等问题的根本原因 |

### 架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                    性能分析器模块架构                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐         ┌─────────────────┐               │
│  │  用户请求        │         │  跟踪文件        │               │
│  │  (自然语言)      │         │  (.perfetto-trace)│              │
│  └────────┬────────┘         └────────┬────────┘               │
│           │                           │                         │
│           ▼                           │                         │
│  ┌─────────────────┐                  │                         │
│  │  perfetto-sql   │                  │                         │
│  │  ─────────────  │                  │                         │
│  │  • 意图解析      │                  │                         │
│  │  • 模式发现      │                  │                         │
│  │  • SQL生成      │                  │                         │
│  │  • 查询验证      │                  │                         │
│  └────────┬────────┘                  │                         │
│           │                           │                         │
│           ▼                           ▼                         │
│  ┌─────────────────────────────────────────────┐               │
│  │              trace_processor                 │               │
│  │  ─────────────────────────────────────────  │               │
│  │  • 查询执行引擎                              │               │
│  │  • 标准库模块                                │               │
│  │  • 数据加载                                  │               │
│  └────────────────────┬────────────────────────┘               │
│                       │                                         │
│                       ▼                                         │
│  ┌─────────────────┐                                           │
│  │  perfetto-trace │                                           │
│  │  -analysis      │                                           │
│  │  ─────────────  │                                           │
│  │  • 假设制定      │                                           │
│  │  • 数据收集      │                                           │
│  │  • 根因分析      │                                           │
│  │  • 报告生成      │                                           │
│  └─────────────────┘                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Perfetto SQL 技能

### 技能元数据

```yaml
name: perfetto-sql
description: 将自然语言数据意图转换为语法有效的Perfetto SQL查询并执行
keywords:
  - Android
  - Perfetto SQL
  - Query Guidelines
  - Performance Profiling
  - Trace Analysis
  - SQL Best Practices
  - SPAN_JOIN
  - Idempotency
```

### 执行协议

Perfetto SQL 技能遵循严格的执行协议，确保查询的正确性和可重复性：

#### Step 0: 工具设置

```bash
# 检查 trace_processor 是否存在
ls trace_processor

# 如果不存在，下载并设置
curl -LO https://get.perfetto.dev/trace_processor
chmod +x trace_processor

# 添加到 .gitignore
echo "trace_processor" >> .gitignore
```

> **⚠️ 重要**: 不要使用 `find` 等搜索工具在整个工作区搜索 `trace_processor`，这会导致超时。

#### Step 1: 解析与模式研究

1. **识别核心问题**: 确定需要查询的数据点和过滤条件
2. **模式搜索**: 在 `perfetto-stdlib.md` 中查找相关表/视图
3. **提取模式**: 获取表结构、列定义和模块导入语句

#### Step 2: 起草与验证循环

验证清单：
- [ ] SQLite语法正确
- [ ] 幂等性保证（使用 `CREATE OR REPLACE`）
- [ ] 表存在于文档中
- [ ] 列名与模式匹配
- [ ] 所有列名使用别名前缀
- [ ] 包含必要的 `INCLUDE PERFETTO MODULE` 语句
- [ ] 使用 `GLOB` 而非 `LIKE`
- [ ] SPAN_JOIN 表正确分区

#### Step 3: 执行与输出

```bash
# 执行查询
./trace_processor --query-string "QUERY" trace_file.perfetto-trace

# 或使用SQL文件
./trace_processor query_file.sql trace_file.perfetto-trace
```

---

## Perfetto Trace Analysis 技能

### 技能元数据

```yaml
name: perfetto-trace-analysis
description: 分析Perfetto跟踪以查找延迟、内存或卡顿问题的根本原因
keywords:
  - Perfetto
  - trace analysis
  - Android performance
  - debugging
  - profiling
  - jank
  - bottleneck
  - SQL
```

### 调查协议

#### 设置阶段

1. **初始化草稿文件**: 在跟踪文件同目录创建分析记录
   ```
   [trace_filename]_analysis.md
   ```

2. **审查领域提示**: 阅读各领域的分析技术
   - `hints_cpu.md` - CPU分析
   - `hints_graphics.md` - 图形分析
   - `hints_io.md` - I/O分析
   - `hints_ipc.md` - IPC分析
   - `hints_memory.md` - 内存分析
   - `hints_power.md` - 电源分析

3. **目标解析**: 如果请求宽泛，先识别活跃应用
   ```sql
   INCLUDE PERFETTO MODULE android.startup.startups;
   SELECT package FROM android_startups;
   ```

#### 调查循环

```
┌─────────────────────────────────────────┐
│           调查循环流程                    │
├─────────────────────────────────────────┤
│                                          │
│  ┌─────────────┐                        │
│  │ 1. 制定假设  │                        │
│  └──────┬──────┘                        │
│         │                               │
│         ▼                               │
│  ┌─────────────┐                        │
│  │ 2. 收集数据  │                        │
│  └──────┬──────┘                        │
│         │                               │
│         ▼                               │
│  ┌─────────────┐                        │
│  │ 3. 分析深入  │                        │
│  └──────┬──────┘                        │
│         │                               │
│         ▼                               │
│  ┌─────────────┐                        │
│  │ 4. 穷尽调查  │ ──▶ 继续循环           │
│  └──────┬──────┘                        │
│         │                               │
│         ▼                               │
│  ┌─────────────┐                        │
│  │ 5. 最终报告  │                        │
│  └─────────────┘                        │
│                                          │
└─────────────────────────────────────────┘
```

---

## 核心概念与数据模型

### 基础表结构

#### track（轨道）

轨道是trace processor中的基本概念，表示相同类型和上下文事件的"时间线"。

| 列名 | 类型 | 描述 |
|------|------|------|
| id | UINT | 唯一标识符 |
| type | STRING | 最具体的子表名称 |
| name | STRING | 轨道名称 |
| parent_id | UINT | 父轨道ID |
| machine_id | UINT | 机器标识符 |

#### thread_state（线程状态）

包含系统中每个线程的调度状态。

| 列名 | 类型 | 描述 |
|------|------|------|
| id | UINT | 唯一标识符 |
| ts | LONG | 开始时间戳（纳秒） |
| dur | LONG | 持续时间（纳秒） |
| cpu | UINT | 执行的CPU |
| utid | UINT | 线程唯一ID |
| state | STRING | 状态（Running/Runnable/Sleeping等） |

#### sched_slice（调度切片）

包含内核线程调度信息的切片。

| 列名 | 类型 | 描述 |
|------|------|------|
| id | UINT | 唯一标识符 |
| ts | LONG | 开始时间戳 |
| dur | LONG | 持续时间 |
| cpu | UINT | CPU核心 |
| utid | UINT | 线程唯一ID |
| end_state | STRING | 结束状态 |
| priority | INT | 内核优先级 |

### 标识符约定

| 标识符 | 描述 | 使用场景 |
|--------|------|----------|
| `utid` | 唯一线程ID | 连接线程相关表 |
| `upid` | 唯一进程ID | 连接进程相关表 |
| `ucpu` | 唯一CPU ID | 多机器场景 |
| `tid` | 线程ID | 可能被回收，不推荐 |
| `pid` | 进程ID | 可能被回收，不推荐 |

> **⚠️ 关键**: 始终使用 `utid`/`upid` 而非 `tid`/`pid`，因为操作系统会回收TID和PID。

---

## 常用查询模式

### 1. 查找应用启动

```sql
INCLUDE PERFETTO MODULE android.startup.startups;

SELECT 
    package,
    ts,
    dur
FROM android_startups
ORDER BY ts DESC
LIMIT 10;
```

### 2. 分析线程状态分布

```sql
SELECT 
    thread_state.state,
    COUNT(*) as count,
    SUM(IIF(thread_state.dur = -1, 
            trace_end() - thread_state.ts, 
            thread_state.dur)) / 1000000.0 as total_dur_ms
FROM thread_state
WHERE thread_state.utid = (SELECT utid FROM thread WHERE name = 'main')
GROUP BY thread_state.state
ORDER BY total_dur_ms DESC;
```

### 3. 查找长时间运行的切片

```sql
SELECT 
    slice.name,
    slice.ts,
    slice.dur / 1000000.0 as dur_ms,
    thread.name as thread_name
FROM slice
JOIN thread_track ON slice.track_id = thread_track.id
JOIN thread ON thread_track.utid = thread.utid
WHERE slice.dur > 16000000  -- > 16ms
ORDER BY slice.dur DESC
LIMIT 20;
```

### 4. CPU频率分析

```sql
INCLUDE PERFETTO MODULE linux.cpu.frequency;

SELECT 
    cpu_frequency_counters.cpu,
    cpu_frequency_counters.freq / 1000.0 as freq_mhz,
    cpu_frequency_counters.ts
FROM cpu_frequency_counters
ORDER BY cpu_frequency_counters.ts;
```

### 5. 计算切片总时间

```sql
SELECT 
    COUNT(*) as total_count,
    SUM(IIF(slice.dur = -1, 
            trace_end() - slice.ts, 
            slice.dur)) / 1000000.0 as total_dur_ms
FROM slice
WHERE slice.name GLOB '*RenderThread*';
```

### 6. 分析调度延迟

```sql
SELECT 
    thread.name,
    AVG(runnable.dur) / 1000000.0 as avg_runnable_ms,
    MAX(runnable.dur) / 1000000.0 as max_runnable_ms
FROM thread_state as runnable
JOIN thread ON runnable.utid = thread.utid
WHERE runnable.state = 'R'  -- Runnable state
GROUP BY thread.name
ORDER BY avg_runnable_ms DESC
LIMIT 10;
```

---

## 性能分析领域指南

### CPU 分析 (`hints_cpu.md`)

#### 关键技术

| 场景 | 分析方法 |
|------|----------|
| 长切片调试 | 检查线程状态（running/sleeping/blocked） |
| 延迟问题 | 递归检查子切片 |
| 唤醒延迟 | 检查CPU的IRQ轨道 |
| 内核线程 | 检查实时优先级 |
| 启动分析 | 聚合主线程不可中断睡眠原因 |
| 频率问题 | 检查cpu_frequency计数器 |
| 并发问题 | 搜索关键线程的阻塞状态 |
| 调度竞争 | 计算调度延迟（Runnable状态时长） |

#### 示例：分析调度延迟

```sql
-- 查找调度延迟最高的线程
SELECT 
    thread.name,
    COUNT(*) as runnable_count,
    SUM(IIF(ts.dur = -1, trace_end() - ts.ts, ts.dur)) / 1000000.0 as total_runnable_ms,
    MAX(ts.dur) / 1000000.0 as max_runnable_ms
FROM thread_state as ts
JOIN thread ON ts.utid = thread.utid
WHERE ts.state = 'R'
GROUP BY thread.name
ORDER BY total_runnable_ms DESC
LIMIT 10;
```

### 内存分析 (`hints_memory.md`)

#### 关键技术

| 场景 | 分析方法 |
|------|----------|
| LMK杀进程 | 查找 `lmk_kill_occurred` 事件 |
| 内存压力 | 检查 `swap_used` 值 |
| 内存颠簸 | 检查 `kswapd` 线程CPU使用 |
| 进程内存 | 检查 `anon_rss` |
| OOM分析 | 比较 median 和 p95 内存使用 |
| Bitmap分析 | 查询 `android.graphics.Bitmap` 实例 |
| DMA内存 | 查询 `dma_heap_stat` 和 `dmabuf_total_size` |

#### 示例：查找LMK杀进程事件

```sql
SELECT 
    ts,
    EXTRACT_ARG(arg_set_id, 'pid') as killed_pid,
    EXTRACT_ARG(arg_set_id, 'oom_score_adj') as oom_score
FROM slice
WHERE slice.name = 'lmk_kill_occurred'
ORDER BY ts DESC;
```

### 图形分析 (`hints_graphics.md`)

#### 关键技术

| 场景 | 分析方法 |
|------|----------|
| UI卡顿 | 检查主线程长切片（>8ms） |
| 图形内存 | 追踪 `gpu_mem_total` 计数器 |
| 大分配 | 查询 `android_graphics_allocs` 表 |
| 双倍内存 | 查找CPU和GPU同时存在的大缓冲区 |
| 帧延迟 | 比较 `actual_frame_timeline` vs `expected_frame_timeline` |
| 纹理上传 | 查找 `texture_upload` 切片 |
| 缓冲交换 | 分析 `eglSwapBuffersWithDamageKHR` 时长 |

#### 示例：检测UI卡顿

```sql
-- 查找主线程上超过8ms的切片
SELECT 
    slice.name,
    slice.ts,
    slice.dur / 1000000.0 as dur_ms
FROM slice
JOIN thread_track ON slice.track_id = thread_track.id
JOIN thread ON thread_track.utid = thread.utid
WHERE thread.name = 'main'
  AND slice.dur > 8000000  -- > 8ms
ORDER BY slice.dur DESC
LIMIT 20;
```

### I/O 分析 (`hints_io.md`)

主要关注点：
- 块设备I/O延迟
- 文件系统操作
- 存储性能瓶颈

### IPC 分析 (`hints_ipc.md`)

主要关注点：
- Binder事务延迟
- 跨进程通信开销
- 服务调用阻塞

### 电源分析 (`hints_power.md`)

主要关注点：
- 唤醒锁持有时间
- CPU活跃时间
- 电池消耗模式

---

## 实战案例

### 案例1：应用启动慢分析

**问题**: 应用启动时间过长

**分析步骤**:

1. **识别启动**
```sql
INCLUDE PERFETTO MODULE android.startup.startups;

SELECT * FROM android_startups 
WHERE package = 'com.example.app';
```

2. **分析主线程状态**
```sql
SELECT 
    state,
    SUM(dur) / 1000000.0 as dur_ms
FROM thread_state
WHERE utid = (SELECT utid FROM thread WHERE name = 'main')
  AND ts BETWEEN {startup_ts} AND {startup_ts} + {startup_dur}
GROUP BY state;
```

3. **查找阻塞原因**
```sql
-- 查找不可中断睡眠的原因
SELECT 
    slice.name,
    SUM(slice.dur) / 1000000.0 as dur_ms
FROM slice
WHERE slice.name GLOB '*blocked*'
  AND slice.ts BETWEEN {startup_ts} AND {startup_end}
GROUP BY slice.name
ORDER BY dur_ms DESC;
```

### 案例2：UI卡顿分析

**问题**: 滚动列表时出现卡顿

**分析步骤**:

1. **识别卡顿帧**
```sql
INCLUDE PERFETTO MODULE android.frames;

SELECT 
    frame.ts,
    frame.dur / 1000000.0 as frame_dur_ms
FROM actual_frame_timeline_frame as frame
WHERE frame.dur > 16600000  -- > 16.6ms
ORDER BY frame.dur DESC;
```

2. **分析卡顿原因**
```sql
-- 查找卡顿期间的主线程活动
SELECT 
    slice.name,
    slice.dur / 1000000.0 as dur_ms
FROM slice
JOIN thread_track ON slice.track_id = thread_track.id
WHERE thread_track.utid = (SELECT utid FROM thread WHERE name = 'main')
  AND slice.ts BETWEEN {jank_ts} AND {jank_ts} + {jank_dur}
  AND slice.dur > 1000000  -- > 1ms
ORDER BY slice.dur DESC;
```

### 案例3：内存泄漏分析

**问题**: 应用内存持续增长

**分析步骤**:

1. **追踪内存趋势**
```sql
SELECT 
    counter.ts,
    counter.value / 1024 / 1024 as rss_mb
FROM counter
JOIN counter_track ON counter.track_id = counter_track.id
WHERE counter_track.name = 'mem.rss'
  AND counter_track.upid = (SELECT upid FROM process WHERE name = 'com.example.app')
ORDER BY counter.ts;
```

2. **识别大对象**
```sql
-- 查找大Bitmap
SELECT 
    heap_graph_object.id,
    heap_graph_object.size_bytes
FROM heap_graph_object
WHERE heap_graph_object.type_name = 'android.graphics.Bitmap'
  AND heap_graph_object.size_bytes > 1000000  -- > 1MB
ORDER BY heap_graph_object.size_bytes DESC;
```

---

## 最佳实践与约束

### SQL编写最佳实践

#### 1. 幂等性保证

```sql
-- ✅ 正确：使用 CREATE OR REPLACE
CREATE OR REPLACE PERFETTO TABLE my_table AS
SELECT * FROM slice;

-- ✅ 正确：先删除再创建虚拟表
DROP TABLE IF EXISTS my_span_join;
CREATE VIRTUAL TABLE my_span_join USING SPAN_JOIN(t1, t2);

-- ❌ 错误：缺少幂等性保证
CREATE TABLE my_table AS
SELECT * FROM slice;
```

#### 2. 字符串匹配

```sql
-- ✅ 正确：使用 GLOB
WHERE slice.name GLOB '*RenderThread*'

-- ✅ 正确：精确匹配使用 =
WHERE slice.name = 'Choreographer#doFrame'

-- ✅ 正确：大小写不敏感
WHERE LOWER(slice.name) GLOB '*renderthread*'

-- ❌ 错误：使用 LIKE
WHERE slice.name LIKE '%RenderThread%'
```

#### 3. 处理不完整切片

```sql
-- ✅ 正确：处理 dur = -1 的情况
SELECT 
    SUM(IIF(slice.dur = -1, 
            trace_end() - slice.ts, 
            slice.dur)) / 1000000.0 as total_ms
FROM slice;

-- ❌ 错误：未处理不完整切片
SELECT SUM(slice.dur) / 1000000.0 as total_ms
FROM slice;
```

#### 4. 列名别名

```sql
-- ✅ 正确：使用表别名前缀
SELECT slice.name, slice.dur, thread.name
FROM slice
JOIN thread ON slice.utid = thread.utid;

-- ❌ 错误：缺少别名
SELECT name, dur, name
FROM slice
JOIN thread ON slice.utid = thread.utid;
```

### SPAN_JOIN 使用约束

```sql
-- ✅ 正确：使用 PARTITIONED 防止重叠
CREATE PERFETTO TABLE t1 AS
SELECT * FROM slice WHERE name = 'foo';

DROP TABLE IF EXISTS span_result;
CREATE VIRTUAL TABLE span_result USING SPAN_JOIN(
    t1 PARTITIONED utid,
    t2 PARTITIONED utid
);

-- ❌ 错误：未分区可能导致崩溃
CREATE VIRTUAL TABLE span_result USING SPAN_JOIN(t1, t2);
```

### 性能约束

| 约束 | 原因 | 解决方案 |
|------|------|----------|
| 不使用 `LIKE` | 性能瓶颈 + 下划线通配符bug | 使用 `GLOB` |
| 不搜索整个工作区 | 超时风险 | 直接检查已知路径 |
| 不跨turn共享状态 | 数据库状态不持久 | 每个查询自包含 |
| 不手动计算时间重叠 | 复杂且易错 | 使用标准库模块 |

### 调试技巧

1. **查询失败时**: 检查语法、表名、列名
2. **结果为空时**: 放宽过滤条件、使用模糊匹配
3. **性能问题时**: 使用更精确的时间窗口
4. **数据不一致时**: 检查标识符使用（utid vs tid）

---

## 快速参考卡

### 常用模块导入

```sql
INCLUDE PERFETTO MODULE android.startup.startups;     -- 应用启动
INCLUDE PERFETTO MODULE android.frames;               -- 帧分析
INCLUDE PERFETTO MODULE linux.cpu.frequency;          -- CPU频率
INCLUDE PERFETTO MODULE sched.runnable;               -- 调度分析
INCLUDE PERFETTO MODULE intervals.overlap;            -- 区间重叠
```

### 时间转换

```sql
ns → ms:  value / 1000000.0
ns → us:  value / 1000.0
ms → ns:  value * 1000000
```

### 状态码

| 状态 | 含义 |
|------|------|
| R | Runnable（可运行） |
| S | Sleeping（睡眠） |
| D | Uninterruptible Sleep（不可中断睡眠） |
| T | Suspended（暂停） |
| X | Exiting（退出中） |
| Z | Zombie（僵尸进程） |

---

*本文档详细描述了 Android Skills 性能分析器模块的使用方法*
