## 知网、搜狗微信、搜狗新闻爬虫

个人项目，只支持**python3**.

需要说明的是，本文中介绍的都是小规模数据的爬虫（数据量<1G），大规模爬取需要会更复杂，本文不涉及这一块。另外，代码细节就不过多说了，只将一个大概思路以及趟过的坑。

本文中涉及的阿布云IP隧道及云打码平台需要自己注册，并在code中相应部分取消注释。

---
### 搜狗微信文章抓取
- 目标：**在搜狗微信模块下搜索关键词的文章，抓取链接保存文章标题，来源，时间，内容的内容**
- 采取的策略：selenium模拟搜索，登陆扫码采用手动扫描的模式，登陆后通过不同的关键词进行搜索，翻页等操作。
- 遇到的坑：
1. SogouWechat这个库只能抓到10个items（自己加入cookies也只能抓10个好像，反正我没成功的抓多个）
2. 登陆只想到手动扫描这一块，没有其他更好的方法
3. 搜索出来的文章链接时临时性的，要及时request并保存
4. 在模拟翻页操作的时候，建议模拟一下页面滚动
5. 网速不好的情况，要有sleep，要不然chrome会报错

### 搜狗新闻抓取
- 目标：**在搜狗新闻搜索中搜索关键，将所有新闻的标题，时间，内容保存下来**
- 采取的策略：
1. request.get关键词，因为搜狗新闻就不涉及到cookies的问题，直接请求
2. ip隧道代理请求（阿布云代理）
3. news的具体页面，如果request获取不到文本，用selenium抓
- 遇到的坑：参照以上第三点。
### 知网摘要信息抓取
- 目标：**指定文献来源或者单位，抓所有的文献的摘要，作者，时间等等**
- 采取的策略：
1. selenium模拟登陆，得到搜索页面
2. ajax抓包，构造请求发送到服务器
3. 自动打码（云打码，效果还可以）
4. ip隧道代理
5. 翻页用request构造
- 遇到的坑：
1. 必须要登陆才能看到所有文献
2. 打码失败的话one more time
3. 数据量有点多，及时保存数据，我没有用数据库，我直接写到文件了

---
### 配置文件、运行文件讲解
项目控制运行模块全部都是在setting文件中修改配置的。

1. 抓取范围配置
注意，START和END是默认为””的，这是指不进行范围限定。如果需要限定范围，必须同时输入START和END，不能只输入一个，另一个为空。
E.g. START =  “20140101”
END =  “ 20180101”
另外在抓知网的时候，由于知网只能浏览300页，所以限定了时间范围也只是在300页内找时间范围内的文献。

2. 抓取源选择
DATA_FROM 是选择抓取哪个模块的参数，只能在以下5个选项内选择
"sogou_news", "sogou_wechat", "cnki_journal", "cnki_from"
E.g.  DATA_FROM = “sogou_wechat”
另外抓取sogou_wechat的时候，刚开始会弹出二维码界面，这是一个微信扫码登陆搜狗的页面，必须扫码登陆，要不然只能访问部分文章

3. 抓取关键词
KEYWORDS 指抓取 sogou_news、sogou_wechat、inner_search需要抓的关键词（知网搜索的关键词与这个无关），以list形式传入
E.g.  KEYWORDS = [“高分一号”, “高分二号”, ...]
另外，搜狗需要精确匹配，程序已处理，只用在这个地方按照以上输入就可以完成。

4. 知网期刊
JOURNAL 指 在 DATA_FROM = “cnki_journal”情况下，需要搜索的期刊，以list形式传入。（建议每次传入一到两个期刊名，因为每次跑的时间过长，有情况及时发现处理）
E.g. 	JOURNAL = [“测绘科学”, ...]

5. 知网来源
FROMS 指 在DATA_FROM = “cnki_from”情况下，需要搜索的单位名称，以list形式传入。
E.g. FROMS = [“武汉测绘院”, ...]
（建议一次性不超过50个）

6. IP、打码配置
一般不要动，除非要修改隧道和打码配置
