第2章使用unittest模塊擴展我們的功能測試
讓我們調整我們的測試，該測試目前檢查默認的Django“it working”頁面，並檢查我們想要在我們網站的真實首頁上看到的一些內容。

是時候揭示我們正在建立什麼樣的網絡應用程序：待辦事項列表網站！在這樣做的過程中，我們非常關注時尚：幾年前所有的網絡教程都是關於建立一個博客。然後是論壇和投票; 現在流行的是待辦事項列表。

原因是待辦事項列表是一個非常好的例子。基本上它非常簡單 - 只是一個文本字符串列表 - 因此很容易獲得並且運行“最小可行的”列表應用程序。而且它可以通過各種方式擴展 - 不同的持久性模型，添加截止日期，提醒，與其他用戶共享以及改進客戶端UI。沒有理由僅限於“待辦事項”清單; 它們可以是任何類型的列表。但重點是它應該足以讓我演示Web編程的所有主要方面，以及如何將TDD應用於它。

## 2.1 使用功能測試來確定最小可行的應用程序的範圍
使用Selenium的測試讓我們驅動一個真正的Web瀏覽器，所以他們真的讓我們從用戶的角度看看應用程序的運行方式。這就是他們被稱為功能測試的原因。

這意味著FT可以是您的應用程序的一種規範。它傾向於跟踪您可能稱之為用戶故事的內容，並跟踪用戶如何使用特定功能以及應用程序應如何響應它們。

> 術語：功能測試==驗收測試==端到端測試
> 我稱之為功能測試，有些人更喜歡稱之為驗收測試或 端到端測試。重點是這些類型的測試從外部看待整個應用程序的運行方式。另一個術語是 黑盒測試，因為測試對被測系統的內部結構一無所知。

FT應該有一個人類可讀的故事，讓我們可以遵循。我們使用測試代碼附帶的註釋使其明確。在創建新FT時，我們可以先編寫註釋，以捕獲用戶素材的關鍵點。作為人類可讀的，您甚至可以與非程序員共享它們，作為討論應用程序的要求和功能的一種方式。

TDD和敏捷軟件開發方法經常在一起，我們經常討論的一個問題是最小的可行的應用程序; 我們可以構建的最簡單的東西仍然有用嗎？讓我們從構建它開始，這樣我們就可以盡快測試水溫。

最小可行的待辦事項列表實際上只需要讓用戶輸入一些待辦事項，並記住它們以便下次訪問。

打開functional_tests.py並寫一個類似這樣的故事：

`functional_tests.py`。 
```py
from selenium import webdriver

browser = webdriver.Firefox()

# Edith has heard about a cool new online to-do app. She goes
# to check out its homepage
browser.get('http://localhost:8000')

# She notices the page title and header mention to-do lists
assert 'To-Do' in browser.title

# She is invited to enter a to-do item straight away

# She types "Buy peacock feathers" into a text box (Edith's hobby
# is tying fly-fishing lures)

# When she hits enter, the page updates, and now the page lists
# "1: Buy peacock feathers" as an item in a to-do list

# There is still a text box inviting her to add another item. She
# enters "Use peacock feathers to make a fly" (Edith is very methodical)

# The page updates again, and now shows both items on her list

# Edith wonders whether the site will remember her list. Then she sees
# that the site has generated a unique URL for her -- there is some
# explanatory text to that effect.

# She visits that URL - her to-do list is still there.

# Satisfied, she goes back to sleep

browser.quit()
```
> 我們有一個註解的詞......
> 當我第一次在Resolver開始時，我曾經用很好的描述性註解來熟練地編寫代碼。我的同事對我說：“哈利，我們對於註解有另一個說法。我們稱他們為謊言。“我很震驚！但我在學校裡了解到註解是好的做法？

> 他們誇大效果。必定有一個註解適用的地方以作為上下文和意圖。但他們的觀點是，寫一條註解只是重複你正在使用代碼做的事情是沒有意義的：
```py
# increment wibble by 1
wibble += 1
```
> 它不僅毫無意義，更新代碼時會忘記更新註釋，而且最終會產生誤導。理想的是努力使你的代碼便於閱讀，使用良好的變量名和函數名，並將它組織好，你就不再需要任何註釋來解釋什麼代碼正在做什麼。這里和那裡只是為了解釋原因。

> 還有其他地方註解非常有用。我們將看到Django在它生成的文件中大量使用它們，以供我們用它來建議其API的有用位。當然，我們在功能測試中使用註釋來解釋用戶故事 - 通過強迫我們從測試中創建一個連貫的故事，它確保我們始終從用戶的角度進行測試。

> 在這個領域有更多的樂趣，比如 行為驅動開發和測試DSL，但它們是其他書籍的主題。

你會注意到，除了把測試寫成註解之外，我已經更新了它`assert`來尋找“To-Do”這個詞而不是“Django”。這意味著我們希望測試現在失敗。我們試試吧吧

首先，啟動服務器：

`$ python3 manage.py runserver`
然後，在另一個shell中運行測試：
```py
$ python functional_tests.py
Traceback (most recent call last):
  File "functional_tests.py", line 10, in <module>
    assert 'To-Do' in browser.title
AssertionError
```
這就是我們所謂的*預期失敗*，這實際上是好消息 - 不如通過的測試那麼好，但至少它是因為正確的原因失敗了; 我們可以確信我們已經正確地編寫了測試。

Python標準庫的unittest模塊
我們應該處理一些小煩惱。首先， 訊息“AssertionError”不是很有用 - 如果測試告訴我們它實際上找到了什麼作為瀏覽器標題會很好。此外，它還會在桌面上掛著一個Firefox視窗，如果可以自動清理掉，那就太好了。

一種選擇是將第二個參數用於`assert`關鍵字，例如：

`assert 'To-Do' in browser.title, "Browser title was " + browser.title`
我們還可以使用a` try/finally`來清理舊的Firefox窗口。但是這些問題在測試中非常普遍，在標準庫的`unittest`模塊中有一些現成的解決方案。讓我們用它！在functional_tests.py中：

functional_tests.py。 
```py
from selenium import webdriver
import unittest

class NewVisitorTest(unittest.TestCase):  #1

    def setUp(self):  #2
        self.browser = webdriver.Firefox()

    def tearDown(self):  #3
        self.browser.quit()

    def test_can_start_a_list_and_retrieve_it_later(self):  #4
        # Edith has heard about a cool new online to-do app. She goes
        # to check out its homepage
        self.browser.get('http://localhost:8000')

        # She notices the page title and header mention to-do lists
        self.assertIn('To-Do', self.browser.title)  #五
        self.fail('Finish the test!')  #6

        # She is invited to enter a to-do item straight away
        [...rest of comments as before]

if __name__ == '__main__':  #7
    unittest.main(warnings='ignore')  #8
```
你可能會注意到這裡的一些事情：

1. 測試被組織成繼承自的類`unittest.TestCase`。

2. 測試的主體是一個叫做`test_can_start_a_list_and_retrieve_it_later`的方法。名稱以`test`開頭的任何方法都是測試方法，並且將由測試運行器運行。每個類可以有多個`test_`方法。我們的測試方法的描述性名稱也是一個好主意。
3. `setUp` 和`tearDown` 是在每次測試之前和之後都會運行的特殊方法。我正在使用它們來啟動和停止我們的瀏覽器 - 注意它們有點像a `try/except`，tearDown即使在測試期間出現錯誤也會運行。[ 2 ] 不再有Firefox視窗到處掛著！

4. 我們使用`self.assertIn`而不是僅僅`assert`來進行測試斷言。`unittest`提供了很多這樣的輔助函式，來進行斷言的測試，如`assertEqual`，`assertTrue`，`assertFalse`，等。您可以在`unittest`文檔中找到更多信息。

5. `self.fail`無論如何都會失敗，產生錯誤信息。我用它作為完成測試的提醒。

6. 最後，我們有了這個`if __name__ == '__main__'`子句（如果你以前沒見過，那就是Python腳本如何檢查它是否是從命令行執行的，而不是僅僅是由另一個腳本導入）。我們調用 `unittest.main()`，啟動`unittest`測試運行器，它將自動在文件中找到測試類和方法並運行它們。

7. `warnings='ignore'`抑制`ResourceWarning`在寫作時發出的多餘的東西。當你讀到這篇文章時它可能已經消失了; 隨意嘗試刪除它！

**注意**
如果您已經閱讀過Django測試文檔，那麼您可能已經看到了一些名為`LiveServerTestCase`的東西，並且想知道我們是否應該現在就使用它。閱讀友好手冊的全部內容！ `LiveServerTestCase`現在有點太複雜，但我保證我會在後面的章節中使用它......

我們來試試吧！
```py
$ python functional_tests.py
F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "functional_tests.py", line 18, in
test_can_start_a_list_and_retrieve_it_later
    self.assertIn('To-Do', self.browser.title)
AssertionError: 'To-Do' not found in 'Welcome to Django'

 ---------------------------------------------------------------------
Ran 1 test in 1.747s

FAILED (failures=1)
```
那有點好不是嗎？它整理了我們的Firefox窗口，它為我們提供了一個格式良好的報告，說明運行了多少測試以及失敗了多少，並且assertIn給了我們一個有用的錯誤消息和有用的調試信息。華麗的！

## 2.3 提交
這是做提交的好點; 這是一個非常自足的變化。我們擴展了功能測試，包括描述我們自己設定的任務的註解，這是我們最不可行的待辦事項列表。我們還重寫了它以使用Python unittest模塊及其各種測試輔助函數。

做一個`git status`- 應該向您保證，唯一已更改的文件是`functional_tests.py`。然後執行a git diff，它會顯示上次提交與當前磁盤上的內容之間的區別。這應該告訴你`functional_tests.py`已經發生了很大變化：
```sh
$ git diff
diff --git a / functional_tests.py b / functional_tests.py
index d333591..b0f22dc 100644
--- a / functional_tests.py
+++ b / functional_tests.py
@@ -1,6 +1,45 @@
 來自selenium import webdriver
+ import unittest

-browser = webdriver.Firefox（）
-browser.get（的“http：//本地主機：8000'）
+ class NewVisitorTest（unittest.TestCase）：

- 在browser.title中插入'Django'
+ def setUp（self）：
+ self.browser = webdriver.Firefox（）
+ self.browser.implicitly_wait（3）
+
+ def tearDown（self）：
+ self.browser.quit（）
[...]
```
現在讓我們做一個：

`$ git commit -a`
的-a意思是“自動添加到跟踪的文件進行任何更改”（即我們之前所犯的任何文件）。它不會添加任何全新的文件（你必須git add自己明確表示），但通常，在這種情況下，沒有任何新文件，所以它是一個有用的快捷方式。

彈出編輯器時，添加描述性提交消息，例如“在註解中推出的First FT，現在使用unittest”。

現在我們處於一個很好的位置，開始為我們的列表應用程序編寫一些真正的代碼。繼續閱讀！

>有用的TDD概念
>**用戶故事**
>從用戶的角度描述應用程序的工作方式。用於構建功能測試。
>**預期的失敗**
>當測試以我們預期的方式失敗時。

[ 2 ]唯一的例外是如果你有一個例外 setUp，那麼tearDown就不會運行。