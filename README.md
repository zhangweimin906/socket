# socket


Fleck是C＃中的WebSocket服务器实现。从Nugget项目分支出来的
Fleck不需要继承，容器或其他引用。
Fleck不依赖于HttpListener或HTTP.sys意味着它将在Windows 7和Server 2008主机上运行。WebSocket备注-Microsoft Docs

例子
以下是将回显到客户端的示例。

var server = new WebSocketServer("ws://0.0.0.0:8181");
server.Start(socket =>
{
  socket.OnOpen = () => Console.WriteLine("Open!");
  socket.OnClose = () => Console.WriteLine("Close!");
  socket.OnMessage = message => socket.Send(message);
});
        

支持的WebSocket版本
Fleck支持多种WebSocket版本的现代Web浏览器

Hixie-Draft-76 / Hybi-00（Safari 5，Chrome <14，Firefox 4（启用时））
Hybi-07（Firefox 6）
Hybi-10（Chrome 14-16，Firefox 7）
Hybi-13（Chrome 17以上版本，Firefox 11以上版本，Safari 6以上版本，Edge 13以上版本（？））


安全WebSocket（wss：//）
启用安全连接需要两件事：使用方案wss代替ws，以及将Fleck指向包含公钥和私钥的x509证书。
var server = new WebSocketServer("wss://0.0.0.0:8431");
server.Certificate = new X509Certificate2("MyCert.pfx");
server.Start(socket =>
{
  //...use as normal
});
制作证书时遇到问题？请参阅本
指南，
以@AdrianBathurst创建x509

子协议协商
要启用子协议协商，请在WebSocketServer.SupportedSubProtocols属性上指定受支持的协议。协商的子协议将在套接字的上可用ConnectionInfo.NegotiatedSubProtocol。
如果在客户端请求（Sec-WebSocket-Protocol标头）上未找到受支持的子协议，则连接将关闭。
var server = new WebSocketServer("ws://0.0.0.0:8181");
server.SupportedSubProtocols = new []{ "superchat", "chat" };
server.Start(socket =>
{
  //socket.ConnectionInfo.NegotiatedSubProtocol is populated
});

自定义日志
Fleck可以登录Log4Net或任何其他第三方日志记录系统。只需FleckLog.LogAction使用所需的行为覆盖该属性。
ILog logger = LogManager.GetLogger(typeof(FleckLog));

FleckLog.LogAction = (level, message, ex) => {
  switch(level) {
    case LogLevel.Debug:
      logger.Debug(message, ex);
      break;
    case LogLevel.Error:
      logger.Error(message, ex);
      break;
    case LogLevel.Warn:
      logger.Warn(message, ex);
      break;
    default:
      logger.Info(message, ex);
      break;
  }
};


禁用Nagle的算法
设置NoDelay于true上WebSocketConnection.ListenerSocket
var server = new WebSocketServer("ws://0.0.0.0:8181");
server.ListenerSocket.NoDelay = true;
server.Start(socket =>
{
  //Child connections will not use Nagle's Algorithm
});

侦听错误后自动重启
设置RestartAfterListenError于true上WebSocketConnection
var server = new WebSocketServer("ws://0.0.0.0:8181");
server.RestartAfterListenError = true;
server.Start(socket =>
{
  //...use as normal
});
