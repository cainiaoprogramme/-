# -
mooc股票信息爬虫
#requests-bs4-re
import requests
import re
from bs4 import BeautifulSoup
import traceback

def getHTMLText(url, code='utf-8'):
    try:
        r = requests.get(url, timeout = 30)
        r.raise_for_status()
        r.encoding = code
        return r.text
    except:
        return ""

def getStockList(lst, stockURL):
    html = getHTMLText(stockURL, 'GB2312')
    soup = BeautifulSoup(html, 'html.parser')   #解析页面
    a = soup.find_all('a')      #找到a标签
    for i in a:
        try:
            href = i.attrs['href']
            lst.append(re.findall(r"[s][hz]\d{6}",href)[0])     #保存各股股票编号
        except:
            continue

def getStockInfo(lst, stockURL, fpath):
    count = 0
    for stock in lst:
        url = stockURL + stock + "html"     #用百度的链接加上各股的链接形成一个访问各股页面的url
        html = getHTMLText(url)     #获取页面内容
        try:
            if html == "":  
                continue    #判断是不是空页面
            infoDict = {}   #用字典记录从当前页面返回的各股信息
            soup = BeautifulSoup(html, 'html.parser')   #解析网页
            stockInfo = soup.find('div', attrs={'class':'stock-bets'})  #根据所有股票信息都封装在div标签下，它的class是stock-bets,所以搜索标签

            name = stockInfo.find_all(attrs={'class':'bets-name'})[0]   #查找的第一个内容是股票名称，封装在这个标签中bets-name的属性的对应标签内
            infoDict.update({'股票名称':name.text.split()[0]})  #获取股票名称的第一部分，加入标签

            #所有的股票信息存储在属性的dt\dd标签中，dt标签是表明信息键的域，dd标签是信息值的域，用find_all找到所有键值域
            keyList = stockInfo.find_all('dt')
            valueList = stockInfo.find_all('dd')
            #对键值对进行还原并存到列表中
            for i in range(len(keyList)):
                key = keyList[i].text
                val = valueList[i].text
                infoDict[key] = val
            #将所有信息存入文件中
            with open(fpath, 'a', encoding='utf-8') as f:
                f.write(str(infoDict) + '\n')
                count = count + 1
                print('\r当前速度:{:.2f}%'.format(count*100/len(lst),end="")
        except:
            count = count + 1
            print('\r当前速度:{:.2f}%'.format(count*100/len(lst),end="")
            continue

def main():
    stock_list_url = 'http://quote.eastmoney.com/stocklist.html'    #获得股票链表的链接
    stock_info_url = 'https://gupiao.baidu.com/stock/'       #获得股票信息发链接的主题部分
    output_file = 'D://BaiduStockInfo.txt'      #输出目录
    slist = []
    getStockList(slist, stock_list_url)
    getStockInfo(slist, stock_info_url, output_file)

main()
