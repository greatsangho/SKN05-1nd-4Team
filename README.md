# SKN05-1nd-4Team

 SK네트웍스 AI과정 1차 프로젝트
 
**1. 팀 소개**
- 팀명
  - # Fundas
     "노력하는 자는 즐기는 자를 이길 수 없다!"
 
  ![image](https://github.com/user-attachments/assets/5c37680e-b382-4798-883f-711e2ba4c821)

 
- 팀원


|**김지연**|**배윤관**|**신혜원**|**허상호**|**황호준**|
|:---:|:---:|:---:|:---:|:---:|
|데이터 입력 자동화|데이터 베이스|데이터 전처리, 깃허브|FAQ, 데이터 시각화|FAQ, 데이터 시각화|


  
 
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
   
 ![erd](https://github.com/user-attachments/assets/4ad68f85-5e44-4dbd-9c40-53b4c31817f4)

   
 
**5. 주요 프로시저**

- 데이터 선택(정보)

  ![화면 캡처 2024-09-06 112025](https://github.com/user-attachments/assets/e87cc110-40f7-43a1-8b6d-55ed7d7de33c)

- 데이터 수집 및 가공, 기획

 - DB 구축

****데이터 자동 입력****

```python
from selenium import webdriver
import os
import pandas as pd
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
import pymysql
import pandas as pd
from sqlalchemy import create_engine

wd = webdriver.Chrome()
wd.get('https://stat.molit.go.kr/portal/cate/statMetaView.do?hRsId=58&hFormId=1244&hSelectId=1244&hPoint=00&hAppr=1&hDivEng=&oFileName=&rFileName=&midpath=&sFormId=1244&sStart=2015&sEnd=2023&sStyleNum=562&settingRadio=xlsx')

element = wd.find_element(By.XPATH, '//*[@id="main"]/div/div[3]/div[1]/div/div[2]/div[2]/div/dl/dd[1]/ul/li[1]/a/em')
wd.execute_script("arguments[0].click();", element)

html = wd.page_source
soup = BeautifulSoup(html,'html.parser')

download_folder = os.path.expanduser('~/Downloads')
file_name = soup.select('#main > div > div.contents-wrap > div.contents > div > div:nth-child(2) > div.mu-accordion-body > div > dl > dd:nth-child(2) > ul > li:nth-child(1) > a > em')[0].text[2:]
file_name = file_name.replace(' ','_')
file_path = os.path.join(download_folder, file_name)

try:
    df = pd.read_excel(file_path,sheet_name ='19.연도별 자동차 등록현황',engine ='openpyxl', header = [2])
    print("파일이 성공적으로 읽혔습니다.")
    print(df.head()) 
except FileNotFoundError:
    print("지정한 파일이 존재하지 않습니다.")
except Exception as e:
    print(f"파일을 읽는 중 오류가 발생했습니다: {e}")

year_data = df.iloc[:,:1]
year_columns = ['year']
year_data.columns = year_columns
#year_data 데이터 인덱스 생성
year_data2 = year_data.reset_index(drop=True)
year_data2 = year_data2.reset_index().rename(columns={'index':'year_id'})

def create_database(database_name, password, username = 'root', host = 'localhost'):

    connection = pymysql.connect(
    host = host,
    user = username, 
    password = password)
    
    with connection.cursor() as cursor:
        cursor.execute(f"create database {database_name}")
    connection.commit()
    connection.close()

def get_connection(database_name, password, username = 'root', host = 'localhost'):

    conn = pymysql.connect(
        host = host,
        user = username,
        password = password,
        database = database_name,
        charset = 'utf8mb4',
        cursorclass = pymysql.cursors.DictCursor
    )
    return conn
    
def Insert_data(file, table_name:str, PK:str, database_name, password, username = 'root', host = 'localhost'):
    engine = create_engine(f'mysql+pymysql://{username}:{password}@{host}/{database_name}')
    file.to_sql(f'{table_name}', con=engine, if_exists='replace', index=False)
    engine.dispose()

    connection = get_connection(database_name, password)
    if PK != '':
        with connection.cursor() as cursor:
            cursor.execute(f"ALTER TABLE {table_name} ADD PRIMARY KEY ({PK});")
        connection.commit()
        connection.close()
        
'''
주의사항!!!
'''

database_name = 'carfirst'
password = '1234'    # 각자 입력!!!

create_database(database_name, password)

Insert_data(year_data2, 'year', 'year_id', database_name, password)
data2 = pd.read_excel(file_path, sheet_name='09.차종별_유형별 현황',engine='openpyxl',header=[2])
data2 = data2.dropna(how='all',axis=1)
data2 = data2.dropna(how='all',axis=0)
data2
kind_data = data2.iloc[1:-1,:1]
kind_data

# # NaN 값을 제외한 리스트로 변환
real_kind_list = kind_data['Unnamed: 0'].dropna().tolist()
# list에 인덱스 생성 안되서 다시 바꿔줌
kind_data=pd.DataFrame(real_kind_list) 
#kind_data 데이터 인덱스 생성
kind_columns = ['kind']
kind_data.columns = kind_columns
kind_data2 = kind_data.reset_index().rename(columns={'index':'kind_id'})

Insert_data(kind_data2, 'kind', 'kind_id', database_name, password)
df = df[['차종', 'Unnamed: 4', 'Unnamed: 8', 'Unnamed: 12', 'Unnamed: 16']]
df = df.drop(0)
df.columns = ['year', '승용', '승합', '화물', '특수']

sum_list = []
year_list = []
kind_list = []

# 각 행을 순회
for i in range(len(df)):
    year_row = df.iloc[i]
    # 각 열을 순회
    for j in range(len(year_row)):
        if j>0:
            kind_value = year_row.iloc[j]
            # 리스트에 값 추가
            year_list.append(i)
            kind_list.append(j-1)
            sum_list.append(kind_value)
        else:
            pass

# 결과를 딕셔너리 형태로 묶기
sum_dict = zip(year_list, kind_list, sum_list)
sum_dict =pd.DataFrame(sum_dict)
# 결과 출력
print(sum_dict)

column_names = ['year_id','kind_id','total']
sum_dict.columns = column_names
sum_dict

Insert_data(sum_dict, 'year_kind_sum','', database_name, password)
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
         0  1         2
    0    0  0  12099779
    1    0  1   1104949
    2    0  2   3171351
    3    0  3     52098
    4    1  0  12483809
    ..  .. ..       ...
    63  15  3    130041
    64  16  0  21390202
    65  16  1    694574
    66  16  2   3726400
    67  16  3    138025
    
    [68 rows x 3 columns]


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





from sqlalchemy import create_engine
username = 'root'
password = '1234'
host = 'localhost'
database = 'mydatabase'

engine = create_engine(f'mysql+pymysql://{username}:{password}@{host}/{database}')

df.to_sql('qna_che', con=engine, if_exists='append', index=False)

engine.dispose()


```python
**조회시스템**

import streamlit as st
import pandas as pd
import pymysql

# 데이터베이스에서 year와 kind 값을 가져오는 함수
def get_years_and_kinds(username='root', password='1234', host='localhost'):
    database_name = "carfirst"  # 고정된 데이터베이스 이름
    connection = pymysql.connect(
        host=host,
        user=username,
        password=password,
        database=database_name,
        cursorclass=pymysql.cursors.DictCursor
    )
    
    # 연도 데이터 가져오기
    with connection.cursor() as cursor:
        cursor.execute("SELECT DISTINCT year FROM year")
        years = cursor.fetchall()
        year_list = [row['year'] for row in years]
    
    # 종류 데이터 가져오기
    with connection.cursor() as cursor:
        cursor.execute("SELECT DISTINCT kind FROM kind")
        kinds = cursor.fetchall()
        kind_list = [row['kind'] for row in kinds]
    
    connection.close()
    
    return year_list, kind_list

# 검색 함수 정의
def search(year, kind, username='root', password='1234', host='localhost'):
    database_name = "carfirst"  # 고정된 데이터베이스 이름
    connection = pymysql.connect(
        host=host,
        user=username,
        password=password,
        database=database_name,
        cursorclass=pymysql.cursors.DictCursor
    )
    
    query = """
    SELECT y.year, k.kind, s.total 
    FROM year_kind_sum s 
    INNER JOIN year y ON s.year_id = y.year_id
    INNER JOIN kind k ON s.kind_id = k.kind_id
    WHERE (%s = 'All' OR y.year = %s)
    AND (%s = 'All' OR k.kind = %s)
    """
    
    with connection.cursor() as cursor:
        cursor.execute(query, (year, year, kind, kind))
        result = cursor.fetchall()

    connection.close()

    # DataFrame으로 변환
    result_df = pd.DataFrame(result, columns=['year', 'kind', 'total'])

    return result_df

# Streamlit UI
st.title("Data Search App")

# year와 kind 데이터를 불러와서 선택 옵션으로 제공
year_list, kind_list = get_years_and_kinds()

# "All" 옵션을 추가
year_list = ["All"] + year_list
kind_list = ["All"] + kind_list

# 사용자가 선택할 수 있는 선택 박스
year = st.selectbox("Select Year:", year_list)
kind = st.selectbox("Select Kind:", kind_list)

# 버튼을 눌렀을 때 검색 실행
if st.button("Search"):
    # 검색 함수 실행 및 결과 표시
    result = search(year, kind)
    
    if not result.empty:
        st.write("Search Results:")
        st.dataframe(result)  # 결과를 테이블 형태로 출력
    else:
        st.write("No results found.")
```
 
**6. 수행결과(테스트/시연 페이지)**
    
<img width="1456" alt="2024-09-06_14 30 41" src="https://github.com/user-attachments/assets/0e3920ca-8401-4ebc-8474-3b60a32babe3">
<img width="1465" alt="2024-09-06_14 30 51" src="https://github.com/user-attachments/assets/d11487b8-5e67-443a-9bf5-464788a6e24a">
<img width="1449" alt="2024-09-06_14 30 15" src="https://github.com/user-attachments/assets/29fa8cf1-d592-488a-8c22-8630ea36c83a">


 
**7. 한 줄 회고**
    
    기획단계에서 주제에 맞는 데이터를 찾고 무엇을 구현할지에 대한 논의 과정이 쉽게 정해져서 일사천리로 프로젝트가 진행 될 줄 알았다. 
    개발단계에서 배운 것을 활용하고 자동화로 데이터가 입력되고 조회가 되게 만드는 과정이 생각보다 쉽지 않아 우왕좌왕, 우당탕탕 시행착오의 연속이였다.
    그래도 중심을 잡고 각자의 과제들을 마무리 할 수 있었던 것 같다.
    기간이 짧아서 거창하게 기획했던 수 많은 아이디어 중에 일부분만 실현이 되어 다음에 기회가 된다고 다른 아이디어도 실현 시켜 보고 싶다.
  
