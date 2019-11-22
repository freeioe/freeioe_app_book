
---

# 通讯模块超时功能封装

将通讯模块的request函数从request(msg, response, padding) 扩展成为 request(msg, response, padding, timeout)

并在指定timeout时，request会在超时后返回而不是默认的永远等待下去。


