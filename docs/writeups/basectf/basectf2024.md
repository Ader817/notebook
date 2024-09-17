## [Week1] 根本进不去啊

最开始通过 `nslookup txt flag.basectf.fun`试图查询DNS记录，发现域名对应的主机地址不存在

那么 flag 可能被隐藏在了 DNS 的 txt 记录中

通过 dig 查询 DNS 解析的 txt 记录，键入命令 `dig txt flag.basectf.fun`

或者使用 nslookup 同理键入命令 `nslookup -query=TXT flag.basectf.fun`即可得到

一段 txt 记录`text = "FLAG: BaseCTF{h0h0_th1s_15_dns_rec0rd}"`



## [Week1] 海上遇到了鲨鱼

附件给了一个.pcapang 文件，用 wireshark 打开这个流量包，主要为 HTTP 和 TCP 的流量

找到了`GET /wireshark/flag.jpg HTTP/1.1`的 Info，我们主要关注 HTTP 协议

因此导出 HTTP 对象，在 flag.php 中找到了一段文本`}67bf613763ca-50b3-4437-7a3a-b683fe51{FTCesaB`

显然为 flag 的倒置，因此倒置回来就可以得到 flag



## [Week1] Base

显然是一串 Base 系列编码后的字符串，直接丢到 Cyberchef 中 ~~magic~~ 一下就拿到了 flag 



## [Week1] 倒计时？海报！

公开了 Flag

`BaseCTF{c0unt_d0wn_fro3_X_every_d@y_i5_re@11y_c0o1_@nd_h@rd_t0_do_1t_ev3ry_n1ght}`

其实是每个海报放大后能看见水印

每一天的倒计时海报中水印的字符拼接起来就能够得到 Flag



## [Week1] 签到！DK盾！

签到题

关注 DK 盾公众号，发送 `BaseCTF2024`，即可获得 Flag



## [Week1] 喵喵太可爱了

娱乐题

貌似命令是

```text
如何把第一个{}中的内容输出呢？教教我嘛，喵~#假设我们有这样的字符串
flag_string = "BaseCTF{m"
```



## [Week1] 人生苦短，我用python
从代码中，可以推测出验证输入的 flag 是否正确的过程是通过一系列的条件检查实现的。为了找到满足所有条件的 flag，我们可以逐步分析每个条件。
分析条件

1. 长度检查：
```python
if len(flag) != 38:
```
flag 的长度必须为 38 个字符。

2. 前缀检查：
```python
if not flag.startswith('BaseCTF{'):
```
flag 必须以 BaseCTF{ 开头。

3. 特定位置字符检查：
```python
if flag.find('Mp') != 10:
```
flag 中 "Mp" 的位置必须是索引 10。

4. 特定末尾检查：
```python
if flag[-3:] * 8 != '3x}3x}3x}3x}3x}3x}3x}3x}':
```
flag 的最后 3 个字符乘 8 必须等于 '3x}3x}3x}3x}3x}3x}3x}3x}'，因此最后 3 个字符为 '3x}'。

5. 最后一个字符检查：
```python
if ord(flag[-1]) != 125:
```
flag 的最后一个字符的 ASCII 码必须为 125，即 }。

6. 下划线检查：
```python
if flag.count('_') // 2 != 2:
```
flag 中的下划线数量必须为 4（因为 4 // 2 == 2）。

7. 下划线分割后的长度检查：
```python
if list(map(len, flag.split('_'))) != [14, 2, 6, 4, 8]:
```
按下划线 split 后，每部分的长度应为 [14, 2, 6, 4, 8]。

8. 步长为 4 的子串检查：
```python
if flag[12:32:4] != 'lsT_n':
```
从索引 12 开始，每 4 个字符一个，必须得到 'lsT_n'。

9. 字符替换检查：
```python
if '馃樅'.join([c.upper() for c in flag[:9]]) != B馃樅A馃樅S馃樅E馃樅C馃樅T馃樅F馃樅{馃樅S'
```
flag 的前 9 个字符的大写必须是 'BASECTF{S'

10. 数值字符检查：
```python
if not flag[-11].isnumeric() or int(flag[-11]) ** 5 != 1024:
```
flag 的倒数第 11 个字符必须是一个数字，并且它的 5 次方为 1024，即 4。
    
11. Base64 编码检查：
```python
if base64.b64encode(flag[-7:-3].encode()) != b'MG1QbA==':
```
flag 的倒数第 7 到第 3 个字符（不含倒数第 3 个）Base64 编码后应为 'MG1QbA=='，即 'G1Pa'。
    
12. 逆序步长检查：
```python
if flag[::-7].encode().hex() != '7d4372733173':
```
逆序每 7 个字符的ASCII码十六进制表示为 '7d4372733173',即'}Crs1s'
    

13. 特定集合检查：
```python
if set(flag[12::11]) != {'l', 'r'}:
```
从索引 12 开始，每 11 个字符取一个，结果的集合应为 {'l', 'r'}。
    
14. 字节序列检查：
```python
if flag[21:27].encode() != bytes([116, 51, 114, 95, 84, 104]):
```
索引 21 到 26 的子字符串的字节序列应为 [116, 51, 114, 95, 84, 104]，即 't3r_Th'。
    
15. 累加校验：
```python
if sum(ord(c) * 2024_08_15 ** idx for idx, c in enumerate(flag[17:20])) != 41378751114180610:
```
索引 17 到 19 的字符经过特定计算应得 41378751114180610。
可以写一个爆破脚本找到18及19位数，17位已知为'_'

```python
key = 41378751114180610 - ord('_')
for i in range(128):
    for j in range(128):
        if i * 2024_08_15 + j * 2024_08_15 * 2 == key:
            print(chr(i),chr(j))
```

得到 B，e

16. 字符属性检查：
```python
if not all([flag[0].isalpha(), flag[8].islower(), flag[13].isdigit()]):
```
索引 0 的字符应为字母，索引 8 的字符应为小写字母，索引 13 的字符应为数字。
    
17. 格式化检查：
```python
if '{whats} {up}'.format(whats=flag[13], up=flag[15]).replace('3', 'bro') != 'bro 1':
```
格式化后的字符串应该是 '3 1'，并替换 '3' 为 'bro' 得 'bro 1'，即 flag[13] 为 '3'，flag[15] 为 '1'。
**这样其实就已经得到了Flag的内容**

`BaseCTF{s1Mpl3_1s_BeTt3r_Th4n_C0mPl3x}`

18. SHA1 校验：
```python
if hashlib.sha1(flag.encode()).hexdigest() != 'e40075055f34f88993f47efb3429bd0e44a7f479':
```
整个 flag 的 SHA-1 哈希应为 'e40075055f34f88993f47efb3429bd0e44a7f479'。

分析完所有检查条件后，我们知道 flag 应该满足所有给定的检查条件，因此我们能推断出 flag 为

`BaseCTF{SLsT_4_Th3rs_3x}`



## [Week1] 你也喜欢圣物吗

打开附件可以得到一个 .zip 文件 where_is_key 和一个 .png 文件

 where_is_key 需要密码来解压，猜测密码被隐写在了 png 图片中，利用 zsteg 自动枚举识别图片中的隐写字段

可以得到密码为`lud1_lud1`

```bash
b1,rgb,lsb,xy       .. text: "key=lud1_lud1"
```

得到密码后继续解压得到一个需要密码的名为 it is fake 的 .zip 文件

根据提示"**where_is_key.zip真的需要密码，再找找**"，猜测可能这个文件利用了 zip 伪加密技术

使用 010 Editor 打开该 zip 文件 , 将数据区和目录区的全局方式位标记都更改为 00 00 并保存

再次解压 zip 文件就可以得到 flag.txt，有两段明显为 Base系列特征的编码

其中上方的是假 flag，From Base64 后得到`flag{0h_n0_it's_f3ke}`

许多个换行后下方的是真正的 flag，两次 From Base64后得到`BaseCTF{1u0_q1_x1_51k1}`



## [Week1] 正着看还是反着看

打开附件能够得到一个 data 类型即纯二进制数据的 flag 文件，并且 cat 将其输出后发现是一段乱码

结合题目名称可以猜测要将二进制数据反向，于是我们可以撰写一个 python 脚本来倒置二进制数据

```python
def reverse_file(input_file, output_file):
    # 打开输入文件并读取其内容
    with open(input_file, 'rb') as f:
        data = f.read()

    # 将数据倒置
    reversed_data = data[::-1]

    # 将倒置后的数据写入输出文件
    with open(output_file, 'wb') as f:
        f.write(reversed_data)

# 示例用法
input_file = 'flag'
output_file = 'flag_reversed'
reverse_file(input_file, output_file)
```

`file flag_reversed`发现该文件其实是一个 JPEG 类型的文件，猜测与图片隐写有关

binwalk 查看附加文件，-e 选项自动提取，可以提取出 flag.txt，查看可以得到 flag 为`BaseCTF{h3ll0_h4cker}`



## [Week1] 捂住X只耳





## [Week2] ez_crypto

```python
import base64
string = "qMfZzunurNTuAdfZxZfZxZrUx2v6x2i0C2u2ngrLyZbKzx0="   # 欲解密的字符串
outtab  = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/="  #  原生字母表
intab   = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/="  #  换表之后的字母表
print (base64.b64decode(string.translate(str.maketrans(intab,outtab))).decode())
```

换表后利用 base64 解码就可以得到 Flag



## [Week2] base?!

 ```plaintexst
 hS5VZPaBjN4IU6G2VFqZqNG-tPrJm64NgMKQuEa3nNIBIFbh0MLBZEpF4LqZn
 9LpBjLoRjPqEV6Lo+
 
 question        response
 
 Base64?         X     
 Base32?         X       
 ```

利用自动编解码识别工具可知这是 **xxencode** 编码，可能题目中 Base64 和 Base32 的 Response "X" 就是告诉这段编码并非传统意义上的 Base64（Base32)，还是有点难想到的

顺带介绍一下 **xxencode** 及其姊妹 **uuencode** --- 他们两者其实都是 Base64 编码的变种

标准 Base64 编码的字符表为A-Za-z0-9+/=

而 xxencode 编码的字符表为+-0-9A-Za-z，uuencode 编码的字符表为 ASCII 的 32-95（包含了较多“奇怪”字符）

---

xxencode 重复以三个字节为一组，如果剩余字节少于三个，则添加尾随零。这 24 位被分成 4 个 6 位数字，6位数字的大小刚好是 0 - 63 ，按照下表将数字替换成字符

```
            1         2         3         4         5         6
 0123456789012345678901234567890123456789012345678901234567890123
 |         |         |         |         |         |         |
 +-0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
```

每组 60 个输出字符（对应于 60/4*3=45  个输入字节）作为单独的行输出，前面有一个编码字符，给出该行上的编码字节数。对于除最后一行之外的所有行，这将是字符“h”（映射到值 45  的字符）。如果输入不能被 45 整除，则最后一行将包含剩余的 N  个输出字符，前面是如上编码的剩余输入字节数。最后，输出一行仅包含一个空格（或加号字符）

**因此**我们发现题目中的文本非常符合 **xxencode**的编码特征

uuencode 与 xxencode 类似，也是重复以三个字节为一组，尾随零。将 24 位二进制数分成 4 个 6 位数字，6位数字的大小为 0 - 63，只不过 uuencode 的字母表替换方式不同，直接在 0 - 63 上加上 32，对应到 ASCII 字符表中的 32 - 95 号字符。同样每组 60 个输出字符作为单独的行输出，也有额外加在最前面的表示该行编码的字节数的编码字符，只不过满行 60 个输出字符所对应的字符应该是"M"（映射到 ASCII 表中第 45+32=77 位字符）



## [Week 2] 二维码1-街头小广告

**--- 来自官方题解（逃**

二维码本身具有纠错能力，只要损坏数量不太多，就能自动纠正。尤其是微信AI的纠错能力特别强，出题时凑了很久才凑出来“刚好补个角才能扫，否则不能扫”的效果。

在合适的位置补上右上角定位块（Windows画图 或者 PPT 就行）：

这样就能扫了。如果直接用微信之类的扫，会直接跳转到`www.basectf.fun`，看不到 flag。（如果你做题时网站没被微信拦截的话）

二维码本身只是数据的表示形式，二维码与它包含的数据（通常是字节）是等价的。为了检查未知来源二维码的安全性，我们希望只查看二维码内容，而不访问网址。生活中可以用微信小程序“草料二维码”的解码功能方便地做到这件事。CTF 中可以用 QR Research 或 QRazyBox 等工具处理二维码。

本题二维码信息：

```
https://www.bilibili.com@qr.xinshi.fun/BV11k4y1X7Rj/mal1ci0us?flag=BaseCTF%7BQR_Code_1s_A_f0rM_Of_m3s5ag3%7D
```

其中`@`前面的`www.bilibili.com`是用户名，在 HTTP 协议中无意义、会被忽略，`qr.xinshi.fun`才是真正的主机名（域名）。%7B 和 %7D 是 URL 编码。比赛时间内，`qr.xinshi.fun`上的 NGINX 把访问者 302 重定向到`www.basectf.fun`，这样就不能打开之后复制链接获得 flag 了。



## [Week 2] 前辈什么的最喜欢了

观察 .txt 文档发现开头提示了是经过 Base64 编码后的 .png 文件

为此可以写一个简单的 python 脚本将其转换为原先的图像文件

```python
import base64
with open("看着我吧，前辈.txt",'r') as f:
    encoded_data = f.read()
decoded_data = base64.b64decode(encoded_data)
with open('data.png','wb') as f:
    f.write(decoded_data)
```

**一把梭：**

1. 利用 ImageMagic 查看图片基本信息 -- xdg-open 尝试打开图片 -- identify 尝试输出图片基本信息

2. 使用 exiftool 检查图片元信息，看看有没有看起来会有用的信息

3. 使用十六进制编辑器打开，观察文件中有无附带信息、图片基本格式是否正确 / 利用 pngcheck 检查图片是否存在异常

4. 使用 binwalk 检查文件末尾是否叠加了多余的文件

5. 使用 stegsolve 打开图片 / 或者使用 CyberChef
  - 观察各个通道的 bit plane
  - 使用 Extract LSB 尝试提取数据格式的 LSB（或者使用 zsteg 猜测）

6. 考虑能否查找原图，如果找到了尝试进行比较

对于本题目，发现图片不能通过 xdg-open 正常显示，但可以直接使用 Windows 系统下的图片查看器打开。并且 pngcheck 后显示有 crc 校验错误，于是我们怀疑篡改了 IHDR 块的宽高数据

:fire:文件头数据块 IHDR (header chunk): 包含了 PNG 文件中储存的图像数据的基本信息，并作为第一个数据块出现在 PNG 数据流中

紧跟 IHDR 后的 8 个字节表示了图像的宽度和高度，例如

`49 48 44 52 00 00 02 25 00 00 03 `

因此我们直接适当增大图像的宽度和高度，在 Windows 自带的图片查看器中查看图片便可以看到因高度截取后而隐藏的 flag 信息（貌似主要是Windows 自带的图片查看器对 png 图像的要求并不是很严格，即使出现校验和错误也会忽视力图能将图像显示出来



## [Week2] 黑丝上的flag

原理是降低flag部分的透明度, 因为取原图的黑色部分, 以黑底显示时基本不影响图片, 以白底显示时flag部分变亮, 所以进行了部分加深.

不推荐直接肉眼读取flag, 出题人预期方法是编程遍历像素的`alpha`（透明度）通道, 重新用黑白写在新的图片上 --- 摘自官方 writeup

先用 zsteg 自动识别隐写发现没有什么成效，然后就直接利用 Cyberchef 的 View Bit Plane 功能，在最低位上对 red blue green alpha 做尝试便能找到 flag



## [Week2] 海上又遇到了鲨鱼

**--- 照搬了官方的 writeup** :smile:

Wireshark 是强大的网络数据捕获与分析工具

打开附件，看到协议为 TCP 与 FTP 的分组，可以一个个点击看看是什么 FTP协议，ftp-data 过滤流量包 发现 文件传输了 flag

追踪数据流

将传输文件的所抓到的数据进行导出 Show data as ASCII 改成 原始数据 另存为 flag.zip

打开压缩包 压缩包有密码且注释也提示我们需要密码而且是重复密码

（可以学一些过滤规则 ftp contains "230"  （但数据流不多 这行为有些多余了

尝试使用 Ba3eBa3e!@#成功解密压缩包

可以看一下整体数据包流量： 一直到流15 攻击者爆破ftp服务密码 用户名为admin 密码为Ba3eBa3e!@# 流16 攻击者拿着爆破成功的密码去登陆ftp服务器 执行了 dir 命令 获取了ftp共享服务目录 发现flag.zip 然后 download 下载了文件 





## [Week2] 反方向的雪

题目给了一张图片，利用 xdg-open 打开没有发现什么异常，identify 查看也没有什么异常，属于 jpeg 文件

我们将图片放在 010 editor 中查看，发现jpg文件尾后还有多余的信息，仔细查看发现是逐字节逆序的zip文件，结合题目名字反方向的提示

将他单独提出来再逐字节逆序，网上可以找到很多类似的工具也可以自己写代码，得到逆序后正常的文件

```python
with open('data.txt','r') as f:
    data = f.read().strip()
    byte_data = bytes.fromhex(data)
    reversed_data = byte_data[::-1]

with open('data.zip','wb') as f:
    f.write(reversed_data)
```

这样就得到了一个压缩包，需要密码，注释里面有一个提示是 The_key_is_n0secr3t，但是这好像并不是压缩包的密码，hint 给出密码为 6 位，利用妙妙小工具 ARCHPR 尝试爆破压缩包密码，得到密码是 123456

（注：无法在Linux环境下用unzip解压该压缩包，因为数据并不是以zip方式压缩的）

利用解压缩软件解压后得到flag.txt，但是并没有出现 flag

其实文件中有很多空白字符，根据题目雪的提示，这里是snow隐写

利用妙妙小工具 SNOW.EXE （注：SNOW Home Page官网提供的破解程序只能在windows系统下运行）解密，结合之前注释得到的key: n0secr3t

```shell
.\SNOW.EXE -C -p n0secr3t flag.txt
```

就可以解密得到flag



## [Week2] 哇！珍德食泥鸭

把 gif 丢到 binwalk 中分离，得到了一个 zip 文件

解压后打开压缩包可以判断文件格式其实是 docx

---

我们补充一下对 Docx 文件格式的理解

**DOCX** 是 Microsoft Office 2007 引入的文件格式，基于 **Open XML** 标准。与 DOC 的二进制格式不同，DOCX 使用了 XML 和压缩的结构，更加开放和可解析。实际上，DOCX 文件就是一个 **ZIP** 文件，包含了多个 XML 文件和其他资源。

**DOCX 文件的结构特点**：

- **ZIP 压缩格式**：DOCX 文件本质上是一个压缩文件，可以用 ZIP 工具（如 7-Zip、WinRAR）解压缩。

- 目录结构：解压 DOCX 文件后，可以看到一个包含多个 XML 文件和资源（如图片、字体等）的文件夹结构。常见的目录结构如下：

  - ```
    word/：包含文档的主要内容和设置。
    ```

    - `document.xml`：存储文档的主要内容（文本）。
    - `styles.xml`：定义文档中的样式信息（字体、段落样式等）。
    - `header1.xml`, `footer1.xml`：存储文档的页眉和页脚。

  - `_rels/`：包含关系文件，定义各个 XML 文件和资源之间的关系。

  - `docProps/`：包含文档的属性（如标题、作者、创建时间等）。

  - `[Content_Types].xml`：描述了文档中所有文件的 MIME 类型。

**解析 DOCX 的 XML 文件**：

- **document.xml**：是核心的文档内容，使用 XML 格式存储文档中的所有文本和格式信息。
- **styles.xml**：包含样式定义，用于指定文档中各种元素（如标题、段落、表格）的外观。
- **附加资源**：如图片、图表等通常保存在 `word/media/` 目录中，并在 XML 文件中引用。

得到 Docx 文件后打开往下翻翻，就可以看到最下面有一张白色图片作为遮挡（通过图片方式是悬浮文字上方判断）

于是我们将其移开，但是并没有发现任何东西，点击工具栏中的显示隐藏文字，再全选将文字颜色改成其他颜色就可以看到 flag ！



## [Week3] 纯鹿人

打开docx文件 全选后发现有一段文字被隐藏，修改文字颜色后发现是一个base64 对其解码 内容为压缩包密码

随后把docx改成zip 提取出里面的docx图片并进行分解，拿到压缩包 输入前面拿到的解压密码即可得到flag



## [Week3] Base Revenge

把 Hint 拖到妙妙工具--编解码工具中一键解码，发现经过 Atbash 解密后得到了 'This is hint' 的提示，说明可能需要用到 Atbash 解码

很明显源文本是一个 Base64 编码后的文本，我们将文本经过 Base64 解码后得到了一串英文文本，提示我们这段文本经过了 Base64 隐写

编写一个 Base64 隐写的解密脚本 ---- 现已归为妙妙工具

```python
import re

path = './base_revenge.txt'  
b64char = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
with open(path, 'r')as f:
	cipher = [i.strip() for i in f.readlines()]
plaintext = ''
for i in cipher:
	if i[-2] == '=':  # There are 4-bit hidden info while end with two '='
		bin_message = bin(b64char.index(i[-3]))[2:].zfill(4)
		plaintext += bin_message[-4:]
	elif i[-1] == '=':  # There are 2-bit hidden info while end with one '='
		bin_message = bin(b64char.index(i[-2]))[2:].zfill(2)
		plaintext += bin_message[-2:]
plaintext = re.findall('.{8}', plaintext)  # 8bits/group
plaintext = ''.join([chr(int(i,2)) for i in plaintext])
print(plaintext)
```

得到隐写的结果后再 Atbash 解密（这里最好使用 Cyberchef，因为妙妙工具在 Atbash 中不区分大小写统一归为小写）

Cyberchef 自动识别经过 Base64 解码后就能得到 Flag



## [Week3] 白丝上的flag

**题目描述：某出题人赠送大家flag时遭遇了信号干扰, 幸好我们在不知名小网站找到了写入flag前的图片, 尝试还原信息吧!**

将题目解压后为两张图片和一个py文件

 打开py文件发现是一个**图像混淆程序**，主要功能是对输入的图像进行混淆处理，然后将结果保存为一个新的图像文件

猜测flag就写在en_image这张图片里，正好这里有原图，我们可以将没有写flag的图片进行相同的加密后再与en_image进行**异或xor**处理即可得到flag

``` python3
from PIL import Image
import numpy as np
from task import confuse_image

image1 = Image.open('en_image.png')
image2 = confuse_image(Image.open('image.png'))

image1_data = np.array(image1)
image2_data = np.array(image2)

result_data = image1_data ^ image2_data
image_result = Image.fromarray(result_data)

image_result.save('flag.png')
```

## [Week3] 

## [Week4] 二维码2-阿喀琉斯之踵

QRazyBox 打开二维码调整Format Info Pattern

Error Correction Level 调整为 H，Mask Pattern 调整为 5，Extract QR Information 即可
