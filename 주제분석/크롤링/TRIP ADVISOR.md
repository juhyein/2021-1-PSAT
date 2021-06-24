```python
import requests
from bs4 import BeautifulSoup
import time

import numpy as np
import pandas as pd
```


```python
import selenium
from selenium import webdriver
from selenium.webdriver import ActionChains
from selenium.webdriver import Chrome

from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By

from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import Select
from selenium.webdriver.support.ui import WebDriverWait
```

# 데이터 정리


```python
base_link = pd.read_csv('가게이름-주소.csv', encoding='EUC-KR')
```


```python
links = base_link[base_link['food_link'].str.contains('https')==True]
links = pd.DataFrame(links['food_link']).reset_index(drop=True)
links
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>food_link</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>https://www.tripadvisor.co.kr/Restaurant_Revie...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>https://www.tripadvisor.co.kr/Restaurant_Revie...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>https://www.tripadvisor.co.kr/Restaurant_Revie...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>https://www.tripadvisor.co.kr/Restaurant_Revie...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>https://www.tripadvisor.co.kr/Restaurant_Revie...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>1448</th>
      <td>https://www.tripadvisor.co.kr/Restaurant_Revie...</td>
    </tr>
    <tr>
      <th>1449</th>
      <td>https://www.tripadvisor.co.kr/Restaurant_Revie...</td>
    </tr>
    <tr>
      <th>1450</th>
      <td>https://www.tripadvisor.co.kr/Restaurant_Revie...</td>
    </tr>
    <tr>
      <th>1451</th>
      <td>https://www.tripadvisor.co.kr/Restaurant_Revie...</td>
    </tr>
    <tr>
      <th>1452</th>
      <td>https://www.tripadvisor.co.kr/Restaurant_Revie...</td>
    </tr>
  </tbody>
</table>
<p>1453 rows × 1 columns</p>
</div>




```python
links.iloc[933][0]
```

# 내 계획
- 일단 링크 하나로 들어가서 어떻게 되는지 보고
- 전체를 돌리려고 해
- 가게이름-주소 데이터의 인덱스가 큰 의미 없는 상황이라 리뷰만 따로 빼서, 리뷰를 키로 잡고 merge할 수 있도록 하려고 함


```python
std_link = links.iloc[0][0]
```


```python
driver = Chrome()
driver.maximize_window()
driver.get(url = std_link)
```

## 음식점 기본 정보 크롤링


```python
# 랭크
main_rank = driver.find_element_by_css_selector('#component_44 > div > div:nth-child(2) > span._13OzAOXO._2VxaSjVD > a > span > b > span').text[-2:]

# 이름
main_name = driver.find_element_by_css_selector('#component_44 > div > div._1hkogt_o > h1').text
# 별점
main_rate = driver.find_element_by_css_selector('#component_44 > div > div:nth-child(2) > span:nth-child(1) > a > svg').get_attribute('aria-label')
main_rate = main_rate[8:]

# 리뷰 전체 개수
main_review = driver.find_element_by_css_selector('#component_44 > div > div:nth-child(2) > span:nth-child(1) > a > span').text
main_review = main_review.replace('건의 리뷰', '')

# 가격대
main_price = driver.find_element_by_css_selector('#component_45 > div.tr9HFDVo > div > div:nth-child(2) > div > div > div._3UjHBXYa > div:nth-child(1) > div._1XLfiSsv').text

# 요리 종류
main_type = driver.find_element_by_css_selector('#component_45 > div.tr9HFDVo > div > div:nth-child(2) > div > div > div._3UjHBXYa > div:nth-child(2) > div._1XLfiSsv').text

# 특별식 제공
main_meal = driver.find_element_by_css_selector('#component_45 > div.tr9HFDVo > div > div:nth-child(2) > div > div > div._3UjHBXYa > div:nth-child(3) > div._1XLfiSsv').text

# 주소
main_address = driver.find_element_by_css_selector('#component_45 > div.tr9HFDVo > div > div:nth-child(3) > div > div > div._2vbD36Hr._36TL14Jn > span:nth-child(2) > a > span._2saB_OSe').text

```


```python
main_df = pd.DataFrame(data = {
    'rank' : main_rank,
    'name' : main_name,
    'rate' : [main_rate],
    'review' : main_review,
    'price' : main_price,
    'typ' : main_type,
    'meal' : main_meal,
    'address' : main_address
})

main_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rank</th>
      <th>name</th>
      <th>rate</th>
      <th>review</th>
      <th>price</th>
      <th>typ</th>
      <th>meal</th>
      <th>address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1곳</td>
      <td>인디언키친</td>
      <td>5.0</td>
      <td>436</td>
      <td>₩15,000 - ₩30,000</td>
      <td>인도 요리, 아시아 요리, 건강식</td>
      <td>채식주의 식단, 채식 옵션, 할랄, 코셔, 글루텐 프리</td>
      <td>제주 애월읍 애원로 191 바그다드하우스</td>
    </tr>
  </tbody>
</table>
</div>



이렇게 만들어야지!!

## 음식점 기본 정보 링크 전체 크롤링


```python
columns = main_df.columns.tolist()
columns
```




    ['rank', 'name', 'rate', 'review', 'price', 'typ', 'meal', 'address']




```python
columns
```




    ['rank', 'name', 'rate', 'review', 'price', 'typ', 'meal', 'address']



### 내 계획
- 일단 beautiful soup으로 html 가져온 다음에
- class 지정해서 각각의 정보들 가져오고,
- 그 정보들 가져오는걸 함수로 빼기


```python
html = driver.page_source
soup = BeautifulSoup(html, 'html.parser')
```

### 정보 불러오는 함수


```python
def try_to_get(soup, method, class_name):
    try : 
        want_to_get = soup.find_all(method, {'class' : class_name})
        return want_to_get
    except:
        pass
```


```python
whole_info = pd.DataFrame(columns = columns)
whole_info
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rank</th>
      <th>name</th>
      <th>rate</th>
      <th>review</th>
      <th>price</th>
      <th>typ</th>
      <th>meal</th>
      <th>address</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
driver = Chrome()
driver.maximize_window()

```


```python
driver.get(url = links.iloc[940][0])
```


```python
whole_info = pd.DataFrame(columns = columns)
whole_info

#driver = Chrome()
#driver.maximize_window()

for i in range(940, len(links)):
    time.sleep(3)
    driver.get(url = links.iloc[i][0])
    
    for column in columns:
        column_tmp = np.nan
        
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')
    
    try:
        rank = driver.find_element_by_class_name('_13OzAOXO._2VxaSjVD').text[21:]
    except:
        pass
    name = try_to_get(soup, 'h1', '_3a1XQ88S')[0].find_all(text=True)
    num_review = str(try_to_get(soup, 'span', "_3Wub8auF")[0])[24:][:-20]
    try:
        price =  try_to_get(soup, 'div', '_1XLfiSsv')[0].find_all(text=True)
    except:
        pass
    typ = soup.select('span._13OzAOXO._34GKdBMV')[0].text
    address=soup.select('span._2saB_OSe')[0].text
    
    try :
        rate = try_to_get(soup, 'svg', 'zWXXYhVR')[0]['aria-label'][-3:]
    except:
        rate = 'check it'
    
    whole_info = whole_info.append({
        'rank' : rank,
        'name' : name,
        'rate' : rate,
        'review' : num_review,
        'price' : price,
        'typ' : typ,
        'meal' : meal,
        'address' : address
    }, ignore_index = True)

```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-25-4349472c88f7> in <module>
         38         'rate' : rate,
         39         'review' : num_review,
    ---> 40         'price' : price,
         41         'typ' : typ,
         42         'meal' : meal,
    

    NameError: name 'price' is not defined



```python
whole_info
```


```python
#Final_info = whole_info.copy()
Final_info = Final_info.append(whole_info)
Final_info = Final_info.reset_index(drop=True)
Final_info
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-24-6145a0b6732e> in <module>
          1 #Final_info = whole_info.copy()
    ----> 2 Final_info = Final_info.append(whole_info)
          3 Final_info = Final_info.reset_index(drop=True)
          4 Final_info
    

    NameError: name 'Final_info' is not defined



```python
FINAL_INFO = Final_info.copy()
```


```python
Final_info.name = [ str(Final_info.name[i]).replace("[","").replace(']','').replace("'","").replace("ㅁ","") for i in range(0,len(Final_info)) ]
Final_info.rate = [ float(Final_info.rate[i])  if Final_info.rate[i] != " 리뷰" else Final_info.rate[i] for i in range(0,len(Final_info))]
Final_info.review = [ int(Final_info.review[i]) if Final_info.review[i] != '' else Final_info.review[i] for i in range(0,len(Final_info)) ]
Final_info.typ = [ Final_info.typ[i].replace("$","").replace("ㅁ","") for i in range(0,len(Final_info))]
Final_info.price = [ str(Final_info.price[i]).replace("₩","").replace("ㅁ","").replace("[","").replace("]","") for i in range(0,len(Final_info))]
#Final_info['rank'] = [int(Final_info['rank'][i].replace("곳","").replace(',','')) if Final_info['rank'][i] !='' else Final_info['rank'][i] for i in range(0,len(Final_info)) ]
```


```python
empty_rank = [ i for i in range(0,len(Final_info)) if type(Final_info['rank'][i]) != int ]
empty_rank_array = np.array(empty_rank)
for i in range(0,len(empty_rank)):
    Final_info['rank' == empty_rank_array[i]] = 9999
```


```python
Final_info['rank'] = [ Final_info['rank'][i].replace('','9999') if i in empty_rank else Final_info['rank'][i] for i in range(0,len(Final_info)) ]
```


```python
Final_info['rank'] = [ int(Final_info['rank'][i])  if i in empty_rank else Final_info['rank'][i] for i in range(0,len(Final_info)) ]
```


```python
FINAL_INFO_RESET_INDEX = Final_info.sort_values(by='rank').head(30)
```


```python
FINAL_INFO_RESET_INDEX.to_csv('tripadvisor_info_sort_by_rank.csv', encoding='utf-8_sig')
```


```python
Final_info.to_csv('tripadvisor_info.csv', encoding='utf_8_sig')
```


```python
Final_info.iloc[7]
```




    rank                   8곳
    name                 자매국수
    rate                  4.0
    review                166
    price      ['아시아 요리, 한국']
    typ              아시아 요리한국
    meal                 None
    address         제주 삼성로 67
    Name: 7, dtype: object




```python
for i in range(0,len(links)):
    if links.iloc[i][0] == 'https://www.tripadvisor.co.kr/Restaurant_Review-g297885-d3933972-Reviews-Gimgane_Donam_Store-Jeju_Jeju_Island.html':
        print(i)
```

    1452
    


```python
whole_info_1.name = [ str(whole_info_1.name[i]).replace("['",'').replace("']",'') for i in range(0, len(whole_info)) ]
```


```python
whole_info_1.rate[0]
```




    {'class': ['zWXXYhVR'],
     'viewbox': '0 0 88 16',
     'width': '88',
     'height': '16',
     'aria-label': '0건의 리뷰',
     'title': '0건의 리뷰'}




```python
whole_info_1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rank</th>
      <th>name</th>
      <th>rate</th>
      <th>review</th>
      <th>price</th>
      <th>typ</th>
      <th>meal</th>
      <th>address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1곳</td>
      <td>인디언키친</td>
      <td>{'class': ['zWXXYhVR'], 'viewbox': '0 0 88 16'...</td>
      <td>436</td>
      <td>[₩15,000 - ₩30,000]</td>
      <td>$$ - $$$인도 요리아시아 요리건강식</td>
      <td>None</td>
      <td>제주 애월읍 애원로 191 바그다드하우스</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2곳</td>
      <td>명진전복</td>
      <td>{'class': ['zWXXYhVR'], 'viewbox': '0 0 88 16'...</td>
      <td>303</td>
      <td>[해산물, 아시아 요리, 한국]</td>
      <td>$$ - $$$해산물아시아 요리한국</td>
      <td>None</td>
      <td>제주 구좌읍 해맞이해안로 1282</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3곳</td>
      <td>우진해장국</td>
      <td>{'class': ['zWXXYhVR'], 'viewbox': '0 0 88 16'...</td>
      <td>195</td>
      <td>[아시아 요리, 한국, 수프]</td>
      <td>$아시아 요리한국수프</td>
      <td>None</td>
      <td>제주 서사로 11</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7곳</td>
      <td>블루버드 바이 맥파이</td>
      <td>{'class': ['zWXXYhVR'], 'viewbox': '0 0 88 16'...</td>
      <td>76</td>
      <td>[자가 맥주 판매pub, 바, 피자, 펍]</td>
      <td>$$ - $$$자가 맥주 판매pub바피자</td>
      <td>None</td>
      <td>제주 탑동로2길 7</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4곳</td>
      <td>올래국수</td>
      <td>{'class': ['zWXXYhVR'], 'viewbox': '0 0 88 16'...</td>
      <td>258</td>
      <td>[아시아 요리, 한국]</td>
      <td>$아시아 요리한국</td>
      <td>None</td>
      <td>제주 귀아랑길 24</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1859</th>
      <td></td>
      <td>김가네 제주한라대점</td>
      <td>{'class': ['zWXXYhVR'], 'viewbox': '0 0 88 16'...</td>
      <td>0</td>
      <td>[아시아 요리, 한국]</td>
      <td>아시아 요리한국</td>
      <td>None</td>
      <td>제주 노형동 759번지 1층</td>
    </tr>
    <tr>
      <th>1860</th>
      <td></td>
      <td>김가네 제주탑동점</td>
      <td>{'class': ['zWXXYhVR'], 'viewbox': '0 0 88 16'...</td>
      <td>0</td>
      <td>[아시아 요리, 한국]</td>
      <td>아시아 요리한국</td>
      <td>None</td>
      <td>제주 일도1동 1347 1층</td>
    </tr>
    <tr>
      <th>1861</th>
      <td></td>
      <td>김가네 제주일도점</td>
      <td>{'class': ['zWXXYhVR'], 'viewbox': '0 0 88 16'...</td>
      <td>0</td>
      <td>[아시아 요리, 한국]</td>
      <td>아시아 요리한국</td>
      <td>None</td>
      <td>제주 이도2동 408-3</td>
    </tr>
    <tr>
      <th>1862</th>
      <td></td>
      <td>바당한그릇</td>
      <td>{'class': ['zWXXYhVR'], 'viewbox': '0 0 88 16'...</td>
      <td>0</td>
      <td>[일본 요리, 아시아 요리, 한국]</td>
      <td>일본 요리아시아 요리한국</td>
      <td>None</td>
      <td>제주 애월읍 애월해안로 552-3</td>
    </tr>
    <tr>
      <th>1863</th>
      <td></td>
      <td>김가네 도남점</td>
      <td>{'class': ['zWXXYhVR'], 'viewbox': '0 0 88 16'...</td>
      <td>0</td>
      <td>[아시아 요리, 한국]</td>
      <td>아시아 요리한국</td>
      <td>None</td>
      <td>제주 도남동 62-11 1층</td>
    </tr>
  </tbody>
</table>
<p>1864 rows × 8 columns</p>
</div>




```python
whole_info = whole_info_1.copy()
```


```python
type(whole_info_1.name[0])
```




    bs4.element.ResultSet




```python
pd.unique(whole_info_1.name)
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-388-7914da0f13e7> in <module>
    ----> 1 pd.unique(whole_info_1.name)
    

    ~\anaconda3\lib\site-packages\pandas\core\algorithms.py in unique(values)
        405 
        406     table = htable(len(values))
    --> 407     uniques = table.unique(values)
        408     uniques = _reconstruct_data(uniques, original.dtype, original)
        409     return uniques
    

    pandas\_libs\hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable.unique()
    

    pandas\_libs\hashtable_class_helper.pxi in pandas._libs.hashtable.PyObjectHashTable._unique()
    

    TypeError: unhashable type: 'ResultSet'



```python
len(links)
```




    1453



## 음식점 별로 review 크롤링


```python
def does_btn_clickable(btn_name):
    try: 
        driver.find_element_by_class_name(btn_name).get_attribute('class')!='nav next ui_button primary disabled'
        return True
    except:
        return False
```


```python
def getting_info(soup):
    plc_name = soup.find_all('h1', {'class' : '_3a1XQ88S'})[0].text
    text = soup.find_all("div", {"class":"column_wrap ui_columns is-multiline"})[0]
    review_tmp = text.find_all("div",{"class":"prw_rup prw_reviews_text_summary_hsx"})
    review_title_tmp = soup.find_all('span' , {'class' : 'noQuotes'})
    ratings_tmp =text.find_all("span", {"class": "ui_bubble_rating"})
    visit_date_tmp = soup.find_all("div", {"class":"prw_rup prw_reviews_stay_date_hsx"})
    
    return plc_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp
```


```python
driver = Chrome()
driver.maximize_window()

whole_review = pd.DataFrame(columns = ['plc_name', 'review','title', 'ratings','visit_date'])
btn_name = 'nav.next.ui_button.primary'

for i in range(874, len(links)):
    driver.get(url = links.iloc[i][0])
    review_tmp_df = pd.DataFrame(columns=['plc_name','review','title','ratings','visit_date'])
    try :
        driver.find_element_by_class_name('taLnk.ulBlueLinks').click() # 더보기 클릭
    except: 
        pass
    time.sleep(2)
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')
    
    plc_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)
    
    if len(review_tmp)!=len(review_title_tmp):
        for j in range(0,len(review_title_tmp)):
            review_tmp_df = review_tmp_df.append(
            {'plc_name' : plc_name,
             'review' : 'check it',
            'title' : 'check it',
            'ratings' : 'check it',
            'visit_date' : 'check it'}, ignore_index=True)
    else:
        for j in range(0,len(review_title_tmp)):
            review_tmp_df = review_tmp_df.append(
                {'plc_name' : plc_name,
                    'review' : str(review_tmp[j].text),
                 'title' : str(review_title_tmp[j].text),
                 'ratings' : int(ratings_tmp[j]["class"][-1][-2:])/10,
                 'visit_date' : str(visit_date_tmp[j].text)}, ignore_index=True)
    

    time.sleep(2)
    try:   
        while str(driver.find_element_by_class_name(btn_name).get_attribute('class'))=='nav next ui_button primary':

                driver.find_element_by_class_name(btn_name).click()
                time.sleep(2)
                try :
                    driver.find_element_by_class_name('taLnk.ulBlueLinks').click() # 더보기 클릭
                except : 
                    pass   

                time.sleep(2)

                html = driver.page_source
                soup = BeautifulSoup(html, 'html.parser')

                pcl_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)

        if len(review_tmp)!=len(review_title_tmp):
            for j in range(0,len(review_title_tmp)):
                review_tmp_df = review_tmp_df.append(
                {'plc_name' : plc_name,
                 'review' : 'check it',
                'title' :'check it',
                'ratings' : 'check it',
                'visit_date' : 'check it'}, ignore_index=True)
        else:
            for j in range(0,len(review_title_tmp)):
                review_tmp_df = review_tmp_df.append(
                    {'plc_name' : plc_name,
                     'review' : str(review_tmp[j].text),
                     'title' : str(review_title_tmp[j].text),
                     'ratings' : int(ratings_tmp[j]["class"][-1][-2:])/10,
                     'visit_date' : str(visit_date_tmp[j].text) } , ignore_index=True)
    except:
        try :
            time.sleep(2)
            driver.find_element_by_class_name('taLnk.ulBlueLinks').click() # 더보기 클릭
        except : 
            pass   
        
        time.sleep(2) 
        num_kr_review = soup.find_all('div', {'data-value' : 'ko'})
        num_kr_review = num_kr_review[0].text[5:]
        try:
            num_kr_review = int(num_kr_review.replace(")",""))
        except:
            num_kr_review = 0
        
        if num_kr_review <= 10 :
            print(f'{i} has only 1 page')
        else:
            time.sleep(2)
            html = driver.page_source
            soup = BeautifulSoup(html, 'html.parser')
    
            plc_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)
    
            if len(review_tmp)!=len(review_title_tmp):
                for j in range(0,len(review_title_tmp)):
                    review_tmp_df = review_tmp_df.append(
                    {'plc_name' : plc_name,
                    'review' : 'check it',
                    'title' : 'check it',
                    'ratings' : 'check it',
                    'visit_date' : 'check it'}, ignore_index=True)
            else:
                for j in range(0,len(review_title_tmp)):
                    review_tmp_df = review_tmp_df.append(
                    {'plc_name' : plc_name,
                    'review' : str(review_tmp[j].text),
                    'title' : str(review_title_tmp[j].text),
                    'ratings' : int(ratings_tmp[j]["class"][-1][-2:])/10,
                    'visit_date' : str(visit_date_tmp[j].text)}, ignore_index=True)
            

        time.sleep(2)
        
        
    whole_review = whole_review.append(review_tmp_df)
    whole_review.reset_index(drop=True, inplace=True)

```

    874 has only 1 page
    875 has only 1 page
    876 has only 1 page
    877 has only 1 page
    878 has only 1 page
    879 has only 1 page
    880 has only 1 page
    881 has only 1 page
    882 has only 1 page
    883 has only 1 page
    884 has only 1 page
    885 has only 1 page
    886 has only 1 page
    887 has only 1 page
    888 has only 1 page
    889 has only 1 page
    890 has only 1 page
    891 has only 1 page
    892 has only 1 page
    893 has only 1 page
    894 has only 1 page
    895 has only 1 page
    896 has only 1 page
    897 has only 1 page
    898 has only 1 page
    899 has only 1 page
    900 has only 1 page
    901 has only 1 page
    902 has only 1 page
    903 has only 1 page
    904 has only 1 page
    905 has only 1 page
    906 has only 1 page
    907 has only 1 page
    908 has only 1 page
    909 has only 1 page
    910 has only 1 page
    911 has only 1 page
    912 has only 1 page
    913 has only 1 page
    914 has only 1 page
    915 has only 1 page
    916 has only 1 page
    917 has only 1 page
    918 has only 1 page
    919 has only 1 page
    920 has only 1 page
    921 has only 1 page
    922 has only 1 page
    923 has only 1 page
    924 has only 1 page
    925 has only 1 page
    926 has only 1 page
    927 has only 1 page
    928 has only 1 page
    929 has only 1 page
    930 has only 1 page
    931 has only 1 page
    933 has only 1 page
    934 has only 1 page
    935 has only 1 page
    936 has only 1 page
    937 has only 1 page
    938 has only 1 page
    939 has only 1 page
    940 has only 1 page
    941 has only 1 page
    942 has only 1 page
    943 has only 1 page
    944 has only 1 page
    945 has only 1 page
    946 has only 1 page
    947 has only 1 page
    948 has only 1 page
    949 has only 1 page
    950 has only 1 page
    951 has only 1 page
    952 has only 1 page
    953 has only 1 page
    954 has only 1 page
    955 has only 1 page
    956 has only 1 page
    957 has only 1 page
    958 has only 1 page
    959 has only 1 page
    960 has only 1 page
    961 has only 1 page
    962 has only 1 page
    963 has only 1 page
    964 has only 1 page
    965 has only 1 page
    966 has only 1 page
    967 has only 1 page
    968 has only 1 page
    969 has only 1 page
    970 has only 1 page
    971 has only 1 page
    972 has only 1 page
    973 has only 1 page
    974 has only 1 page
    975 has only 1 page
    976 has only 1 page
    977 has only 1 page
    978 has only 1 page
    979 has only 1 page
    980 has only 1 page
    981 has only 1 page
    982 has only 1 page
    983 has only 1 page
    984 has only 1 page
    985 has only 1 page
    986 has only 1 page
    987 has only 1 page
    988 has only 1 page
    989 has only 1 page
    990 has only 1 page
    991 has only 1 page
    992 has only 1 page
    993 has only 1 page
    994 has only 1 page
    995 has only 1 page
    996 has only 1 page
    997 has only 1 page
    998 has only 1 page
    999 has only 1 page
    1000 has only 1 page
    1001 has only 1 page
    1002 has only 1 page
    1003 has only 1 page
    1004 has only 1 page
    1005 has only 1 page
    1006 has only 1 page
    1007 has only 1 page
    1008 has only 1 page
    1009 has only 1 page
    1010 has only 1 page
    1011 has only 1 page
    1012 has only 1 page
    1013 has only 1 page
    1014 has only 1 page
    1015 has only 1 page
    1016 has only 1 page
    1017 has only 1 page
    1018 has only 1 page
    1019 has only 1 page
    1020 has only 1 page
    1021 has only 1 page
    1022 has only 1 page
    1023 has only 1 page
    1024 has only 1 page
    1025 has only 1 page
    1026 has only 1 page
    1027 has only 1 page
    1028 has only 1 page
    1029 has only 1 page
    1030 has only 1 page
    1031 has only 1 page
    1032 has only 1 page
    1033 has only 1 page
    1034 has only 1 page
    1035 has only 1 page
    1036 has only 1 page
    1037 has only 1 page
    1038 has only 1 page
    1039 has only 1 page
    1040 has only 1 page
    1041 has only 1 page
    1042 has only 1 page
    1043 has only 1 page
    1044 has only 1 page
    1045 has only 1 page
    1046 has only 1 page
    1047 has only 1 page
    1048 has only 1 page
    1049 has only 1 page
    1050 has only 1 page
    1051 has only 1 page
    1052 has only 1 page
    1053 has only 1 page
    1054 has only 1 page
    1055 has only 1 page
    1056 has only 1 page
    1057 has only 1 page
    1058 has only 1 page
    1059 has only 1 page
    1060 has only 1 page
    1061 has only 1 page
    1062 has only 1 page
    1063 has only 1 page
    1064 has only 1 page
    1065 has only 1 page
    1066 has only 1 page
    1067 has only 1 page
    1068 has only 1 page
    1069 has only 1 page
    1070 has only 1 page
    1071 has only 1 page
    1072 has only 1 page
    1073 has only 1 page
    1074 has only 1 page
    1075 has only 1 page
    1076 has only 1 page
    1077 has only 1 page
    1078 has only 1 page
    1079 has only 1 page
    1080 has only 1 page
    1081 has only 1 page
    1082 has only 1 page
    1083 has only 1 page
    1084 has only 1 page
    1085 has only 1 page
    1086 has only 1 page
    1087 has only 1 page
    1088 has only 1 page
    1089 has only 1 page
    1090 has only 1 page
    1091 has only 1 page
    1092 has only 1 page
    1093 has only 1 page
    1094 has only 1 page
    1095 has only 1 page
    1096 has only 1 page
    1097 has only 1 page
    1098 has only 1 page
    1099 has only 1 page
    1100 has only 1 page
    1101 has only 1 page
    1102 has only 1 page
    1103 has only 1 page
    1104 has only 1 page
    1105 has only 1 page
    1106 has only 1 page
    


    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    <ipython-input-20-6b3d1eecfbfd> in <module>
         16     soup = BeautifulSoup(html, 'html.parser')
         17 
    ---> 18     plc_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)
         19 
         20     if len(review_tmp)!=len(review_title_tmp):
    

    <ipython-input-18-1bee652c8964> in getting_info(soup)
          1 def getting_info(soup):
    ----> 2     plc_name = soup.find_all('h1', {'class' : '_3a1XQ88S'})[0].text
          3     text = soup.find_all("div", {"class":"column_wrap ui_columns is-multiline"})[0]
          4     review_tmp = text.find_all("div",{"class":"prw_rup prw_reviews_text_summary_hsx"})
          5     review_title_tmp = soup.find_all('span' , {'class' : 'noQuotes'})
    

    IndexError: list index out of range



```python
soup.find_all('h1', {'class' : '_3a1XQ88S'})[0].text
```


```python
whole_review
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>plc_name</th>
      <th>review</th>
      <th>title</th>
      <th>ratings</th>
      <th>visit_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>종이시계</td>
      <td>在去酒店途中，發現這淺藍色的小屋(印有咖啡店)，當時已經駛過了，但心裡想著要回頭。進入咖啡店...</td>
      <td>感覺非常好</td>
      <td>4.0</td>
      <td>방문 날짜: 2018년 8월</td>
    </tr>
    <tr>
      <th>1</th>
      <td>호떡분식</td>
      <td>스위트 쌀과 설탕 - 한국산 팬케이크 거리 음식으로 독특한 음식 개념을 사랑했습니다...</td>
      <td>독특한</td>
      <td>4.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>2</th>
      <td>원항아리칼국수보쌈</td>
      <td>夏に韓国へ行くとやっぱりコングクス（豆乳の冷たい麺）を食べたくなります。済州島にもあるかなと...</td>
      <td>夏にはやっぱりコングクス</td>
      <td>4.0</td>
      <td>방문 날짜: 2018년 8월</td>
    </tr>
    <tr>
      <th>3</th>
      <td>새큰이가든</td>
      <td>녹두 삼계탕이 맛있어요오리탕은..저는 추천ㄴㄴ 김치나 깍두기가 곁들여져서 나오면 좋...</td>
      <td>숙소근처라서 몇번 갔는데...</td>
      <td>4.0</td>
      <td>방문 날짜: 2018년 8월</td>
    </tr>
    <tr>
      <th>4</th>
      <td>삼영식당</td>
      <td>길가에 있는 작은 가게입니다. 현지 손님들이 많이 찾는듯 합니다. 가격도 다른 관광...</td>
      <td>맛있어요</td>
      <td>4.0</td>
      <td>방문 날짜: 2018년 6월</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>305</th>
      <td>장마루보양탕</td>
      <td>Jang Floor Dog Meat Soup - I certainly had app...</td>
      <td>Hot soup</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>306</th>
      <td>이어도횟집</td>
      <td>이도 사시미 (Ieodo Sashimi) 레스토랑에서 정통 한국 요리를 제공하는 방...</td>
      <td>진정한</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>307</th>
      <td>뽕뜨락피자</td>
      <td>요새 쌀피자가 은근히 괜찮아서 여러곳 가보게 되는데 우연히 들렀지만 정성들여 친절하...</td>
      <td>무난한 쌀피자. 친절한 기억</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 5월</td>
    </tr>
    <tr>
      <th>308</th>
      <td>해오름다방</td>
      <td>마을에서 먹을 곳 중 하나 - 손 내려! . 나는 음식이 뜨겁고 실망하지 않는 것을...</td>
      <td>그것을 좋아했다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>309</th>
      <td>장미다방</td>
      <td>Rose Coffee Shop의 분위기는 나에게 즐거웠습니다. 카운터 녀석은 그들의...</td>
      <td>대단한 분위기</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
  </tbody>
</table>
<p>310 rows × 5 columns</p>
</div>




```python
whole_review.to_csv("1106까지.csv",encoding='utf-8-sig')
```

# 무시


```python
whole_review
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>review</th>
      <th>title</th>
      <th>ratings</th>
      <th>visit_date</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
reviews_tmp.text
```




    '주말 저녁 친구들과 가든 커리, 새우 커리, 탄두리\n\n치킨과 플레인, 갈릭, 버터난들과 함께 먹었습니다.\n\n가격은 제주도 관광지 음식보다 상대적으로 저렴\n\n하다고 느꼈고 음식양도 꽤 많았습니다.\n\n인도 현지보다 더 맛있었습니다\n\n엄지 척!!!'




```python
soup.find_all('p.partial_entry')
```




    []




```python
reviews_tmp = driver.find_elements_by_css_selector('p.partial_entry')
reviews_titles_tmp = driver.find_element_by_css_selector('span.noQuotes')
ratings_tmp = soup.find_all("div", {"class": "ui_column is-9"})
visit_date_tmp = soup.find_all("div", {'class' : 'prw_rup prw_reviews_stay_date_hsx'})

for i in range(0,len(review_tmp)):
    
```


      File "<ipython-input-321-4400ecde52e4>", line 7
        
        ^
    SyntaxError: unexpected EOF while parsing
    



```python
visit_date_tmp
```




    [<span class="stay_date_label">방문 날짜:</span>,
     <span class="stay_date_label">방문 날짜:</span>,
     <span class="stay_date_label">방문 날짜:</span>,
     <span class="stay_date_label">방문 날짜:</span>,
     <span class="stay_date_label">방문 날짜:</span>,
     <span class="stay_date_label">방문 날짜:</span>,
     <span class="stay_date_label">방문 날짜:</span>,
     <span class="stay_date_label">방문 날짜:</span>,
     <span class="stay_date_label">방문 날짜:</span>,
     <span class="stay_date_label">방문 날짜:</span>]




```python
ratings = soup.find_all("div", {"class": "ui_column is-9"})
ratings[9].find_all('span', {'class' : 'ui_bubble_rating'})
```




    [<span class="ui_bubble_rating bubble_30"></span>]




```python
html = driver.page_source
soup = BeautifulSoup(html, 'html.parser')
ratings_tmp = soup.find_all("span", {"class": "ui_bubble_rating"})
ratings_tmp
```




    [<span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_30"></span>,
     <span class="ui_bubble_rating bubble_40"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_40"></span>,
     <span alt="풍선 5개 중 4.5" class="ui_bubble_rating bubble_45" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 3" class="ui_bubble_rating bubble_30" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 3.5" class="ui_bubble_rating bubble_35" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4" class="ui_bubble_rating bubble_40" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4" class="ui_bubble_rating bubble_40" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 5" class="ui_bubble_rating bubble_50" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4" class="ui_bubble_rating bubble_40" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4" class="ui_bubble_rating bubble_40" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4.5" class="ui_bubble_rating bubble_45" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4" class="ui_bubble_rating bubble_40" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 3.5" class="ui_bubble_rating bubble_35" style="font-size:16px;"></span>]




```python
ratings_tmp
```




    [<span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_10"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_50"></span>,
     <span class="ui_bubble_rating bubble_40"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_45"></span>,
     <span class="ui_bubble_rating bubble_40"></span>,
     <span alt="풍선 5개 중 4.5" class="ui_bubble_rating bubble_45" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 3" class="ui_bubble_rating bubble_30" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 3.5" class="ui_bubble_rating bubble_35" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4" class="ui_bubble_rating bubble_40" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4" class="ui_bubble_rating bubble_40" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 5" class="ui_bubble_rating bubble_50" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4" class="ui_bubble_rating bubble_40" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4" class="ui_bubble_rating bubble_40" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4.5" class="ui_bubble_rating bubble_45" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 4" class="ui_bubble_rating bubble_40" style="font-size:16px;"></span>,
     <span alt="풍선 5개 중 3.5" class="ui_bubble_rating bubble_35" style="font-size:16px;"></span>]




```python
#rn703762323 > span
```


```python
      
        # reviews
        review_tmp = driver.find_element_by_css_selector('p.partial_entry').text
        reviews = reviews.append(review_tmp)
            
        #review title
        review_title_tmp = soup.find_all('span' , {'class' : 'noQuotes'})
        review_titles = review_titles.append(review_title_tmp[0])
        
        # ratings
        ratings_tmp = soup.find_all("span", {"class": "ui_bubble_rating"})
        ratings = ratings.append(ratings_tmp[0])
        
        #visit_date
        visit_date_tmp = soup.find_all("span", {'class' : 'stay_date_label'})
        visit_date = visit_date.append(visit_date_tmp[0])
    
    review_tmp_df = review_tmp_df.append({
        'review' : reviews,
        'title' : review_titles,
        'ratings' : ratings,
        'visit_date' : visit_date_tmp
    })
    
    whole_review = whole_review.append(review_tmp_df, ignore_index=True)
```


      File "<ipython-input-266-5e9671e0e1d0>", line 17
        review_tmp_df = review_tmp_df.append({
        ^
    IndentationError: unexpected indent
    



```python
driver = Chrome()
driver.maximize_window()

whole_review = pd.DataFrame(columns = ['plc_name', 'review','title', 'ratings','visit_date'])
btn_name = 'nav.next.ui_button.primary'

for i in range(1107, len(links)):
    driver.get(url = links.iloc[i][0])
    review_tmp_df = pd.DataFrame(columns=['plc_name','review','title','ratings','visit_date'])
    try :
        driver.find_element_by_class_name('taLnk.ulBlueLinks').click() # 더보기 클릭
    except: 
        pass
    time.sleep(2)
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')
    
    plc_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)
    
    if len(review_tmp)!=len(review_title_tmp):
        for j in range(0,len(review_title_tmp)):
            review_tmp_df = review_tmp_df.append(
            {'plc_name' : plc_name,
             'review' : 'check it',
            'title' : 'check it',
            'ratings' : 'check it',
            'visit_date' : 'check it'}, ignore_index=True)
    else:
        for j in range(0,len(review_title_tmp)):
            review_tmp_df = review_tmp_df.append(
                {'plc_name' : plc_name,
                    'review' : str(review_tmp[j].text),
                 'title' : str(review_title_tmp[j].text),
                 'ratings' : int(ratings_tmp[j]["class"][-1][-2:])/10,
                 'visit_date' : str(visit_date_tmp[j].text)}, ignore_index=True)
    

    time.sleep(2)
    try:   
        while str(driver.find_element_by_class_name(btn_name).get_attribute('class'))=='nav next ui_button primary':

                driver.find_element_by_class_name(btn_name).click()
                time.sleep(2)
                try :
                    driver.find_element_by_class_name('taLnk.ulBlueLinks').click() # 더보기 클릭
                except : 
                    pass   

                time.sleep(2)

                html = driver.page_source
                soup = BeautifulSoup(html, 'html.parser')

                pcl_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)

        if len(review_tmp)!=len(review_title_tmp):
            for j in range(0,len(review_title_tmp)):
                review_tmp_df = review_tmp_df.append(
                {'plc_name' : plc_name,
                 'review' : 'check it',
                'title' :'check it',
                'ratings' : 'check it',
                'visit_date' : 'check it'}, ignore_index=True)
        else:
            for j in range(0,len(review_title_tmp)):
                review_tmp_df = review_tmp_df.append(
                    {'plc_name' : plc_name,
                     'review' : str(review_tmp[j].text),
                     'title' : str(review_title_tmp[j].text),
                     'ratings' : int(ratings_tmp[j]["class"][-1][-2:])/10,
                     'visit_date' : str(visit_date_tmp[j].text) } , ignore_index=True)
    except:
        try :
            time.sleep(2)
            driver.find_element_by_class_name('taLnk.ulBlueLinks').click() # 더보기 클릭
        except : 
            pass   
        
        time.sleep(2) 
        num_kr_review = soup.find_all('div', {'data-value' : 'ko'})
        num_kr_review = num_kr_review[0].text[5:]
        try:
            num_kr_review = int(num_kr_review.replace(")",""))
        except:
            num_kr_review = 0
        
        if num_kr_review <= 10 :
            print(f'{i} has only 1 page')
        else:
            time.sleep(2)
            html = driver.page_source
            soup = BeautifulSoup(html, 'html.parser')
    
            plc_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)
    
            if len(review_tmp)!=len(review_title_tmp):
                for j in range(0,len(review_title_tmp)):
                    review_tmp_df = review_tmp_df.append(
                    {'plc_name' : plc_name,
                    'review' : 'check it',
                    'title' : 'check it',
                    'ratings' : 'check it',
                    'visit_date' : 'check it'}, ignore_index=True)
            else:
                for j in range(0,len(review_title_tmp)):
                    review_tmp_df = review_tmp_df.append(
                    {'plc_name' : plc_name,
                    'review' : str(review_tmp[j].text),
                    'title' : str(review_title_tmp[j].text),
                    'ratings' : int(ratings_tmp[j]["class"][-1][-2:])/10,
                    'visit_date' : str(visit_date_tmp[j].text)}, ignore_index=True)
            

        time.sleep(2)
        
        
    whole_review = whole_review.append(review_tmp_df)
    whole_review.reset_index(drop=True, inplace=True)

```

    1107 has only 1 page
    1108 has only 1 page
    1109 has only 1 page
    1110 has only 1 page
    1111 has only 1 page
    1112 has only 1 page
    1113 has only 1 page
    1114 has only 1 page
    1115 has only 1 page
    1116 has only 1 page
    1117 has only 1 page
    1118 has only 1 page
    1119 has only 1 page
    1120 has only 1 page
    1121 has only 1 page
    1122 has only 1 page
    1123 has only 1 page
    1124 has only 1 page
    1125 has only 1 page
    1126 has only 1 page
    1127 has only 1 page
    1128 has only 1 page
    1129 has only 1 page
    1130 has only 1 page
    1131 has only 1 page
    1132 has only 1 page
    1133 has only 1 page
    1134 has only 1 page
    1135 has only 1 page
    1136 has only 1 page
    1137 has only 1 page
    1138 has only 1 page
    1139 has only 1 page
    1140 has only 1 page
    1141 has only 1 page
    1142 has only 1 page
    1143 has only 1 page
    1144 has only 1 page
    1145 has only 1 page
    1146 has only 1 page
    1147 has only 1 page
    


    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    <ipython-input-58-4d6dd83620a8> in <module>
         16     soup = BeautifulSoup(html, 'html.parser')
         17 
    ---> 18     plc_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)
         19 
         20     if len(review_tmp)!=len(review_title_tmp):
    

    <ipython-input-18-1bee652c8964> in getting_info(soup)
          1 def getting_info(soup):
    ----> 2     plc_name = soup.find_all('h1', {'class' : '_3a1XQ88S'})[0].text
          3     text = soup.find_all("div", {"class":"column_wrap ui_columns is-multiline"})[0]
          4     review_tmp = text.find_all("div",{"class":"prw_rup prw_reviews_text_summary_hsx"})
          5     review_title_tmp = soup.find_all('span' , {'class' : 'noQuotes'})
    

    IndexError: list index out of range



```python
r2=whole_review #1107~1147
r2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>plc_name</th>
      <th>review</th>
      <th>title</th>
      <th>ratings</th>
      <th>visit_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>우정식당</td>
      <td>Friendship Sik Dang에서의 멋진 추억은이 윙윙 거리는 곳에서 음식과 ...</td>
      <td>사랑스러운 분위기</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>1</th>
      <td>도다리새꼬시회</td>
      <td>제주도에서 최고의 식사 장소 중 하나 인 Do Bridge Sae Kko Si Sa...</td>
      <td>그것을 좋아했다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>2</th>
      <td>조아찌</td>
      <td>바로 옆 아일랜드팩토리를 가려했으나오픈시간 이전이라 옆에있는 조아찌로 왔어요음료는 ...</td>
      <td>이호 해수욕장 근처 로컬카페.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>3</th>
      <td>조아찌</td>
      <td>주말에 갔더니 넘 사람이 늘어서일까.예전에 황량한 바닷가에 있던 독특한 느낌이 사라...</td>
      <td>많이 손님이 늘어서일까.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 5월</td>
    </tr>
    <tr>
      <th>4</th>
      <td>뚱땡이밥집</td>
      <td>위치는 쉽게 도달 할 수 있습니다. 기운은 장소의 전반적인 분위기와 잘 어울립니다....</td>
      <td>밥 마니아!</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>5</th>
      <td>용우동</td>
      <td>주변에 다른 가게들도 많아서 특별하게 뛰어나다고 말할수는 없는데요. 그냥 우동먹고싶...</td>
      <td>간단하게 우동 한그릇</td>
      <td>3.0</td>
      <td>방문 날짜: 2015년 11월</td>
    </tr>
    <tr>
      <th>6</th>
      <td>뽀글뽀글</td>
      <td>혼자 밥먹기엔 적합하지 않고 점심때 가면 김치찌게만 가능한 집 그리고 저녁엔 일찍 ...</td>
      <td>점심시간 단일메뉴</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 10월</td>
    </tr>
    <tr>
      <th>7</th>
      <td>세화쌍둥이횟집</td>
      <td>Se Hwa Twins Sashimi Restaurant에서 전통적인 제주 요리를 ...</td>
      <td>세화 쌍둥이 사시미 식당</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>8</th>
      <td>세화해녀잠수촌</td>
      <td>Se Hwa 여성 잠수부 잠수 마을은 정말로 그들의 개념이 마음에 들었다. 나는 그...</td>
      <td>재미있는</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>9</th>
      <td>세화오리촌</td>
      <td>Se Hwa Duck Village is a small and beautiful v...</td>
      <td>Nice one</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>10</th>
      <td>올레파이</td>
      <td>식당에서 먹었는데 파이를 다시 데워주지는 않는다. 따뜻하게 먹었으면 맛있었을까? 싶...</td>
      <td>보통 파이</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 8월</td>
    </tr>
    <tr>
      <th>11</th>
      <td>용궁반점</td>
      <td>짜장면 맛있어요 탕수육은 그냥 그래요 짬뽕은 평균이에요간짜장 삼선짜장 맛있고 배달 ...</td>
      <td>짱꼐집</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 6월</td>
    </tr>
    <tr>
      <th>12</th>
      <td>미스터곰</td>
      <td>떡볶이 양념에 튀김과 떡볶이가 같이나오는 모닥치기를 전문적으로 파는 곳. 맛은 보통...</td>
      <td>모닥치기 전문점</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 11월</td>
    </tr>
    <tr>
      <th>13</th>
      <td>자전거빵</td>
      <td>노형동에 자전거집이 별루 없는거같은데 여기 수리도 잘해주시고 해외 고가 자전거도 취...</td>
      <td>노형동 자전거포</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 8월</td>
    </tr>
    <tr>
      <th>14</th>
      <td>자전거빵</td>
      <td>간단히 먹을 수 있어서 좋았던것 같아요허기질때 짱자전거 타고 가면서 먹어서 정말 맛...</td>
      <td>자전거 타서 배고플때 먹는빵</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 3월</td>
    </tr>
    <tr>
      <th>15</th>
      <td>풍경채</td>
      <td>제주공항근처 이호동에 위치한 동네맛집.지역주민들이 많이 찾는 식당.양념갈비와 오겹살...</td>
      <td>동네정식&amp;오겹살</td>
      <td>3.0</td>
      <td>방문 날짜: 2019년 4월</td>
    </tr>
    <tr>
      <th>16</th>
      <td>안세미가든</td>
      <td>여행 첫 날 저녁으로 먹었습니다. 일단 저희가 5시반쯤 갔는데 일찍가서 그런지 손님...</td>
      <td>양은 적지만 괜찮았습니다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 7월</td>
    </tr>
    <tr>
      <th>17</th>
      <td>엔돌핀다방</td>
      <td>En Dolphin Coffee Shop의 커피는 신선하고 신선한 상태로 제공되었습...</td>
      <td>그것을 좋아했다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>18</th>
      <td>수목원가는길</td>
      <td>몇달전에 망했어요 이제 거기 없어요 문닫았어요 폐업했다고 써있더군요 이걸 리뷰로 남...</td>
      <td>저런..</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 4월</td>
    </tr>
    <tr>
      <th>19</th>
      <td>꺼멍도새기</td>
      <td>인터넷 맛집 검색으로 간곳인데, 동문시장내 지하에 유리파티션으로 구분된 식당칸에 위...</td>
      <td>분위기있는 곳 한식집 찾는 분 말고</td>
      <td>3.0</td>
      <td>방문 날짜: 2019년 12월</td>
    </tr>
    <tr>
      <th>20</th>
      <td>꺼멍싸바</td>
      <td>나는 우연히 우연히이 곳을 우연히 발견했다. 나는이 장소를 꾸미기 위해 그들이 한 ...</td>
      <td>좋은 발견</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>21</th>
      <td>황제의밤</td>
      <td>네, 여기에 쉽게 도착할 수 있습니다. 레스토랑의 인테리어는 기다림을 가치있게 만들...</td>
      <td>예스 장소!</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>22</th>
      <td>숙경식당</td>
      <td>나는 수경 식당에 특별한 날에 내 친구들과 함께 있었고, 즐거운 하루였다.</td>
      <td>맛있는 음식</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>23</th>
      <td>본죽</td>
      <td>저의 이웃입니다.특히 잦죽과 양송이 버섯죽이 맛있습니다.가끔 식구들과 외식을 합니다...</td>
      <td>잦죽 좋아요</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 5월</td>
    </tr>
    <tr>
      <th>24</th>
      <td>노형탑부평참숯구이</td>
      <td>요즘 너무 많아진 고깃집... 몸에도 않좋은 연탄에 고기 구워먹지말고 요즘에는 다시...</td>
      <td>노형동 고깃집</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 6월</td>
    </tr>
    <tr>
      <th>25</th>
      <td>카페띠아모</td>
      <td>아메리카노가 맛있었던 카페 디아모입니다. 알바생이 친절하고 분위기도 좋습니다. 근처...</td>
      <td>카페</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 5월</td>
    </tr>
    <tr>
      <th>26</th>
      <td>용두바당횟집</td>
      <td>Lokasinya di pinggir laut, jadi bisa lihat pem...</td>
      <td>Restoran seafood</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 7월</td>
    </tr>
    <tr>
      <th>27</th>
      <td>우미정</td>
      <td>이 장소의 장식은 단순하고 우아했습니다. 나는지도를 조사하고 몇 분 안에 거기에있었...</td>
      <td>그것을 사랑.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>28</th>
      <td>쉬는팡</td>
      <td>수민 팡은 노동자들에게 훌륭한 봉사를했다. 이 메뉴는 마을의 모든 레스토랑에서와 같...</td>
      <td>격투기</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>29</th>
      <td>엄바위</td>
      <td>내가 머무르고있는 나의 호텔의 지방 주민은이 장소를 찾아 가기 위해 나에게 말했다....</td>
      <td>좋은 장소</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>30</th>
      <td>엔제리너스</td>
      <td>공항내 입점한 엔제리너스 공간은 협소하고 사람은 많아서 복잡합니다. 맛은 타지점과 ...</td>
      <td>엔제리너스</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 9월</td>
    </tr>
    <tr>
      <th>31</th>
      <td>월랑봉</td>
      <td>이 레스토랑의 위치는 분명했습니다. 정말 아름답게 꾸며진 곳. . . .</td>
      <td>월란봉</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>32</th>
      <td>신라제과</td>
      <td>여기의 환경은 나를 매우 기쁘게했다. 정말 운 좋은 곳을 찾아 먹을 수 있습니다.</td>
      <td>신라 제과점!</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>33</th>
      <td>큰언니국수</td>
      <td>나쁘진 않았지만 그렇다고 특색있진 않았어요. 줄서서 기다리기 싫은 분들은 편하게 한...</td>
      <td>나쁘진 않았어요.</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 1월</td>
    </tr>
    <tr>
      <th>34</th>
      <td>섬나라일식초밥</td>
      <td>와우, 내가 여기서 말할 수있는 것은 Islands Nation Japanese R...</td>
      <td>그것을 좋아했다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>35</th>
      <td>시암 타이랜드 레스토랑 앤 카페</td>
      <td>As a major fan of Thai food, I was excited abo...</td>
      <td>Pleasant Surroundings, Buffet a Bit Too Spicy</td>
      <td>3.0</td>
      <td>방문 날짜: 2015년 4월</td>
    </tr>
    <tr>
      <th>36</th>
      <td>상하이반점</td>
      <td>Loved the local flavor at the Shanghai Chinese...</td>
      <td>Good chinese</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>37</th>
      <td>올레파이</td>
      <td>식당에서 먹었는데 파이를 다시 데워주지는 않는다. 따뜻하게 먹었으면 맛있었을까? 싶...</td>
      <td>보통 파이</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 8월</td>
    </tr>
    <tr>
      <th>38</th>
      <td>용궁반점</td>
      <td>짜장면 맛있어요 탕수육은 그냥 그래요 짬뽕은 평균이에요간짜장 삼선짜장 맛있고 배달 ...</td>
      <td>짱꼐집</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 6월</td>
    </tr>
    <tr>
      <th>39</th>
      <td>미스터곰</td>
      <td>떡볶이 양념에 튀김과 떡볶이가 같이나오는 모닥치기를 전문적으로 파는 곳. 맛은 보통...</td>
      <td>모닥치기 전문점</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 11월</td>
    </tr>
    <tr>
      <th>40</th>
      <td>자전거빵</td>
      <td>노형동에 자전거집이 별루 없는거같은데 여기 수리도 잘해주시고 해외 고가 자전거도 취...</td>
      <td>노형동 자전거포</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 8월</td>
    </tr>
    <tr>
      <th>41</th>
      <td>자전거빵</td>
      <td>간단히 먹을 수 있어서 좋았던것 같아요허기질때 짱자전거 타고 가면서 먹어서 정말 맛...</td>
      <td>자전거 타서 배고플때 먹는빵</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 3월</td>
    </tr>
    <tr>
      <th>42</th>
      <td>카페아프리카</td>
      <td>우리는 다른 레스토랑을 찾고, 그가 찾을 수 없습니다. 하지만 이 곳이 식당인 것 ...</td>
      <td>새로운, 아마?</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 8월</td>
    </tr>
    <tr>
      <th>43</th>
      <td>소레노래연습장</td>
      <td>음식점으로 분류되어있는데 음식점이 아니라 노래방이다.안에서 파는 술들이 맛있다. 안...</td>
      <td>노래방이다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 6월</td>
    </tr>
  </tbody>
</table>
</div>




```python
r22=r2
r22
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>plc_name</th>
      <th>review</th>
      <th>title</th>
      <th>ratings</th>
      <th>visit_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>우정식당</td>
      <td>Friendship Sik Dang에서의 멋진 추억은이 윙윙 거리는 곳에서 음식과 ...</td>
      <td>사랑스러운 분위기</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>1</th>
      <td>도다리새꼬시회</td>
      <td>제주도에서 최고의 식사 장소 중 하나 인 Do Bridge Sae Kko Si Sa...</td>
      <td>그것을 좋아했다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>2</th>
      <td>조아찌</td>
      <td>바로 옆 아일랜드팩토리를 가려했으나오픈시간 이전이라 옆에있는 조아찌로 왔어요음료는 ...</td>
      <td>이호 해수욕장 근처 로컬카페.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>3</th>
      <td>조아찌</td>
      <td>주말에 갔더니 넘 사람이 늘어서일까.예전에 황량한 바닷가에 있던 독특한 느낌이 사라...</td>
      <td>많이 손님이 늘어서일까.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 5월</td>
    </tr>
    <tr>
      <th>4</th>
      <td>뚱땡이밥집</td>
      <td>위치는 쉽게 도달 할 수 있습니다. 기운은 장소의 전반적인 분위기와 잘 어울립니다....</td>
      <td>밥 마니아!</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>5</th>
      <td>용우동</td>
      <td>주변에 다른 가게들도 많아서 특별하게 뛰어나다고 말할수는 없는데요. 그냥 우동먹고싶...</td>
      <td>간단하게 우동 한그릇</td>
      <td>3.0</td>
      <td>방문 날짜: 2015년 11월</td>
    </tr>
    <tr>
      <th>6</th>
      <td>뽀글뽀글</td>
      <td>혼자 밥먹기엔 적합하지 않고 점심때 가면 김치찌게만 가능한 집 그리고 저녁엔 일찍 ...</td>
      <td>점심시간 단일메뉴</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 10월</td>
    </tr>
    <tr>
      <th>7</th>
      <td>세화쌍둥이횟집</td>
      <td>Se Hwa Twins Sashimi Restaurant에서 전통적인 제주 요리를 ...</td>
      <td>세화 쌍둥이 사시미 식당</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>8</th>
      <td>세화해녀잠수촌</td>
      <td>Se Hwa 여성 잠수부 잠수 마을은 정말로 그들의 개념이 마음에 들었다. 나는 그...</td>
      <td>재미있는</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>9</th>
      <td>세화오리촌</td>
      <td>Se Hwa Duck Village is a small and beautiful v...</td>
      <td>Nice one</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>10</th>
      <td>올레파이</td>
      <td>식당에서 먹었는데 파이를 다시 데워주지는 않는다. 따뜻하게 먹었으면 맛있었을까? 싶...</td>
      <td>보통 파이</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 8월</td>
    </tr>
    <tr>
      <th>11</th>
      <td>용궁반점</td>
      <td>짜장면 맛있어요 탕수육은 그냥 그래요 짬뽕은 평균이에요간짜장 삼선짜장 맛있고 배달 ...</td>
      <td>짱꼐집</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 6월</td>
    </tr>
    <tr>
      <th>12</th>
      <td>미스터곰</td>
      <td>떡볶이 양념에 튀김과 떡볶이가 같이나오는 모닥치기를 전문적으로 파는 곳. 맛은 보통...</td>
      <td>모닥치기 전문점</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 11월</td>
    </tr>
    <tr>
      <th>13</th>
      <td>자전거빵</td>
      <td>노형동에 자전거집이 별루 없는거같은데 여기 수리도 잘해주시고 해외 고가 자전거도 취...</td>
      <td>노형동 자전거포</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 8월</td>
    </tr>
    <tr>
      <th>14</th>
      <td>자전거빵</td>
      <td>간단히 먹을 수 있어서 좋았던것 같아요허기질때 짱자전거 타고 가면서 먹어서 정말 맛...</td>
      <td>자전거 타서 배고플때 먹는빵</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 3월</td>
    </tr>
    <tr>
      <th>15</th>
      <td>풍경채</td>
      <td>제주공항근처 이호동에 위치한 동네맛집.지역주민들이 많이 찾는 식당.양념갈비와 오겹살...</td>
      <td>동네정식&amp;오겹살</td>
      <td>3.0</td>
      <td>방문 날짜: 2019년 4월</td>
    </tr>
    <tr>
      <th>16</th>
      <td>안세미가든</td>
      <td>여행 첫 날 저녁으로 먹었습니다. 일단 저희가 5시반쯤 갔는데 일찍가서 그런지 손님...</td>
      <td>양은 적지만 괜찮았습니다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 7월</td>
    </tr>
    <tr>
      <th>17</th>
      <td>엔돌핀다방</td>
      <td>En Dolphin Coffee Shop의 커피는 신선하고 신선한 상태로 제공되었습...</td>
      <td>그것을 좋아했다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>18</th>
      <td>수목원가는길</td>
      <td>몇달전에 망했어요 이제 거기 없어요 문닫았어요 폐업했다고 써있더군요 이걸 리뷰로 남...</td>
      <td>저런..</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 4월</td>
    </tr>
    <tr>
      <th>19</th>
      <td>꺼멍도새기</td>
      <td>인터넷 맛집 검색으로 간곳인데, 동문시장내 지하에 유리파티션으로 구분된 식당칸에 위...</td>
      <td>분위기있는 곳 한식집 찾는 분 말고</td>
      <td>3.0</td>
      <td>방문 날짜: 2019년 12월</td>
    </tr>
    <tr>
      <th>20</th>
      <td>꺼멍싸바</td>
      <td>나는 우연히 우연히이 곳을 우연히 발견했다. 나는이 장소를 꾸미기 위해 그들이 한 ...</td>
      <td>좋은 발견</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>21</th>
      <td>황제의밤</td>
      <td>네, 여기에 쉽게 도착할 수 있습니다. 레스토랑의 인테리어는 기다림을 가치있게 만들...</td>
      <td>예스 장소!</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>22</th>
      <td>숙경식당</td>
      <td>나는 수경 식당에 특별한 날에 내 친구들과 함께 있었고, 즐거운 하루였다.</td>
      <td>맛있는 음식</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>23</th>
      <td>본죽</td>
      <td>저의 이웃입니다.특히 잦죽과 양송이 버섯죽이 맛있습니다.가끔 식구들과 외식을 합니다...</td>
      <td>잦죽 좋아요</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 5월</td>
    </tr>
    <tr>
      <th>24</th>
      <td>노형탑부평참숯구이</td>
      <td>요즘 너무 많아진 고깃집... 몸에도 않좋은 연탄에 고기 구워먹지말고 요즘에는 다시...</td>
      <td>노형동 고깃집</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 6월</td>
    </tr>
    <tr>
      <th>25</th>
      <td>카페띠아모</td>
      <td>아메리카노가 맛있었던 카페 디아모입니다. 알바생이 친절하고 분위기도 좋습니다. 근처...</td>
      <td>카페</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 5월</td>
    </tr>
    <tr>
      <th>26</th>
      <td>용두바당횟집</td>
      <td>Lokasinya di pinggir laut, jadi bisa lihat pem...</td>
      <td>Restoran seafood</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 7월</td>
    </tr>
    <tr>
      <th>27</th>
      <td>우미정</td>
      <td>이 장소의 장식은 단순하고 우아했습니다. 나는지도를 조사하고 몇 분 안에 거기에있었...</td>
      <td>그것을 사랑.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>28</th>
      <td>쉬는팡</td>
      <td>수민 팡은 노동자들에게 훌륭한 봉사를했다. 이 메뉴는 마을의 모든 레스토랑에서와 같...</td>
      <td>격투기</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>29</th>
      <td>엄바위</td>
      <td>내가 머무르고있는 나의 호텔의 지방 주민은이 장소를 찾아 가기 위해 나에게 말했다....</td>
      <td>좋은 장소</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>30</th>
      <td>엔제리너스</td>
      <td>공항내 입점한 엔제리너스 공간은 협소하고 사람은 많아서 복잡합니다. 맛은 타지점과 ...</td>
      <td>엔제리너스</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 9월</td>
    </tr>
    <tr>
      <th>31</th>
      <td>월랑봉</td>
      <td>이 레스토랑의 위치는 분명했습니다. 정말 아름답게 꾸며진 곳. . . .</td>
      <td>월란봉</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>32</th>
      <td>신라제과</td>
      <td>여기의 환경은 나를 매우 기쁘게했다. 정말 운 좋은 곳을 찾아 먹을 수 있습니다.</td>
      <td>신라 제과점!</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>33</th>
      <td>큰언니국수</td>
      <td>나쁘진 않았지만 그렇다고 특색있진 않았어요. 줄서서 기다리기 싫은 분들은 편하게 한...</td>
      <td>나쁘진 않았어요.</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 1월</td>
    </tr>
    <tr>
      <th>34</th>
      <td>섬나라일식초밥</td>
      <td>와우, 내가 여기서 말할 수있는 것은 Islands Nation Japanese R...</td>
      <td>그것을 좋아했다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>35</th>
      <td>시암 타이랜드 레스토랑 앤 카페</td>
      <td>As a major fan of Thai food, I was excited abo...</td>
      <td>Pleasant Surroundings, Buffet a Bit Too Spicy</td>
      <td>3.0</td>
      <td>방문 날짜: 2015년 4월</td>
    </tr>
    <tr>
      <th>36</th>
      <td>상하이반점</td>
      <td>Loved the local flavor at the Shanghai Chinese...</td>
      <td>Good chinese</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>37</th>
      <td>올레파이</td>
      <td>식당에서 먹었는데 파이를 다시 데워주지는 않는다. 따뜻하게 먹었으면 맛있었을까? 싶...</td>
      <td>보통 파이</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 8월</td>
    </tr>
    <tr>
      <th>38</th>
      <td>용궁반점</td>
      <td>짜장면 맛있어요 탕수육은 그냥 그래요 짬뽕은 평균이에요간짜장 삼선짜장 맛있고 배달 ...</td>
      <td>짱꼐집</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 6월</td>
    </tr>
    <tr>
      <th>39</th>
      <td>미스터곰</td>
      <td>떡볶이 양념에 튀김과 떡볶이가 같이나오는 모닥치기를 전문적으로 파는 곳. 맛은 보통...</td>
      <td>모닥치기 전문점</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 11월</td>
    </tr>
    <tr>
      <th>40</th>
      <td>자전거빵</td>
      <td>노형동에 자전거집이 별루 없는거같은데 여기 수리도 잘해주시고 해외 고가 자전거도 취...</td>
      <td>노형동 자전거포</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 8월</td>
    </tr>
    <tr>
      <th>41</th>
      <td>자전거빵</td>
      <td>간단히 먹을 수 있어서 좋았던것 같아요허기질때 짱자전거 타고 가면서 먹어서 정말 맛...</td>
      <td>자전거 타서 배고플때 먹는빵</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 3월</td>
    </tr>
    <tr>
      <th>42</th>
      <td>카페아프리카</td>
      <td>우리는 다른 레스토랑을 찾고, 그가 찾을 수 없습니다. 하지만 이 곳이 식당인 것 ...</td>
      <td>새로운, 아마?</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 8월</td>
    </tr>
    <tr>
      <th>43</th>
      <td>소레노래연습장</td>
      <td>음식점으로 분류되어있는데 음식점이 아니라 노래방이다.안에서 파는 술들이 맛있다. 안...</td>
      <td>노래방이다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 6월</td>
    </tr>
  </tbody>
</table>
</div>




```python
driver = Chrome()
driver.maximize_window()

whole_review = pd.DataFrame(columns = ['plc_name', 'review','title', 'ratings','visit_date'])
btn_name = 'nav.next.ui_button.primary'

for i in range(1148, len(links)):
    driver.get(url = links.iloc[i][0])
    review_tmp_df = pd.DataFrame(columns=['plc_name','review','title','ratings','visit_date'])
    try :
        driver.find_element_by_class_name('taLnk.ulBlueLinks').click() # 더보기 클릭
    except: 
        pass
    time.sleep(2)
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')
    
    plc_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)
    
    if len(review_tmp)!=len(review_title_tmp):
        for j in range(0,len(review_title_tmp)):
            review_tmp_df = review_tmp_df.append(
            {'plc_name' : plc_name,
             'review' : 'check it',
            'title' : 'check it',
            'ratings' : 'check it',
            'visit_date' : 'check it'}, ignore_index=True)
    else:
        for j in range(0,len(review_title_tmp)):
            review_tmp_df = review_tmp_df.append(
                {'plc_name' : plc_name,
                    'review' : str(review_tmp[j].text),
                 'title' : str(review_title_tmp[j].text),
                 'ratings' : int(ratings_tmp[j]["class"][-1][-2:])/10,
                 'visit_date' : str(visit_date_tmp[j].text)}, ignore_index=True)
    

    time.sleep(2)
    try:   
        while str(driver.find_element_by_class_name(btn_name).get_attribute('class'))=='nav next ui_button primary':

                driver.find_element_by_class_name(btn_name).click()
                time.sleep(2)
                try :
                    driver.find_element_by_class_name('taLnk.ulBlueLinks').click() # 더보기 클릭
                except : 
                    pass   

                time.sleep(2)

                html = driver.page_source
                soup = BeautifulSoup(html, 'html.parser')

                pcl_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)

        if len(review_tmp)!=len(review_title_tmp):
            for j in range(0,len(review_title_tmp)):
                review_tmp_df = review_tmp_df.append(
                {'plc_name' : plc_name,
                 'review' : 'check it',
                'title' :'check it',
                'ratings' : 'check it',
                'visit_date' : 'check it'}, ignore_index=True)
        else:
            for j in range(0,len(review_title_tmp)):
                review_tmp_df = review_tmp_df.append(
                    {'plc_name' : plc_name,
                     'review' : str(review_tmp[j].text),
                     'title' : str(review_title_tmp[j].text),
                     'ratings' : int(ratings_tmp[j]["class"][-1][-2:])/10,
                     'visit_date' : str(visit_date_tmp[j].text) } , ignore_index=True)
    except:
        try :
            time.sleep(2)
            driver.find_element_by_class_name('taLnk.ulBlueLinks').click() # 더보기 클릭
        except : 
            pass   
        
        time.sleep(2) 
        num_kr_review = soup.find_all('div', {'data-value' : 'ko'})
        num_kr_review = num_kr_review[0].text[5:]
        try:
            num_kr_review = int(num_kr_review.replace(")",""))
        except:
            num_kr_review = 0
        
        if num_kr_review <= 10 :
            print(f'{i} has only 1 page')
        else:
            time.sleep(2)
            html = driver.page_source
            soup = BeautifulSoup(html, 'html.parser')
    
            plc_name, review_tmp, review_title_tmp, ratings_tmp, visit_date_tmp = getting_info(soup)
    
            if len(review_tmp)!=len(review_title_tmp):
                for j in range(0,len(review_title_tmp)):
                    review_tmp_df = review_tmp_df.append(
                    {'plc_name' : plc_name,
                    'review' : 'check it',
                    'title' : 'check it',
                    'ratings' : 'check it',
                    'visit_date' : 'check it'}, ignore_index=True)
            else:
                for j in range(0,len(review_title_tmp)):
                    review_tmp_df = review_tmp_df.append(
                    {'plc_name' : plc_name,
                    'review' : str(review_tmp[j].text),
                    'title' : str(review_title_tmp[j].text),
                    'ratings' : int(ratings_tmp[j]["class"][-1][-2:])/10,
                    'visit_date' : str(visit_date_tmp[j].text)}, ignore_index=True)
            

        time.sleep(2)
        
        
    whole_review = whole_review.append(review_tmp_df)
    whole_review.reset_index(drop=True, inplace=True)
```

    1148 has only 1 page
    1149 has only 1 page
    1150 has only 1 page
    1151 has only 1 page
    1152 has only 1 page
    1153 has only 1 page
    1154 has only 1 page
    1155 has only 1 page
    1156 has only 1 page
    1157 has only 1 page
    1158 has only 1 page
    1159 has only 1 page
    1160 has only 1 page
    1161 has only 1 page
    1162 has only 1 page
    1163 has only 1 page
    1164 has only 1 page
    1165 has only 1 page
    1166 has only 1 page
    1167 has only 1 page
    1168 has only 1 page
    1169 has only 1 page
    1170 has only 1 page
    1171 has only 1 page
    1172 has only 1 page
    1173 has only 1 page
    1174 has only 1 page
    1175 has only 1 page
    1176 has only 1 page
    1177 has only 1 page
    1178 has only 1 page
    1179 has only 1 page
    1180 has only 1 page
    1181 has only 1 page
    1182 has only 1 page
    1183 has only 1 page
    1184 has only 1 page
    1185 has only 1 page
    1186 has only 1 page
    1187 has only 1 page
    1188 has only 1 page
    1189 has only 1 page
    1190 has only 1 page
    1191 has only 1 page
    1192 has only 1 page
    1193 has only 1 page
    1194 has only 1 page
    1195 has only 1 page
    1196 has only 1 page
    1197 has only 1 page
    1198 has only 1 page
    1199 has only 1 page
    1200 has only 1 page
    1201 has only 1 page
    1202 has only 1 page
    1203 has only 1 page
    1204 has only 1 page
    1205 has only 1 page
    1206 has only 1 page
    1207 has only 1 page
    1208 has only 1 page
    1209 has only 1 page
    1210 has only 1 page
    1211 has only 1 page
    1212 has only 1 page
    1213 has only 1 page
    1214 has only 1 page
    1215 has only 1 page
    1216 has only 1 page
    1217 has only 1 page
    1218 has only 1 page
    1219 has only 1 page
    1220 has only 1 page
    1221 has only 1 page
    1222 has only 1 page
    1223 has only 1 page
    1224 has only 1 page
    1225 has only 1 page
    1226 has only 1 page
    1227 has only 1 page
    1228 has only 1 page
    1229 has only 1 page
    1230 has only 1 page
    1231 has only 1 page
    1232 has only 1 page
    1233 has only 1 page
    1234 has only 1 page
    1235 has only 1 page
    1236 has only 1 page
    1237 has only 1 page
    1238 has only 1 page
    1239 has only 1 page
    1240 has only 1 page
    1241 has only 1 page
    1242 has only 1 page
    1243 has only 1 page
    1244 has only 1 page
    1245 has only 1 page
    1246 has only 1 page
    1247 has only 1 page
    1248 has only 1 page
    1249 has only 1 page
    1250 has only 1 page
    1251 has only 1 page
    1252 has only 1 page
    1253 has only 1 page
    1254 has only 1 page
    1255 has only 1 page
    1256 has only 1 page
    1257 has only 1 page
    1258 has only 1 page
    1259 has only 1 page
    1260 has only 1 page
    1261 has only 1 page
    1262 has only 1 page
    1263 has only 1 page
    1264 has only 1 page
    1265 has only 1 page
    1266 has only 1 page
    1267 has only 1 page
    1268 has only 1 page
    1269 has only 1 page
    1270 has only 1 page
    1271 has only 1 page
    1272 has only 1 page
    1273 has only 1 page
    1274 has only 1 page
    1275 has only 1 page
    1276 has only 1 page
    1277 has only 1 page
    1278 has only 1 page
    1279 has only 1 page
    1280 has only 1 page
    1281 has only 1 page
    1282 has only 1 page
    1283 has only 1 page
    1284 has only 1 page
    1285 has only 1 page
    1286 has only 1 page
    


    ---------------------------------------------------------------------------

    NoSuchElementException                    Traceback (most recent call last)

    <ipython-input-61-bea14e9e1784> in <module>
         39     try:
    ---> 40         while str(driver.find_element_by_class_name(btn_name).get_attribute('class'))=='nav next ui_button primary':
         41 
    

    ~\anaconda3\lib\site-packages\selenium\webdriver\remote\webdriver.py in find_element_by_class_name(self, name)
        563         """
    --> 564         return self.find_element(by=By.CLASS_NAME, value=name)
        565 
    

    ~\anaconda3\lib\site-packages\selenium\webdriver\remote\webdriver.py in find_element(self, by, value)
        975                 value = '[name="%s"]' % value
    --> 976         return self.execute(Command.FIND_ELEMENT, {
        977             'using': by,
    

    ~\anaconda3\lib\site-packages\selenium\webdriver\remote\webdriver.py in execute(self, driver_command, params)
        320         if response:
    --> 321             self.error_handler.check_response(response)
        322             response['value'] = self._unwrap_value(
    

    ~\anaconda3\lib\site-packages\selenium\webdriver\remote\errorhandler.py in check_response(self, response)
        241             raise exception_class(message, screen, stacktrace, alert_text)
    --> 242         raise exception_class(message, screen, stacktrace)
        243 
    

    NoSuchElementException: Message: no such element: Unable to locate element: {"method":"css selector","selector":".nav.next.ui_button.primary"}
      (Session info: chrome=90.0.4430.85)
    

    
    During handling of the above exception, another exception occurred:
    

    IndexError                                Traceback (most recent call last)

    <ipython-input-61-bea14e9e1784> in <module>
         79         time.sleep(2)
         80         num_kr_review = soup.find_all('div', {'data-value' : 'ko'})
    ---> 81         num_kr_review = num_kr_review[0].text[5:]
         82         try:
         83             num_kr_review = int(num_kr_review.replace(")",""))
    

    IndexError: list index out of range



```python
r3=whole_review
```


```python
r3#!148~1286 이제 더이상 리뷰 없다.
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>plc_name</th>
      <th>review</th>
      <th>title</th>
      <th>ratings</th>
      <th>visit_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>피자가기기막혀</td>
      <td>피자 가기 김치 - 맛있는 음식은 모든 사람에게 제공되지만, 서비스가 독특하기 때문...</td>
      <td>피자 가기 김치조</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>1</th>
      <td>루스트플레이스 아라점</td>
      <td>개인적으로는 고급스러운 내부에 비쌀것 같다는 생각이 들게 만들었다. 하지만 생각보다...</td>
      <td>고급스런 내부 푸짐한 양</td>
      <td>3.0</td>
      <td>방문 날짜: 2016년 8월</td>
    </tr>
    <tr>
      <th>2</th>
      <td>평대블루스</td>
      <td>평창 블루 수 (青 大平)가 메뉴를 확장 한 방식을 사랑했습니다. 뜨겁게 먹을 때 ...</td>
      <td>대단히 좋아했습니다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>3</th>
      <td>뜨로</td>
      <td>가게도 나름 깔끔하고 먹을만했어요. 시내에 있으니 오며가며 앞으로 몇번 정도는 찾지...</td>
      <td>맛있어요.</td>
      <td>3.0</td>
      <td>방문 날짜: 2017년 6월</td>
    </tr>
    <tr>
      <th>4</th>
      <td>전통주점창</td>
      <td>이 레스토랑의 위치는 아무런 문제가되지 않았습니다. 모두를 위해 먹을 수있는 물건이...</td>
      <td>반드시 방문해야합니다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>208</th>
      <td>용출횟집</td>
      <td>성인 5 + 아기 1명은 갔더니 4명짜리 테이블로 안내하며 아기는 낑겨 앉고 성인 ...</td>
      <td>제주도의 역갑질 식당</td>
      <td>1.0</td>
      <td>방문 날짜: 2017년 8월</td>
    </tr>
    <tr>
      <th>209</th>
      <td>용출횟집</td>
      <td>제주도 북부쪽에 위치한 횟집입니다. 식당앞에서 비행기들이 제주공항에 착륙하기 위해서...</td>
      <td>제주도 횟집</td>
      <td>4.0</td>
      <td>방문 날짜: 2016년 5월</td>
    </tr>
    <tr>
      <th>210</th>
      <td>용출횟집</td>
      <td>용담 해안도로를 따라 횟집들이 쭉 늘어서 있으나 그 중 유난히 회가 쫄깃쫄깃하고 감...</td>
      <td>신선한 회</td>
      <td>5.0</td>
      <td>방문 날짜: 2015년 7월</td>
    </tr>
    <tr>
      <th>211</th>
      <td>요레</td>
      <td>여러가지 다양한 퓨전요리라 입맛에 맞는요리를 선택해서 먹을수있어 좋아요~^^제주시 ...</td>
      <td>맛집~!</td>
      <td>5.0</td>
      <td>방문 날짜: 2016년 12월</td>
    </tr>
    <tr>
      <th>212</th>
      <td>요레</td>
      <td>서버는 고객을 대하는 방법을 정확히 알고있었습니다. . 정말 아름답게 꾸며진 곳. ...</td>
      <td>옛날!</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
  </tbody>
</table>
<p>213 rows × 5 columns</p>
</div>




```python
review2=pd.concat([r2,r3],axis=0)
```


```python
review2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>plc_name</th>
      <th>review</th>
      <th>title</th>
      <th>ratings</th>
      <th>visit_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>우정식당</td>
      <td>Friendship Sik Dang에서의 멋진 추억은이 윙윙 거리는 곳에서 음식과 ...</td>
      <td>사랑스러운 분위기</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>1</th>
      <td>도다리새꼬시회</td>
      <td>제주도에서 최고의 식사 장소 중 하나 인 Do Bridge Sae Kko Si Sa...</td>
      <td>그것을 좋아했다.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>2</th>
      <td>조아찌</td>
      <td>바로 옆 아일랜드팩토리를 가려했으나오픈시간 이전이라 옆에있는 조아찌로 왔어요음료는 ...</td>
      <td>이호 해수욕장 근처 로컬카페.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>3</th>
      <td>조아찌</td>
      <td>주말에 갔더니 넘 사람이 늘어서일까.예전에 황량한 바닷가에 있던 독특한 느낌이 사라...</td>
      <td>많이 손님이 늘어서일까.</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 5월</td>
    </tr>
    <tr>
      <th>4</th>
      <td>뚱땡이밥집</td>
      <td>위치는 쉽게 도달 할 수 있습니다. 기운은 장소의 전반적인 분위기와 잘 어울립니다....</td>
      <td>밥 마니아!</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>208</th>
      <td>용출횟집</td>
      <td>성인 5 + 아기 1명은 갔더니 4명짜리 테이블로 안내하며 아기는 낑겨 앉고 성인 ...</td>
      <td>제주도의 역갑질 식당</td>
      <td>1.0</td>
      <td>방문 날짜: 2017년 8월</td>
    </tr>
    <tr>
      <th>209</th>
      <td>용출횟집</td>
      <td>제주도 북부쪽에 위치한 횟집입니다. 식당앞에서 비행기들이 제주공항에 착륙하기 위해서...</td>
      <td>제주도 횟집</td>
      <td>4.0</td>
      <td>방문 날짜: 2016년 5월</td>
    </tr>
    <tr>
      <th>210</th>
      <td>용출횟집</td>
      <td>용담 해안도로를 따라 횟집들이 쭉 늘어서 있으나 그 중 유난히 회가 쫄깃쫄깃하고 감...</td>
      <td>신선한 회</td>
      <td>5.0</td>
      <td>방문 날짜: 2015년 7월</td>
    </tr>
    <tr>
      <th>211</th>
      <td>요레</td>
      <td>여러가지 다양한 퓨전요리라 입맛에 맞는요리를 선택해서 먹을수있어 좋아요~^^제주시 ...</td>
      <td>맛집~!</td>
      <td>5.0</td>
      <td>방문 날짜: 2016년 12월</td>
    </tr>
    <tr>
      <th>212</th>
      <td>요레</td>
      <td>서버는 고객을 대하는 방법을 정확히 알고있었습니다. . 정말 아름답게 꾸며진 곳. ...</td>
      <td>옛날!</td>
      <td>3.0</td>
      <td>방문 날짜: 2018년 7월</td>
    </tr>
  </tbody>
</table>
<p>257 rows × 5 columns</p>
</div>




```python
review2.to_csv("1107에서1286.csv",encoding='utf-8-sig')
```
