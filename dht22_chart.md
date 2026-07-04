# 树莓派：DHT22 温湿度监控 + 图表



## 说明

用 DHT22 读取温湿度，matplotlib 绘制实时趋势图，保存为 PNG 和历史 CSV 数据。学习传感器读取、数据可视化和文件存储的完整流程。



## 硬件

- 树莓派 ×1

- DHT22 ×1

- 可选：4.7KΩ 上拉电阻



## 代码

```python

import Adafruit_DHT

import csv

import time

from datetime import datetime

import matplotlib.pyplot as plt



SENSOR = Adafruit_DHT.DHT22

PIN = 4  # GPIO4

LOG_FILE = "/home/pi/temperature_log.csv"



def log_data(temp, hum):

    """追加写入 CSV"""

    with open(LOG_FILE, "a", newline="") as f:

        writer = csv.writer(f)

        writer.writerow([datetime.now().isoformat(), temp, hum])



def plot_history():

    """绘制历史曲线"""

    times, temps, hums = [], [], []

    with open(LOG_FILE, "r") as f:

        reader = csv.reader(f)

        next(reader)  # 跳过表头

        for row in reader:

            times.append(datetime.fromisoformat(row[0]))

            temps.append(float(row[1]))

            hums.append(float(row[2]))



    fig, ax1 = plt.subplots(figsize=(10, 5))

    ax2 = ax1.twinx()

    ax1.plot(times, temps, "r-", label="温度 °C")

    ax2.plot(times, hums, "b-", label="湿度 %")

    ax1.set_xlabel("时间")

    ax1.set_ylabel("温度 (°C)")

    ax2.set_ylabel("湿度 (%)")

    fig.legend(loc="upper left")

    plt.title("温湿度监测")

    plt.savefig("/home/pi/temperature_chart.png", dpi=100)

    plt.close()

    print("图表已保存为 temperature_chart.png")



def main():

    # 初始化 CSV 表头

    with open(LOG_FILE, "w", newline="") as f:

        csv.writer(f).writerow(["time", "temperature", "humidity"])



    count = 0

    try:

        while True:

            hum, temp = Adafruit_DHT.read_retry(SENSOR, PIN)

            if hum is not None and temp is not None:

                print(f"{datetime.now():%H:%M:%S} | {temp:.1f}°C | {hum:.1f}%")

                log_data(temp, hum)

                count += 1

            else:

                print("传感器读取失败")

            time.sleep(10)  # 每 10 秒采样一次

    except KeyboardInterrupt:

        print(f"\n共采集 {count} 条数据")



    if count > 0:

        plot_history()



if __name__ == "__main__":

    main()

```



## 教学重点

- `Adafruit_DHT.read_retry()` 自动重试，比 `read()` 可靠

- CSV 是物联网数据存储的标准格式，Excel 可打开

- `matplotlib` 双 Y 轴 `twinx()` 同时显示温度和湿度

- `datetime.isoformat()` 生成机器可读的时间字符串



## 常见错误

- DHT 库安装：`pip3 install Adafruit_DHT`

- 传感器读取偶尔失败（NaN）→ 用 `read_retry` 而非 `read`

- 文件权限：`/home/pi/` 下用 `sudo` 可能导致不同用户

- matplotlib 在无显示器环境需用 `Agg` 后端

