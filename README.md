# web-scrapping-fedor

import requests
from bs4 import BeautifulSoup
import json
from time import sleep

keywords = ['django', 'flask']
results = []

for count in range(0, 41):
    url = f'https://spb.hh.ru/search/vacancy?text=python&area=1&area=2&page={count}'
    headers = {
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36',
    }
    response = requests.get(url, headers=headers)

    if response.status_code == 200:
        html = response.text
        soup = BeautifulSoup(html, 'html.parser')
        vacancies = soup.find_all("div", class_="vacancy-serp-item")

        for vacancy in vacancies:
            link = vacancy.find('a', class_='bloko-link').get('href')
            company = vacancy.find("a", class_="bloko-link bloko-link_secondary").text
            city = vacancy.find("span", class_="vacancy-serp-item__meta-info").text
            salary = vacancy.find("div", class_="vacancy-serp-item__compensation")
            if salary:
                salary = salary.text.strip()
            else:
                salary = 'не указана'

            response_vacancy = requests.get(link, headers=headers)
            sleep(2)

            soup_vacancy = BeautifulSoup(response_vacancy.text, 'html.parser')
            description = soup_vacancy.find("div", class_="vacancy-description")
            if description:
                description = description.text
            else:
                description = 'описание вакансии не найдено'

            if all(keyword.lower() in description.lower() for keyword in keywords):
                vacancy_info = {
                    'link': link,
                    'company': company,
                    'city': city,
                    'salary': salary,
                    'description': description
                }
                results.append(vacancy_info)
                
with open('vacancies.json', 'w', encoding='utf-8') as file:
    json.dump(results, file, ensure_ascii=False, indent=4)

print("parsing zaconchen")




##  так же был и такой вариант кода




import requests
from bs4 import BeautifulSoup
import json
from time import sleep

url = 'https://spb.hh.ru/search/vacancy?text=python&area=1&area=2'
keywords = ['Django', 'Flask']
headers = {
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/58.0.3029.110 ',
}
response = requests.get(url, headers = headers)

if response.status_code == 200:
    html = response.text
    soup = BeautifulSoup(html, 'html.parser')
    vacancies = soup.find_all("div", class_="vacancy-card--z_UXteNo7bRGzxWVcL7y font-inter")
    results = []
    for vacancy in vacancies:
        link = vacancy.find('a', class_='bloko-link').get('href')
        company = vacancy.find("span", class_="company-info-text--vgvZouLtf8jwBmaD1xgp").text
        city = vacancy.find("span", class_="fake-magritte-primary-text--Hdw8FvkOzzOcoR4xXWni").text
        salary = vacancy.find("span", class_="fake-magritte-primary-text--Hdw8FvkOzzOcoR4xXWni compensation-text--kTJ0_rp54B2vNeZ3CTt2 separate-line-on-xs--mtby5gO4J0ixtqzW38wh")
        if salary:
            salary = salary.text.strip()
        else:
            salary = 'Не указана'

        def get_url():
            for count in range(1, 41):
                url = f"https://spb.hh.ru/search/vacancy?text=python&area=1&area={count}"
                response = requests.get(url, headers = headers)
                soup = BeautifulSoup(response.text, 'html.parser')
                data = soup.find_all('div', class_="vacancy-card--z_UXteNo7bRGzxWVcL7y font-inter")

                for i in data:
                    cart_url = i.find('a', class_='bloko-link').get('href')
                    yield cart_url

        for cart_url in get_url():
            response = requests.get(cart_url, headers = headers)
            sleep(2)

            soup = BeautifulSoup(response.text, 'html.parser')
            vacanc = soup.find("div", class_="row-content")
            description = vacanc.find("div", class_="g-user-content").text 

        if all(keyword.lower() in description.lower() for keyword in keywords):
            vacancy_info = {
                'link': link,
                'company': company,
                'city': city,
                'salary': salary
            }
            print(vacancy_info)

            with open('vacancies.json', 'w', encoding='utf-8') as file:
                json.dump(results, file, ensure_ascii=False, indent=4)

            print('Парсинг завершен. Результаты сохранены в файле vacancies.json.')
        else:
            print('Ошибка при выполнении запроса.')
