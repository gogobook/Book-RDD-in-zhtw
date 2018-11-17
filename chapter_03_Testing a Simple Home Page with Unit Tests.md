# 3.使用單元測試測試簡單主頁
我們完成了最後一章功能測試失敗，告訴我們它希望我們網站的主頁在其標題中有“待辦事項”。是時候開始處理我們的應用程序了。

警告：事情即將成真
前兩章故意很好，很輕鬆。從現在開始，我們進入了一些更加豐富的編碼。這是一個預測：在某些時候，事情會出錯。你會看到我應該看到的不同結果。這是一件好事，因為它將是一個真正的角色建設學習體驗™。

一種可能性是我給出了一些含糊不清的解釋，你做了一些與我的意圖不同的事情。退一步思考一下我們在本書的這一點上想要實現的目標。我們正在編輯哪個文件，我們希望用戶能夠做什麼，我們測試什麼以及為什麼？可能是您編輯了錯誤的文件或功能，或者運行了錯誤的測試。我認為你會從這些“停止和思考”的時刻中了解更多有關TDD的信息，而不是按照說明和復制粘貼順利進行的所有時間。

或者它可能是一個真正的錯誤。要頑強，仔細閱讀錯誤信息（請參閱本章後面的閱讀回溯），然後你就會深究它。它可能只是一個缺少的逗號，或尾隨斜線，或者可能是Selenium查找方法中的缺失s。但是，正如Zed Shaw所說的那樣，這種調試也是學習的絕對重要部分，所以一定要堅持下去！

如果你真的陷入困境，你可以隨時給我發電子郵件（或試試 谷歌集團）。快樂的調試！

## 3.1. Our First Django App, and Our First Unit Test
Django鼓勵您將代碼構建到應用程序中：理論上，一個項目可以擁有許多應用程序，您可以使用由其他人開發的第三方應用程序，甚至可以在不同的項目中重用您自己的應用程序之一......我承認我自己從未實際管理過它！不過，應用程序是保持代碼組織的好方法。

讓我們為我們的待辦事項列表啟動一個應用程序：

`$ python manage.py startapp lists`
這將創建一個名為lists的文件夾，在manage.py和現有的 superlists文件夾旁邊，並在其中包含許多佔位符文件，用於模型，視圖以及我們感興趣的測試：
```sh
.
├── db.sqlite3
├── functional_tests.py
├── lists
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
├── superlists
│   ├── __init__.py
│   ├── __pycache__
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── virtualenv
    ├── [...]
```
3.2。單元測試，以及它們與功能測試的不同之處
正如我們對物品的許多標籤所做的那樣，單元測試和功能測試之間的界限有時會變得有點模糊。但是，基本的區別在於，功能測試從外部測試應用程序，是從用戶的角度來看的。，單元測試從內部測試應用程序，是從程序員的角度來看。

我正在遵循的TDD方法希望我們的應用程序能夠被兩種類型的測試所覆蓋。我們的工作流程看起來有點像這樣：

1. 我們首先編寫一個功能測試，從用戶的角度描述新功能。

2. 一旦我們進行了失敗的功能測試，我們就會開始思考如何編寫可以讓它通過的代碼（或者至少要通過它當前的失敗）。我們現在使用一個或多個單元測試來定義我們希望代碼行為的方式 - 我們編寫的每個生產代碼行都應該通過（至少）我們的單元測試來測試。

3. 一旦我們進行了失敗的單元測試，我們就會編寫盡可能少的 應用程序代碼，足以讓單元測試通過。我們可能會在第2步和第3步之間迭代幾次，直到我們認為功能測試會更進一步。

4. 現在我們可以重新運行我們的功能測試，看看它們是否通過，或者進一步了解。這可能會促使我們編寫一些新的單元測試，以及一些新的代碼，等等。

您可以看到，在整個過程中，功能測試從高層驅動開發，而單元測試從底層驅動開發。

這看起來有點多餘嗎？有時它會有這種感覺，但功能測試和單元測試確實有著截然不同的目標，而且它們通常看起來會有很大不同。

> 功能測試應該幫助您構建具有正確功能的應用程序，並確保您不會意外地破壞它。單元測試應該幫助您編寫乾淨且沒有bug的代碼。
現在有足夠的理論讓我們看看它在實踐中的表現。

3.3。Django中的單元測試
讓我們看看如何為我們的主頁視圖編寫單元測試。在*lists/tests.py*中打開新文件，你會看到如下內容：

lists/tests.py
```py
from django.test import TestCase

# Create your tests here.
```
Django幫我們建議使用`TestCase`它提供的特殊版本。它是該標準的增強版本`unittest.TestCase`，還有一些額外的Django特性，我們將在接下來的幾章中發現它們。

Django建議我們使用一個特殊版本的`TestCase`，這是它提供的。它是標準單元測試的擴充版。`unittest.TestCase`，還有一些django特有的特性，我們將在接下來的幾章中發現。

您已經看到，TDD週期包括從失敗的測試開始，然後編寫代碼使其通過。好吧，在我們能走到這一步之前，我們想知道我們正在編寫的單元測試肯定會由我們的自動化測試運行器運行，不管它是什麼。在functional_tests.py，我們直接運行它，但是Django做的這個文件更像魔術。所以，為了確保，讓我們做一個故意愚蠢的失敗測試:

lists/tests.py
```py
from django.test import TestCase

class SmokeTest(TestCase):

    def test_bad_maths(self):
        self.assertEqual(1 + 1, 3)
```
現在讓我們調用這個神秘的Django測試運行器。像往常一樣，它是一個 manage.py 命令：
```sh
$ python manage.py test
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_bad_maths (lists.tests.SmokeTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/lists/tests.py", line 6, in test_bad_maths
    self.assertEqual(1 + 1, 3)
AssertionError: 2 != 3

 ---------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```
優秀。機器似乎正在起作用。這是提交的一個好點：

$ git status   ＃應顯示列表/未跟踪
$ git add lists 
$ git diff --staged   ＃將顯示你即將提交的差異
$ git commit -m“為列表添加app，故意單元測試失敗”
毫無疑問，該`-m`標誌允許您在命令行傳遞提交消息，因此您不需要使用編輯器。您可以選擇使用Git命令行的方式; 我只會告訴你我見過的主要用品。關鍵規則是：確保在執行之前始終查看您要提交的內容。

## 3.4. Django’s MVC, URLs, and View Functions
Django是按照典型的模型-視圖-控制器(MVC)模式構建的。廣泛。它確實有模型，但它的視圖更像一個控制器，而模板實際上是View部分，但大意是這樣的。如果您感興趣，您可以在Django FAQs中查找討論的細節。
無論如何，與任何Web服務器一樣，Django的主要工作是決定當用戶在我們的網站上詢問特定URL時該怎麼做。Django的工作流程如下：

1. HTTP 請求是針對特定URL的。

2. Django使用一些規則來決定哪個視圖函數應該處理請求（這被稱為解析 URL）。

3. view函數處理請求並返回HTTP 響應。

所以我們要測試兩件事：

- 我們可以將網站根目錄（“/”）的URL解析為我們製作的特定視圖函數嗎？
- 我們可以讓這個視圖函數返回一些HTML，它將通過功能測試嗎？

讓我們從第一個開始吧。打開lists/tests.py，將我們的愚蠢測試改為：
lists/tests.py
```py
from django.urls import resolve
from django.test import TestCase
from lists.views import home_page  

class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')  
        self.assertEqual(found.func, home_page)  
```
這裡發生了什麼？

1. `resolve`是Django在內部使用的函數來解析URL並找到它們應映射到的視圖函數。我們正在檢查 `resolve`，當用“/”調用時，站點的根目錄找到一個名為的函數home_page。
2. 那是什麼函數?這是我們接下來要寫的視圖函數，它會返回我們想要的HTML。從導入中可以看到，我們計畫將其存儲在list /view .py中

$ python manage.py test
ImportError: cannot import name 'home_page'

這是一個非常可預測且無趣的錯誤：我們嘗試導入一些我們尚未編寫的東西。但對於TDD而言，這仍然是個好消息，預計這個例外將被視為預期的失敗。由於我們有一個失敗的功能測試和一個失敗的單元測試，所以我們有測試山羊的全部祝福去編碼。

3.5。最後！我們實際上寫了一些應用程序代碼！
令人興奮，不是嗎？需要注意的是，TDD意味著長期的預期只是逐漸消除，並且只是微小的增量。特別是因為我們正在學習並且剛剛開始，我們只允許自己一次更改（或添加）一行代碼 - 每次，我們只做出解決當前測試失敗所需的最小變化。

我在這裡刻意偏執，但我們目前的測試失敗是什麼？我們不能導入home_page來自lists.views？好吧，讓我們解決這個問題 - 只有那個。在lists/views.py中：

lists/views.py
from django.shortcuts import render

# Create your views here.
home_page = None
“你一定是在開玩笑！” 我能聽到你說。

我可以聽到你，因為當我的同事們首次向我演示TDD時，我曾經說過（有感覺）。好吧，請耐心等待，我們將討論這一點是否會在一段時間內走得太遠。但是現在，讓自己跟進，即使有些惱怒，看看我們的測試是否可以幫助我們編寫正確的代碼，一次只做一小步。

我們再次運行測試：
```py
$ python manage.py test
Creating test database for alias 'default'...
E
======================================================================
ERROR: test_root_url_resolves_to_home_page_view (lists.tests.HomePageTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/lists/tests.py", line 8, in
test_root_url_resolves_to_home_page_view
    found = resolve('/')
  File ".../django/urls/base.py", line 27, in resolve
    return get_resolver(urlconf).resolve(path)
  File ".../django/urls/resolvers.py", line 394, in resolve
    raise Resolver404({'tried': tried, 'path': new_path})
django.urls.exceptions.Resolver404: {'tried': [[<RegexURLResolver
<RegexURLPattern list> (admin:admin) ^admin/>]], 'path': ''}

 ---------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (errors=1)
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```
> 閱讀回溯
讓我們花一點時間討論如何閱讀回溯，因為這是我們在TDD中必須做的很多事情。您很快就會學會瀏覽它們並獲取相關線索：
```py
======================================================================
ERROR: test_root_url_resolves_to_home_page_view (lists.tests.HomePageTest)  
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "...python-tdd-book/lists/tests.py", line 8, in
test_root_url_resolves_to_home_page_view
    found = resolve('/')  
  File ".../django/urls/base.py", line 27, in resolve
    return get_resolver(urlconf).resolve(path)
  File ".../django/urls/resolvers.py", line 394, in resolve
    raise Resolver404({'tried': tried, 'path': new_path})
django.urls.exceptions.Resolver404: {'tried': [[<RegexURLResolver  
<RegexURLPattern list> (admin:admin) ^admin/>]], 'path': ''}  
 ---------------------------------------------------------------------
[...]
```
1. 你看的第一個地方通常是錯誤本身。有時這就是你需要看到的全部內容，它會讓你立即發現問題。但有時候，就像在這種情況下，它不是很明顯。
2. 接下來要仔細檢查的是：哪個測試失敗了？它絕對是我們所期望的 - 也就是我們剛剛寫的那個？在這種情況下，答案是肯定的。
3. 然後我們在測試代碼中尋找啟動失敗的位置。我們從回溯的頂部開始，尋找測試文件的文件名，檢查哪個測試函數以及失敗來自哪個代碼行。在這種情況下，它是我們稱為resolve “/” URL 的函數的行。
通常會有第四步，我們會進一步了解與問題有關的任何自己的應用程序代碼。在這種情況下，它是所有Django代碼，但我們將在本書的後面部分看到第四步的大量示例。

將它們全部拉到一起，我們將回溯解釋為告訴我們，當嘗試解析“/”時，Django引發了404錯誤 - 換句話說，Django無法找到“/”的URL映射。讓我們幫忙吧。

## 3.6. urls.py
我們的測試告訴我們，我們需要一個URL映射。Django使用名為urls.py的文件 將URL映射到視圖函數。有一個主urls.py在整個網站`superlists/superlists`文件夾。我們來看看：

superlists/urls.py
```py
"""superlists URL Configuration

The `urlpatterns` list routes URLs to views. For more information please see:
    https://docs.djangoproject.com/en/1.11/topics/http/urls/
Examples:
Function views
    1. Add an import:  from my_app import views
    2. Add a URL to urlpatterns:  url(r'^$', views.home, name='home')
Class-based views
    1. Add an import:  from other_app.views import Home
    2. Add a URL to urlpatterns:  url(r'^$', Home.as_view(), name='home')
Including another URLconf
    1. Import the include() function: from django.conf.urls import url, include
    2. Add a URL to urlpatterns:  url(r'^blog/', include('blog.urls'))
"""
from django.conf.urls import url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
]
```
像往常一樣，Django有很多有用的評論和默認建議。

如果你的urls.py看起來不同或者它提到了一個名為`path`而不是的函數`url`，那是因為你得到了錯誤的Django版本。本書是為Django v1.11編寫的。再看一下“ 先決條件和假設 ”部分，並在繼續之前獲得正確的版本。
一個`url`條目以一個正則表達式開頭，該表達式定義了它應用的URL，並繼續說明它應該將這些請求發送到哪裡 - 要么是你導入的視圖函數，要么是其他地方的另一個urls.py文件。

第一個示例條目有正則表達式`^$`，這意味著一個空字符串 - 這可能與我們網站的根目錄相同，我們一直用“/”進行測試嗎？讓我們看看 - 如果我們加入它會發生什麼？

如果你從來沒有遇到過正則表達式，那麼你現在就可以忘掉它，但是你應該記下去了解它們。
我們還將刪除 admin URL，因為我們暫時不會使用Django管理站點：

```py
superlists/urls.py
from django.conf.urls import url
from lists import views

urlpatterns = [
    url(r'^$', views.home_page, name='home'),
]
```
再次運行單元測試，使用`python manage.py test`：
```
[...]
TypeError: view must be a callable or a list/tuple in the case of include().
```
那是進步！我們不再獲得404了。

回溯是凌亂的，但最後的消息告訴我們發生了什麼事情：單元測試實際上已經作出的URL“/”和之間的鏈接 home_page = None中列出/ views.py，而現在抱怨 home_page的觀點是不調用。這使我們有理由將其從None實際功能轉變為實際功能。每個代碼更改都由測試驅動！

回到*lists/views.py*：
```py
lists/views.py
from django.shortcuts import render

# Create your views here.
def home_page():
    pass
```
現在？
```py
$ python manage.py test
Creating test database for alias 'default'...
.
 ---------------------------------------------------------------------
Ran 1 test in 0.003s

OK
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...

```
萬歲！我們的第一個單元測試通行證！這太重要了，我覺得它值得承諾：

$ git diff   ＃應顯示對urls.py，tests.py和views.py的更改
$ git commit -am “第一個單元測試和url映射，虛擬視圖”
這是git commit我將展示的最後一個變體，a和m標記在一起，它將所有更改添加到跟踪文件並使用命令行中的提交消息。

git commit -am是最快的表述，但也給你最少的反饋意見，所以確保你已經做了 git status一個git diff事先，並明確了將要進行的變化。

## 3.7。單元測試測試一個視圖
在為我們的視圖編寫測試時，它可以不僅僅是一個無用的函數，而是一個向HTML返回真實響應的函數。打開lists/tests.py，並添加一個新的 測試方法。我會解釋每一點：

lists/tests.py
```py
from django.urls import resolve
from django.test import TestCase
from django.http import HttpRequest

from lists.views import home_page


class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)


    def test_home_page_returns_correct_html(self):
        request = HttpRequest()                             # 1
        response = home_page(request)                       # 2
        html = response.content.decode('utf8')              # 3
        self.assertTrue(html.startswith('<html>'))          # 4
        self.assertIn('<title>To-Do lists</title>', html)   # 5
        self.assertTrue(html.endswith('</html>'))           # 6
```
這個新測試發生了什麼？

1. 我們創建了一個`HttpRequest`對象，這是Django在用戶瀏覽器要求頁面時會看到的對象。
1. 我們將它傳遞給我們的`home_page` 視圖，這給了我們一個回應。聽到這個對像是一個被調用的類的實例，你不會感到驚訝 `HttpResponse`。
1. 然後，我們提取`.content` 響應。這些是原始字節，將通過線路發送到用戶瀏覽器的1和0。我們調用 `.decode()`它們將它們轉換為發送給用戶的HTML字符串。
1. 我們希望它從一個`<html>`最終關閉的標籤開始。
1. 我們希望`<title>`在中間某處有一個標籤，其中包含“待辦事項列表” - 因為這是我們在功能測試中指定的內容。
再次，單元測試由功能測試驅動，但它也更接近實際代碼 - 我們現在正在考慮程序員。

讓我們現在運行單元測試，看看我們如何繼續：



`TypeError: home_page() takes 0 positional arguments but 1 was given`

3.7.1。單元測試/代碼週期
我們現在可以開始進入TDD 單元測試/代碼循環了：

1. 在終端中，運行單元測試並查看它們是如何失敗的。

1. 在編輯器中，進行最小的代碼更改以解決當前的測試失敗。

並重複！

我們越是緊張地使我們的代碼正確，我們使每個代碼更改的更小和更小 - 這個想法是絕對確保每個代碼都通過測試來證明。

這可能看起來很費勁，而且起初會是這樣。但是一旦你掌握了一切，即使你採取微觀步驟，你也會發現自己編碼很快 - 這就是我們在工作中編寫所有生產代碼的方式。

讓我們看看我們能夠以多快的速度完成這個循環：

最小的代碼更改：

lists/views.py
```py
def home_page(request):
    pass
```
測試：
```py
html = response.content.decode（'utf8'）
AttributeError：'NoneType'對像沒有屬性'content'
```
代碼 - 我們使用1`，如預測：

lists/views.py
```py
from django.http import HttpResponse

# Create your views here.
def home_page(request):
    return HttpResponse()
```
再次測試：
```py
    self.assertTrue(html.startswith('<html>'))
AssertionError: False is not true
```
再次編碼：

lists/views.py
```py
def home_page(request):
    return HttpResponse('<html>')
```
測試：

`AssertionError: '<title>To-Do lists</title>' not found in '<html>'`

碼：

lists/views.py
```py
def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title>')
```
測試 - 幾乎在那裡？
```py
    self.assertTrue（html.endswith（'</ HTML>'））
AssertionError：False is not true
```
來吧，最後的努力：
```py
lists/views.py
def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title></html>')
```
一定？
```py
$ python manage.py test
Creating test database for alias 'default'...
..
 ---------------------------------------------------------------------
Ran 2 tests in 0.001s

OK
System check identified no issues (0 silenced).
Destroying test database for alias 'default'...
```
是! 現在，讓我們運行我們的功能測試。如果它還沒有運行，請不要忘記再次啟動dev服務器。這感覺就像是比賽的最後熱度; 這肯定是......它可能嗎？
```py
$ python functional_tests.py
F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "functional_tests.py", line 19, in
test_can_start_a_list_and_retrieve_it_later
    self.fail('Finish the test!')
AssertionError: Finish the test!

 ---------------------------------------------------------------------
Ran 1 test in 1.609s

FAILED (failures=1)
```
失敗？什麼？哦，這只是我們的小提醒？是？是! 我們有一個網頁！

咳咳。好吧，我認為這章真是令人興奮的結局。你可能仍然有點困惑，或許有興趣聽到所有這些測試的理由，並且不要擔心，所有這一切都將會到來，但我希望你在那裡盡頭感受到一絲興奮。

稍微承諾冷靜下來，並反思我們所涵蓋的內容：

$ git diff   ＃應該在tests.py中顯示我們的新測試，在views.py中顯示視圖
$ git commit -am“基本視圖現在返回最小的HTML”
那是一個很大的篇章！為什麼不嘗試輸入`git log`，可能使用 `--oneline`旗幟，以提醒我們做了什麼：
```sh
$ git log --oneline
a6e6cc9基本視圖現在返回最小的HTML
450c0f3第一個單元測試和url映射，虛擬視圖
ea2b037為故障添加應用程序，故意單元測試失敗
[...]
```
不錯 - 我們涵蓋了：

- 啟動Django應用程序
- Django單元測試運行器
- FT和單元測試之間的區別
- Django URL解析和urls.py
- Django查看函數，請求和響應對象
- 並返回基本的HTML



Running the Django dev server

    python manage.py runserver
Running the functional tests

    python functional_tests.py
Running the unit tests

    python manage.py test
The unit-test/code cycle

        Run the unit tests in the terminal.

        Make a minimal code change in the editor.

        Repeat!

