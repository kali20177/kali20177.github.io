# 批量提取行程码/健康码信息到表格


## 1. 说明

学校老师希望能实现一个自动处理行程码、健康码信息的程序，本文记录一下实现过程。笔者非计算机专业人员，代码看看就好哈 :disappointed_relieved:。

## 2. 实现思路

程序执行逻辑：

1. 学生的行程码/健康码截图由管理人员汇总，全部保存到文件夹。
2. 遍历文件夹，通过文件名分别处理行程码和健康码。
3. 对于行程码，调用 OCR API 获取行程码信息，获取服务器 json 字符串，解析 json 处理结果。
4. 对于健康码，使用二维码相关的包得到轮廓坐标，计算截取部分的颜色。
5. 将结果对应保存到表格中。

## 3. 健康码处理

健康码本身解析后是一个网页链接，普通人并不能获取持码人个人信息，只能考虑直接识别颜色。

### 3.1 读取文件

考虑到收上来的截图会以中文来命名，但是 opencv 无法直接读取中文文件名的图片，需要借助 numpy 模块。

```python
def cv_imread(file_path):
    cv_img = cv2.imdecode(np.fromfile(file_path, dtype=np.uint8), cv2.IMREAD_COLOR)
    return cv_img
```

### 3.2 二维码处理

Python 中用于处理二维码的包有：pyzbar、opencv-python 和 zxing。在实际使用过程中，pyzbar 无法很好处理黄码，zxing 表现较好，但是 zxing 本身由 java 实现，最后使用打包工具生成的 exe 无法运行。最后选择了 opencv-python 中的 QRCodeDetector 模块，实现了获取二维码轮廓坐标的功能。

```python
def scan_qrcode(image):
    qrCodeDetector = cv2.QRCodeDetector()
    decodedText, points, _ = qrCodeDetector.detectAndDecode(image)
    return decodedText, points
```

拿到 points 坐标后可以直接截取出来这个部分用于下一步识别颜色。

### 3.3 颜色识别

定义 HSV 颜色范围

```python
# 颜色范围定义
color_dist = {
    'red': {'Lower': np.array([0, 60, 60]), 'Upper': np.array([6, 255, 255])},
    'blue': {'Lower': np.array([100, 80, 46]), 'Upper': np.array([124, 255, 255])},
    'green': {'Lower': np.array([35, 43, 35]), 'Upper': np.array([90, 255, 255])},
    'yellow': {'Lower': np.array([26, 43, 46]), 'Upper': np.array([34, 255, 255])},
}

def detect_color(image, color):
    gs = cv2.GaussianBlur(image, (5, 5), 0)
    hsv = cv2.cvtColor(gs, cv2.COLOR_BGR2HSV)
    mask = cv2.inRange(hsv, color_dist[color]['Lower'], color_dist[color]['Upper'])  
    image_s = image.shape[0] * image.shape[1] 
    ratio = sum(sum(mask // 255)) / image_s

    if ratio > 0.02:
        return True
    else:
        return False
```

## 4. 处理行程码

从行程码截图提取出的信息和健康码信息并无重合，所以无法通过图片信息进行匹配。只能依赖上传时的命名。
这里直接使用百度智能云的 API 获取行程码信息。先注册百度智能云账号，获取 API Key 和 Secret Key。将行程码用 base64 编码后传给服务器，解析获取的 json 字符串，提取手机号码 (不全)、途径地、风险等级和更新时间。

## 5. 生成表格

使用 xlwt 模块生成表格，用当天日期命名。

```python
import datetime
...
    xl = xlwt.Workbook(encoding='utf-8')
    sheet = xl.add_sheet('sheet1', cell_overwrite_ok=True)
    sheet.write(0, 0, '姓名')
    sheet.write(0, 1, '手机号码')
    sheet.write(0, 2, '途径地')
    sheet.write(0, 3, '风险等级')
    sheet.write(0, 4, '行程码更新时间')
    sheet.write(0, 5, '健康码颜色')
    now = datetime.datetime.now()
    excel_name = now.strftime('%Y-%m-%d') + '.xls'
```

## 6. 打包

将 python 程序打包成 exe 文件一般使用 pyinstaller 包，让程序能够在无 python 环境的机器上也能运行。假设现在为 main.py 进行打包。

```bash
pip3 install pyinstaller
```

然后进入项目目录，执行命令：

```bash
pyinstaller -F main.py
```

**错误 1**:

```text
The 'typing' package is an obsolete backport of a standard library package and is incompatible with PyInstaller. Please `conda remove typing` then try again.
```

直接 `conda remove typing` 会很慢并有可能不成功。可以运行 `pip uninstall typing`。

**错误 2**:

```text
PermissionError: [Errno 13] Permission denied: xxx
```

将 main.py 文件所在目录的属性选项中的 `只读` 选项去掉，然后再执行命令。

**错误 3**:

```text
TypeError: _get_sysconfigdata_name() missing 1 required positional argument: 'check_exists'
```

参考文末链接 4 的解决方法。

最后在路径下生成的文件夹中找到 `dist` 文件夹，将其中的 `main.exe` 文件移动到项目目录下，双击打开即可运行。main.py 文件，dist 等文件夹都可以删除。

如需自定义图标并不显示控制台，使用命令：

```bash
Pyinstaller -F -w -i xx.ico main.py 
```

## 参考

1. [stackoverflow typing error](https://stackoverflow.com/questions/70710731/the-typing-package-is-an-obsolete-backport-of-a-standard-library-package-and-i)
2. [Python 报错：PermissionError: [Errno 13] Permission denied](https://blog.csdn.net/weixin_44630029/article/details/118021429)
3. [别再问我怎么 Python 打包成 exe 了！](https://zhuanlan.zhihu.com/p/162237978)
4. [使用 pyinstaller 将 py 文件打包成 exe 注意事项及报错](https://blog.csdn.net/weixin_44120025/article/details/122961857)

