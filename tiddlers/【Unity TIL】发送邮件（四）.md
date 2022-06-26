```c#
void SendEmail ()
{
    string email = "me@example.com";
    string subject = MyEscapeURL("My Subject");
    string body = MyEscapeURL("My Body\r\nFull of non-escaped chars");
    Application.OpenURL("mailto:" + email + "?subject=" + subject + "&body=" + body);
}
string MyEscapeURL (string url)
{
    return WWW.EscapeURL(url).Replace("+","%20");
}
```

## 参考链接
[Application.OpenURL for email on mobile - Unity Answers](https://answers.unity.com/questions/61669/applicationopenurl-for-email-on-mobile.html)


