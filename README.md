# 简述

>python作为一门解释型脚本语言是没办法做到像C、java 等语言那样编译发布的，所以代码基本是开源的，如作为商业用途必须对其进行加密。
现行环境是主机的C程序通过导入python脚本的方式进行调用。这样的缺点是python必须会以源码的文件出现在主机上。目前使用的生成pyc文件并不

>是有效的解决保密手段。因为pyc只是对py文件文件序列化编译成二进制的文件，通过python marshal模块就可将其恢复文本运行，其实反向还原源码

>是相当容易的。本文时候说明对Python进行更高级的加密方法。

## 步骤

### 加密步骤

首先编译源码生成Pyc文件
py3.6以二进制 读取pyc文件并偏移12位（去头部）
使用zlib压缩文本
在文件头加入自定义字节（干扰）
使用a85或者b16编码
生成密文保存云端
解密步骤

导入文本
逆向解编码
去掉干扰字节
使用python 内建zlib库解压
使用python marshal 导入解压文本运行
使用exec直接运行文本 exec(import("marshal").load(str))
python演示

源代码



```python
import os
print(os.path)
print("running success!")
```
#### 加密脚本

```python
import zlib
import base64
import os

with open(os.path.dirname(__file__)+"/t36.pyc","rb") as f:
    f.seek(12)
    metaData = f.read()

zData = b"\x00\x02"+zlib.compress(metaData)
bData = base64.b85encode(zData)

with open(os.path.dirname(__file__)+"/testMi.pd","wb") as f:
    f.write(bData)

print("密文:",bData)
```

加密后:
```text
c$|C8fCQL;*a3)(<$y#ALkeRKLli?QV=6-yQ!^tYkd*?aQy7C8G?`z5bof;X6qV-XW#*+T6qhC^rxq70YBJwq$}hgfT2PdkS8|J`Ah9H4B|{M_P%)VJCFf!llayMTni3P7T$EW*662beTvS<55>OdaqE}FPi^C>2KczG$)s7M55*8rA!N|n~0L5P^a{
```

#### 解密/运行脚本
```python
import os

data = "c$|C8fCQL;*a3)(<$y#ALkeRKLli?QV=6-yQ!^tYkd*?aQy7C8G?`z5bof;X6qV-XW#*+T6qhC^rxq70YBJwq$}hgfT2PdkS8|J`Ah9H4B|{M_P%)VJCFf!llayMTni3P7T$EW*662beTvS<55>OdaqE}FPi^C>2KczG$)s7M55*8rA!N|n~0L5P^a{"
exec(__import__("marshal").loads(__import__("zlib").decompress(__import__("base64").b85decode(data)[2:])))
```

### 原理

实际上就是对pyc文件进行多重编码，破译者无法真正知道该怎样去解码，因为解码的过程在C语言中，对python 的破译相当于对C程序的反编译。
而且就算知道根据编码特征知道编码方式，但是在其中加入干扰也会导致解码失败。所以这种加密方式破解难度很大。
