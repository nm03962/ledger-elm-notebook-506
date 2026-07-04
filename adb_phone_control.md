# 树莓派：安卓手机 ADB 控制



## 说明

用树莓派通过 ADB(Android Debug Bridge) 控制连接的安卓手机：截图、点击、滑动。可制作自动化测试机器人。



## 硬件

- 树莓派 ×1

- 安卓手机 ×1（需开 USB 调试）

- USB 数据线



## 代码

```python

import subprocess

import time

import os



class PhoneController:

    def __init__(self):

        self.check_adb()



    def check_adb(self):

        """检查 ADB 是否可用"""

        result = subprocess.run(["adb", "version"],

                                 capture_output=True, text=True)

        if "Android Debug Bridge" not in result.stdout:

            raise RuntimeError("ADB 未安装！sudo apt install adb")



    def get_device(self):

        """获取连接的设备"""

        result = subprocess.run(["adb", "devices"],

                                 capture_output=True, text=True)

        lines = result.stdout.strip().split("\n")

        return [l.split()[0] for l in lines[1:] if l.strip() and "device" in l]



    def screenshot(self, path="screenshot.png"):

        """截取手机屏幕"""

        subprocess.run(["adb", "shell", "screencap", "-p", "/sdcard/sc.png"])

        subprocess.run(["adb", "pull", "/sdcard/sc.png", path])

        print(f"截图已保存: {path}")



    def tap(self, x, y):

        """点击屏幕坐标"""

        subprocess.run(["adb", "shell", "input", "tap", str(x), str(y)])

        print(f"点击 ({x}, {y})")



    def swipe(self, x1, y1, x2, y2, duration_ms=300):

        """滑动屏幕"""

        subprocess.run(["adb", "shell", "input", "swipe",

                        str(x1), str(y1), str(x2), str(y2), str(duration_ms)])

        print(f"滑动 ({x1},{y1}) -> ({x2},{y2})")



    def type_text(self, text):

        """输入文字"""

        # 空格需转义

        text = text.replace(" ", "%s")

        subprocess.run(["adb", "shell", "input", "text", text])



    def auto_demo(self):

        """自动演示：打开设置→截图→返回"""

        print("开始自动化演示...")

        # 从底部上滑打开应用抽屉

        self.swipe(540, 1800, 540, 300)

        time.sleep(0.5)

        # 点击设置图标（按你的手机调坐标）

        self.tap(540, 800)

        time.sleep(1)

        self.screenshot("settings.png")

        # 返回桌面

        subprocess.run(["adb", "shell", "input", "keyevent", "4"])  # KEYCODE_BACK

        print("演示完成")



if __name__ == "__main__":

    phone = PhoneController()

    devices = phone.get_device()



    if not devices:

        print("未检测到设备！请确保:")

        print("1. 手机已开启 USB 调试")

        print("2. 已授权此电脑调试")

        exit(1)



    print(f"已连接: {devices[0]}")



    # 交互式菜单

    while True:

        cmd = input("\n命令: tap/screenshot/swipe/demo/exit > ").strip()

        if cmd == "tap":

            x, y = input("坐标 x y: ").split()

            phone.tap(int(x), int(y))

        elif cmd == "screenshot":

            phone.screenshot()

        elif cmd == "swipe":

            x1, y1, x2, y2 = input("起始x y 终点x y: ").split()

            phone.swipe(int(x1), int(y1), int(x2), int(y2))

        elif cmd == "demo":

            phone.auto_demo()

        elif cmd == "exit":

            break

```



## 教学重点

- ADB 是 Android 调试桥，`adb shell` 执行远程命令

- `input tap/swipe/text` 模拟用户操作

- `screencap` + `pull` 截取屏幕

- KEYCODE 码：4=返回, 3=Home, 26=电源

- 可用此框架写自动化测试脚本



## 常见错误

- 手机没开 USB 调试 → 设置→关于→连点版本号→开发者选项

- 没授权 → 手机上弹窗点"始终允许"

- ADB 版本不匹配 → `adb kill-server && adb start-server`

- 坐标与手机分辨率不匹配（通常 1080x1920）

