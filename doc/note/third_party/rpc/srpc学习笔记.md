# srpc 学习笔记
<!-- TOC -->

- [srpc 学习笔记](#srpc-学习笔记)
  - [1. Q&A](#1-qa)
    - [1.1. 加锁不生效](#11-加锁不生效)

<!-- /TOC -->

## 1. Q&A

### 1.1. 加锁不生效

```c++
//! 将 fix 会话的延迟衰减到原来的 80%，并更新到db
void FixDelayAttenuation::attenuate()
{
    // 在这里加锁主要有两个原因：
    //    1. 保证只有一个线程触发衰减逻辑，避免多个线程同时触发 fix 延迟衰减，造成重复写 DB 的情况（存在多个线程同时调用 timeout_work_fix 的情况）
    //    2. 保证多个线程同步更新 fix 延迟
    std::lock_guard<std::mutex> lock(m_mutex_cache_fix_delay);
    for (MapFixDelayAttenuation::iterator it = m_session_fix_delay_atten.begin(); it != m_session_fix_delay_atten.end(); ++it)
    {
        //! 检查 fix 会话延迟的更新时间，超过指定阈值没有更新后（该会话可能因为延迟过高，导致没有被作为可用路由返回给inbound，从而导致一直没有数据）
        if (it->second.is_out_of_date())
        {
            FLOG_DEBUG("fix delay info is out of date: thid=" << std::this_thread::get_id() << ", " << it->second.short_debug_string());
            // ...
        }
    }
}
```

启动后，查看日志，发现打印了多次日志，并且线程id还不一样

```txt
[2022-07-22 03:54:58.927928] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140055729403648, prev_update_time=1658476488, fix_delay_info=fix_delay_time: 4500 fix_no_rsp_ratio: 60
[2022-07-22 03:54:58.932979] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=139977346250496, prev_update_time=1658476488, fix_delay_info=fix_delay_time: 4500 fix_no_rsp_ratio: 60
[2022-07-22 03:54:58.934860] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140195509143296, prev_update_time=1658476488, fix_delay_info=fix_delay_time: 4500 fix_no_rsp_ratio: 60
[2022-07-22 03:55:08.938889] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140055729403648, prev_update_time=1658476498, fix_delay_info=fix_delay_time: 3600 fix_no_rsp_ratio: 48
[2022-07-22 03:55:08.943887] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=139977346250496, prev_update_time=1658476498, fix_delay_info=fix_delay_time: 3600 fix_no_rsp_ratio: 48
[2022-07-22 03:55:08.947866] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140195509143296, prev_update_time=1658476498, fix_delay_info=fix_delay_time: 3600 fix_no_rsp_ratio: 48
[2022-07-22 03:55:18.950925] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140055729403648, prev_update_time=1658476508, fix_delay_info=fix_delay_time: 2880 fix_no_rsp_ratio: 38
[2022-07-22 03:55:18.954905] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=139977346250496, prev_update_time=1658476508, fix_delay_info=fix_delay_time: 2880 fix_no_rsp_ratio: 38
[2022-07-22 03:55:18.960870] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140195509143296, prev_update_time=1658476508, fix_delay_info=fix_delay_time: 2880 fix_no_rsp_ratio: 38
```

问题原因：srpc配置错误，开启了多个进程，所以加锁失效，能同时进入到该逻辑，通过在日志输出进程 id 即可发现问题：

```txt
[2022-07-22 03:54:58.927928] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140055729403648, pid=2378, prev_update_time=1658476488, fix_delay_info=fix_delay_time: 4500 fix_no_rsp_ratio: 60
[2022-07-22 03:54:58.932979] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=139977346250496, pid=2382, prev_update_time=1658476488, fix_delay_info=fix_delay_time: 4500 fix_no_rsp_ratio: 60
[2022-07-22 03:54:58.934860] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140195509143296, pid=2380, prev_update_time=1658476488, fix_delay_info=fix_delay_time: 4500 fix_no_rsp_ratio: 60
[2022-07-22 03:55:08.938889] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140055729403648, pid=2378, prev_update_time=1658476498, fix_delay_info=fix_delay_time: 3600 fix_no_rsp_ratio: 48
[2022-07-22 03:55:08.943887] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=139977346250496, pid=2382, prev_update_time=1658476498, fix_delay_info=fix_delay_time: 3600 fix_no_rsp_ratio: 48
[2022-07-22 03:55:08.947866] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140195509143296, pid=2380, prev_update_time=1658476498, fix_delay_info=fix_delay_time: 3600 fix_no_rsp_ratio: 48
[2022-07-22 03:55:18.950925] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140055729403648, pid=2378, prev_update_time=1658476508, fix_delay_info=fix_delay_time: 2880 fix_no_rsp_ratio: 38
[2022-07-22 03:55:18.954905] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=139977346250496, pid=2382, prev_update_time=1658476508, fix_delay_info=fix_delay_time: 2880 fix_no_rsp_ratio: 38
[2022-07-22 03:55:18.960870] [DEBUG][mq_sub.cpp][325][attenuate][1]: fix delay info is out of date: thid=140195509143296, pid=2380, prev_update_time=1658476508, fix_delay_info=fix_delay_time: 2880 fix_no_rsp_ratio: 38
```

---

**srpc 服务的配置**：

```yaml
global:
  listen:
    - address: 0.0.0.0:14005/tcp            # 服务监听的端口与所用的网络协议
  timeout: 700                              # 连接超时时间
service:
  - id: 1
    module: ../client/bin/futc_ems_write.so # 加载的模块（业务逻辑所在）
    conf: ../client/conf/config.ini         # 配置文件路径
    procnum: 4  # 开启的进程数
    flog:
      level: 3  # 远程日志等级（该远程日志不一定时 srpc 的功能，可能是扩展出来的功能）
    log:
      level: 3  # 本地日志等级
    packetdump: off # 是否输出dump信息

```
