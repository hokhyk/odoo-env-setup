PEP3333指出，WSGI(Web Server Gateway Interface)是WEB服务器和web框架或web应用之间建立的一种简单通用的接口规范。有了wsgi这份接口规范，在web开发的过程中，能更加自由的选择服务器端和框架；在服务器端和框架的开发过程能够分离开来，不用过多的考虑双方具体的实现，使得服务器端和框架开发者能够专心自己领域的开发工作。

　WSGI主要由三大部分组成WSGI SERVER、WSGI Middleware 、WSGI Application，他们之间的关系是：wsgi server 接收来自客户端的request请求，封装environ环境变量，给app提供回调函数；调用app,并将environ和回调函数一起传递给app；最后接收来自app的header/status/body响应信息，传给客户端。wsgi Middleware是服务器和应用程序之间的桥梁，中间件同时扮演两种角色，在服务端看来，中间是app能够被调用，在应用程序看来，它是服务端，能够进一步包装需要出来的信息，并将出来的信息传递调用app。应用程序是我们开发程序的主体，必须可调用，必须能够接受服务端传过来的两个参数environ和callback函数。将传入进来的参数，进一步出来生成responce响应信息，传回服务器端。下面是三者的关系图。

　

　　实现一个服务端WSGI SERVE：　
复制代码

#!/usr/bin/python
#coding=utf-8

def run(application):
    environ={}                     #设定环境信息
    
    def start_response(status,headers):  #define callback function
        pass

    result=application(environ,start_response) #call app pass two parameters
   
    def write(data):                #访问result iterator object
        pass 
    
    for data in result:
        write(data)

复制代码

        从以上代码可以看出，:server:run是服务端程序，调用可调用应用程序，app。定义一个start_response函数传递给app,作为回调函数。run服务端调用app后，返回一个iterator 迭代对象，将其赋值给result。write()函数在start_response将响应头信息发送后，发送响应body信息。

　　

　　Middleware

       middleware 即是服务端又是应用端。那么就将满足服务端条件：能够配置envion,具有start_response并将其传递给应用程序调用应用程序；应用端条件：可调用，接收两个参数，返回迭代对象。
复制代码

def dispath(url_app_mapping):
    def midware_app(envion,start_responce):      #callable function
        url=environ['PATH_INFO']
        app=url_app_mapping[url]                 #get the view function
        result=app(environ, start_response)      #call app
        return result
    return midware_app

class dispath():
    
    def __init__(self,application):              #调用合适的app
        self.app=application

    def __call__(self,environ,start_response):
        #do sometiong 
        return self.app(environ,start_response)  #自身可调用

复制代码

　　从函数的角度实现的中间件和从类的角度实现的中间件，类的角度实现的中间件更加直观明显。

　WSGI application

   
复制代码

#!/usr/bin/python
#coding=utf-8

def app(environ,start_response):              #可调用接收两个参数
    status='200 ok'
    headers=[('Content-type','text/plain')]   #设置响应状态码和响应信息

    start_response(status,headers)            #调用回调函数            

    return ["hello world"]                    #返回body 可迭代对象

复制代码

    一个简易的app应用程序，注意返回的body要是一个可迭代对象。



# 
从WSGI源码中可以看到，第一个函数是get_current_url,源码如下，仔细分析一下这段代码的作用：

def get_current_url(environ, root_only=False, strip_querystring=False, host_only=False, trusted_hosts=None):
    
    tmp = [environ['wsgi.url_scheme'], '://', get_host(environ, trusted_hosts)]
    cat = tmp.append
    if host_only:
        return uri_to_iri(''.join(tmp) + '/') 

    cat(url_quote(wsgi_get_bytes(environ.get('SCRIPT_NAME', ''))).rstrip('/'))
    cat('/')              
    if not root_only:
        cat(url_quote(wsgi_get_bytes(environ.get('PATH_INFO', '')).lstrip(b'/')))
        if not strip_querystring:
            qs = get_query_string(environ)
            if qs:
                cat('?' + qs)
    return uri_to_iri(''.join(tmp))

get_current_url接受５个参数，其中后四个是默认参数都设置为False或者是None。
第一句list中，索引为0的值获取environ字典中key为wsgi.url_scheme的值，scheme有两种：http或者是https。索引为2的值是一个地址,其中函数get_host和trusted_host的作用我们会在下面的源码中看到。　　

cat是一个函数，相当于list1.append(i)。　　
接下来如果host_only为True,那么直接返回字符串，格式如：https://www.xxxxx.xxxx/ 。注意这个uri_to_iri函数位于urls模块中，它的作用是： 将URI转换成用Unicode编码的IRI，IRI的介绍可以查看w3 。

cat(url_quote(wsgi_get_bytes(environ.get('SCRIPT_NAME', ''))).rstrip('/'))中首先get环境字典元素中的script_name,接着wsgi_get_bytes方法将编码方式改成为latin1,在_compat模块中wsgi_get_bytes=operator.methoncaller('encode','latin1')。url_quote传入要转换的编码格式的字符串和需要转成的编码格式，将字符串的编码方式改变成给定的形式，默认参数是utf-8。　　
methodcaller的作用如下：

url_quote源码中有一个bytearray方法是一个内建函数：使用方法

>>>d1 = b"12345"
>>>print(type(d1))
>>>d2 = bytearray(d1)
>>>print(type(d2))
>>>for i in d2:
>>>    print d2

由输出结果可知d2是一个"bytearry"类型，且d2是一个可迭代对象。

下一个条件语句，针对URL中path和querying进行格式化，并将path和query加到URL路径中。
类似get_query_string函数，在WSGI中有很多，主要得到的是特定的字符串，get_query_string的理解如下：
里面有一个方法:try_coerce_native其作用是：将字符Unicode字符格式转换成默认的编码格式。　　
　
函数最后返回一个经过url_parse处理过的tuple。url_parse返回经过is_test_based、URL或者BytesURL修饰过的字符串。而URL和BytesURL两个类从BaseURL中继承。最中还是返回URL中各个信息段组成的tuple。
所以get_current_url最后得到一个由URL各信息段组成的tuple。



