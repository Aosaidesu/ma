# from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
import time
import requests
import os
設定
targeturl = "https://www.dlsite.com/maniax/work/=/productid/RJ01680712.html"
ここに取得したい画像の保存先フォルダを指定
savedir = "dliteimages"
os.makedirs(savedir, existok=True)

def setup_driver():
chrome_options = Options()
chromeoptions.addargument("--headless") # 画面表示なし
chromeoptions.addargument("--no-sandbox")
chromeoptions.addargument("--disable-dev-shm-usage")
chromeoptions.addargument("--disable-blink-features=AutomationControlled") # 自動化検知回避
chromeoptions.addexperimental_option("excludeSwitches", ["enable-automation"])
chromeoptions.addexperimental_option("useAutomationExtension", False)

driver = webdriver.Chrome(options=chrome_options)
driver.execute_script("Object.defineProperty(navigator, 'webdriver', {get: () => undefined})")
return driver

def getunlockedimages(driver, url):
print(f"[*] ブラウザを起動して {url} にアクセス中...")
driver.get(url)

# JavaScript が完全に実行されるまで待機（画像が復号されるまで）
# DLsite はロード後に画像を動的に生成するため、十分待つ必要がある
print("[*] JavaScript 実行待機中 (画像復号化等待)...")
time.sleep(5)

# 画面をスクロールして遅延読み込み画像をトリガーする
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
time.sleep(3)

# 復号された後の DOM から画像 URL を抽出
# DLsite の構造により、class が 'img' で src が 'https://dlssl.com...' のものを拾う
# または 'data-src' が復号されている場合を想定
images = driver.findelements(By.CSSSELECTOR, "img[src='dlssl.com'], img[data-src='dlssl.com']")

found_urls = []
for img in images:
src = img.get_attribute('src')
datasrc = img.getattribute('data-src')
# サンプル画像（低解像度）を除外するロジック（必要に応じて調整）
# 本番では URL に 'thumb' や 'sample' が入っていないか確認する
potentialurl = src or datasrc

if potentialurl and 'dlssl.com' in potentialurl:
if 'thumb' not in potentialurl.lower() and 'sample' not in potentialurl.lower():
foundurls.append(potentialurl)

return found_urls

def downloadimages(urls, drivercookies):
print(f"[*] {len(urls)} 個の画像をダウンロード中...")

# ブラウザから取得した Cookie を付与してリクエストを送る
session = requests.Session()
for cookie in driver_cookies:
session.cookies.set(cookie.get('name'), cookie.get('value'), domain=cookie.get('domain'))

headers = {
"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
"Referer": target_url
}
session.headers.update(headers)

count = 0
for url in urls:
try:
print(f" 処理中: {url.split('/')[-1]}")
response = session.get(url, timeout=15)
if response.status_code == 200:
filename = f"{savedir}/img{count}.jpg"
with open(filename, 'wb') as f:
f.write(response.content)
print(f" -> 保存完了: {filename}")
count += 1
else:
print(f" -> 失敗 (Status: {response.status_code})")
except Exception as e:
print(f" -> エラー: {str(e)}")

return count

def main():
driver = setup_driver()
try:
# 1. ブラウザでページを開き、JS 復号を待つ
imageurls = getunlockedimages(driver, targeturl)

if not image_urls:
print("[!] 復号された画像 URL が検出されませんでした。")
print(" (注: 完全な購入者限定画像は、ログイン済み Cookie を付与しないと表示されない場合があります)")
else:
print(f"[+] 合計 {len(image_urls)} 個の画像 URL を取得しました。")

# 2. Cookie を取得してダウンロード
cookies = driver.get_cookies()
downloadedcount = downloadimages(image_urls, cookies)

if downloaded_count == 0:
print("[!] ダウンロードされた画像はありませんでした。")
else:
print(f"[+] 処理終了。{downloaded_count} 個の画像を保存しました。")

finally:
driver.quit()

if _name_ == "_main_":
main()
