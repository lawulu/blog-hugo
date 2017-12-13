+++
title = "广协的IP库到底准不住"
draft = false
date = "2017-02-25"
Categories = ["促进大数据发展行动纲要"] 
Description = "" 
Tags = ["ip","python"] 
toc = true

+++

## 缘起
对于CPM广告，地域定向是非常重要的。广告协会提供了ip库，方便AdNetwork和第三方监控做地域定向。百度默认以ip138做为其"IP"关键字的第一个入口，我相信ip138的库是比较准的，那广协的IP库和ip138的库差别大吗？
## 数据分析
### 随机从线上提取10000个ip，并使用广协库解析成对应城市
从某个时段原始Nginx日志中的ip(300W+)随机取10000条
使用pandas
```
csv = pd.read_csv('/tmp/ip.log')
np.savetxt('/tmp/ipSample.log',csv.sample(10000).values,fmt='%s')
```

### 根据ip获取ip138对应的城市
```
reload(sys)
sys.setdefaultencoding('utf-8')

# get data from web
def get_data(query):
    url = "http://m.ip138.com/ip.asp?ip=" + query
    res = requests.get(url)
    print 'get data for '+ query +' result:' + str(res.status_code)
    if(res.status_code != 200):
        return 'Failed Response'
    else:
        return parse_html(res.content)

def parse_html(html):
    res = BeautifulSoup(html,"html.parser").find_all("p", "result")
    result =split(res[0])
    if len(res)>1:
        result=result+","+ split(res[1])
    return result

def split(str):
    try:
        splits=str.string.encode("utf-8").split('：')
        return splits[1]
    except RuntimeError:
        return 'Split Error:'+str

with open('/tmp/ipSample.log') as source:
    with open('ip138Result.csv','aw') as result:
        for line in source:
            ip = line.strip('\n')
            ipStr=get_data(ip)
            time.sleep(0.1)
            result.write(ip+'|'+ipStr+'\n')

```
### 取全国60强城市，统计两个结果的差距

http://www.mnw.cn/news/china/886245.html 60强城市

```
citys=['上海','北京','广州','深圳','成都','重庆','杭州','南京','沈阳','苏州','天津','武汉','西安','长沙','大连','济南','宁波','青岛','无锡','厦门','郑州','长春','常州','哈尔滨','福州','昆明','合肥','东莞','石家庄','呼和浩特','南昌','温州','佛山','贵阳','南宁','海口','湖州','唐山','临沂','嘉兴','绍兴','南通','徐州','泉州','太原','烟台','乌鲁木齐','潍坊','珠海','洛阳','中山','兰州','金华','淮安','吉林','威海','淄博','银川','扬州','芜湖','盐城','宜昌','西宁','襄阳','绵阳']
files =["ip138Result.csv","ipResultV2.log"]
import subprocess, datetime, sys
def checkNums(file,city):
    p = subprocess.Popen(""" cat """ + file + """| grep """+ city +"""  |wc -l """, shell=True, stdout=subprocess.PIPE );
    p.wait()
    out = p.stdout.readlines()[0]
    return str(out).strip()

for city in citys:
    result = city + "\t"
    for file in files:
        result = result +checkNums(file,city)+ "\t"
    print  result
```

## 结论
**北京相差比较大，系统库比Ip138库多出一倍**

其他城市 上海 深圳 南京 苏州 大连 宁波等也相差在20%以上。
可以说跟IP138差别还是比较大的。

但是三线城市比较准。
