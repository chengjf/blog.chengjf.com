+++
title = '从阿里云bucket下载整个文件夹下的文件'
date = 2023-07-31T12:37:36+08:00
draft = false
comment = true
tags = ["Python", "脚本", "OSS", "阿里云"]
categories = "编程技术"
+++

首先，先安装里云oss的lib：


> pip install -i http://mirrors.aliyun.com/pypi/simple/ oss2

脚本如下：

```python
import oss2
from itertools import islice
import shutil

accessKeyId = "access_key_id"
accessKeySecret = "access_key_secret"

auth = oss2.Auth(accessKeyId, accessKeySecret)

# auth = oss2.Auth(accessKeyId, accessKeySecret)
bucket = oss2.Bucket(auth, "https://your.endpoint.com/", "your bucket name")

count = 0
for obj in oss2.ObjectIterator(bucket, prefix = "your/path"):
    print(obj.key)
    count = count + 1
    # if count > 100:
    #     break
    print(count)
    object_stream = bucket.get_object(obj.key)
```