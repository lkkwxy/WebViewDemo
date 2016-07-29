# 一、WKWebView的基本使用
    
  1、初始化方法
<pre>- (instancetype)initWithFrame:(CGRect)frame configuration:(WKWebViewConfiguration *)configuration</pre>2、简单介绍一下configuration，WKWebViewConfiguration里面有个userContentController。可通过它为webview注入javaScript代码。并且可以添加监听javaScript的回调。
(1)初始化configuration<pre> WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc]init];</pre>(2)获取javaScript代码,我把javaScript代码写在了一个文件中<pre>NSString *jsStr = [NSString stringWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"WKWebViewJS" ofType:@"js"] encoding:NSUTF8StringEncoding error:nil];</pre>

(3)初始化script对象
<pre>
	 /*@abstract 初始script对象
     @param source javaScript代码
     @param injectionTime 注入javaScript代码的时机
     @param forMainFrameOnly 是不是仅为MainFrame注入
     */

    WKUserScript *userScript = [[WKUserScript alloc]initWithSource:jsStr injectionTime:WKUserScriptInjectionTimeAtDocumentStart forMainFrameOnly:NO];</pre>

(4)添加script对象<pre>
[configuration.userContentController addUserScript:userScript];</pre>
(5)添加scriptMessage回调
window.webkit.messageHandlers.<name>.postMessage(<messageBody>) 在JS中实现这个方法，系统会调用这个方法- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message

<pre>
[configuration.userContentController addScriptMessageHandler:self name:@"imageClick"];
[configuration.userContentController addScriptMessageHandler:self name:@"popToPreviousVC"];
</pre>
3、WKWebView 执行javaScript代码
<pre>self.webview evaluateJavaScript:@"changeFontSize(12)" completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        }];</pre>
4、WKWebView的一些简单属性的介绍
<pre>backForwardList            浏览历史
title                      网页标题             支持KVO
URL                        正在显示的URL        支持KVO
loading                    是否正在加载          支持KVO
estimatedProgress          加载进度             支持KVO
canGoBack  canGoForward    能否后退 前进         支持KVO
reload                     重新加载
stopLoading                停止加载
allowsBackForwardNavigationGestures  是否允许侧滑返回上一页
</pre>

5、WKWebView的加载方法跟UIWebView基本一样
<pre>
- (nullable WKNavigation *)loadRequest:(NSURLRequest *)request;
- (nullable WKNavigation *)loadHTMLString:(NSString *)string baseURL:(nullable NSURL *)baseURL;
- (nullable WKNavigation *)loadData:(NSData *)data MIMEType:(NSString *)MIMEType characterEncodingName:(NSString *)characterEncodingName baseURL:(NSURL *)baseURL;
</pre>

#二 WKWebViewUIDelegate

<pre>
//当需要打开一个新窗口的时候的调用，如a标签的target='_blank'，需要返回一个新的Webview
- (nullable WKWebView *)webView:(WKWebView *)webView createWebViewWithConfiguration:(WKWebViewConfiguration *)configuration forNavigationAction:(WKNavigationAction *)navigationAction windowFeatures:(WKWindowFeatures *)windowFeatures

以下三个类似alert的代理，一定要调用completionHandler(),这个回调，告诉webview结果
//webview上需要弹出alert的时候调用此方法，如果不实现此方法，则webview的alert是显示不出来,alert类似于只有确定按钮的UIAlertView
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler

//webview上需要弹出confirm的时候调用此方法，如果不实现此方法，则webview的confirm是显示不出来,confirm类似于有确定按钮和取消按钮的UIAlertView
- (void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL result))completionHandler

//webview上需要弹出prompt的时候调用此方法，如果不实现此方法，则webview的prompt是显示不出来,prompt类似于带一个textField的UIAlertView。defaultText相当于textField的placeholed
- (void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(nullable NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * __nullable result))completionHandler

//window.close() 的时候调用
- (void)webViewDidClose:(WKWebView *)webView
</pre>

#三 WKWebViewNavigationDelegate

<pre>
//决定是否允许发起这个请求
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler

//在webview有响应之后，再次决定是否允许这个请求
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandle

//webview开始加载的时候调用
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(null_unspecified WKNavigation *)navigation

//webview内容已经加载结束，但是上面的某些资源比如图片加载之前调用
- (void)webView:(WKWebView *)webView didCommitNavigation:(null_unspecified WKNavigation *)navigation

//webview加载结束的时候调用
- (void)webView:(WKWebView *)webView didCommitNavigation:(null_unspecified WKNavigation *)navigation

//webview加载失败的时候调用
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error

//webview在commit过程中失败的时候调用，例如在didCommitNavigation这个代理方法中调用webview的stopLoading方法
- (void)webView:(WKWebView *)webView didFailNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error
</pre>

#四 WKScriptMessageHandler

<pre>
//收到JavaScript回调的时候调用
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message</pre>

#五 接下来用一下做个小demo。
功能如下：
1、点击按钮可以控制webView上的字体的大小和颜色
2、点击webView上的返回，能pop到上一个VC
3、点击webView图片,弹出alert告诉我弹出的是第几张图片，和图片的大小和位置以及所有的图片的地址
4、验证代理方法
效果如下：

![1.gif](http://upload-images.jianshu.io/upload_images/128249-5b9580381ec982c3.gif?imageMogr2/auto-orient/strip)

  (1)首先为webview注入JS代码，JS代码里主要实现的是：当window加载之后，为img标签添加click事件，定义改变字体颜色和大小的函数
  <pre>		WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc]init];
    NSString *jsStr = [NSString stringWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"WKWebViewJS" ofType:@"js"] encoding:NSUTF8StringEncoding error:nil];
    WKUserScript *userScript = [[WKUserScript alloc]initWithSource:jsStr injectionTime:WKUserScriptInjectionTimeAtDocumentStart forMainFrameOnly:NO];
    [configuration.userContentController addUserScript:userScript];
    [configuration.userContentController addScriptMessageHandler:self name:@"imageClick"];
    [configuration.userContentController addScriptMessageHandler:self name:@"popToPreviousVC"];</pre>
(2)调用JS代码,实现改变字体颜色和大小
<pre>
- (void)runJavaAcript:(UIButton *)button{
    if (button.tag == 0) {
        [self.webview evaluateJavaScript:@"changeTextColor('#f00')" completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        }];
    }else if (button.tag == 1){
        [self.webview evaluateJavaScript:@"changeTextColor('#666')" completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        }];
    }else if (button.tag == 2){
        [self.webview evaluateJavaScript:@"changeFontSize(38)" completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        }];
    }else if (button.tag == 3){
        [self.webview evaluateJavaScript:@"changeFontSize(12)" completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        }];
    }
}</pre>

(3)点击webview上的返回按钮pop到上一个VC
1）在html中的定义
<pre><code>
//返回标签的定义
<button onclick="popToPreviousVC()">返回</button>
//函数实现
function popToPreviousVC(){
    webkit.messageHandlers.popToPreviousVC.postMessage(true);
}
</code></pre>
2）webview收到回调的处理
  <pre>
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{
    if ([message.name isEqualToString:@"imageClick"]) {
        NSInteger current = [message.body[@"current"] integerValue];
        UIAlertView *alert = [[UIAlertView alloc]initWithTitle:[NSString stringWithFormat:@"你点击了第%ld张图片,该图片的坐标是%@",(long)current,message.body[@"rect"]] message:[NSString stringWithFormat:@"图片的地址分别是：%@",message.body[@"srcs"]] delegate:nil cancelButtonTitle:nil otherButtonTitles:@"OK", nil];
        [alert show];
    }else if([message.name isEqualToString:@"popToPreviousVC"]){
        [self.navigationController popViewControllerAnimated:YES];
    }
}</pre>

<blockquote>
  <p>1、要想接收到回调，一定要先在先添加回调
  [configuration.userContentController addScriptMessageHandler:self name:@"popToPreviousVC"];
 2、添加的名字一定要和发送的名字一致
 </p>
</blockquote>

(4)点击webView图片,弹出alert告诉我弹出的是第几张图片，和图片的大小和位置以及所有的图片的地址,回调的信息如上，下面展示js代码
<pre>
function initImageClick(obj,index) {
    obj.onclick = function imgOnclick() {
        //获取image的位置
        var rect = obj.getBoundingClientRect();
        //获取所有的img标签
        var imgs = document.getElementsByTagName('img');
        var srcs = new Array();
        for(var i in imgs){
            //判断img的src是否为未定义，加入srcs数组
            if(imgs[i].src != undefined)
            srcs.push(imgs[i].src);
        }
        //由于OC不识别rect这个对象，在这把rect转成了符合CGRect格式的字符串
        var message = {'rect':'{{'+rect.left+', '+rect.top+'}, {'+rect.width+', '+rect.height+'}}}'};
        //为message这个js对象的属性赋值
        message.current = index;
        message.srcs = srcs;
        //发送消息
        webkit.messageHandlers.imageClick.postMessage(message);
    }
}</pre>
代理方法的验证具体看代码
