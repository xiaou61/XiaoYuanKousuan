## 小猿搜题冲榜/刷排名/专用思路-理论速度1小时/3.6w分 附带0s教程
2024-10-13 16:36 
**最新真正意义上的0s** 不管你做题速度多慢，提交后为0s 这个的整体思路是通过frida进行拦截，修改做题用时。
先看结果
![image](https://github.com/user-attachments/assets/01c4333e-979e-48bd-8716-e91674e169bb)

最先是这个项目所提出来的方案
https://github.com/Hawcett/XiaoYuanKouSuan_Frida_hook
之后我这里简单教学一下我对于这个方案实操的过程。
代码方面使用的是
x781078959用户写出的
https://github.com/x781078959/xyks/tree/master/frida/submit
首先是mumu模拟器安装firda-service 这里推荐的是14.2.18版本。(前俩天更新的那个版本没有适配mumu模拟器，安装会出现问题)

之后把包移动到data/local/tmp这里
![image](https://github.com/user-attachments/assets/041ea03b-cfbc-4d48-96b9-21b67e8c8f9e)

之后用adb shell进入adb控制台把他启动起来就可以
![image](https://github.com/user-attachments/assets/9efe9d0a-7ff6-4cd0-86c3-af9688525635)


这个的https://github.com/x781078959/xyks/tree/master/frida/submit代码可以直接用。如果你出现无法获得pid，(这个我刚才操作出现过，不过后面就没有再出现了。)
可以使用gg修改器去查看pid。
![image](https://github.com/user-attachments/assets/d3c8ffd2-42d5-49dd-8237-853652bacdf5)
之后运行这个代码就没问题了
```python
import json
import sys

import frida

# 获取 PID
pid = 3602

# 通过链接到虚拟机frida-server
device = frida.get_usb_device()

# 附加到已有进程的PID
session = device.attach(pid)

# 加载js文件
with open("submit.js", encoding='utf-8') as f:
    script = session.create_script(f.read())

# 设置控制台消息处理程序
def on_message(message, data):
    print(1)
    if message['type'] == 'send':
        print(1)
        # 复制粘贴 https://github.com/Hawcett/XiaoYuanKouSuan_Frida_hook 的代码
        bytes_obj = bytes.fromhex(message['payload'])
        string = bytes_obj.decode('utf-8')
        string = string.replace(r'\"', "%")
        # 修正null不加引号导致的错误
        string = string.replace("null", '"null"')
        print("字符串: ", string)
        json_data = json.loads(string)
        print(json_data)
        print('原始花费时间：', json_data['costTime'])
        # TODO 时间不要修改为0 最低为1 否则会有难以解决的错误
        json_data['costTime'] = 1
        print('现在花费时间：', json_data['costTime'])
        print(json_data)
        data = str(json_data).replace('%', r'\"')
        data = str(data).replace('\'', '\"')
        data = str(data).replace(' ', '')
        script.post({'type': 'send', 'my_data': data})
    else:
        print("[{}] {}".format(message['type'], message['description']))

# 设置消息处理程序
script.on('message', on_message)

# 加载js文件并获取脚本输出的信息
script.load()
# 保持脚本运行
sys.stdin.read()
```
⚠️：这个脚本不会修改题目数量和答案，只是在提交的时候会显示0s。



之后是另外一个xp模块的项目(这个适合想要冲全国榜的)
https://github.com/zhangLog08/Simian
这个可以方便快速的完成下面的功能
**自定义答案(题数不变)
模式二:自定义答案(只有一道题)**
冲榜方法可以看这里：https://www.bilibili.com/video/BV1Y52vYVEEd/?spm_id_from=333.999.0.0






最新10-12 21:48 可直接看这个项目 已经测试pk可用，https://github.com/cr4n5/XiaoYuanKouSuan  顺便补一句，如果冲榜也可选择1w道题，可以改成功

pk的代码精简版(要去那个项目里面下载上那个js文件)Mitmproxy的端口和host自己去改，代码里面是我的：
```python
import argparse
import sys

from mitmproxy.tools.main import mitmdump

# 命令行参数解析
parser = argparse.ArgumentParser(description="Mitmproxy script")
parser.add_argument("-P", "--port", type=int, default=8080, help="Port to listen on")
parser.add_argument("-H", "--host", type=str, default="192.168.190.1", help="Host to listen on")
args = parser.parse_args()

sys.argv = ["mitmdump", "-s", __file__, "--listen-host", args.host, "--listen-port", str(args.port)]


# 直接处理请求和响应
def handle_flow(flow):
    print(f"Response: {flow.response.status_code} {flow.request.url}")

    if "https://leo.fbcontent.cn/bh5/leo-web-oral-pk/exercise_" in flow.request.url:
        with open("exercise.js", "r", encoding="utf-8") as f:
            text = f.read()

        if text:
            flow.response.text = text
            print("修改成功")
        else:
            print("未能修改响应文本")


# 启动 mitmdump
mitmdump()
```



最新10-11 20:12:现在pk改包虽然不能用了，但是识别题目已经有人解出来了，那就可以用抓包后获取答案加adb继续炸鱼。
https://github.com/xmexg/xyks/issues/9
⚠️：改方法在pk模式热更新加密数据后依旧可用。不知道什么时候会被修复

先放战绩：
![f70b43fe42abdf1d34d3e17e08b46e3](https://github.com/user-attachments/assets/d0ea44a9-78f6-4462-989f-c2aabbab88aa)
![01a1dc5bb84ded88a33d1d254619f25](https://github.com/user-attachments/assets/7165d5af-4033-43fc-becd-faaf79feb94e)


⚠️：这个方法很多还需要手动操作，我目前无法用代码完全实现，如果你有兴趣可以给我提issue我们一起讨论。

## 冲榜思路(这个已经不是最优解，可以往下看我的最优解)

先说整体思路：**抓包+改答案+adb模拟**

之后详细教程：

我这里用到的是小黄鸟，你用别的抓包软件也可以。



过ssl用这个

![image-20241011150157919](https://xiaou-1305448902.cos.ap-nanjing.myqcloud.com/img/202410111502023.png)



之后刷分最快的是练习里面去修改题数

![image-20241011150322675](https://xiaou-1305448902.cos.ap-nanjing.myqcloud.com/img/202410111503730.png)

修改题数我这里是改的这个的请求体

![image-20241011150344852](https://xiaou-1305448902.cos.ap-nanjing.myqcloud.com/img/202410111503896.png)

![image-20241011150358481](https://xiaou-1305448902.cos.ap-nanjing.myqcloud.com/img/202410111503513.png)

之后修改完成后，要自己手答一遍。建议直接点**跳过**

之后正常打完后会继续结算页面。(`注意这个时候要关闭抓包软件，建议是在修改过后，是你想要的题目数量后就关闭抓包，不然会结算不了`)

![image-20241011150813675](https://xiaou-1305448902.cos.ap-nanjing.myqcloud.com/img/202410111508879.png)

结算完成后开启抓包，(此时开修改答案为1)

即可完成操作。

之后抓包软件就可以一直开启着。

改答案我这里用到的是我基于这个 [XiaoYuanKouSuan/main.py at main · cr4n5/XiaoYuanKouSuan (github.com)](https://github.com/cr4n5/XiaoYuanKouSuan)的昨天的一个版本修改的当时他还没有练习的模式，现在我看已经有了，可以参考他的代码 当然我的这个也可以用 (这个更新速度很快)里面很多没用的东西我都没删掉。



```python
import re
import time
import sys
import threading
import argparse

import adbutils

from mitmproxy import http
from mitmproxy.tools.main import mitmdump

auto_jump = True




# 拦截 HTTP 响应，检查 URL 并逐个字段替换
def response(flow: http.HTTPFlow):
    # 匹配目标 URL
    url_pattern = re.compile(r"https://xyks.yuanfudao.com/leo-math/android/exams.+")

    # 如果 URL 匹配
    if url_pattern.match(flow.request.url):
        # 如果响应类型是 JSON
        if "application/json" in flow.response.headers.get("Content-Type", ""):

            response_text = flow.response.text

            # 使用正则表达式逐个替换字段
            response_text = re.sub(r'"answer":"[^"]+"', '"answer":"1"', response_text)
            response_text = re.sub(r'"answers":\[[^\]]+\]', '"answers":["1"]', response_text)

            # 更新修改后的响应体
            flow.response.text = response_text


        threading.Thread(target=answer_write, args=(len(re.findall(r'answers', flow.response.text)),)).start()




def jump_to_next():
    # 结束，自动进下一局
    device = adbutils.adb.device()
    time.sleep(3)
    command = "input tap 540 1520"
    device.shell(command)  # “开心收下”按钮的坐标
    time.sleep(0.3)
    command = "input tap 780 1820"
    device.shell(command)  # “继续”按钮的坐标
    time.sleep(0.3)
    command = "input tap 510 1700"
    device.shell(command)  # “继续PK”按钮的坐标


def swipe_screen():
    xy = [[1480, 1050], [1440, 1470]]
    command = f"input swipe {xy[0][1]} {xy[0][0]} {xy[1][1]} {xy[1][0]} 0"

    device = adbutils.adb.device()
    device.shell(command)


def answer_write(answer):
    time.sleep(6)
    for _ in range(5500):
        print("11111")
        swipe_screen()
        time.sleep(0.01)


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Mitmproxy script")
    parser.add_argument("-P", "--port", type=int, default=8080, help="Port to listen on")
    parser.add_argument("-H", "--host", type=str, default="192.168.190.1", help="Host to listen on")
    args = parser.parse_args()

    sys.argv = ["mitmdump", "-s", __file__, "--listen-host", args.host, "--listen-port", str(args.port)]
    mitmdump()
```



## 0s教学(现在已不可用，小猿搜题会删除过于离谱的排名)
0s这些是我昨天弄的，如果你按照我的方法是可行的
![01a1dc5bb84ded88a33d1d254619f25](https://github.com/user-attachments/assets/4de9cf89-9a42-4e4f-a5b2-b8d61a87b3b4)

关于0s

就是改为一道题，然后用adb

这里就不建议开启抓包了，改好一次后(这里指的是改好题目数量)，关闭抓包，不断的点**再练一次**在刷成绩，也不建议**修改答案**。

因为这个0s是我昨天弄成的，我不知道今天是否还有用。

还是很容易刷出来的，基本上我刷俩分钟就出来了。

**刷出来的标志**：就是在图标还没有完全消失之前adb已经绘制好了答案。这样就是0s。

但是0s的话，当前账号就无法看到你取得0s的那个排行榜的榜单了，别人那里是显示的，不过排名是0


## 个人刷榜思路(适合小白技术难度较低)
后台挂着改包的脚本(改包现在pk没办法用，但是练习还可以用)
之后我这里用的自动精灵，其他的也可以，可以看演示视频：https://www.bilibili.com/video/BV1Ri2SYsEk4/?spm_id_from=333.999.0.0
**亲测了一下午发现改题目数量与不改题目数量差距不大，这个操作起来比较简单。**

## 碎碎念

介绍一下自己，大三学生，计算机专业。

从昨天一直搞到今天，大概也弄了十来个小时了。

一路看着各种思路的出现

从最开始的autojs ocr

到后面的抓包提前拿答案。到改包改答案。 pk30题改1题。

我这个思路不见得是最优的方案，并且只是思路，代码实现方面还需要有人去优化。

网络安全路漫漫，这次的事件，不只是给小猿平台，**更是给更多的程序员提了醒**，其实多一次验证，就可以完美避开这个问题。



