---
title: <Python> Asyncio - 개념이해부터 실전적용까지 (끝)
date: 2019/11/24
category: Python
comments: true
tag: 
- Python
keywords:
- Python
- Asyncio
- Concurrency
---
실제로 돌려본 코드를 뜯어보자.

```Python
import asyncio
import aiohttp
import time
import write_read_module_for_article_crawling as wrm
import re
from pathlib import Path
from bs4 import BeautifulSoup
from datetime import datetime
```
write_read_module_for_article_crawling은 크롤링에 사용했던 입출력 함수들을 모아 놓은 모듈이다.
<br>


```Python
current_folder = str(Path(__file__).absolute().parent) # 현재 스크립트가 들어있는 폴더
result_folder = '/Cnet_result' # 결과들을 저장할 폴더

def list_spliter_by_num(target_list, number):
    """
    넣은 리스트를 number 단위로 쪼개는 함수
    """
    for i in range(0, len(target_list), number):
        yield target_list[i:i+number]


def decomposer(target_element, tag_conditionList_param):
    """
    What are the structure of this function's Parameters?
    
    target_element > 
        the object that has 'decompose', 'find_all' method
    
    tag_conditionList_param >
        [{'tag' : the tag which you want to delete, 'Condition' = {Search Condition}}, 
        \{another}, {others}, ...]
        e.g [{'tag' : 'span', 'Condition' : {"data-item" : True}}]
    """
    for each_cmd in tag_conditionList_param:
        for each in target_element.find_all(each_cmd['tag'], each_cmd['Condition']):
            each.decompose()
```
`list_spliter_by_num` : 리스트와 숫자를 받아서, 숫자 단위로 리스트를 쪼개서 반환한는 제너레이터다.
`decomposer` : 없애고 싶은 조건의 태그를 딕셔너리로 받아서, beautifulsoup의 decompose 함수로 일괄 제거하는 함수다.
<br>


```Python
async def get_page_lastnum(archive_page_url):
    """
    cnet 아카이브 페이지의 url을 받는다.
    아카이브 페이지는 연도별로 되어 있다.
    해당 연도의 마지막 페이지 숫자를 받아온다.
    """
    async with aiohttp.ClientSession() as session:
        async with session.get(archive_page_url, headers={'user-agent': 'Mozilla/5.0'}) as req:
            if req.status == 404:
                return None # 없는 페이지라면 반환값은 None이 된다.
            html = await req.text()
            soup = BeautifulSoup(html, 'html.parser')
            lastpage_num = soup.find('a', class_='page last').getText() # 해당 연도의 마지막 페이지 가져옴
    return lastpage_num
```
Cnet archive 페이지에서 해당 연도의 마지막 페이지 숫자를 받아온다.
함수 정의와 HTTP 요쳥 시, async 키워드가 쓰였음에 주목하자.
함수 내부의 `html = await req.text()`, `await `이 키워드도 중요하다. html 문서 구조를 가져오는 동안 다른 작업을 실행 가능하다는 키워드이다.
<br>


```Python
async def get_article_link(archive_page_url, prefix):
    """
    cnet 아카이브 페이지의 url을 받아서, 그 페이지의 모든 기사 url을 수집한다.
    아카이브 페이지는 연도별로 되어 있다.
    """
    article_links_in_one_page = []
    async with aiohttp.ClientSession() as session:
        async with session.get(archive_page_url, headers={'user-agent': 'Mozilla/5.0'}) as req:
            if req.status == 404:
                return None # 없는 페이지라면 반환값은 None이 된다.
            html = await req.text()
            soup = BeautifulSoup(html, 'html.parser')
        article_link_table = soup.find_all('ul', class_='items')[1]
        # 페이지에 unordered list가 2개이고, 2번째 리스트가 기사 목록 리스트이다.
        article_link_tags = article_link_table.find_all('a')
        # 기사 리스트 테이블에서 a 태그 전부 가져온다. find_all이기 때문에 리스트를 반환한다.
        for each_tag in article_link_tags:
            article_links_in_one_page.append(prefix + each_tag.get('href')) 
            # 완전한 기사 url 리스트가 만들어짐
    return article_links_in_one_page


async def get_article_link_run(start_year):
    current_year = datetime.now().year # 현재년도를 받는다
    year = start_year
    prefix = 'https://www.cnet.com' # 기사 URL 조합에 필요한 접두어
    prefix_archive = 'https://www.cnet.com/sitemaps/articles/' # 아카피브 페이지 URL 조합에 필요한 접두어

    tasks_for_lastpage = []
    # 각 연도별 마지막 페이지 받아옴
    for each in range(start_year, current_year+1):
        tasks_for_lastpage.append(get_page_lastnum(prefix_archive + str(each)))
    
    lastpage_num_list = await asyncio.gather(*tasks_for_lastpage)
    # 마지막 페이지를 연도순으로, 리스트로 받아옴 
    lastpage_yearnum_dict = {key:val for key,val in enumerate(lastpage_num_list, 1995)}
    # dictionary comprehension으로 { 연도 : 마지막 페이지 } 구조를 생성함
    while year <= current_year:
        try:
            tasks = []
            for each in range(1,int(lastpage_yearnum_dict[year])+1): # 아카이브 연도 페이지의 첫 페이지부터 마지막 페이지까지
                url = prefix_archive + str(year) + '/' + str(each) # 각 페이지 url 생성
                tasks.append(get_article_link(url ,prefix)) # task 리스트에 코루틴 하나 만들어서 넣음
            
            splitted_list = list_spliter_by_num(tasks, 10) 
            # 리스트를 10개 단위로 쪼개서, 10개씩으로 구성된 리스트를 원소로 가지는 
            # 리스트를 생성항. 10개가 안되면 거기까지만 생성됨.

            year_total_list = [] # 10개씩 쪼개서 발생한 결과들을 다 담을 리스트

            for each in splitted_list:
                results = await asyncio.gather(*each)
                    # task를 10개씩 끊어서 이벤트 루프에 보내서 실행시키고 결과를 리스트로 받음
                    # 결과의 순서는 들어간 awaitable objects의 순서를 따름. 순서가 보장된다는게 너무 좋다.
                year_total_list.extend(results) # year_total_list에 넣음
            
            for each in year_total_list:
                wrm.write_to_txt(each, folder=current_folder + result_folder, filename='url_list')
            print('저장함') # url 목록 저장

            year += 1
            error_limit = 0
            
        except Exception as e:
            print(e)
            print(f'중단된 연도 : {year}')
            time.sleep(120)
            error_limit += 1
            if error_limit >= 5:
                print('좀 쉬고 오세요. 5회 이상 차단됨.')
                break
```
`lastpage_num_list = await asyncio.gather(*tasks_for_lastpage)` 부분을 보자.
asyncio.gather는 전달된 task들의 반환값을 모아서 리스트로 전달한다. 코루틴이 전달되면 알아서 task로 wrapping해주기 때문에, tasks_for_lastpage 리스트에 get_article_link라는 코루틴들을 넣어서 asyncio.gather에 전달하였다.
각 연도 아카이브의 마지막 페이지 숫자를 모두 받아오면, 본격적으로 기사 링크들을 받아오기 시작한다.
`splitted_list = list_spliter_by_num(tasks, 10)` 이걸로 task들을 10개 단위로 쪼갠 이유는, 한 번에 너무 많은 요청을 넣으면 IP가 막히기 때문이다.
되는대로 task들을 한 번에 때려넣었으면, Synchronous-Blocking 방식에 비해 16배 정도의 성능 향상을 기대할 수 있는데, 10개 단위로 쪼개 넣을 수 밖에 없었기 때문에 성능 향상은 4배에서 만족해야 했다.

이후로는 원리는 다 똑같고, 함수들이 하는 행동이 달라질 뿐이다.
모든 기사의 URL을 가지고 있는 텍스트 파일에서 url을 읽어서 내용을 가져오고, 이를 10개씩 Asynchronous 방식으로 처리했을 뿐이다.
이렇게 1.1일정도 돌리고 나면, 36만개의 Cnet 기사를 모두 가져올 수 있게 된다.
<br>


```Python
async def get_article_contents(article_url, f1=None, args_f1=None):
    """
    cnet 기사 페이지의 url을 받아서, 그 기사의 제목, 날짜, 이름을 수집한다.
    f1에는 decomposer가 들어간다.
    """
    async with aiohttp.ClientSession() as session:
        async with session.get(article_url, headers={'user-agent': 'Mozilla/5.0'}) as req:
            if req.status == 404:
                # print('그런 페이지가 없다')
                return None # 없는 페이지라면 반환값은 None이 된다.
            html = await req.text()
            soup = BeautifulSoup(html, 'html.parser')

            try :
                title_pa = soup.find('div', class_='c-head').find('h1', class_='speakableText').getText()
            except AttributeError :
                title_pa = None

            try :
                contents_pa_row = soup.find('div', class_='article-main-body')
                f1(contents_pa_row, args_f1) # decomposer : button 이름 등 노이즈 요소 날려버리기
                contents_pa = contents_pa_row.getText().replace('\n','').replace('\r','')
            except AttributeError :
                contents_pa = None

            try:
                #작성날짜 받아옴
                published_time_row = soup.find('time', {'datetime' : True}).get('datetime')
                T_loc = published_time_row.find('T')
                published_time_pa = published_time_row[0:T_loc]
            except AttributeError :
                published_time_pa = None

            article = dict(title = title_pa, time = published_time_pa, contents = contents_pa, link = article_url)
    return article


async def get_article_contents_run(last_progress=0):
    """
    last progress는 모종의 이유로 스크립트가 도중에 꺼졌을 경우, 이어하기를 위해 있는 변수임
    """
    # decomposer에 사용될 argument
    command_list = [{'tag': 'span', 'Condition': {'class' : [re.compile(r'credit'), re.compile(r'Credit$')]}}\
    ,{'tag': 'span', 'Condition': {'data-item' : True}}\
    ,{'tag': 'div', 'Condition': {'class' : ['on', 'off']}}\
    ,{'tag': 'script', 'Condition': {'type' : 'application/javascript'}}\
    ,{'tag': 'footer', 'Condition' : ''}
    ]
    # 저장 시 사용될 헤더
    header_list = ['Title', 'Published_time', 'Contents', 'Link']

    # url 파일 읽어올 제너레이터
    reader_gen = wrm.read_txt_linebyline(current_folder + result_folder + '/url_list')
    
    # 진행상황 체크
    progress = 0

    if last_progress != 0: # 입력한 last progress까지 generator를 돌림
        check = 0
        while check < last_progress:
            next(reader_gen)
            check += 1

    while True:
        try :
            try:
                tasks = []
                for _ in range(1,11):
                    url = next(reader_gen)
                    # print(url)
                    tasks.append(get_article_contents(url, f1=decomposer, args_f1=command_list))
                result = await asyncio.gather(*tasks)
                wrm.write_to_sthsv_dict(result, folder=current_folder + result_folder, filename='article_contents', header_list=header_list, delimiter_pa='|')
                progress += 10
            except StopIteration:
                print('모든 URL에 대한 수집이 완료되었습니다.')
                break
        except Exception as e:
            error = str(e)
            wrm.write_to_txt([error, progress], folder=current_folder + result_folder, filename='err_and_progress')
        print(f'수집한 기사 수 : {progress}')


########## 기사 URL 수집부분 ##################
start = time.time()
asyncio.run(get_article_link_run(1995))
end = time.time()
print(f'totaltime_url = {end-start}')
###########################################

######### 기사 내용 수집부분 ###################
last_progress = int(input("마지막 진행상황을 입력하세요 : "))

start = time.time()
asyncio.run(get_article_contents_run(last_progress))
end = time.time()
print(f'totaltime_article = {end-start}')
###########################################
```