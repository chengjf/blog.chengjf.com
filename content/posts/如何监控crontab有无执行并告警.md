+++
title = '如何监控crontab有无执行并告警'
date = 2019-05-14T13:40:36+08:00
draft = false
comment = true
tags = ["Linux"]
categories = "运维"
+++

首先，crontab要执行的任务要有输出日志，表明任务执行过。

然后，另起一个crontab任务，去监控上面的这个日志，比较日志的更新时间，判断任务是否执行，从而提醒。

下面是我在实际生产环境中使用的一个脚本示例：

```bash
timestamp() {
  date +"%s"
}
url="http://报警url"

last_mofidy_timestamps="$(stat -c %Y /home/example/example.log)"
echo $last_mofidy_timestamps

current_timestamps="$(timestamp)"
echo $current_timestamps

if (($current_timestamps - $last_mofidy_timestamps > 3600)); then
    curl $url
fi
```