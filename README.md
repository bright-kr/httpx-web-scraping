# Python에서 HTTPX로 Web스크레이핑하기

[![Promo](https://github.com/bright-kr/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/) 

이 가이드는 강력한 Python HTTP 클라이언트인 HTTPX를 사용하여 Web스크레이핑하는 방법을 설명합니다:

- [HTTPX란 무엇인가요?](#what-is-httpx)
- [HTTPX로 스크레이핑하기: 단계별 가이드](#scraping-with-httpx-step-by-step-guide)
  - [단계 #1: 프로젝트 설정](#step-1-project-setup)
  - [단계 #2: 스크레이핑 라이브러리 설치](#step-2-install-the-scraping-libraries)
  - [단계 #3: 대상 페이지의 HTML 가져오기](#step-3-retrieve-the-html-of-the-target-page)
  - [단계 #4: HTML 파싱](#step-4-parse-the-html)
  - [단계 #5: 데이터 스크레이핑](#step-5-scrape-data-from-it)
  - [단계 #6: スクレイ핑한 데이터 내보내기](#step-6-export-the-scraped-data)
  - [단계 #7: 모두 합치기](#step-7-put-it-all-together)
- [HTTPX Web스크레이핑 고급 기능 및 기법](#httpx-web-scraping-advanced-features-and-techniques)
  - [커스텀 헤더 설정](#set-custom-headers)
  - [커스텀 User Agent 설정](#set-a-custom-user-agent)
  - [Cookie 설정](#set-cookies)
  - [프록시 통합](#proxy-integration)
  - [에러 처리](#error-handling)
  - [세션 처리](#session-handling)
  - [Async API](#async-api)
  - [실패한 요청 재시도](#retry-failed-requests)
- [Web스크레이핑에서 HTTPX vs Requests](#httpx-vs-requests-for-web-scraping)
- [결론](#conclusion)


## What Is HTTPX?

[HTTPX](https://github.com/projectdiscovery/httpx)는 [`httpcore`](https://pypi.org/project/httpcore/) 라이브러리 위에 구축된, 완전한 기능을 갖춘 Python 3용 HTTP 클라이언트입니다. 고부하 멀티스레딩 환경에서도 신뢰할 수 있는 성능을 제공하도록 설계되었습니다. HTTPX는 동기 및 비동기 API를 모두 제공하며, HTTP/1.1 및 HTTP/2 프로토콜을 지원합니다.

**Features**

- **간단하고 모듈식 코드베이스**: 기여와 확장이 쉽도록 설계되었습니다.
- **빠르고 구성 가능**: 여러 요소를 프로빙하기 위한 완전 맞춤형 플래그를 제공합니다.
- **다양한 프로빙 방식**: 다양한 HTTP 기반 프로빙 기법을 지원합니다.
- **자동 프로토콜 폴백**: 기본적으로 HTTPS에서 HTTP로 스마트 폴백합니다.
- **유연한 입력 옵션**: host, URL, CIDR을 입력으로 받을 수 있습니다.
- **고급 기능**: 프록시, 커스텀 HTTP 헤더, 구성 가능한 타임아웃, 기본 인증 등 다양한 기능을 지원합니다.

**Pros**

- **커맨드라인 사용 가능**: [`httpx[cli]`](https://pypi.org/project/httpx/)를 통해 접근할 수 있습니다.
- **풍부한 기능**: HTTP/2 및 비동기 API 지원을 포함합니다.
- **활발한 개발**: 정기적인 업데이트로 지속적으로 개선됩니다.

**Cons**

- **잦은 업데이트**: 새 릴리스에서 호환성이 깨지는 변경이 도입될 수 있습니다.
- **낮은 대중성**: [`requests`](https://requests.readthedocs.io/en/latest/) 라이브러리만큼 널리 사용되지는 않습니다.

## Scraping with HTTPX: Step-By-Step Guide

HTTPX는 HTTP 클라이언트이므로, 가져온 HTML에서 데이터를 파싱하고 추출하려면 [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) 같은 HTML 파서가 필요합니다.

> **Warning**:\
> HTTPX는 프로세스의 초기 단계에서만 사용되지만, 전체 워크플로를 안내해 드립니다. 더 고급 HTTPX 스크레이핑 기법에 관심이 있으시다면, Step 3 이후 다음 장으로 건너뛰어도 됩니다.

### Step #1: Project Setup

머신에 Python 3+를 설치하고 HTTPX 스크레이핑 프로젝트용 디렉터리를 생성합니다:

```bash
mkdir httpx-scraper
```

해당 디렉터리로 이동한 다음 [가상 환경](https://docs.python.org/3/library/venv.html)을 초기화합니다:

```bash
cd httpx-scraper
python -m venv env
```

Python IDE에서 프로젝트 폴더를 열고, 폴더 안에 `scraper.py` 파일을 만든 다음 가상 환경을 활성화합니다. Linux 또는 macOS에서는 다음을 실행합니다:

```bash
./env/bin/activate
```

Windows에서는 다음을 실행합니다:

```
env/Scripts/activate
```

### Step #2: Install the Scraping Libraries

HTTPX와 BeautifulSoup를 설치합니다:

```bash
pip install httpx beautifulsoup4
```

추가한 의존성을 `scraper.py` 스크립트로 import합니다:

```python
import httpx
from bs4 import BeautifulSoup
```

### Step #3: Retrieve the HTML of the Target Page

이 예제에서 대상 페이지는 “[Quotes to Scrape](https://quotes.toscrape.com/)” 사이트입니다:

![The Quotes To Scrape homepage](https://github.com/bright-kr/httpx-web-scraping/blob/main/Images/image-70.png)

HTTPX의 `get()` 메서드를 사용하여 홈 페이지의 HTML을 가져옵니다:

```python
# Make an HTTP GET request to the target page
response = httpx.get("http://quotes.toscrape.com")
```

내부적으로 HTTPX는 서버에 HTTP GET 요청를 수행하며, 서버는 페이지의 HTML로 응답합니다. `response.text` 속성을 사용하여 HTML 콘텐츠에 접근할 수 있습니다:

```python
html = response.text
print(html)
```

이는 원시 HTML 콘텐츠를 출력합니다:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Quotes to Scrape</title>
    <link rel="stylesheet" href="/static/bootstrap.min.css">
    <link rel="stylesheet" href="/static/main.css">
</head>
<body>
    <!-- omitted for brevity... -->
</body>
</html>
```

### Step #4: Parse the HTML

BeautifulSoup 생성자에 HTML 콘텐츠를 전달하여 파싱합니다:

```python
# Parse the HTML content using
 soup = BeautifulSoup(html, "html.parser")
```

이제 `soup` 변수에는 파싱된 HTML이 들어 있으며, 데이터 추출을 위한 메서드가 제공됩니다.

### Step #5: Scrape Data From It

페이지에서 인용구 데이터를 스크레이핑합니다:

```python
# Where to store the scraped data
quotes = []

# Extract all quotes from the page
quote_elements = soup.find_all("div", class_="quote")

# Loop through quotes and extract text, author, and tags
for quote_element in quote_elements:
    text = quote_element.find("span", class_="text").get_text().replace("“", "").replace("”", "")
    author = quote_element.find("small", class_="author")
    tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

    # Store the scraped data
    quotes.append({
        "text": text,
        "author": author,
        "tags": tags
    })
```

이 스니펫은 스크레이핑한 데이터를 저장하기 위해 `quotes`라는 리스트를 정의합니다. 그런 다음 모든 인용구 HTML 요소를 선택하고 루프를 돌면서 인용문 텍스트, 저자, 태그를 추출합니다. 추출된 각 인용구는 `quotes` 리스트 안에서 딕셔너리로 저장되어, 이후 사용 또는 내보내기를 위한 데이터 구조를 갖추게 됩니다.

### Step #6: Export the Scraped Data

스크레이핑한 데이터를 CSV 파일로 내보냅니다:

```python
# Specify the file name for export
with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])

    # Write the header row
    writer.writeheader()

    # Write the scraped quotes data
    writer.writerows(quotes)
```

이 스니펫은 `quotes.csv`라는 파일을 쓰기 모드로 열고, 컬럼 헤더(`text`, `author`, `tags`)를 정의한 뒤, 파일에 헤더를 기록하고 `quotes` 리스트의 각 딕셔너리를 CSV 파일에 작성합니다. `csv.DictWriter`가 포맷팅을 처리하므로 구조화된 데이터를 쉽게 저장할 수 있습니다.

Python 표준 라이브러리에서 `csv`를 import합니다:

```python
import csv
```

### Step #7: Put It All Together

최종 HTTPX Web스크레이핑 스크립트에는 다음 코드가 포함됩니다:

```python
import httpx
from bs4 import BeautifulSoup
import csv

# Make an HTTP GET request to the target page
response = httpx.get("http://quotes.toscrape.com")

# Access the HTML of the target page
html = response.text

# Parse the HTML content using BeautifulSoup
soup = BeautifulSoup(html, "html.parser")

# Where to store the scraped data
quotes = []

# Extract all quotes from the page
quote_elements = soup.find_all("div", class_="quote")

# Loop through quotes and extract text, author, and tags
for quote_element in quote_elements:
    text = quote_element.find("span", class_="text").get_text().replace("“", "").replace("”", "")
    author = quote_element.find("small", class_="author").get_text()
    tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

    # Store the scraped data
    quotes.append({
        "text": text,
        "author": author,
        "tags": tags
    })

# Specify the file name for export
with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])

    # Write the header row
    writer.writeheader()

    # Write the scraped quotes data
    writer.writerows(quotes)
```

다음으로 실행합니다:

```bash
python scraper.py
```

또는 Linux/macOS에서는 다음을 실행합니다:

```bash
python3 scraper.py
```

프로젝트 루트 폴더에 다음 내용이 포함된 `quotes.csv` 파일이 생성됩니다:

![The CSV containing the scraped data](https://github.com/bright-kr/httpx-web-scraping/blob/main/Images/image-71.png)

## HTTPX Web Scraping Advanced Features and Techniques

더 복잡한 예제를 사용해 보겠습니다. 대상 사이트는 [HTTPBin.io `/anything` endpoint](https://httpbin.io/anything)입니다. 이는 호출자가 전송한 IP 주소, 헤더 및 기타 정보를 반환하는 특수 API입니다.

### Set Custom Headers

HTTPX는 `headers` 인수를 사용하여 [커스텀 헤더를 지정](https://www.python-httpx.org/quickstart/#custom-headers)할 수 있습니다:

```python
import httpx

# Custom headers for the request
headers = {
    "accept": "application/json",
    "accept-language": "en-US,en;q=0.9,fr-FR;q=0.8,fr;q=0.7,es-US;q=0.6,es;q=0.5,it-IT;q=0.4,it;q=0.3"
}

# Make a GET request with custom headers
response = httpx.get("https://httpbin.io/anything", headers=headers)
# Handle the response...
```

### Set a Custom User Agent

[`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent)는 [Web스크레이핑을 위한 가장 중요한 HTTP 헤더](/blog/web-data/http-headers-for-web-scraping) 중 하나입니다. 기본적으로 HTTPX는 다음 `User-Agent`를 사용합니다:

```
python-httpx/<VERSION>
```

이 값은 요청가 자동화되었음을 쉽게 드러낼 수 있으며, 그 결과 대상 사이트가 이를 차단할 가능성이 있습니다.

이를 피하기 위해, 다음과 같이 실제 브라우저를 모방하도록 커스텀 `User-Agent`를 설정할 수 있습니다:

```python
import httpx

# Define a custom User-Agent
headers = {
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Safari/537.36"
}

# Make a GET request with the custom User-Agent
response = httpx.get("https://httpbin.io/anything", headers=headers)
# Handle the response...
```

### Set Cookies

[`cookies` 인수](https://www.python-httpx.org/quickstart/#cookies)를 사용하여 HTTPX에서 Cookie를 설정합니다:

```python
import httpx

# Define cookies as a dictionary
cookies = {
    "session_id": "3126hdsab161hdabg47adgb",
    "user_preferences": "dark_mode=true"
}

# Make a GET request with custom cookies
response = httpx.get("https://httpbin.io/anything", cookies=cookies)
# Handle the response...
```

이를 통해 Web스크레이핑 요청에 필요한 세션 데이터를 포함할 수 있습니다.

### Proxy Integration

Web스크레이핑 수행 중 신원을 보호하고 IP 주소 밴을 피하기 위해 [HTTPX 요청를 프록시로 라우팅](https://www.python-httpx.org/advanced/proxies/)합니다. 이를 위해 `proxies` 인수를 사용합니다:

```python
import httpx

# Replace with the URL of your proxy server
proxy = "<YOUR_PROXY_URL>"

# Make a GET request through a proxy server
response = httpx.get("https://httpbin.io/anything", proxy=proxy)
# Handle the response...
```

### Error Handling

기본적으로 HTTPX는 [연결 또는 네트워크 이슈](https://www.python-httpx.org/exceptions/)에 대해서만 에러를 발생시킵니다. [`4xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#client_error_responses) 및 [`5xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#server_error_responses) 상태 코드의 HTTP 응답에 대해서도 예외를 발생시키려면, 아래와 같이 `raise_for_status()` 메서드를 사용합니다:

```python
import httpx

try:
    response = httpx.get("https://httpbin.io/anything")
    # Raise an exception for 4xx and 5xx responses
    response.raise_for_status()
    # Handle the response...
except httpx.HTTPStatusError as e:
    # Handle HTTP status errors
    print(f"HTTP error occurred: {e}")
except httpx.RequestError as e:
    # Handle connection or network errors
    print(f"Request error occurred: {e}")
```

### Session Handling


HTTPX에서 top-level API를 사용하면 각 リクエ스트마다 새로운 연결이 생성되며, 즉 TCP 연결이 재사용되지 않습니다. 이 접근 방식은 호스트에 대한 リクエ스트 수가 증가할수록 비효율적이 됩니다.

반면 `httpx.Client` 인스턴스를 사용하면 [HTTP connection pooling](https://devblogs.microsoft.com/premier-developer/the-art-of-http-connection-pooling-how-to-optimize-your-connections-for-peak-performance/)을 활성화할 수 있습니다. 이는 동일한 호스트로 여러 リクエ스트를 보낼 때, 매번 새로운 TCP 연결을 생성하는 대신 기존 TCP 연결을 재사용할 수 있음을 의미합니다.

top-level API 대신 `Client`를 사용할 때의 이점은 다음과 같습니다:

- 반복적인 핸드셰이크가 없으므로 リクエ스트 전반의 레이턴시 감소
- CPU 사용량 감소 및 왕복(round-trip) 횟수 감소
- 네트워크 트래픽 감소

또한 `Client` 인스턴스는 top-level API에는 없는 기능을 포함하여 세션 처리를 지원합니다:

- リクエ스트 간 Cookie 지속성
- 모든 발신 リクエ스트에 설정 적용
- HTTP プロキ시를 통한 リクエ스트 전송

일반적으로 HTTPX에서는 컨텍스트 매니저(`with` 문)와 함께 `Client`를 사용하는 것이 권장됩니다:

```python
import httpx

with httpx.Client() as client:
    # Make an HTTP request using the client
    response = client.get("https://httpbin.io/anything")

    # Extract the JSON response data and print it
    response_data = response.json()
    print(response_data)
```

또는 클라이언트를 수동으로 관리하고 `client.close()`로 connection pool을 명시적으로 닫을 수 있습니다:

```python
import httpx

client = httpx.Client()
try:
    # Make an HTTP request using the client
    response = client.get("https://httpbin.io/anything")

    # Extract the JSON response data and print it
    response_data = response.json()
    print(response_data)
except:
  # Handle the error...
  pass
finally:
  # Close the client connections and release resources
  client.close()
```

> **Note**:\
> `requests` 라이브러리에 익숙하시다면, `httpx.Client()`는 [`requests.Session()`](https://requests.readthedocs.io/en/latest/user/advanced/#session-objects)과 유사한 목적을 수행합니다.

### Async API

기본적으로 HTTPX는 표준 동기 API를 노출합니다. 동시에, 필요할 때 사용할 수 있는 [비동기 클라이언트](https://www.python-httpx.org/async/)도 제공합니다. [`asyncio`](https://docs.python.org/3/library/asyncio.html)로 작업하는 경우, 효율적으로 발신 HTTP リクエ스트를 전송하기 위해 async 클라이언트 사용이 필수적입니다.

HTTPX에서 비동기 リクエ스트를 수행하려면 `AsyncClient`를 초기화한 뒤, 아래와 같이 이를 사용하여 GET リクエ스트를 보냅니다:

```python
import httpx
import asyncio

async def fetch(url):
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        return response.text

async def main():
    urls = ["https://httpbin.io/anything"] * 5
    responses = await asyncio.gather(*(fetch(url) for url in urls))
    for response in responses:
        print(response)

asyncio.run(main())
```

[`with`](https://docs.python.org/3/reference/compound_stmts.html#with) 문은 블록이 끝날 때 클라이언트가 자동으로 닫히도록 보장합니다. 또는 클라이언트를 수동으로 관리하는 경우 `await client.close()`로 명시적으로 닫을 수 있습니다.

`AsyncClient`를 사용할 때는 모든 HTTPX リクエ스트 메서드(`get()`, `post()` 등)가 비동기입니다. 따라서 응답를 얻기 위해 호출 전에 반드시 `await`를 추가해야 합니다.

### Retry Failed Requests

Web스크레이핑 중 네트워크 불안정으로 인해 연결 실패 또는 타임아웃이 발생할 수 있습니다. HTTPX는 [`HTTPTransport`](https://www.python-httpx.org/advanced/transports/) 인터페이스를 통해 이러한 이슈 처리를 단순화합니다. 이 메커니즘은 `httpx.ConnectError` 또는 `httpx.ConnectTimeout`이 발생할 때 リクエ스트를 재시도합니다.

다음 코드는 최대 3회까지 リクエ스트를 재시도하도록 transport를 구성하는 방법을 보여줍니다:

```python
import httpx

# Configure transport with retry capability on connection errors or timeouts
transport = httpx.HTTPTransport(retries=3)

# Use the transport with an HTTPX client
with httpx.Client(transport=transport) as client:
    # Make a GET request
    response = client.get("https://httpbin.io/anything")
    # Handle the response...
```

연결 관련 에러만 リト라이를 트리거합니다. read/write 에러 또는 특정 HTTP 상태 코드를 처리하려면 [`tenacity`](https://tenacity.readthedocs.io/en/latest/) 같은 라이브러리로 커스텀 재시도 로직을 구현해야 합니다.

## HTTPX vs Requests for Web Scraping

다음 표는 HTTPX와 [Web스크레이핑을 위한 Requests](/blog/web-data/python-requests-guide)를 비교합니다:

| **Feature** | **HTTPX** | **Requests** |
| --- | --- | --- |
| **GitHub stars** | 8k  | 52.4k |
| **Async support** | ✔️  | ❌   |
| **Connection pooling** | ✔️  | ✔️  |
| **HTTP/2 support** | ✔️  | ❌   |
| **User-agent customization** | ✔️  | ✔️  |
| **Proxy support** | ✔️  | ✔️  |
| **Cookie handling** | ✔️  | ✔️  |
| **Timeouts** | 연결 및 읽기에 대해 커스터마이즈 가능 | 연결 및 읽기에 대해 커스터마이즈 가능 |
| **Retry mechanism** | transports를 통해 사용 가능 | `HTTPAdapter`s를 통해 사용 가능 |
| **Performance** | 높음 | 중간 |
| **Community support and popularity** | 성장 중 | 큼 |

## Conclusion

자동화된 HTTP リクエ스트는 공개 IP 주소를 노출하여 신원과 위치가 드러날 수 있으며, 이는 프라이버시를 침해합니다. 보안과 프라이버시를 강화하려면 프록시 서버를 사용하여 IP 주소를 숨기십시오.

Bright Data는 전 세계 최고의 프록시 서버를 운영하며, Fortune 500 기업과 20,000명 이상의 고객에게 서비스를 제공하고 있습니다. 제공 범위에는 다양한 プロキ시 유형이 포함됩니다:

- [Datacenter proxies](https://brightdata.co.kr/proxy-types/datacenter-proxies) – 770,000개 이상의 データセンター프록시 IP.
- [Residential proxies](https://brightdata.co.kr/proxy-types/residential-proxies) – 195개 이상의 국가에서 7,200만 개 이상의 レジデンシャル프록시 IP.
- [ISP proxies](https://brightdata.co.kr/proxy-types/isp-proxies) – 700,000개 이상의 ISP프록시 IP.
- [Mobile proxies](https://brightdata.co.kr/proxy-types/mobile-proxies) – 700만 개 이상의 モバイル프록시 IP.

지금 무료 Bright Data 계정을 생성하여 당사의 스크레이핑 솔루션과 프록시를 테스트해 보십시오!