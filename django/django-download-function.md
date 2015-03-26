#Django 实现下载文件功能

基于Django建立的网站，如果提供文件下载功能，最简单的方式莫过于将静态文件交给Nginx等处理，但有些时候，由于网站本身逻辑，需要通过Django提供下载功能，如页面数据导出功能（下载动态生成的文件）、先检查用户权限再下载文件等。因此，有必要研究一下文件下载功能在Django中的实现。

##最简单的文件下载功能的实现

将文件流放入HttpResponse对象即可，如：

```python
def file_download(request):
	# do something...
	with open('file_name.txt') as f:
		c = f.read()
	return HttpResponse(c)
```

这种方式简单粗暴，适合小文件的下载，但如果这个文件非常大，这种方式会占用大量的内存，甚至导致服务器崩溃

##更合理的文件下载功能

Django的`HttpResponse`对象[允许将迭代器作为传入参数](https://docs.djangoproject.com/en/dev/ref/request-response/#passing-iterators)，将上面代码中的传入参数`c`换成一个迭代器，便可以将上述下载功能优化为对大小文件均适合；而Django更进一步，推荐使用 `StreamingHttpResponse`对象取代`HttpResponse`对象，`StreamingHttpResponse`对象用于[将文件流发送给浏览器](https://docs.djangoproject.com/en/dev/ref/request-response/#django.http.StreamingHttpResponse)，与HttpResponse对象非常相似，对于文件下载功能，使用`StreamingHttpResponse`对象更合理。

因此，更加合理的文件下载功能，应该先写一个迭代器，用于处理文件，然后将这个迭代器作为参数传递给`StreaminghttpResponse`对象，如：

```python
from django.http import StreamingHttpResponse

def big_file_download(request):
	# do something...
 
	def file_iterator(file_name, chunk_size=512):
		with open(file_name) as f:
			while True:
				c = f.read(chunk_size)
				if c:
					yield c
				else:
					break
 
	the_file_name = "file_name.txt"
	response = StreamingHttpResponse(file_iterator(the_file_name))
 
	return response
```

##文件下载功能再次优化

上述的代码，已经完成了将服务器上的文件，通过文件流传输到浏览器，但文件流通常会以乱码形式显示到浏览器中，而非下载到硬盘上，因此，还要在做点优化，让文件流写入硬盘。优化很简单，给StreamingHttpResponse对象的`Content-Type`和`Content-Disposition`字段赋下面的值即可，如：

```python
response['Content-Type'] = 'application/octet-stream'
response['Content-Disposition'] = 'attachment;filename="test.pdf"'
```

完整代码如下：
```python
from django.http import StreamingHttpResponse

def big_file_download(request):
	# do something...
 
	def file_iterator(file_name, chunk_size=512):
		with open(file_name) as f:
			while True:
				c = f.read(chunk_size)
				if c:
					yield c
				else:
					break
 
	the_file_name = "big_file.pdf"
	response = StreamingHttpResponse(file_iterator(the_file_name))
	response['Content-Type'] = 'application/octet-stream'
    response['Content-Disposition'] = 'attachment;filename="{0}"'.format(the_file_name)
 
	return response
```


