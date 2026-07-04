# 树莓派：Web 服务器 + GPIO 控制面板



## 说明

用 Flask 框架搭建一个网页控制面板，通过浏览器控制树莓派的引脚。这是物联网面板的基础。



## 代码

```python

from flask import Flask, render_template, request, jsonify

import RPi.GPIO as GPIO



app = Flask(__name__)



# GPIO 配置

PINS = {

    "LED 红灯": 17,

    "LED 绿灯": 27,

    "蜂鸣器":   22,

}



GPIO.setmode(GPIO.BCM)

for pin in PINS.values():

    GPIO.setup(pin, GPIO.OUT)

    GPIO.output(pin, GPIO.LOW)



# HTML 模板（内嵌，也可放 templates 目录）

HTML = """

<!DOCTYPE html>

<html><head>

<meta charset='utf-8'><meta name='viewport' content='width=device-width'>

<title>控制面板</title>

<style>

body{text-align:center;font-family:Arial;margin-top:30px;}

.btn{padding:20px 40px;margin:15px;font-size:24px;border:none;border-radius:10px;color:white;cursor:pointer;}

.on{background:#4CAF50;}.off{background:#f44336;}

.status{font-size:20px;margin:20px;}

</style></head><body>

<h1>🏠 树莓派控制面板</h1>

{% for name, state in states.items() %}

<div class='status'>{{ name }}: <b>{{ state }}</b></div>

{% endfor %}

{% for name in pins %}

<div>

<a href='/toggle/{{ name }}'><button class='btn on'>{{ name }}</button></a>

</div>

{% endfor %}

</body></html>

"""



from flask import render_template_string

import jinja2



@app.route("/")

def index():

    states = {n: "🟢 亮" if GPIO.input(p) else "⚫ 灭" for n, p in PINS.items()}

    return render_template_string(HTML, pins=PINS.keys(), states=states)



@app.route("/toggle/<name>")

def toggle(name):

    pin = PINS.get(name)

    if pin is not None:

        current = GPIO.input(pin)

        GPIO.output(pin, not current)

    return "<script>window.location='/'</script>"



@app.route("/api/status")

def api_status():

    return jsonify({n: bool(GPIO.input(p)) for n, p in PINS.items()})



@app.route("/api/set/<name>/<state>")

def api_set(name, state):

    pin = PINS.get(name)

    if pin is not None:

        GPIO.output(pin, state.lower() == "on")

    return jsonify({"ok": True})



if __name__ == "__main__":

    GPIO.cleanup()

    GPIO.setmode(GPIO.BCM)

    for pin in PINS.values():

        GPIO.setup(pin, GPIO.OUT)



    try:

        app.run(host="0.0.0.0", port=5000, debug=False)

    finally:

        GPIO.cleanup()

```



## 教学重点

- Flask 是 Python 最流行的 Web 微框架

- `render_template_string` 可以对内嵌模板渲染（不用文件）

- `@app.route()` 装饰器定义 URL 路由

- GPIO 操作需在 `try/finally` 中确保 cleanup

- JSON API 方便被手机 App/AI 助手调用



## 常见错误

- `host="0.0.0.0"` 监听所有接口（局域网可访问），仅开发用 `127.0.0.1`

- 端口被占用（5000 是 macOS AirPlay 常用端口）

- GPIO 不是线程安全的 → Flask 多线程模式下需加锁

