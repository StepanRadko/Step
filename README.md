import requests
from bs4 import BeautifulSoup
import csv

CSV = 'cards.csv'
HOST = 'http://kolpachok.com.ua/'
URL = 'http://kolpachok.com.ua/prochee'
HEADERS = {
'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36'
}


def get_html(url, params=''):
    r = requests.get(url, headers=HEADERS, params=params)
    return r


def get_content(html):
    soup = BeautifulSoup(html, 'html.parser')
    items = soup.find_all('div', class_='productdisplay')
    cards = []

    for item in items:
        cards.append(
            {
                'title': item.find('h2', class_='prodtitles').get_text(strip=True),
                'title2': item.find('span', class_='pricedisplay').get_text(strip=True),
                'card_img': HOST + item.find('div', class_='imagecol').find('img').get('src')

            }
        )

    return cards

def save_doc(items, path):
    with open(path,'w', newline='') as file:
        writer = csv.writer(file, delimiter=';')
        writer.writerow(['Название продукта','Цена','Фото'])
        for item in items:
            writer.writerow([item['title'], item['title2'],item['card_img']])


def parser():
    PAGENATION = input('УКАЖИТЕ КОЛЛИЧЕСТВО СТРАНИЦ ДЛЯ ПАРСИНГА: ')
    PAGENATION = int(PAGENATION.strip())
    html = get_html(URL)
    if html.status_code == 200:
        cards = []
        for page in range(1, PAGENATION):
            print(f'парсинг страницы № {page}')
            html = get_html(URL, params={'page': page})
            cards.extend(get_content(html.text))
            save_doc(cards, CSV)
        pass
    else:
        print('Error')


parser()
