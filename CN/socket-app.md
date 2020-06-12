# socket 双机通信                           [返回主页](../index.md)

## Demo下载

[客户端](https://hsupengbo.github.io/Client.exe)
[服务端](https://hsupengbo.github.io/Server.exe)

## 任务描述
利用 Socket 来实现双机通信，理解 TCP 状态机图
实验内容：使用 Socket 编程，采用其中的 TCP 面向连接方式，实现计算机数
据的交换。
## 实验软件和硬件环境：
### 实验软件：  
VS2019, MFC 类库  
### 实验硬件:  
Intel® Core™ i5-8250U CPU @1.60GHz 1.80GHz
## 技术原理:
1. 利用 socket 实现通信
2. 利用多线程实现异步通信
## 测试结果:

## 关键代码： 
### 服务端：
(输入端口后)
点击开启按钮:
客户端再次连接通信: 
```
void CServerDlg::OnBnClickedButton1()
{
// TODO: 在此添加控件通知处理程序代码
UpdateData(TRUE);
int myport = _ttoi(s_port);
UpdateData(FALSE);
if (myport < 1024 || myport > 65535) {
UpdateData(FALSE);
MessageBox(TEXT("输入的端口不合适"), TEXT("提示"));
return;
}
EndFlag = 0;
GetDlgItem(IDC_BUTTON1)->ShowWindow(SW_HIDE);
GetDlgItem(IDC_BUTTON2)->ShowWindow(SW_SHOW);
GetDlgItem(IDC_BUTTON3)->ShowWindow(SW_SHOW);
WSADATA wsData;
WSAStartup(MAKEWORD(2, 2), &wsData);
 CString IP= _T("ip:0.0.0.0");
AfxBeginThread(&StartFunc, 0); //开启线程
WSACleanup();
}
```
线程函数 StartFunc():
```
UINT StartFunc(LPVOID p) {
WSADATA wsaData;
WORD wVersion;
wVersion = MAKEWORD(2, 2);
WSAStartup(wVersion, &wsaData);
SOCKADDR_IN local_addr;
SOCKADDR_IN client_addr;
CString port;
CServerDlg* dlg = (CServerDlg*)AfxGetApp()->GetMainWnd(); //得到主窗口指针
dlg->GetDlgItemTextW(IDC_EDIT1, port);
//创建socket
local_addr.sin_family = AF_INET;
local_addr.sin_port = htons(_ttoi(port));
local_addr.sin_addr.s_addr = htonl(INADDR_ANY);
if ((serSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)) == INVALID_SOCKET)
{
dlg->update( _T("创建失败"));
return 0;
}
BOOL debug = true; //端口复用
setsockopt(serSocket, SOL_SOCKET, SO_REUSEADDR, (const char*)&debug, sizeof(BOOL));
//绑定
bind(serSocket, (struct sockaddr*) & local_addr, sizeof(SOCKADDR_IN));
//监听
listen(serSocket, 2);
dlg->update( _T("已成功开启...."));
while (1) {
AfxBeginThread(&AcceptFunc, 0);
if (EndFlag) break; //EndFlag表示已经点击关闭按钮，需要停止线程
}
closesocket(cliSocket);
return 0;
}
```
线程函数 AcceptFunc():
```
UINT AcceptFunc(LPVOID p) {
CString port;
CServerDlg* dlg = (CServerDlg*)AfxGetApp()->GetMainWnd(); //得到主窗口指针
SOCKADDR_IN client_addr;
int iaddrSize = sizeof(SOCKADDR_IN);
int res;
char msg[256] = { 0 };
if ((cliSocket = accept(serSocket, (struct sockaddr*) & client_addr, &iaddrSize)) ==
INVALID_SOCKET)
{
dlg->update(_T("连接失败"));
return 0;
}
while (1) //接收信息
{
if (EndFlag) {
return 0;
}
if ((res = recv(cliSocket, msg, 256, 0)) <=0)
{
dlg->update(_T("失去连接")); return 0;
}
else
{
USES_CONVERSION;
CString strRecv = A2T(msg);
dlg->update( _T("客户端>>") + strRecv);
}
}
return 0;
}
```
客户端：
(输入 IP 地址，端口后)
客户端点击连接按钮:
```
void CClientDlg::OnBnClickedButton1()
{
// TODO: 在此添加控件通知处理程序代码
UpdateData(TRUE);
int myport = _ttoi(sPort);
if (myport < 1024 || sPort.IsEmpty()) {
UpdateData(FALSE);
MessageBox(TEXT("输入的端口不合适"), TEXT("提示"));
return;
}
ClearAll(); //清屏
UpdateData(FALSE);
WSADATA wsaData;
SOCKADDR_IN server_addr;
WORD wVersion;
wVersion = MAKEWORD(2, 2);
WSAStartup(wVersion, &wsaData);
server_addr.sin_addr.s_addr = htonl(sIP);
server_addr.sin_family = AF_INET;
server_addr.sin_port = htons(_ttoi(sPort));
//创建socket
if ((cliSocket = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)) == INVALID_SOCKET)
{
update(_T("创建Soket错误!"));
return;
}
//socket连接
if (connect(cliSocket, (struct sockaddr*) & server_addr, sizeof(SOCKADDR_IN)) == SOCKET_ERROR)
{
update(_T("连接失败"));
return;
}
else
{
update(_T("连接成功"));
GetDlgItem(IDC_BUTTON1)->ShowWindow(SW_HIDE);
GetDlgItem(IDC_BUTTON2)->ShowWindow(SW_SHOW);
GetDlgItem(IDC_BUTTON3)->ShowWindow(SW_SHOW); 
EndFlag = 0;
AfxBeginThread(&RecvFunc, 0); //开启线程
}
}
```
线程函数 RecvFunc():
```
UINT RecvFunc(LPVOID p) {
int res;
char msg[256] = { 0 };
CString s;
CClientDlg* dlg = (CClientDlg *)AfxGetApp()->GetMainWnd(); //得到主窗口指针
dlg->update(_T("开始聊天"));
while (1) //接收数据
{
if (EndFlag) {
dlg->ClearAll();
EndFlag = 0;
break;
}
if ((res = recv(cliSocket, msg, 256, 0)) == -1)
{
dlg->update(_T("失去连接"));
break;
}
else
{
USES_CONVERSION;
CString strRecv = A2T(msg);
dlg->update(_T("服务器>>") + strRecv);
}
}
return 0;
}
```
服务端与客户端近似的功能函数:
点击发送按钮:
```
void CServerDlg::OnBnClickedButton3()
{
// TODO: 在此添加控件通知处理程序代码
UpdateData(TRUE);
char msg[256] = { 0 };
USES_CONVERSION;
strcpy_s(msg, 256, T2A(s_send));
UpdateData(FALSE);
if (s_send.GetLength() == 0)
{
MessageBox(_T("请输入信息"));
}
else if (send(cliSocket, msg, 256, 0) == SOCKET_ERROR) //发送数据
{
update( _T("发送失败"));
}
else
{
update( _T("服务器>>") + s_send);
}
UpdateData(TRUE);
s_send = _T("");
UpdateData(FALSE);
}
```
点击关闭或者断开按钮:
```
void CServerDlg::OnBnClickedButton2()
{
// TODO: 在此添加控件通知处理程序代码
UpdateData(TRUE);
s_page = _T("");
UpdateData(FALSE);
EndFlag = 1;
GetDlgItem(IDC_BUTTON1)->ShowWindow(SW_SHOW);
GetDlgItem(IDC_BUTTON2)->ShowWindow(SW_HIDE);
GetDlgItem(IDC_BUTTON3)->ShowWindow(SW_HIDE);
}
```
获取时间函数:
```
CString getNowtime() {
CTime tm; tm = CTime::GetCurrentTime();
CString tim;
tim = tm.Format("%Y年%m月%d日 %X");
return tim;
}
```
将消息更新到全局消息框中:
```
void CServerDlg::update(CString s)
{
UpdateData(TRUE);
s_page += getNowtime() + _T("\r\n") + s +_T("\r\n");
UpdateData(FALSE);
}
```
## 实验过程中遇到的问题及解决处理方法
1. 服务端点击关闭按钮后，再次点击启动发现不能使用同一端口:
setsockopt()函数实现端口复用
```
BOOL m=TRUE;
setsockopt() (serSocket, SOL_SOCKET, SO_REUSEADDR, (const char*)&(m), sizeof(BOOL));
```
2. 实现异步通信
服务端点击启动按钮响应函数内调用 StartFunc()线程:
```
StartFunc(){bind();listen();accept(); while(1){ recv();} close();return 0;}
```
客户端点击连接按钮响应函数内调用 RecvFunc()线程:
```
RecvFunc(){while(1){ recv();} return 0;}
```
3. 客户端不能多次连接，再一次就连不上:
服务端中，把 Accept()和 Recv()放到一个线程中 AcceptFunc();在 StartFunc()线程中循环调用线
程 AcceptFunc()。
```
StartFunc(){bind();listen(); while(1){ AcceptFunc();} close();return 0;}
AcceptFunc(){ accept(); while(1){ recv();} return 0;}
```
## 总结：
 +  对 socket 编程有了更深的理解，socket 利用三元组[IP,协议,端口]进行网络间通
信，服务端调用 listen()进入监听状态，通过 accept()来接收客户端请求；客户端
调用 connect()函数请求连接。双方通过调用 recv()和 send()来接收和发送数据。
 +  对于同步和异步通信的概念和应用有了更深的理解。实现双向异步通信，可以利用
多线程实现，通过线程来使得接收信息和发送信息功能不相关联，不必接收一步发
送一步。


