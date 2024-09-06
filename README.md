# SKN05-1nd-4Team

 SK네트웍스 AI과정 1차 프로젝트
 
**1. 팀 소개**
- 팀명
  - 작성 필요
- 팀원

신혜원

![KakaoTalk_20240906_121451675](https://github.com/user-attachments/assets/9019c838-54bf-4ece-b27d-0e4917bd4001)




  
 
**2. 프로젝트 개요**

- 프로젝트 명
  - MySQL을 활용한 자동차 등록 현황 조회 및 FAQ 제공 프로그램 구현

-프로젝트 소개
  - 자동차 등록 현황 데이터를 수집하고, MySQL 데이터베이스에 저장한다. 그리고 MySQL 데이터베이스로부터 원하는 데이터들을 조회할 수 있도록 하고, FAQ도 제공해주는 프로그램을 구현한다.

- 프로젝트 필요성(배경)
  1) 자동차 등록 현황과 관련하여 원하는 데이터를 쉽게 조회 가능
  2) 그래프를 통한 전반적인 데이터 분석 및 이를 활용한 자동차 관련 정책 수립 

- 프로젝트 목표
  1) 지금까지 배운 학습 내용 복습 및 활용 (함수, 크롤링, DB 등)
  2) 데이터 수집 및 분석 능력 향상
  3) 적절한 작업 분배 및 지속적인 커뮤니케이션을 통한 협업 수행 능력 향상
  
**3. 기술 스택**
   <img src="https://img.shields.io/badge/python-3776AB?style=for-the-badge&logo=python&logoColor=white">
   <img src="https://img.shields.io/badge/mysql-4479A1?style=for-the-badge&logo=mysql&logoColor=white"> 
   <img src="https://img.shields.io/badge/streamlit%20-%23FF0000.svg?style=for-the-badge&logo=streamlit&logoColor=white">
   <img src="https://img.shields.io/badge/github-181717?style=for-the-badge&logo=github&logoColor=white">
   
  
**4. ERD**
   
   ![화면 캡처 2024-09-06 103022](https://github.com/user-attachments/assets/6cc28547-be58-405b-bdda-ce84532d0fb8)
   
 
**5. 주요 프로시저**

- 데이터 선택(정보)

  ![화면 캡처 2024-09-06 112025](https://github.com/user-attachments/assets/e87cc110-40f7-43a1-8b6d-55ed7d7de33c)

- 데이터 가공
  
  ![화면 캡처 2024-09-06 113001](https://github.com/user-attachments/assets/a4a5bf0a-7ff9-440e-ae70-4fa35ebabc9f)

 - DB 구축

```python
# 필요한 라이브러리

import pymysql
import pandas as pd
```


```python
# MySQL 데이터 베이스 생성 함수

def create_database(database_name:str, username = 'root', password = 'BEA02050509!', host = 'localhost'):
    '''
    원하는 데이터베이스 이름을 문자열로 해서 함수 인수로 넣어주면 됩니다.
    그리고 password 인수 값에는 각자의 MySQL 패스워드 값으로 바꿔주세요.
    '''
    
    connection = pymysql.connect(
    host = host,
    user = username, 
    password = password)
    
    with connection.cursor() as cursor:
        cursor.execute(f"create database {database_name}")
    connection.commit()
    connection.close()
```


```python
# 데이터 베이스 생성 함수 실행

database_name = "first_project"

create_database(database_name)
```


```python
# 데이터 베이스 접근 함수

def get_connection(database_name, username = 'root', password = 'BEA02050509!', host = 'localhost'):
    conn = pymysql.connect(
        host = host,
        user = username,
        password = password,
        database = database_name,
        charset = 'utf8mb4',
        cursorclass = pymysql.cursors.DictCursor
    )
    return conn
```


```python
# 연도 테이블 생성 함수

def year_table(update:list=None):
    '''
    처음에 2019~2023년의 연도 테이블을 만든다.
    그리고 나중에 이후 연도를 추가로 넣고 싶으면,
    update 인수에 리스트를 넣어준다.
    '''
    
    if update == None:
        year_values = list(range(2019, 2024))
    
        connection = get_connection(database_name)
        with connection.cursor() as cursor:
            cursor.execute("CREATE TABLE IF NOT EXISTS year (year_id INT AUTO_INCREMENT NOT NULL PRIMARY KEY, year INT NOT NULL)")
            for i in year_values:
                sql = f"INSERT INTO year (year) VALUES ({i})"   # INSERT INTO 테이블_이름 (열_이름) VALUES (값)
                cursor.execute(sql)

        connection.commit()
        connection.close()
    else:
        connection = get_connection(database_name)
        with connection.cursor() as cursor:
            for i in update:
                sql = f"INSERT INTO year (year) VALUES ({i})"
                cursor.execute(sql)

        connection.commit()
        connection.close()
```


```python
# 연도 테이블 생성 함수 실행

year_table()
```


```python
# 차종 테이블 생성 함수

def car_type():
    '''
    차종 테이블은 차종이 고정되어 있으므로, 데이터를 따로 받지 않는다.
    '''

    car_type = ['승용', '승합', '화물', '특수']

    connection = get_connection(database_name)
    with connection.cursor() as cursor:
        cursor.execute("CREATE TABLE IF NOT EXISTS car_type (car_id INT AUTO_INCREMENT NOT NULL PRIMARY KEY, type VARCHAR(2) NOT NULL)")
        for i in car_type:
            sql = f"INSERT INTO car_type (type) VALUES ('{i}')"
            cursor.execute(sql)
            
    connection.commit()
    connection.close()
```


```python
# 차종 테이블 생성 함수 실행

car_type()
```


```python
# 데이터 입력 함수

def Insert_data(filename:str, update:list=None):
    '''
    연도별 차종별 데이터가 함께 있는 파일의 이름을 문자열로 넣어 준다.
    초기 데이터 삽입 이후에 추가로 데이터를 넣고 싶다면,
    데이터 파일명에다가 연도 리스트 또한 update 인수를 통해 전달해준다.
    '''
    
    len_type = list(range(1,5))
    data = pd.read_csv(filename)

    connection = get_connection(database_name)
    with connection.cursor() as cursor:
        cursor.execute("CREATE TABLE IF NOT EXISTS car_amount (id INT AUTO_INCREMENT NOT NULL PRIMARY KEY, year_id INT NOT NULL, car_id INT NOT NULL, amount INT NOT NULL, FOREIGN KEY (year_id) REFERENCES year(year_id), FOREIGN KEY (car_id) REFERENCES car_type(car_id))")
    connection.commit()
    connection.close()
        
    connection = get_connection(database_name)
    with connection.cursor() as cursor:
        sql = "SELECT COUNT(year_id) AS total FROM year"
        cursor.execute(sql)
        result = cursor.fetchone()
        len_year = result['total'] + 1
        
        if update == None:
            year_list = list(range(1, len_year))
            for i in year_list:
                for j in len_type:
                    value = int(data.iloc[i-1,j])
                    sql = f"INSERT INTO car_amount (year_id, car_id, amount) VALUES ({i}, {j}, {value})"
                    cursor.execute(sql)
        else:
            year_table(update)
            new_year_list = list(range(len_year, len_year + len(update)))
            for i in new_year_list:
                for j in len_type:
                    value = int(data.iloc[i-len_year,j])
                    sql = f"INSERT INTO car_amount (year_id, car_id, amount) VALUES ({i}, {j}, {value})"
                    cursor.execute(sql)

    connection.commit()
    connection.close()
```


```python
# 데이터 입력 함수 실행

Insert_data('car.csv')
```


**FAQ**

```python
pip install selenium webdriver-manager
```

```python
import urllib.request
from sqlalchemy import create_engine
from bs4 import BeautifulSoup
import urllib.request
import pandas as pd
import re
```

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager

# Optional: Set up Chrome options if needed
chrome_options = Options()
# chrome_options.add_argument("--headless")  # Uncomment to run in headless mode

# Set up the Chrome WebDriver using webdriver-manager
service = Service(ChromeDriverManager().install())
wd = webdriver.Chrome(service=service, options=chrome_options)

# Navigate to the desired URL
wd.get('https://www.chevrolet.co.kr/faq/')

# Maximize the browser window
wd.maximize_window()

# Your additional code for interaction or scraping would go here

# Close the browser after use
# wd.quit()

```


```python
result = []
faq_list = []
category = []
question = []
answer = []

for j in range(2,11):
    try:
        selenium_el = wd.find_element(By.XPATH,f'//*[@id="q-main-content"]/adv-grid[2]/adv-col[1]/div/adv-grid/adv-col[{j}]/div/a')
        selenium_el.click()
        
        html = wd.page_source  #  현재 페이지 정보를 html로 변환
        soup = BeautifulSoup(html, 'html.parser') # html 정보를 파싱
        
        faq = soup.select('#q-main-content > adv-grid.large-margin > adv-col > div > div > div > div')
        for i in range(0,len(faq),2):
            temp = faq[i].text.strip().split(']')
            k, q = temp[0][1:],temp[1]
            a = faq[i+1].text.strip()
        
            # 정규 표현식 개선: 숫자, 한글, 공백, 특수 문자 등을 더 많이 포함
            re_a = re.findall(r'[가-힣0-9\s\.\,\?\!]+', a)
            an = ' '.join(re_a).replace('\xa0', '').replace('\n', '').replace('\t', '')
            
            category.append(k)
            question.append(q)
            answer.append(an)
            
    except Exception as e:
        print(i)
        print(j)
```


```python
import pandas as pdS

data = {
    'category':category,
    'question' : question,
    'answer': answer}

df = pd.DataFrame(data)
df
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
      <th>category</th>
      <th>question</th>
      <th>answer</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>구매관련</td>
      <td>근저당 설정 해지 방법 및 절차를 알고싶습니다.</td>
      <td>자동차 근저당 설정 해지는 할부를 진행한 할부사의 지점을 방문하여 근저당 설정 해지...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>구매관련</td>
      <td>저공해 증명서 재발급 철차 및 장소를 알고싶습니다.</td>
      <td>저공해 자동차 증명서는 차량 구입을 담당한 전시장을 통해 재 발급이 가능합니다.차량...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>구매관련</td>
      <td>재고차 구입 조건시 할인율을 알고싶습니다.</td>
      <td>쉐보레 전시장을 통해 자세한 상담 받아주시기 바랍니다.</td>
    </tr>
    <tr>
      <th>3</th>
      <td>구매관련</td>
      <td>신차 구입시 출고 예정일 확인 방법을 알고싶습니다.</td>
      <td>출고예정일은 고객님께서 선택하신 차종, 옵션, 색상 등에 따라 다소 차이가 있습니다...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>구매관련</td>
      <td>할부금 중도 상환 방법을 알고싶습니다.</td>
      <td>할부금 중도 상환은 가능하며, 중도상환에 따른 수수료가 발생됩니다.할부금 잔액과 수...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>123</th>
      <td>OnlineShop</td>
      <td>선포인트가 무엇인가요?</td>
      <td>선포인트는 고객님께서 대상카드를 이용하여 쉐보레 차량을 구입하실 경우, 최대 30만...</td>
    </tr>
    <tr>
      <th>124</th>
      <td>OnlineShop</td>
      <td>온라인 구매를 위해 사전에 준비할 것들은?</td>
      <td>구매하실 차종과 차량대금 지불 방법에 따라 다릅니다. 자세한 내용은 주문 진행하신 ...</td>
    </tr>
    <tr>
      <th>125</th>
      <td>OnlineShop</td>
      <td>개명했는데 휴대폰 인증에서 에러가 나는 경우는 어떻게 해야하나요?</td>
      <td>쉐보레 홈페이지에 가입된 회원 정보와 휴대폰 인증의 이름이 일치하지 않아 발생하는 ...</td>
    </tr>
    <tr>
      <th>126</th>
      <td>OnlineShop</td>
      <td>본인 명의 휴대폰이 아니면 서비스이용이 불가한지?</td>
      <td>본인을 식별하기 위한 휴대폰 인증을 사용하고 있어 반드시 본인 명의의 휴대폰이 필요...</td>
    </tr>
    <tr>
      <th>127</th>
      <td>OnlineShop</td>
      <td>비회원은 온라인 구매가 불가한지?</td>
      <td>회원만 온라인 구매가 가능합니다. 본인 확인, 기존 구매 이력, 오토포인트 등을 조...</td>
    </tr>
  </tbody>
</table>
<p>128 rows × 3 columns</p>
</div>




```python
from sqlalchemy import create_engine
username = 'root'
password = '1234'
host = 'localhost'
database = 'mydatabase'

engine = create_engine(f'mysql+pymysql://{username}:{password}@{host}/{database}')

df.to_sql('qna_che', con=engine, if_exists='append', index=False)

engine.dispose()
```


**조회시스템**

```python
from selenium import webdriver
import pandas as pd
import os
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
import pymysql
import pandas as pd
from sqlalchemy import create_engine
from data_db import create_database
```


```python


wd = webdriver.Chrome()
wd.get('https://stat.molit.go.kr/portal/cate/statMetaView.do?hRsId=58&hFormId=1244&hSelectId=1244&hPoint=00&hAppr=1&hDivEng=&oFileName=&rFileName=&midpath=&sFormId=1244&sStart=2015&sEnd=2023&sStyleNum=562&settingRadio=xlsx')


element = wd.find_element(By.XPATH, '//*[@id="main"]/div/div[3]/div[1]/div/div[2]/div[2]/div/dl/dd[1]/ul/li[1]/a/em')
wd.execute_script("arguments[0].click();", element)

html = wd.page_source
soup = BeautifulSoup(html,'html.parser')

```


```python
download_folder = os.path.expanduser('~/Downloads')
file_name = soup.select('#main > div > div.contents-wrap > div.contents > div > div:nth-child(2) > div.mu-accordion-body > div > dl > dd:nth-child(2) > ul > li:nth-child(1) > a > em')[0].text[2:]
file_name = file_name.replace(' ','_')
file_name
file_path = os.path.join(download_folder, file_name)

try:
    df = pd.read_excel(file_path,sheet_name ='19.연도별 자동차 등록현황',engine ='openpyxl', header = [2])
    print("파일이 성공적으로 읽혔습니다.")
    print(df.head()) 
except FileNotFoundError:
    print("지정한 파일이 존재하지 않습니다.")
except Exception as e:
    print(f"파일을 읽는 중 오류가 발생했습니다: {e}")
```

    파일이 성공적으로 읽혔습니다.
         차종     승용 Unnamed: 2 Unnamed: 3 Unnamed: 4     승합 Unnamed: 6 Unnamed: 7  \
    0    년도     관용        자가용        영업용          계     관용        자가용        영업용   
    1  2007  20714   11674085     404980   12099779  12650     999807      92492   
    2  2008  21388   12025715     436706   12483809  13269     987448      95981   
    3  2009  22267   12551833     449719   13023819  14177     967890      98620   
    4  2010  22872   13124972     483925   13631769  15039     931740     102946   
    
      Unnamed: 8     화물  ... Unnamed: 11 Unnamed: 12    특수 Unnamed: 14  \
    0          계     관용  ...         영업용           계    관용         자가용   
    1    1104949  25230  ...      334584     3171351  2090       10945   
    2    1096698  25535  ...      338711     3160338  2110       11372   
    3    1080687  25970  ...      341745     3166512  2070       11890   
    4    1049725  26306  ...      345805     3203808  2055       12604   
    
      Unnamed: 15 Unnamed: 16     합계 Unnamed: 18 Unnamed: 19 Unnamed: 20  
    0         영업용           계     관용         자가용         영업용           계  
    1       39063       52098  60684    15496374      871119    16428177  
    2       39892       53374  62302    15820627      911290    16794219  
    3       40232       54192  64484    16330410      930316    17325210  
    4       41395       56054  66272    16901013      974071    17941356  
    
    [5 rows x 21 columns]
    


```python
df = df.dropna(how='all',axis=1)
df = df.dropna(how='all',axis=0)
car_data = df.loc[7:]
car_data

car_data = car_data[['차종','Unnamed: 4','Unnamed: 8','Unnamed: 12','Unnamed: 16']]
```


```python
car_data.columns = ['year','승용','승합','화물','특수']
car_data = car_data.reset_index(drop=True)
car_data
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
      <th>year</th>
      <th>승용</th>
      <th>승합</th>
      <th>화물</th>
      <th>특수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2013</td>
      <td>15078354</td>
      <td>970805</td>
      <td>3285707</td>
      <td>65998</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2014</td>
      <td>15747171</td>
      <td>947012</td>
      <td>3353683</td>
      <td>70089</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015</td>
      <td>16561665</td>
      <td>920320</td>
      <td>3432937</td>
      <td>74963</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016</td>
      <td>17338160</td>
      <td>892539</td>
      <td>3492173</td>
      <td>80479</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017</td>
      <td>18034540</td>
      <td>867522</td>
      <td>3540323</td>
      <td>85910</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2018</td>
      <td>18676924</td>
      <td>843794</td>
      <td>3590939</td>
      <td>90898</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2019</td>
      <td>19177517</td>
      <td>811799</td>
      <td>3592586</td>
      <td>95464</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2020</td>
      <td>19860955</td>
      <td>783842</td>
      <td>3615245</td>
      <td>105937</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2021</td>
      <td>20410648</td>
      <td>749968</td>
      <td>3631975</td>
      <td>118510</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2022</td>
      <td>20952759</td>
      <td>723961</td>
      <td>3696317</td>
      <td>130041</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2023</td>
      <td>21390202</td>
      <td>694574</td>
      <td>3726400</td>
      <td>138025</td>
    </tr>
  </tbody>
</table>
</div>




```python
year = car_data[['year']]
year
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
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2013</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2014</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2015</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2018</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2019</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2020</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2021</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2022</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2023</td>
    </tr>
  </tbody>
</table>
</div>




```python
car_type = pd.DataFrame(list(car_data.columns[1:]),columns=['차종'])
car_type
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
      <th>차종</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>승용</td>
    </tr>
    <tr>
      <th>1</th>
      <td>승합</td>
    </tr>
    <tr>
      <th>2</th>
      <td>화물</td>
    </tr>
    <tr>
      <th>3</th>
      <td>특수</td>
    </tr>
  </tbody>
</table>
</div>




```python
create_database('test')
```

 
**6. 수행결과(테스트/시연 페이지)**
    
![화면 캡처 2024-09-06 111509](https://github.com/user-attachments/assets/a4e1cd77-7fb9-40be-b5d3-592c7e8264e3)

 
**7. 한 줄 회고**
    - 작성 필요
