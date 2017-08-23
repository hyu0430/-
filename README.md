# -
page request session application作用域简单理解 
几乎所有的Web开发语言都支持Session功能，Servlet也不例外。 Servlet/JSP中的Session功能是通过作用域(scope)这个概念来实现的。
作用域分为四种，分别为：

page
在当前页面有效(仅用于JSP中)
request
在当前请求中有效
session
在当前会话中有效
application
在所有应用程序中有效

是不是看不太明白？page因为仅用于JSP中，这里只讲述其他三种作用域。 首先要声明的一点，所谓“作用域”就是“信息共享的范围”， 也就是说一个信息能够在多大的范围内有效。 

话说武松一日来到景阳岗，见一旗帜迎风飘扬，旗子上书五个大字“三碗不过岗”。 武松叫道：“店家，拿三碗酒来，再切两斤熟牛肉！”店小二应声道：“三碗好酒， 二斤熟牛肉啰～～”里面厨师赶忙当当当当切好牛肉，店小二倒上三碗酒，店小二端上前来。
武松咕咚咕咚连干三碗，叫一声“好酒！店家，再来三碗！”小二忙又倒上三碗好酒， 武松一饮而尽。就这样前前后后武松一共喝了十八大腕。付了帐刚要走，店小二道： “客官，这前面山上有大虫，客官刚刚喝完十八碗酒恐怕过不得岗，不如在小店暂住一夜， 待明天和猎户一同过岗岂不是好？”
之后武松说什么就留待各位看官自己去回忆啦。在这段武松打虎中， 大家有没有看到些熟悉的东西？
•武松：浏览器。
•酒馆： 服务器。
•店小二、厨师： Servlet或者JSP。
•来三碗好酒！：浏览器向服务器发出HTTP请求。
•店小二上酒：服务器的响应。
•武松从进店到离开： 一个HTTP会话（即 Session）。
我们可以看到，Web交互的最基本单位为HTTP请求(‘武松点菜‘)。 每个用户从进入网站到离开网站这段过程称为一个HTTP会话 (“武松进店到出店”)，一个服务器的运行过程中会有多个用户访问， 就是多个HTTP会话(“酒馆当然不可能只接待武松一个客人”)。 那么作用域就可以理解为：

request
HTTP请求开始到结束这段时间
session
HTTP会话开始到结束这段时间
application
服务器启动到停止这段时间

request
一个HTTP请求的处理可能需要多个Servlet合作(“武松点菜时店小二就要吩咐厨房做菜”)， 几个Servlet之间可以通过某种方式传递信息(“店小二就用吆喝的方式通知厨房”)， 但这个信息在请求结束后就无效了(“厨房在做完菜之后就不用再管这道菜的事儿了”)。 

Servlet之间的信息共享是通过HttpServletRequest接口的两个方法来实现的：
void setAttribute(String name, Object value)
将对象 value 以 name 为名称保存到request作用域中。
Object getAttribute(String name)
从request作用域中取得指定名字的信息。
doGet()、doPost()函数的第一个参数就是 HttpServletRequest 对象， 使用这个对象的 setAttribute 即可传递信息。
那么设置好信息之后，如何将信息传给其他Servlet？ 这就要用到 RequestDispatcher 接口的 forward 方法，将请求转发给其他Servlet。
RequestDispatcher ServletContext.getRequestDispatcher(String path)
取得Dispatcher以便转发。path为转发的目的Servlet。
void RequestDispatcher.forward(ServletRequest request, ServletResponse response)
将request和response转发。
因此，只要在当前Servlet中先 setAttribute，然后forward，最后在forward到的Servlet中 getAttribute即可实现信息传递。
PHP的程序员可能不太好理解这一段，因为php中没有转发的概念， 一个请求只能由一个PHP文件来处理，所以PHP中根本没有request作用域的概念。 而Servlet则不同，请求可以在应用程序中任意转发，所以用request作用域在不同Servlet之间传递信息。 需要注意两点：
1.转发不是重定向，转发是在Web应用内部进行的。PHP支持重定向但没有转发。
2.转发对浏览器是透明的，也就是说，无论在服务器上如何转发，浏览器地址栏中显示的仍然是最初那个Servlet的地址。

session
session作用域比较容易理解，同一浏览器访问多次，在这多次访问之间传递信息，就是session作用域。 (武松每次点菜，帐房先生都要记一笔账，等武松走之前结帐用。 这笔帐在武松吃饭过程中始终有效，即位于session作用域中)
session是通过HttpSession接口实现的。
Object HttpSession.getAttribute(String name)
从session中获取信息
void HttpSession.setAttribute(String name, Object value)
向session中保存信息
而通过HttpServletRequest.getSession()方法可以获得HttpSession对象。
HttpSession HttpServletRequest.getSession()
获取当前请求所在的session的对象。
session的开始容易判断(浏览器发出第一个HTTP请求即可认为会话开始)， 但结束就不好判断了(因为浏览器关闭时不会通知服务器“我关了，会话可以结束了”)， 所以只能通过这种方法判断：如果一定的时间内客户端没有反应，则认为会话结束。 Tomcat的默认值为120分钟，但这个值也可以通过 HttpSession 的 setMaxInactiveInterval() 方法来设置。
void setMaxInactiveInterval(int interval)
设置会话的超时值。
如果想主动让会话结束，如用户单击“注销”的时候，可以使用HttpSession 的 invalidate() 方法：
void invalidate()
强制结束当前session。
application
application作用域就是服务器启动到关闭的整段时间， 在这个作用域内设置的信息可以被所有应用程序使用。 (餐馆打烊后结帐，用到的即是开张到打烊之间的所有信息。)
还记得上一节提到的ServetContext吗？ application作用域上的信息传递就是通过ServetContext实现的。
Object getAttribute(String name)
从application中获取信息。
void setAttribute(String name, Object value)
向application作用域中设置信息。
总结
可以看到，每个作用域除了实现接口不同、意义不同之外，它们的使用方法和作用都是相同的， 都是通过 getAttribute 和 setAttribute 方法进行信息传递。

作用域
意义
实现接口
request
HTTP请求内
HttpServletRequest
session
HTTP会话内
HttpSession
application
服务器生命周期内
ServletContext


示例程序
示例程序
这一节的示例程序是一个用户登录的模拟程序。文件较多。
•login.html 登录表单
•DoLogin.Java 处理登录动作的Servlet
•LoginSuccess.java 用于显示登录成功信息的Servlet
•SessionTest.java 登录后的处理程序
•DoLogout.java 注销的处理程序
为了演示 request、application、session 各个作用域的使用方法， Servlet之间进行了数据传递，数据传递方式如下：


数据产生
数据接受
数据内容
作用域
DoLogin
LoginSuccess
登录时间
request
DoLogin
SessionTest
登录用户名
session
DoLogin
SessionTest
系统登录次数
application
