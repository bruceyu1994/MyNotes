二维码资产管理项目
===
## 1. C#模拟登陆

* CookieContainer: 装载返回的cookies的容器
* C#示例代码,用户登陆指定的网站：

```csharp
HttpWebRequest request = null;  //提供 WebRequest 类的 HTTP 特定的实现。
HttpWebResponse response = null;    //提供 WebResponse 类的 HTTP 特定的实现

try 
{
    //提交的信息，包括用户名和密码等信息
    string postdata = "uid=" + account + "&pwd=" + password;

    //抽象类的静态方法可直接调用,初始化新的WebRequest实例化web访问类
    request = (HttpWebRequest)WebRequest.Create(loginUrl);

    //request.Credentials:获取或设置请求的身份验证信息;
    //CredentialCache.DefaultCredentials:获取应用程序的系统凭据,
    //对于 ASP.NET应用程序，默认凭据是已登录的用户或正被模拟的用户的用户凭据。
    request.Credentials = CredentialCache.DefaultCredentials;

    request.Method = "POST";//数据提交方式为POST

    //request.ContentType:获取或设置传入请求的 MIME 内容类型。
    request.ContentType = "application/x-www-form-urlencoded";    //模拟头 

    //request.AllowAutoRedirect：获取或设置一个值，该值指示请求是否应跟随重定向响应；
    //如果请求应自动跟随 Internet 资源的重定向响应，则为 true，否则为 false。默认值为 true。
    request.AllowAutoRedirect = false;   // 不用需自动跳转

    //必须设置CookieContainer存储请求返回的Cookies
    if (CookieContainer != null)
    {
        request.CookieContainer = CookieContainer;
    }
    else
    {
        request.CookieContainer = new CookieContainer();
        CookieContainer = request.CookieContainer;
    }

    //保持TCP连接
    request.KeepAlive = true;

    //提交请求  
    byte[] postdatabytes = Encoding.UTF8.GetBytes(postdata); //对指定字符集进行编码
    request.ContentLength = postdatabytes.Length;

    Stream stream;
    stream = request.GetRequestStream();  //返回一个数据流为HttpWebRequest发送数据

    //设置POST 数据，向当前流中写入postdatabytes字节序列，其中0为postdatabytes从0开始的字节偏移量
    //postdatabytes.Length要写入当前流的字节数。
    stream.Write(postdatabytes, 0, postdatabytes.Length); 
    stream.Close();

    //接收响应
    response = (HttpWebResponse)request.GetResponse(); //返回对 Internet 请求的响应。

    //获取包含与特定 URI（request.RequestUri） 关联的 Cookie 实例的 CookieCollection。
    response.Cookies = request.CookieContainer.GetCookies(request.RequestUri);

    CookieCollection cook = response.Cookies;
    //返回一个 HTTP Cookie 标头，其中包含表示由分号分隔的与特定 URI 关联的Cookie 实例的字符串。
    string strcookieheader = request.CookieContainer.GetCookieHeader(request.RequestUri);
    CookiesString = strcookieheader;

    //取下一次GET跳转地址  
    //实现一个 TextReader，使其以gb2312编码从字节流中读取字符。
    StreamReader sr = new StreamReader(response.GetResponseStream(), Encoding.GetEncoding("gb2312"));

    string content = sr.ReadToEnd(); //读取来自流的当前位置到结尾的所有字符。
    sr.Close(); //关闭 StreamReader 对象和基础流，并释放与读取器关联的所有系统资源

    request.Abort();    //取消对 Internet 资源的请求。
    response.Close();   //Close 方法关闭响应流，并通过其他请求将释放与重复使用的资源的连接。

    //依据登陆成功后返回的Page信息，求出下次请求的url
    //每个网站登陆后加载的Url和顺序不尽相同，以下两步需根据实际情况做特殊处理，从而得到下次请求的URL
    string[] substr = content.Split(new char[] { '"' });
    NextRequestUrl = "http://zcgl.wzu.edu.cn/plat/" + substr[1];   
}
catch (WebException ex)
{
    MessageBox.Show(string.Format("登陆时出错，详细信息：{0}", ex.Message));
}
```

补充：
form的enctype属性为编码方式，分两种：  

1. application/x-www-form-urlencoded，默认值。action为get时，把form数据转换成一个字串（name1=value1&name2=value2...），append到url后面，用?分割；当action为post时，浏览器把form数据封装到http body中，然后发送到server。  

2. multipart/form-data，type=file时使用，浏览器会把整个表单以控件为单位分割，并为每个部分加上Content-Disposition(form-data或者file),Content-Type(默认为text/plain),name(控件name)等信息，并加上分割符(boundary)。

* 获取指定页面的内容,C#示例代码:
```csharp
HttpWebRequest request = null;
HttpWebResponse response = null;
try
{
    request = (HttpWebRequest)WebRequest.Create(NextRequestUrl);
    request.Credentials = CredentialCache.DefaultCredentials;
    request.Method = "GET";
    request.KeepAlive = true;
    request.Headers.Add("Cookie:" + CookiesString);
    request.CookieContainer = CookieContainer;
    request.AllowAutoRedirect = false;
    response = (HttpWebResponse)request.GetResponse();

    //设置cookie  
    CookiesString = request.CookieContainer.GetCookieHeader(request.RequestUri);
    //取再次跳转链接  
    StreamReader sr = new StreamReader(response.GetResponseStream(), Encoding.GetEncoding("gb2312"));
    string ss = sr.ReadToEnd();
    sr.Close();
    request.Abort();
    response.Close();

    //依据登陆成功后返回的Page信息，求出下次请求的url
    //每个网站登陆后加载的Url和顺序不尽相同，以下两步需根据实际情况做特殊处理，从而得到下次请求的URL
    //string[] substr = ss.Split(new char[] { '"' });
    //NextRequestUrl = substr[1];
    //NextRequestUrl = "http://zcgl.wzu.edu.cn/plat/" + substr[1];
    ResultHtml = ss;
}
catch (WebException ex)
{
    MessageBox.Show(string.Format("获取页面HTML信息出错，详细信息：{0}", ex.Message));
}
```

## 2. 项目规划

1. 通过仪器编号获取获网页信息  
>模拟登陆（通过URL,详见上模拟登陆）

2. 通过正则匹配需要的信息
```csharp
private static string GetValue(string pattern)
{
    Match m = Regex.Match(WebAutoLogin.ResultHtml, pattern);
    int start = m.ToString().LastIndexOf("value=\"");
    int end = m.ToString().LastIndexOf("\"");
    requesteturn m.ToString().Substring(start + 7, end - (start + 7));
}
```

3. 通过zxing库将采集的信息生成二维码和条形码
```csharp
//C#代码
using com.google.zxing;
using ByteMatrix = com.google.zxing.common.ByteMatrix;
/// <summary>
/// 生成二维码
/// </summary>
public void CreateQRCode()
{
    //构造二维码内容
    string content =
        Num_columnHeader.Text + "：" + Info_listView.Items[1].SubItems[0].Text + "\n" +
        Name_columnHeader.Text + "：" + Info_listView.Items[1].SubItems[1].Text + "\n" +
        Person_columnHeader.Text + "：" + Info_listView.Items[1].SubItems[2].Text + "\n" +
        Place_columnHeader.Text + "：" + Info_listView.Items[1].SubItems[3].Text + "\n" +
        Type_columnHeader.Text + "：" + Info_listView.Items[1].SubItems[4].Text + "\n" +
        Univalence_columnHeader.Text + "：" + Info_listView.Items[1].SubItems[5].Text + "\n" +
        Date_columnHeader.Text + "：" + Info_listView.Items[1].SubItems[6].Text;

    ByteMatrix byteMatrix = new MultiFormatWriter().encode(content, BarcodeFormat.QR_CODE, 200, 200);
    Bitmap bitmap = toBitmap(byteMatrix);
    QRCode_pictureBox.Image = bitmap;
}

/// <summary>
/// 生成条形码
/// </summary>
private void CreateBarCode(string content)
{
    Regex rg = new Regex(@"^\d{8}$");
    if(rg.IsMatch(content))
    {
        ByteMatrix byteMatrix = new MultiFormatWriter().encode(content, BarcodeFormat.EAN_8, 200, 30);
        Bitmap bitmap = toBitmap(byteMatrix);
        BarCodeList.Add(bitmap);
        QRCode_pictureBox.Image = bitmap;
    }
    else
    {
        MessageBox.Show("非数字，无法生成条形码");
    }
}
        
/// <summary>
/// 生成位图
/// </summary>
/// <param name="byteMatrix"></param>
/// <returns></returns>
private Bitmap toBitmap(ByteMatrix byteMatrix)
{
    int width = byteMatrix.Width;
    int height = byteMatrix.Height;
    Bitmap bitmap = new Bitmap(width, height, System.Drawing.Imaging.PixelFormat.Format32bppArgb);
    for(int i=0; i<width; i++)
    {
        for(int j=0; j<height; j++)
        {
            bitmap.SetPixel(i, j, byteMatrix.get_Renamed(i, j) != -1 ? ColorTranslator.FromHtml("#436EEE") : ColorTranslator.FromHtml("#C7C7C7"));
        }
    }
    return bitmap;
}
```

4. 通过二维码扫描软件获取信息（建议用微信，QQ只能识别链接，无法识别文本内容）

5. 显示信息列表(listView)
```csharp
//向listView中添加内容
ListViewItem lvi = new ListViewItem(str0);
lvi.SubItems.Add(str1);
Info_listView.Items.Add(lvi);

//读取listView列标题内容
string str = columnHeader.Text;

//读取listView中的index1行，index2+1列的内容
string str = Info_listView.Items[index1].SubItems[index2].Text;
```

6. 删除listView列表项
```csharp
//删除一条记录
if (Info_listView.Items.Count > 0)
{
    Info_listView.Items.Remove(Info_listView.FocusedItem);
}
//删除多条记录
if (Info_listView.Items.Count > 0)
{
    Info_listView.MultiSelect = true;
    foreach (ListViewItem item in Info_listView.Items)
    {
        if (item.Selected)
            item.Remove();
    }
}
//清空
if (Info_listView.Items.Count > 0)
{
    Info_listView.Items.Clear();
}
```

7. listView显示二维码信息
>listView有5种视图:LargeIcon, SmallIcon, List, Details, Tile
```csharp
//listView显示图片,视图为LargeIcon
private void showCode()
{
    if (QRCodeList.Count > 0)
    {
        //QRCode_listView属性设置
        QRCode_listView.View = View.LargeIcon;
        QRCode_listView.LargeImageList = QRCode_imageList;
        QRCode_listView.BeginUpdate();

        QRCode_listView.LargeImageList.ImageSize = new System.Drawing.Size(200, 200);

        for (int i = 0; i < QRCodeList.Count; i++)
        {
            QRCode_imageList.Images.Add(QRCodeList[i]);
            ListViewItem lvi = new ListViewItem(Info_listView.Items[i].SubItems[0].Text);
            QRCode_listView.Items.Add(lvi);
            QRCode_listView.Items[i].ImageIndex = QRCode_imageList.Images.Count - 1 ;
        }

        QRCode_listView.EndUpdate();
    }
}
```

8. 保存二维码图片
```csharp
FolderBrowserDialog folderDlg = new FolderBrowserDialog();
folderDlg.Description = "请选择文件夹";

if (folderDlg.ShowDialog() == DialogResult.OK)
{
    string path = folderDlg.SelectedPath;
    for (int i = 0; i < QRCodeList.Count; i++)
    {
        QRCodeList[i].Save(path + @"\" + Info_listView.Items[i].SubItems[0].Text + ".bmp", System.Drawing.Imaging.ImageFormat.Bmp);
    }
}
```

9. 打印及预览二维码(一个或多个)


10. [C#将图片保存到资源文件并调用](http://blog.csdn.net/mengdong_zy/article/details/8971154)



>PS:
1. 有想法很重要
2. 做任何事情时，千万别去想它以后会不会给你带来什么，不然你又会陷入迷茫，无从下手





