# Python Tips

## 介绍
Python 自带了很多好用的工具和有意思的小技巧，这些‘小’而实用的技巧，可以使用一行代码或者一条命令，带来很多便利。

# 小技巧
Python 代码中的便利的代码片段。

## Mac 下发送通知
```python
import os

def notify(title, text):
	# 调用 mac osascript 命令
    os.system("""
              osascript -e 'display notification "{}" with title "{}"'
              """.format(text, title))

notify("通知", "三年二班xxx，到教导处来一趟。")
```

## print 输出到文件中
```
print >>  file("myfile", "w"), "hello world"
```

## 变量值交换
```
a = 1
b = 2
a, b = b, a # a=2, b=1
```


# 小工具
Python 自带电池（battery included），直接通过命令行使用。

## json 格式化工具
```
echo '{"foo": "lorem", "bar": "ipsum"}' | python -m json.tool
curl https://randomuser.me/api/ | python -m json.tool
```

## 本地启动 server（可用于文件下载）
```
python -m SimpleHTTPServer 8000
```
会直接在机器上起一个 FTP 服务，内容为当前目录下的内容。

## 使用默认浏览器打开url
```
python -m webbrowser
参数：
-n open new window
-t open new tab
url target url
```
