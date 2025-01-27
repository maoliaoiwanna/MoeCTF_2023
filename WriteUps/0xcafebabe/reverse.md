# Reverse
### Author: 0xcafebabe


## CHALLENGE: Reverse入门指北

将INTRO_RE.exe拖入x32dbg，找到flag: moectf{{F1rst_St3p_1s_D0ne}}

---
## CHALLENGE: base_64

[使用在线pyc反编译工具](https://tool.lu/pyc)
```py
#!/usr/bin/env python
# visit https://tool.lu/pyc/ for more information
# Version: Python 3.7

import base64
from string import *
str1 = 'yD9oB3Inv3YAB19YynIuJnUaAGB0um0='
string1 = 'ZYXWVUTSRQPONMLKJIHGFEDCBAzyxwvutsrqponmlkjihgfedcba0123456789+/'
string2 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
flag = input('welcome to moectf\ninput your flag and I wiil check it:')
enc_flag = base64.b64encode(flag.encode()).decode()
enc_flag = enc_flag.translate(str.maketrans(string2, string1))
if enc_flag == str1:
    print('good job!!!!')
else:
    print('something wrong???')
    exit(0)
```
显然base64编码后进行了文本替换，替换回来再解密就可以
```py
import base64
from string import *
str1 = 'yD9oB3Inv3YAB19YynIuJnUaAGB0um0='
string1 = 'ZYXWVUTSRQPONMLKJIHGFEDCBAzyxwvutsrqponmlkjihgfedcba0123456789+/'
string2 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
result = str1.translate(str.maketrans(string1, string2))
print(base64.b64decode(result).decode())
```
结果是moectf{pYc_And_Base64~}

---
## CHALLENGE: UPX!
```
CHALLENGE: UPX!
DESCRIPTION: 浮小云把自己珍藏的flag锁在UPX里了，这样应该就万无一失了吧？
注意：可以先做xor。如果对这道题感到困难，自行搜索学习UPX是什么。
```
下载upx.exe，随后```upx -d UPX!.exe```或者直接在x64dbg里面通过栈平衡，过解压部分

寻找关键点，发现是把输入的文本每个xor 0x67之后和已经xor的flag进行逐字节比较，并且通过分析发现字符长度是0x2A(42)bytes


直接访问```0x00007FF65C432150``` hexdump如下
```
00007FF65C466000  0A 08 02 04 13 01 1C 57 0F 38 1E 57 12 38 2C 09  .......W.8.W.8,.  
00007FF65C466010  57 10 38 2F 57 10 38 13 08 38 35 02 11 54 15 14  W.8/W.8..85..T..  
00007FF65C466020  02 38 32 37 3F 46 46 46 1A 00 00 00 00 00 00 00  .827?FFF........  
```
也就是```0A08020413011C570F381E5712382C095710382F571038130838350211541514023832373F4646461A```

编写python脚本
```py
enFlag = '0A08020413011C570F381E5712382C095710382F571038130838350211541514023832373F4646461A'
enFlag = bytes.fromhex(enFlag)
for i in enFlag:
    print(chr(i^0x67), end='')
```
获得flag:moectf{0h_y0u_Kn0w_H0w_to_Rev3rse_UPX!!!}

---
## CHALLENGE: ANDROID
```
DESCRIPTION: 简单的android，需要一点点java基础。
看不懂Java代码也可以问问AI？
如果不知道一个apk文件应该怎么分析，可以试试JADX

下载有问题看外链:https://wwi.lanzoup.com/iyX2p14ou4cb
``
hint1:找不到题目的关键逻辑在哪？可以试试使用手机或是模拟器运行观察，并查找关键字符串
```
拖入GDA4.0.8 发现如下代码
```java
       this.enc = new char[31]{25,7,0,14,27,3,16,'/',24,2,9,':',4,1,':','*',11,29,6,7,12,9,'0','T',24,':',28,21,27,28,16};
       this.key = new char[9]{'t','h','e','m','o','e','k','e','y'};

    public void onClick(View p0){
       int i2;
       String str = this.val$input.getText().toString();
       if (str.length() != 31) {
          Toast.makeText(this.this$0.getApplicationContext(), "长度不对哦", 0).show();
          return;
       }else {
          byte[] bytes = str.getBytes();
          int i = 0;
          while (true) {
             if (i < 31) {
                int i1 = i % this.this$0.key.length;
                if ((i2 = bytes[i] ^ this.this$0.key[i1]) != this.this$0.enc[i]) {
                   break ;
                }else {
                   i = i + 1;
                }
             }else {
                Toast.makeText(this.this$0.getApplicationContext(), "恭喜！回答正确", 0).show();
                return;
             }
          }
          Toast.makeText(this.this$0.getApplicationContext(), "好像有哪里不对", 0).show();
          return;
       }
    }

```
显然，flag长度31且在mod key.length的情况下逐个获取key，并与加密后的flag进行异或，我们可以直接编写代码
```py
enFlag = [25,7,0,14,27,3,16,ord('/'),24,2,9,ord(':'),4,1,ord(':'),ord('*'),11,29,6,7,12,9,ord('0'),ord('T'),24,ord(':'),28,21,27,28,16]
enKey = ['t','h','e','m','o','e','k','e','y']

for i in range(len(enFlag)):
    print(chr(enFlag[i]^ord(enKey[i%len(enKey)])), end='')
```
```
注:
使用ord是因为GDA反编译时候把一些ASCII显式字符直接写出来了，在java中直接可以当作ascii code使用但是python列表中则成为了str
```
得到结果moectf{Java_in_Android_1s_easy}

---

## CHALLENGE: RRRRRc4

直接载入x64dbg

发现```cmp rax,0x25```即key的长度

进一步调试发现又是经典的加密后比较，拿到rc4密钥moectf2023后进入在线工具 https://cyberchef.org/ 解密

```
00007FF797A26000  1B 9B FB 19 06 6A B5 3B 7C BA 03 F3 91 B8 B6 3D  ..û..jµ;|º.ó.¸¶=  
00007FF797A26010  8A C1 48 2E 50 11 E7 C7 4F B1 27 CF F3 AE 03 09  .ÁH.P.çÇO±'Ïó®..  
00007FF797A26020  B2 08 FB DC 22 00 00 00 00 00 00 00 00 00 00 00  ².ûÜ"...........  
```


拿到flag:moectf{y0u_r3a11y_understand_rc4!!!!}

## CHALLENGE: SMC
```
CHALLENGE: SMC
DESCRIPTION: smc是一种程序在运行时修改自身的方法
建议了解一些C语言及逆向基础知识再来尝试
需要的话可以尝试搜索CTF SMC并尝试例题
下载有问题看外链:https://wwi.lanzoup.com/iOqD614ou4dc
```
直接拖入x64dbg，发现流程为输入->VirtualProtect->关键段代码xor解密->运行关键段


```小提示:不要在解密前的第一个字节下断，这样会导致第一个字节变成0xCC进而解密错误，从而运行错误```

观察汇编代码，显然strlen后和[ebp-8]比较，来确认用户输入长度是否正确，随后把用户输入内容进行加密

直接拿到加密后的东西
```
0040A000  9F 91 A7 A5 94 A6 8D B5 A7 9C A6 A1 BF 91 A4 53  ..§¥.¦.µ§.¦¡¿.¤S  
0040A010  A6 53 A5 A3 94 9B 91 9E 8F 00 00 00 50 6C 7A 20  ¦S¥£........Plz   
```

然后写解密
```py
encFlag = '9F91A7A594A68DB5A79CA6A1BF91A453A653A5A3949B919E8F'
encFlag = bytes.fromhex(encFlag)
for i in encFlag:
    print(chr((i^0x39) - 0x39), end = '')
```
得到结果:moectf{Self_Mod1f1cation}

```注意:由于运算符优先性，减号大于位异或，所以要加括号！```

---

## CHALLENGE: EQUATION
```
CHALLENGE: EQUATION
DESCRIPTION: 浮小云正在挑战一个神必的数学难题，但看起来他被困住了！听说你是数学领域的大佬，现在就是你大显身手的时候了！（z3这个工具也许能帮到你
hint:ida分析变量时，可能会将数组长度分析错误，手动改一下可能会好看很多.
部分编辑器（如vscode)有批量处理数据的功能，尝试一下吧～
另外使用z3写解题脚本时，如果用BitVec设未知数解不出来不妨试试用Int设未知数。
``` 
### 直接载入ida查看，发现数组v4[24]的长度有问题，考虑更改为scanf中确定的长度(31)

### 更改后，所有变量都正常了

可以观察到，出题人非常险恶，给了一个if，然后一堆的变量，让你解，我们可以发现，if中存在着30个不等于式子，然后用||连起来，相当于如果每个条件都满足等于了，那么就可以完成题目，获得flag。
- 但是，30个式子解31个变量，显然有一个自由度，所以我们必须知道其中一部分变量的确切取值，然后再进行解决

这里我用的是z3-solver
安装方法:

```pip install z3-solver```


- 然后对于扣下来的文本处理，我们先去除文本里面所有的空格和\r\n（换行符），然后我们需要把不等号换成等号，之后再把||换成\r\n


然后我们编写python代码，通过z3-solver进行求解

```py
from z3 import *
s = Solver()

flag_length = 31
v4=[i for i in range(flag_length)]

for i in range(flag_length):
    v4[i] = BitVec("v{}".format(i), 8)

#增加flag特殊条件
s.add(v4[0]==ord('m'))
s.add(v4[1]==ord('o'))
s.add(v4[2]==ord('e'))
s.add(v4[3]==ord('c'))
s.add(v4[4]==ord('t'))
s.add(v4[5]==ord('f'))
s.add(v4[6]==ord('{'))

s.add(0x14E*v4[28]+0x64*v4[27]+369*v4[26]+124*v4[25]+278*v4[24]+158*v4[23]+162*v4[22]+145*v4[19]+27*v4[17]+91*v4[15]+195*v4[14]+342*v4[13]+391*v4[10]+204*v4[9]+302*v4[8]+153*v4[7]+292*v4[6]+382*v4[5]+221*v4[4]+316*v4[3]+118*v4[2]+295*v4[1]+247*v4[0]+236*v4[11]+27*v4[12]+361*v4[16]+81*v4[18]+105*v4[20]+65*v4[21]+67*v4[29]+41*v4[30]==596119)
s.add(371*v4[29]+338*v4[28]+269*v4[27]+312*v4[26]+67*v4[25]+299*v4[24]+235*v4[23]+294*v4[22]+303*v4[21]+211*v4[20]+122*v4[19]+333*v4[18]+341*v4[15]+111*v4[14]+253*v4[13]+68*v4[12]+347*v4[11]+44*v4[10]+262*v4[9]+357*v4[8]+323*v4[5]+141*v4[4]+329*v4[3]+378*v4[2]+316*v4[1]+235*v4[0]+59*v4[6]+37*v4[7]+264*v4[16]+73*v4[17]+126*v4[30]==634009)
s.add(337*v4[29]+338*v4[28]+118*v4[27]+82*v4[26]+239*v4[21]+58*v4[20]+304*v4[19]+330*v4[18]+377*v4[17]+306*v4[16]+221*v4[13]+345*v4[12]+124*v4[11]+272*v4[10]+270*v4[9]+229*v4[8]+377*v4[7]+373*v4[6]+297*v4[5]+112*v4[4]+386*v4[3]+90*v4[2]+361*v4[1]+236*v4[0]+386*v4[14]+73*v4[15]+315*v4[22]+33*v4[23]+141*v4[24]+129*v4[25]+123*v4[30]==685705)
s.add(367*v4[29]+55*v4[28]+374*v4[27]+150*v4[24]+350*v4[23]+141*v4[22]+124*v4[21]+366*v4[20]+230*v4[19]+307*v4[18]+191*v4[17]+153*v4[12]+383*v4[11]+145*v4[10]+109*v4[9]+209*v4[8]+158*v4[7]+221*v4[6]+188*v4[5]+22*v4[4]+146*v4[3]+306*v4[2]+230*v4[1]+13*v4[0]+287*v4[13]+257*v4[14]+137*v4[15]+7*v4[16]+52*v4[25]+31*v4[26]+355*v4[30]==557696)
s.add(100*v4[29]+191*v4[28]+362*v4[27]+55*v4[26]+210*v4[25]+359*v4[24]+348*v4[21]+83*v4[20]+395*v4[19]+350*v4[16]+291*v4[15]+220*v4[12]+196*v4[11]+399*v4[8]+68*v4[7]+84*v4[6]+281*v4[5]+334*v4[4]+53*v4[3]+399*v4[2]+338*v4[0]+18*v4[1]+148*v4[9]+21*v4[10]+174*v4[13]+36*v4[14]+2*v4[17]+41*v4[18]+137*v4[22]+24*v4[23]+368*v4[30]==538535)
s.add(188*v4[29]+(v4[26]<<7)+93*v4[25]+248*v4[24]+83*v4[23]+207*v4[22]+217*v4[19]+309*v4[16]+16*v4[15]+135*v4[14]+251*v4[13]+200*v4[12]+49*v4[11]+119*v4[10]+356*v4[9]+398*v4[8]+303*v4[7]+224*v4[6]+208*v4[5]+244*v4[4]+209*v4[3]+189*v4[2]+302*v4[1]+395*v4[0]+314*v4[17]+13*v4[18]+310*v4[20]+21*v4[21]+67*v4[27]+127*v4[28]+100*v4[30]==580384)
s.add(293*v4[29]+343*v4[28]+123*v4[27]+387*v4[26]+114*v4[25]+303*v4[24]+248*v4[23]+258*v4[21]+218*v4[20]+180*v4[19]+196*v4[18]+398*v4[17]+398*v4[14]+138*v4[9]+292*v4[8]+38*v4[7]+179*v4[6]+190*v4[5]+57*v4[4]+358*v4[3]+191*v4[2]+215*v4[1]+88*v4[0]+22*v4[10]+72*v4[11]+357*v4[12]+9*v4[13]+389*v4[15]+81*v4[16]+85*v4[30]==529847)
s.add(311*v4[29]+202*v4[28]+234*v4[27]+272*v4[26]+55*v4[25]+328*v4[24]+246*v4[23]+362*v4[22]+86*v4[21]+75*v4[20]+142*v4[17]+244*v4[16]+216*v4[15]+281*v4[14]+398*v4[13]+322*v4[12]+251*v4[11]+357*v4[8]+76*v4[7]+292*v4[6]+389*v4[5]+275*v4[4]+312*v4[3]+200*v4[2]+110*v4[1]+203*v4[0]+99*v4[9]+21*v4[10]+269*v4[18]+33*v4[19]+356*v4[30]==631652)
s.add(261*v4[29]+189*v4[26]+55*v4[25]+23*v4[24]+202*v4[23]+185*v4[22]+182*v4[21]+285*v4[20]+217*v4[17]+157*v4[16]+232*v4[15]+132*v4[14]+169*v4[13]+154*v4[12]+121*v4[11]+389*v4[10]+376*v4[9]+292*v4[6]+225*v4[5]+155*v4[4]+234*v4[3]+149*v4[2]+241*v4[1]+312*v4[0]+368*v4[7]+129*v4[8]+226*v4[18]+288*v4[19]+201*v4[27]+288*v4[28]+69*v4[30]==614840)
s.add(60*v4[29]+118*v4[28]+153*v4[27]+139*v4[26]+23*v4[25]+279*v4[24]+396*v4[23]+287*v4[22]+237*v4[19]+266*v4[18]+149*v4[17]+193*v4[16]+395*v4[15]+97*v4[14]+16*v4[13]+286*v4[12]+105*v4[11]+88*v4[10]+282*v4[9]+55*v4[8]+134*v4[7]+114*v4[6]+101*v4[5]+116*v4[4]+271*v4[3]+186*v4[2]+263*v4[1]+313*v4[0]+149*v4[20]+129*v4[21]+145*v4[30]==510398)
s.add(385*v4[29]+53*v4[28]+112*v4[27]+8*v4[26]+232*v4[25]+145*v4[24]+313*v4[23]+156*v4[22]+321*v4[21]+358*v4[20]+46*v4[19]+382*v4[18]+144*v4[16]+222*v4[14]+329*v4[13]+161*v4[12]+335*v4[11]+50*v4[10]+373*v4[9]+66*v4[8]+44*v4[7]+59*v4[6]+292*v4[5]+39*v4[4]+53*v4[3]+310*v4[0]+154*v4[1]+24*v4[2]+396*v4[15]+81*v4[17]+355*v4[30]==558740)
s.add(249*v4[29]+386*v4[28]+313*v4[27]+74*v4[26]+22*v4[25]+168*v4[24]+305*v4[21]+358*v4[20]+191*v4[19]+202*v4[18]+14*v4[15]+114*v4[14]+224*v4[13]+134*v4[12]+274*v4[11]+372*v4[10]+159*v4[9]+233*v4[8]+70*v4[7]+287*v4[6]+297*v4[5]+318*v4[4]+177*v4[3]+173*v4[2]+270*v4[1]+163*v4[0]+77*v4[16]+25*v4[17]+387*v4[22]+18*v4[23]+345*v4[30]==592365)
s.add(392*v4[29]+385*v4[28]+302*v4[27]+13*v4[25]+27*v4[24]+99*v4[22]+343*v4[19]+324*v4[18]+223*v4[17]+372*v4[16]+261*v4[15]+181*v4[14]+203*v4[13]+232*v4[12]+305*v4[11]+393*v4[10]+325*v4[9]+231*v4[8]+92*v4[7]+142*v4[6]+22*v4[5]+86*v4[4]+264*v4[3]+300*v4[2]+387*v4[1]+360*v4[0]+225*v4[20]+127*v4[21]+2*v4[23]+80*v4[26]+268*v4[30]==619574)
s.add(270*v4[28]+370*v4[27]+235*v4[26]+96*v4[22]+85*v4[20]+150*v4[19]+140*v4[18]+94*v4[17]+295*v4[16]+19*v4[14]+176*v4[12]+94*v4[11]+258*v4[10]+302*v4[9]+171*v4[8]+66*v4[7]+278*v4[6]+193*v4[5]+251*v4[4]+284*v4[3]+218*v4[2]+(v4[1]<<6)+319*v4[0]+125*v4[13]+24*v4[15]+267*v4[21]+160*v4[23]+111*v4[24]+33*v4[25]+174*v4[29]+13*v4[30]==480557)
s.add(87*v4[28]+260*v4[27]+326*v4[26]+210*v4[25]+357*v4[24]+170*v4[23]+315*v4[22]+376*v4[21]+227*v4[20]+43*v4[19]+358*v4[18]+364*v4[17]+309*v4[16]+282*v4[15]+286*v4[14]+365*v4[13]+287*v4[12]+377*v4[11]+74*v4[10]+225*v4[9]+328*v4[6]+223*v4[5]+120*v4[4]+102*v4[3]+162*v4[2]+123*v4[1]+196*v4[0]+29*v4[7]+27*v4[8]+352*v4[30]==666967)
s.add(61*v4[29]+195*v4[28]+125*v4[27]+(v4[26]<<6)+260*v4[25]+202*v4[24]+116*v4[23]+230*v4[22]+326*v4[21]+211*v4[20]+371*v4[19]+353*v4[16]+124*v4[13]+188*v4[12]+163*v4[11]+140*v4[10]+51*v4[9]+262*v4[8]+229*v4[7]+100*v4[6]+113*v4[5]+158*v4[4]+378*v4[3]+365*v4[2]+207*v4[1]+277*v4[0]+190*v4[14]+320*v4[15]+347*v4[17]+11*v4[18]+137*v4[30]==590534)
s.add(39*v4[28]+303*v4[27]+360*v4[26]+157*v4[25]+324*v4[24]+77*v4[23]+308*v4[22]+313*v4[21]+87*v4[20]+201*v4[19]+50*v4[18]+60*v4[17]+28*v4[16]+193*v4[15]+184*v4[14]+205*v4[13]+140*v4[12]+311*v4[11]+304*v4[10]+35*v4[9]+356*v4[8]+23*v4[5]+85*v4[4]+156*v4[3]+16*v4[2]+26*v4[1]+157*v4[0]+150*v4[6]+72*v4[7]+58*v4[29]==429108)
s.add(157*v4[29]+137*v4[28]+71*v4[27]+269*v4[26]+161*v4[25]+317*v4[20]+296*v4[19]+385*v4[18]+165*v4[13]+159*v4[12]+132*v4[11]+296*v4[10]+162*v4[7]+254*v4[4]+172*v4[3]+132*v4[0]+369*v4[1]+257*v4[2]+134*v4[5]+384*v4[6]+53*v4[8]+255*v4[9]+229*v4[14]+129*v4[15]+23*v4[16]+41*v4[17]+112*v4[21]+17*v4[22]+222*v4[23]+96*v4[24]+126*v4[30]==563521)
s.add(207*v4[29]+83*v4[28]+111*v4[27]+35*v4[26]+67*v4[25]+138*v4[22]+223*v4[21]+142*v4[20]+154*v4[19]+111*v4[18]+341*v4[17]+175*v4[16]+259*v4[15]+225*v4[14]+26*v4[11]+334*v4[10]+250*v4[7]+198*v4[6]+279*v4[5]+301*v4[4]+193*v4[3]+334*v4[2]+134*v4[0]+37*v4[1]+183*v4[8]+5*v4[9]+270*v4[12]+21*v4[13]+275*v4[23]+48*v4[24]+163*v4[30]==493999)
s.add(393*v4[29]+176*v4[28]+105*v4[27]+162*v4[26]+148*v4[25]+281*v4[24]+300*v4[23]+342*v4[18]+262*v4[17]+152*v4[12]+43*v4[11]+296*v4[10]+273*v4[9]+75*v4[6]+18*v4[4]+217*v4[2]+132*v4[1]+112*v4[0]+210*v4[3]+72*v4[5]+113*v4[7]+40*v4[8]+278*v4[13]+24*v4[14]+77*v4[15]+11*v4[16]+55*v4[19]+255*v4[20]+241*v4[21]+13*v4[22]+356*v4[30]==470065)
s.add(369*v4[29]+231*v4[28]+285*v4[25]+290*v4[24]+297*v4[23]+189*v4[22]+390*v4[21]+345*v4[20]+153*v4[19]+114*v4[18]+251*v4[17]+340*v4[16]+44*v4[15]+58*v4[14]+335*v4[13]+359*_v4[12]+392*v4[11]+181*v4[8]+103*v4[7]+229*v4[6]+175*v4[5]+208*v4[4]+92*v4[3]+397*v4[2]+349*v4[1]+356*v4[0]+(v4[9]<<6)+5*v4[10]+88*v4[26]+40*v4[27]+295*v4[30]==661276)
s.add(341*v4[27]+40*v4[25]+374*v4[23]+201*v4[22]+77*v4[21]+215*v4[20]+283*v4[19]+213*v4[18]+392*v4[17]+224*v4[16]+v4[15]+270*v4[12]+28*v4[11]+75*v4[8]+386*v4[7]+298*v4[6]+170*v4[5]+287*v4[4]+247*v4[3]+204*v4[2]+103*v4[1]+21*v4[0]+84*v4[9]+27*v4[10]+159*v4[13]+192*v4[14]+213*v4[24]+129*v4[26]+67*v4[28]+27*v4[29]+361*v4[30]==555288)
s.add(106*v4[29]+363*v4[28]+210*v4[27]+171*v4[26]+289*v4[25]+240*v4[24]+164*v4[23]+342*v4[22]+391*v4[19]+304*v4[18]+218*v4[17]+32*v4[16]+350*v4[15]+339*v4[12]+303*v4[11]+222*v4[10]+298*v4[9]+47*v4[8]+48*v4[6]+264*v4[4]+113*v4[3]+275*v4[2]+345*v4[1]+312*v4[0]+171*v4[5]+384*v4[7]+175*v4[13]+5*v4[14]+113*v4[20]+19*v4[21]+263*v4[30]==637650)
s.add(278*v4[29]+169*v4[28]+62*v4[27]+119*v4[26]+385*v4[25]+289*v4[24]+344*v4[23]+45*v4[20]+308*v4[19]+318*v4[18]+270*v4[17]+v4[16]+323*v4[15]+332*v4[14]+287*v4[11]+170*v4[10]+163*v4[9]+301*v4[8]+303*v4[7]+23*v4[6]+327*v4[5]+169*v4[3]+28*v4[0]+365*v4[1]+15*v4[2]+352*v4[12]+72*v4[13]+140*v4[21]+65*v4[22]+346*v4[30]==572609)
s.add(147*v4[29]+88*v4[28]+143*v4[27]+237*v4[26]+63*v4[24]+281*v4[22]+388*v4[21]+142*v4[20]+208*v4[19]+60*v4[18]+354*v4[15]+88*v4[14]+146*v4[13]+290*v4[12]+349*v4[11]+43*v4[10]+230*v4[9]+267*v4[6]+136*v4[5]+383*v4[4]+35*v4[3]+226*v4[2]+385*v4[1]+238*v4[0]+348*v4[7]+20*v4[8]+158*v4[16]+21*v4[17]+249*v4[23]+9*v4[25]+343*v4[30]==603481)
s.add(29*v4[29]+323*v4[26]+159*v4[25]+118*v4[20]+326*v4[19]+211*v4[18]+225*v4[17]+355*v4[16]+201*v4[15]+149*v4[14]+296*v4[13]+184*v4[12]+315*v4[11]+364*v4[10]+142*v4[9]+75*v4[8]+313*v4[7]+142*v4[6]+396*v4[5]+348*v4[4]+272*v4[3]+26*v4[2]+206*v4[1]+173*v4[0]+155*v4[21]+144*v4[22]+366*v4[23]+257*v4[24]+148*v4[27]+24*v4[28]+253*v4[30]==664504)
s.add(4*v4[29]+305*v4[28]+226*v4[27]+212*v4[26]+175*v4[25]+93*v4[24]+165*v4[23]+341*v4[20]+14*v4[19]+394*v4[18]+(v4[17]<<8)+252*v4[16]+336*v4[15]+38*v4[14]+82*v4[13]+155*v4[12]+215*v4[11]+331*v4[10]+230*v4[9]+241*v4[8]+225*v4[7]+186*v4[4]+90*v4[3]+50*v4[2]+62*v4[1]+34*v4[0]+237*v4[5]+11*v4[6]+336*v4[21]+36*v4[22]+29*v4[30]==473092)
s.add(353*v4[29]+216*v4[28]+252*v4[27]+8*v4[26]+62*v4[25]+233*v4[24]+254*v4[23]+303*v4[22]+234*v4[21]+303*v4[20]+(v4[19]<<8)+148*v4[18]+324*v4[17]+317*v4[16]+213*v4[15]+309*v4[14]+28*v4[13]+280*v4[11]+118*v4[10]+58*v4[9]+50*v4[8]+155*v4[7]+161*v4[6]+(v4[5]<<6)+303*v4[4]+76*v4[3]+43*v4[2]+109*v4[1]+102*v4[0]+93*v4[30]==497492)
s.add(89*v4[29]+148*v4[28]+82*v4[27]+53*v4[26]+274*v4[25]+220*v4[24]+202*v4[23]+123*v4[22]+231*v4[21]+169*v4[20]+278*v4[19]+259*v4[18]+208*v4[17]+219*v4[16]+371*v4[15]+181*v4[12]+104*v4[11]+392*v4[10]+285*v4[9]+113*v4[8]+298*v4[7]+389*v4[6]+322*v4[5]+338*v4[4]+237*v4[3]+234*v4[0]+261*v4[1]+10*v4[2]+345*v4[13]+3*v4[14]+361*v4[30]==659149)
s.add(361*v4[29]+359*v4[28]+93*v4[27]+315*v4[26]+69*v4[25]+137*v4[24]+69*v4[23]+58*v4[22]+300*v4[21]+371*v4[20]+264*v4[19]+317*v4[18]+215*v4[17]+155*v4[16]+215*v4[15]+330*v4[14]+239*v4[13]+212*v4[12]+88*v4[11]+82*v4[10]+354*v4[9]+85*v4[8]+310*v4[7]+84*v4[6]+374*v4[5]+380*v4[4]+215*v4[3]+351*v4[2]+141*v4[1]+115*v4[0]+108*v4[30]==629123)


flag = []
if s.check() == sat:
    ans = s.model()
    for i in range(flag_length):   
        flag.append(ans[v4[i]])
else:
    print("unable to solve...")
for x in flag:
    print(chr(int(str(x))),end="")
```

运行后得到结果
```cmd
C:\Users\Administrator\Desktop\ctf>python test.py
moectf{y0u_s0lv3d_Equati0ns!!!}
```

---

## CHALLENGE: junk_code
```
CHALLENGE: junk_code
DESCRIPTION: 只要一两个花指令就能让IDA不能f5，IDA一片飘红，真不戳啊~
```
注:花指令根本不会影响x32dbg的动态调试，这个题是送给你300分的送分题

拖进x32dbg，找到关键点，发现%36s，说明flag长度36，然后继续找


观察到sub ecx, 0x5，每个字节都是这么处理的，然后下面又和
```"hj`^oavt+pZm`h+q._"```进行比较，所以我们直接写脚本
```py
str1 = '"hj`^oavt+pZm`h+q._"'
for i in str1:
   print(chr(ord(i)+ 0x5), end='')
```
拿到```moectf{y0u_rem0v3d```，显然这是一半的flag，我们输入正确的前半部分flag，然后后面随便输入，长度超过36就行，然后在后半部分下硬件断点，运行程序。

### 发现程序将后半部分逐个xor 0x66，然后调用了一个函数，传递了junk_code.B26350

```
00B26E50  39 12 0E 55 39 0C 13 08 0D 39 05 56 02 55 47 47  9..U9....9.V.UGG  
00B26E60  47 1B 00 00 00 00 00 00 77 65 6C 63 6F 6D 65 20  G.......welcome   
```

很显然，这就是我们需要的flag，我们编写脚本

```py
enFlag = '39120E55390C13080D39055602554747471B'
enFlag = bytes.fromhex(enFlag)
for i in enFlag:
    print(chr(i^0x66), end='')
```

获得```_th3_junk_c0d3!!!}```

将所得到的flag拼接，得到最终答案```moectf{y0u_rem0v3d_th3_junk_c0d3!!!}```

### 小结：这个题不值300分，不管从花指令还是从两段不同的解密

---
## CHALLENGE: ezandroid

我们直接找到libezandroid.so，拖入IDA 64，查看JNI_OnLoad函数


于是我们发现了一串奇怪的字符串```******************@**************.************...****#..*****.********.*****.****.....*****.****.*********......***********************```，我们先继续往下看

由于懒得导入JNI.h文件，看到了```sub_17F0(v4, (__int64)"com/doctor3/ezandroid/MainActivity");```后，我们可以大胆猜测这个函数是FindClass函数，v4应该是jni_env，然后我直接猜测是动态注册(RegisterNatives)，点开sub_1820的第三个参数

我们发现了JAVA函数名及其sig，大快人心，看来猜对了，我们就可以直接点进去sig下面的那个，就是动态注册的函数地址

### 点进去后找到了如下代码

w a s d...在看到+=15 -=15 可以基本确定是一个"走迷宫题目"，而且迷宫的长度是15
我们将神秘字符串按每行15个字符进行分行
```
***************
***@***********
***.***********
*...****#..****
*.********.****
*.****.....****
*.****.********
*......********
***************
```
然后走迷宫:```ssaassssdddddwwddddwwa```

```moectf{ssaassssdddddwwddddwwa}```即为flag


## CHALLENGE: unwind

虽然题目很长，但是看到SEH，就说明又是一个静态不友好题目，我们直接上x32dbg

### 先大致看一眼

发现确实是通过SEH来改变控制流，所以我们不和他正面刚

在input后下断，拿到你刚刚输入的文本地址，然后直接下硬件访问断点，跑起来，然后就会断在读取的地方。

显然有是一处加密算法，我们获取到函数头这里的RVA，上IDA查看

我们再去搜索TEA加密算法（因为题目已经提醒你是TEA了）
编写cpp代码
```cpp
//加密函数
void encrypt(unsigned int* v, unsigned int* k) {
	unsigned int v0 = v[0], v1 = v[1], sum = 0, i; /* set up */
	unsigned int delta = 0x9e3779b9; /* a key schedule constant */
	unsigned int k0 = k[0], k1 = k[1], k2 = k[2], k3 = k[3]; /* cache key */
	for (i = 0; i < 32; i++) { /* basic cycle start */
		sum += delta;
		v0 += ((v1 << 4) + k0) ^ (v1 + sum) ^ ((v1 >> 5) + k1);
		v1 += ((v0 << 4) + k2) ^ (v0 + sum) ^ ((v0 >> 5) + k3);
	} /* end cycle */
	v[0] = v0; v[1] = v1;
}
//解密函数
void decrypt(unsigned int* v, unsigned int* k) {
	unsigned int v0 = v[0], v1 = v[1], sum = 0xC6EF3720, i; /* set up */
	unsigned int delta = 0x9e3779b9; /* a key schedule constant */
	unsigned int k0 = k[0], k1 = k[1], k2 = k[2], k3 = k[3]; /* cache key */
	for (i = 0; i < 32; i++) { /* basic cycle start */
		v1 -= ((v0 << 4) + k2) ^ (v0 + sum) ^ ((v0 >> 5) + k3);
		v0 -= ((v1 << 4) + k0) ^ (v1 + sum) ^ ((v1 >> 5) + k1);
		sum -= delta;
	} /* end cycle */
	v[0] = v0; v[1] = v1;
}
```

经过对比，我们发现善良的出题人并没有改变TEA加密算法的一些参数或数据，我们直接开始解密

观察到这个函数的Caller又两个参数，经过进一步剖析，我们发现```DX3906,doctor3, FUX1AOYUN, R3verier分别是四次TEA加密的密钥，总共进行8轮，每一轮加密4*2字节，前四轮加密一遍，而后面4次进行了两次加密```，由于TEA加密算法是对称加密，我们可以直接进行解密。(在输入数据时记得注意大小端问题)

找到加密后的flag
```
// 下面的4(每次加密字节)*2(每轮)*4(轮数)字节是加密一次后的
00F5A000  5A E3 6B E4 06 87 02 4F | 43 DF CD C1 77 98 6B DB  Zãkä...OCßÍÁw.kÛ  
00F5A010  8F 38 43 99 E3 93 22 B5 | 23 FD B0 1C E5 E3 EE CE  .8C.ã."µ#ý°.åãîÎ  

// 下面的4(每次加密字节)*2(每轮)*4(轮数)字节是加密两次后的
00F5A020  2F 1D AD 2B A4 15 98 F9 | D8 EB 25 FA 6B 21 B7 72  /..+¤..ùØë%úk!·r  
00F5A030  B9 03 33 2E D9 4C EB 7B | F5 A7 48 F9 90 9D 38 FC  ¹.3.ÙLë{õ§Hù..8ü  
```

```cpp
unsigned int k[4] = { 0 };

unsigned int v1[2] = { 0xe46be35a, 0x4f028706 };
memcpy_s(k, sizeof(k), "DX3906" "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0", sizeof(k));
decrypt(v1, k);
printf("1 - 解密后的数据：%u %u\n", v1[0], v1[1]);

unsigned int v2[2] = { 0xc1cddf43, 0xdb6b9877 };
memcpy_s(k, sizeof(k), "doctor3" "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0", sizeof(k));
decrypt(v2, k);
printf("2 - 解密后的数据：%u %u\n", v2[0], v2[1]);

unsigned int v3[2] = { 0x9943388f, 0xb52293e3 };
memcpy_s(k, sizeof(k), "FUX1AOYUN" "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0", sizeof(k));
decrypt(v3, k);
printf("3 - 解密后的数据：%u %u\n", v3[0], v3[1]);

unsigned int v4[2] = { 0x1cb0fd23, 0xceeee3e5 };
memcpy_s(k, sizeof(k), "R3verier" "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0", sizeof(k));
decrypt(v4, k);
printf("4 - 解密后的数据：%u %u\n", v4[0], v4[1]);

unsigned int v5[2] = { 0x2bad1d2f, 0xf99815a4 };
memcpy_s(k, sizeof(k), "DX3906" "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0", sizeof(k));
decrypt(v5, k);
decrypt(v5, k);
printf("5 - 解密后的数据：%u %u\n", v5[0], v5[1]);

unsigned int v6[2] = { 0xfa25ebd8, 0x72b7216b };
memcpy_s(k, sizeof(k), "doctor3" "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0", sizeof(k));
decrypt(v6, k);
decrypt(v6, k);
printf("6 - 解密后的数据：%u %u\n", v6[0], v6[1]);

unsigned int v7[2] = { 0x2e3303b9, 0x7beb4cd9 };
memcpy_s(k, sizeof(k), "FUX1AOYUN" "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0", sizeof(k));
decrypt(v7, k);
decrypt(v7, k);
printf("7 - 解密后的数据：%u %u\n", v7[0], v7[1]);

unsigned int v8[2] = { 0xf948a7f5, 0xfc389d90 };
memcpy_s(k, sizeof(k), "R3verier" "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0", sizeof(k));
decrypt(v8, k);
decrypt(v8, k);
printf("8 - 解密后的数据：%u %u\n", v8[0], v8[1]);
```

得到
```
1 - 解密后的数据： 1667592045 1467704948
2 - 解密后的数据： 812601199 2003783472
3 - 解密后的数据： 1597002591 1702129257
4 - 解密后的数据： 1953719666 1600613993
5 - 解密后的数据： 1599418489 1601057859
6 - 解密后的数据： 1815163971 1949519199
7 - 解密后的数据： 862138711 1599685215
8 - 解密后的数据： 829910613 2105435214
```
转换为hex
```
63656F6D 577B6674 306F4F6F 776F4F30
5F30535F 65746E69 74736572 5F676E69
5F553079 5F6E3443 6C313443 74334D5F
33633157 5F59425F 31776E55 7D7E644E
```


### 得到flag
moectf{WoOo00Oow_S0_interesting_y0U_C4n_C41l_M3tW1c3_BY_Unw1Nd~}

### 本题总结
本题较难，不过如果使用python解密TEA可能在代码编写上会简单一些


## CHALLENGE: GUI 

### 由于时间仓促，直接上x64dbg了，本题没有进行详细分析，直接输入
```
abcdef...zABCDEF...Z012...9
~!@#$%^&*()-=[]\;',./_+{}|:"<>?
```
分别查看它们加密后的值（因为本题的加密是字节无关的，不会影响其他字节），然后直接查看加密后的flag，一一对应回来就行！

### 赛后思考：其实可以拖入IDA，控制流显然（不过本人IDA用的不熟练。


## CHALLENGE: "天网"

### 没在赛中做出来，因为FlagMachine太多了，感觉应该是可以做自动化分析ILcode。
