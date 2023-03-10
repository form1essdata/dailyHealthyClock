# 如何优雅的完成每日健康打卡？

## 1.脚本代码部分

### 采用python语言， 使用selenium库完成脚本自动化。

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium import webdriver
import datetime
#引入selenium、time和datetime库。


chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--ignore-certificate-errors')
chrome_options.add_argument('--ignore-ssl-errors')         
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-gpu')
chrome_options.add_argument('--disable-dev-shm-usage')
chrome_options.add_experimental_option("excludeSwitches", ['enable-automation', 'enable-logging'])
# 使用chrome驱动chrome浏览器，并使用上述代码避免命令行输出过多日志。


driver = webdriver.Chrome(executable_path="D:/dk/chromedriver.exe", options=chrome_options) #  定义驱动器目录


#用selenium功能创建下面两个函数（定位点击和定位输入）来简化代码，
def f_c(xpath):
    a = driver.find_element(By.XPATH,xpath).click()                 
#创建函数：定位(find)并点击(click)，xpath是定位位置


def f_s(xpath,b):
    a = driver.find_element(By.XPATH,xpath).send_keys(b)          
#创建函数：定位(find)并输入(shuru)，xpath是定位位置,b是文本内容


#创建页面操作全过程函数
def fullProcess(name,psw):                                         
    driver.get('目标网址')   
#打开健康打卡网址，部署在企业微信、微信小程序的打卡程序有很大概率能找到网页链接。    
    f_s('//*[@id="userId"]',name)
    f_s('//*[@id="pwd"]',psw)
    f_c('//*[@id="qdtj_btn"]')
    time.sleep(1)
#分析学校打卡页面为两部分，第一部分为输入账号密码并点击登录按钮。
#完成后需等待1秒

    f_c('/html/body/div[1]/div[4]/div[2]/div/div[2]/div/div/div/div[5]/label')#住宿地：省内
    f_c('/html/body/div[1]/div[4]/div[4]/div/div/div/div/div/div[1]/label')#是否返回校本部：否
    f_c('/html/body/div[1]/div[4]/div[5]/div/div/div/div/div/div[1]/label')#体温数据:正常
    f_c('/html/body/div[1]/div[4]/div[6]/div/div/div/div/div/div[1]/label')#家庭成员：否
    f_c('/html/body/div[1]/div[4]/div[7]/div/div/div/div/div/div[1]/label')#本人感染期间：否
    f_c('/html/body/div[1]/div[4]/label')#本人承诺
    f_c('/html/body/div[1]/div[5]/a')#提交
#第二部分为填写各项指标内容最后点击提交


#这时，你只需要运行 fullProcess('输入你的账号'，'输入你的密码')就可以完成半自动打卡。
#但如果你乐于助人，想帮助你的爱人，室友和同学优雅地进行每日打卡，那么可以继续向下编写。



#因为人数变得多，所以需要一环节来判断是否签到成功
#使用try-execpt判断操作是否失败，并将fullProcess封装到sec_jud(second judgement)函数中
def sec_jud(a,b,c):                  
    try:
        fullProcess(a,b)
        print('签到成功   此时时间是：',datetime.datetime.now(),c)
    except:
        print('*WARNING签到失败 此时时间是：',datetime.datetime.now(),a)
    time.sleep(1)
    
    
#创建类，更好的管理人员名单。
class userDate:
    def __init__(self,name,studentID,password):
        self.name = name
        self.studentID = studentID
        self.password = password
#定义go函数，层层封装，最终通过go运行second judgement函数
    def go(self):
        sec_jud(self.studentID,self.password,self.name)

        
#将各个账户数据以空格隔开，以账号密码昵称顺序隔开，换行继续输入，存入list.txt文件
#读取list文件，存入类，并运行go函数
def nuadk():
    with open('D:/nuadk/nuadk/list.txt') as f:
        content = f.readline()
        num = 0
        while content != '':
            num += 1
            res = content.split()
            print("读取第%d位，%s"%(num,res[2]))
            tem_name = userDate(res[2],res[0],res[1]).go()
            content = f.readline()
	driver.quit()


#运行最终函数
nuadk()

```



## 2.服务器部署自动化（linux-ubuntu）

因为校内定时断网等操作，必须将程序稳定运行在可靠环境中。各大厂服务器选择配置最低的即可。

```shell
crontab -e
#使用crontab定时运行上述脚本程序

01 0 * * * python /home/daka.py
#在crontab中写入定时指令
```





```shell
#服务器每日准时打卡
#打卡完毕后将数据发送至邮箱
```

