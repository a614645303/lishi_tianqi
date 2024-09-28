# lishi_tianqi
获取本年度之前本地区的历史天气
在中国天气网历史天气查询网站（https://lishi.tianqi.com）爬取某地区某时间段内的天气。（仅供学习）
注意*
1、仅用于查询本地区有历史记录以来至本年度之前的历史天气
2、本地区代码为网址中间的地区代码：
  例如北京：首页网址为：https://lishi.tianqi.com/beijing/index.html
          地区代码为：beijing

主文件为：lishi_tianqi


根据所提供的地区及日期查询爬区当月每日的天气情况并返回一个列表。
格式为：[日期, 当日最高温, 当日最低温, 天气, 风力]
def crawl_historical_weather(city, date):
    try:
        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'
                          ' AppleWebKit/537.36 (KHTML, like Gecko) '
                          'Chrome/91.0.4472.124 Safari/537.36'
        }
        url = f'https://lishi.tianqi.com/{city}/{date}.html'
        print(url)
        response = requests.get(url, headers=headers)
        print(response)
        if response.status_code == 200:
            soup = BeautifulSoup(response.text, 'html.parser')
            # print(soup)
            # 根据具体网页结构提取天气信息
            temperature = soup.find('ul', class_='thrui')
            # print(temperature)
            weather_data = []

            for li in temperature.find_all('li'):
                date_text = li.find('div', class_='th200').text
                high_temp = li.find_all('div', class_='th140')[0].text
                low_temp = li.find_all('div', class_='th140')[1].text
                weather_condition = li.find_all('div', class_='th140')[2].text
                weather_wind = li.find_all('div', class_='th140')[3].text
                weather_data.append([date_text, high_temp, low_temp, weather_condition, weather_wind])
            # print(weather_data) # 打印调试
            return weather_data
        else:  # 错误提示
            raise ValueError(f"Failed to fetch data for {date}. Status code: {response.status_code}")
    except Exception as e:
        print(f'error:{e}')
        return None


# 定义地区
place = 'mudanqu'
# 起止年份(不要获取本年度）
start_year = 2021
stop_year = 2023


# 创建一个列表容器，储存函数返回值
weather_database = []
# 日期生成器
for date_year in range(start_year, stop_year+1):
    for date_month in range(1, 13):
        time.sleep(1)  # 延迟一秒，减少对网站压力。
        date_month_str = f"{date_month:02d}"  # 使用 f-string 格式化月份，确保是两位数
        date_f = f"{date_year}{date_month_str}"  # 拼接
        print(date_f)
        # 调用函数 放入列表
        weather_database += crawl_historical_weather(place, date_f)
        # print(weather_database) #打印调试

# 写入excel
if weather_database:
    df = pd.DataFrame(weather_database,
                      columns=['Date', 'High Temperature', 'Low Temperature', 'Weather Condition', 'weather_wind'])
    df.to_excel('historical_weather.xlsx', index=False)
else:
    print('Failed to fetch historical weather data.')
