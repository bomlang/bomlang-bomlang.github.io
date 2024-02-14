---
layout: post
title: 프로젝트에 필요한 게임 카드의 데이터를 크롤링하는 작업
tag: Ensemble_stars, project
---

# 데이터가 있어야 무언가 시작할 수 있다...?!?!!

말 그대로, 현재 프론트엔드를 배운 나는 큰 벽에 막혀있다. 웹사이트 만들기? 할수야 있지. 근데 생각해보자. 
무슨 웹사이트를 만들건지, 어떤 기능을 토대로 할건지, 그러기 위해서 어떤 정보가 필요한지 등등에 대한 정확한 명세서가 없다.

그래서 일단 내가 앞으로 제작할 프로젝트의 메인 포인트인 카드정보와 카드 이미지등의 데이터가 필요하다.
이 데이터가 있어야 나중에 만들어질 '효율덱 추천기능'을 만들 수 있기 때문이다.

## 데이터를 얻기위한 방법에 대해서

이번 프로젝트에 필요한 '앙상블 스타즈!!'라는 게임에선 따로 정보에 대한 API가 없으며, 심지어 공식사이트에서도 제공해주지 않는다.
단지, 카드가 기간에맞게 공개되고, 그 카드에 대한 정보를 여러 커뮤니티에서 유저 또는 특정 팬덤사이트에서 기입하는 느낌이다. (내가 알기론 현재 그렇다.)

그래서 카드에 대한 정보부터 시작해서 이미지, CG데이터를 얻는 방법에 대해 생각해 보았다.

1. [Fandom-Ensemble-Stars](https://ensemble-stars.fandom.com/) 사이트에서 크롤링하여 데이터를 가져온다.
2. 한국의 한 유저분이 만든 '구글 스프레드시트'로 제작된 **효율덱 추천시트**라는 엑셀시트에서 데이터를 가져온다.
3. [Enstars.info](https://enstars.info/)라는 사이트에서 데이터를 크롤링하여 가져온다.

라는 크게 3가지 방법이 존재했다.

먼저 3번부터 보자면, 해외의 한 유저가 취미로 만든 웹사이트로 보여졌다. 다만, 지금으로부터 약 2년전부터 업데이트 되지않아, 최신 데이터를 가져오기에 힘들다는 점이 있었다.

그 다음 2번같은 경우에 '구글 스프레드시트'를 사용하여 제작한 엑셀시트가 존재하는데, 이건 한국어로 번역도 되어있고 이미지와 각종 여러 데이터가 존재하여 나에게 가장 필요한 데이터였다.
하지만 권한의 문제로 해당 시트를 완전히 크롤링하는 부분에서 문제가 있었다.

마지막 1번같은 경우는 크롤링에 성공했다.
다만, 사이트가 영어로된 사이트이기에 카드의 정보또한 영어로 기입되어 있다는 치명적 단점이 존재했다.

## 파이썬으로 크롤링하여 데이터 가져오기

```python
import requests
from bs4 import BeautifulSoup
import json

class EnsembleCard:
    def __init__(self, name, unbloomed_card, bloomed_card, table_data):
        self.name = name
        self.unbloomed_card = unbloomed_card
        self.bloomed_card = bloomed_card
        self.table_data = self.process_table_data(table_data)

    def process_table_data(self, table_data):
        processed_data = {}
        for key, values in table_data.items():
            processed_data[key] = [self.process_value(value) for value in values]
        return processed_data

    def process_value(self, value):
        parts = value.split()
        if len(parts) == 2:
            category, amount = parts
            return {category: amount}
        return {}

    def __str__(self):
        return f"EnsembleCard(name={self.name}, " \
               f"unbloomed_card={self.unbloomed_card}, bloomed_card={self.bloomed_card}, table_data={self.table_data})"

def scrape_and_save_data(url, output_file_path):
    # 사이트에 접속
    response = requests.get(url)

    # 응답이 정상적인지 확인
    if response.status_code == 200:
        soup = BeautifulSoup(response.text, 'html.parser')

        # 데이터 추출 및 객체 생성
        card_name_tag = soup.find('h1', class_='page-header__title')
        card_name = card_name_tag.text.strip() if card_name_tag else "Unknown"

        # 추가로 가져올 이미지 데이터
        unbloomed_card_tag = soup.find('div', class_='card-single card-unbloomed')
        unbloomed_card = unbloomed_card_tag.find('img')['src'] if unbloomed_card_tag else None

        bloomed_card_tag = soup.find('div', class_='card-single card-bloomed')
        bloomed_card = bloomed_card_tag.find('img')['src'] if bloomed_card_tag else None

        # 추가로 가져올 테이블 데이터 (data-item-name="card-stats-music" 속성을 가진 section 내부의 테이블)
        music_section = soup.find('section', {'data-item-name': 'card-stats-music'})
        table_data = {}
        if music_section:
            table = music_section.find('table')
            if table:
                tbody = table.find('tbody')
                if tbody:
                    for tr in tbody.find_all('tr'):
                        th = tr.find('th')
                        tds = tr.find_all('td')
                        if th and tds:
                            key = th.text.strip()
                            values = [td.text.strip() for td in tds]
                            table_data[key] = values

            # 객체 생성
            ensemble_card = EnsembleCard(name=card_name,
                                         unbloomed_card=unbloomed_card, bloomed_card=bloomed_card, table_data=table_data)

            # JSON 형식으로 변환
            ensemble_card_json = json.dumps(ensemble_card.__dict__, ensure_ascii=False, indent=2)

            # JSON 파일에 쓰기
            with open(output_file_path, 'w', encoding='utf-8') as json_file:
                json_file.write(ensemble_card_json)

            print(f'Data saved to {output_file_path}')
        else:
            print('Error: Section not found with data-item-name="card-stats-music".')
    else:
        print(f'Error: {response.status_code}')

  

# 함수 호출 예시
scrape_and_save_data('https://ensemble-stars.fandom.com/wiki/(Crown_of_the_Blue_Sea)_Tori_Himemiya', 'Tori.json')

```

사실 자바스크립트만 하다가 파이썬을 하는건 처음이라, AI의 도움을 받아서 코드를 작성했다.

코드의 로직을 살펴보자면,
1. 매개변수로 받은 사이트에 접속한다.
2. 정상적으로 진입이 완료되면, 추출할 데이터를 찾고, 가져온다.
3. 가져온 데이터를 객체화 시키고 저장한다.

라는 로직이다.


## 삽질하다 시간을 허비한 과거의 나 🥲

사실 이 사이트에서 좀 시간을 많이 허비한 부분이 있는데, 결론적으로 뻘짓한거였다.

그 삽질이 무엇인가에 대해서 간단히 이야기 하려한다.

먼저 사이트의 이미지를 보자면,

<img width="553" alt="스크린샷 2023-12-29 오후 3 02 18" src="https://github.com/bomlang/bomlang.github.io/assets/61110237/deea0693-35a2-4859-8b3e-682c2daa3405">

<img width="553" alt="스크린샷 2023-12-29 오후 3 02 02" src="https://github.com/bomlang/bomlang.github.io/assets/61110237/da008048-4ec2-450a-ade0-cea25163c533">

위에 두개의 사진이 있는데, 첫번째는 Basic이라는 예전 버전의 게임데이터다.
현재 내가 프로젝트로 하려는 게임은 두가지 버전이 있는데, Basicr과 Music이다. 
내가 필요로 하는 데이터는 두번째 사진인 Music데이터이다.
심지어 이미지마저 다르기(색이 다르다)때문에 이 부분에 대해서 고민이 많이 필요했다.

처음에는 버튼을 눌렀을 때, 데이터가 바뀌는 방식인줄 알았다.
하지만 크롤링은 처음 웹사이트를 들어간 기준으로 정적인 데이터를 가져오기 때문에, 초기 화면인 Basic을 가져오는 줄 알았고, 나는 Music 버튼이 눌린 데이터가 필요했기에 고민 할 수밖에 없었다.

그래서 선택한 수단이 바로 selenium이다.

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from bs4 import BeautifulSoup
import json

class EnsembleCard:
    def __init__(self, name, unbloomed_card, bloomed_card, table_data):
        self.name = name
        self.unbloomed_card = unbloomed_card
        self.bloomed_card = bloomed_card
        self.table_data = table_data

    def __str__(self):
        return f"EnsembleCard(name={self.name}, " \
               f"unbloomed_card={self.unbloomed_card}, bloomed_card={self.bloomed_card}, table_data={self.table_data})"

# 크롬 드라이버 경로 설정 (본인의 환경에 맞게 설정)
chrome_driver_path = '/Users/holee/desktop/Ensamble/Ensemble_cardData/chromedriver-mac-arm64'

# 크롬 옵션 설정
chrome_options = Options()
chrome_options.add_argument('--headless')  # 만약 화면이 필요 없다면 headless 모드로 설정

# 크롤링할 페이지 URL
url = 'https://ensemble-stars.fandom.com/wiki/(Crown_of_the_Blue_Sea)_Tori_Himemiya'

# Selenium으로 버튼 클릭하여 데이터 로드
with webdriver.Chrome(options=chrome_options, executable_path=chrome_driver_path) as driver:
    driver.get(url)

    # 두 번째 탭으로 이동
    music_tab = driver.find_element_by_css_selector('li[data-item-name="card-music"]')
    music_tab.click()

    # 페이지 소스를 BeautifulSoup으로 파싱
    soup = BeautifulSoup(driver.page_source, 'html.parser')

    # 추가로 가져올 테이블 데이터 (두 번째 테이블 식별자 사용)
    second_table = soup.find('table', {'class': 'wikitable'})
    table_data = {}
    if second_table:
        tbody = second_table.find('tbody')
        if tbody:
            for tr in tbody.find_all('tr'):
                th = tr.find('th')
                tds = tr.find_all('td')
                if th and tds:
                    key = th.text.strip()
                    values = [td.text.strip() for td in tds]
                    table_data[key] = values

    # 추가로 가져올 이미지 데이터
    unbloomed_card_tag = soup.find('div', class_='card-single card-unbloomed')
    unbloomed_card = unbloomed_card_tag.find('img')['src'] if unbloomed_card_tag else None

    bloomed_card_tag = soup.find('div', class_='card-single card-bloomed')
    bloomed_card = bloomed_card_tag.find('img')['src'] if bloomed_card_tag else None

    # 객체 생성
    ensemble_card = EnsembleCard(name="Tori Himemiya",
                                 unbloomed_card=unbloomed_card, bloomed_card=bloomed_card, table_data=table_data)

    # JSON 형식으로 변환
    ensemble_card_json = json.dumps(ensemble_card.__dict__, ensure_ascii=False, indent=2)

    # JSON 파일에 쓰기
    output_file_path = 'torii.json'
    with open(output_file_path, 'w', encoding='utf-8') as json_file:
        json_file.write(ensemble_card_json)

    print(f'Data saved to {output_file_path}')
```

이 코드를 보면 selenium을 사용하고, 구글 드라이버를 다운받아 사용했다.

물론 코드는 정상적으로 잘 작동했다.
하지만 내가 원하는 Music데이터가 아닌 Basic데이터가 가져와졌다.

## 문제의 원인은 바로바로...

문제의 원인은 처음부터 Basic과 Music이 버튼에 의해 값이 바뀌는게 아니라, 처음부터 section단위로 페이지가 두개가 있었고, 버튼을 누를때 해당 섹션이 display에 의해서 숨겨졌다 보여졌다 하는 형식이었던 거같다.
내가 원하는 데이터는 `table`태그의 데이터였는데, html을 자세히 살펴보니 가장 아랫단에 있었다.

하하.. 😵‍💫

이제 데이터는 가져와졌으니 supabase에 넣어야 하는데, 테이블을 어떤 형식으로 해야할지 고민이다.
앞으로 이 부분에 대해서 기획을 좀 더 구상해봐야겠다.
