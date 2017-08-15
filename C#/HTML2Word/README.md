### HTML转出到Word中 ###
通过系统剪切板把HTML复制到Word中，并且保留原来的格式

```C#

    class HTMLUtils
    {
        // 将HTML代码复制到Windows剪贴板，并保证中
        [DllImport("user32.dll")]
        static extern bool OpenClipboard(IntPtr hWndNewOwner);
        [DllImport("user32.dll")]
        static extern bool EmptyClipboard();
        [DllImport("user32.dll")]
        static extern IntPtr SetClipboardData(uint uFormat, IntPtr hMem);
        [DllImport("user32.dll")]
        static extern bool CloseClipboard();
        [DllImport("user32.dll", SetLastError = true)]
        static extern uint RegisterClipboardFormatA(string lpszFormat);

        [DllImport("kernel32.dll", SetLastError = true)]
        static extern IntPtr GlobalLock(IntPtr hMem);
        [DllImport("kernel32.dll", SetLastError = true)]
        static extern uint GlobalSize(IntPtr hMem);
        [DllImport("kernel32.dll", SetLastError = true)]
        static extern IntPtr GlobalUnlock(IntPtr hMem);

        /// <summary>
        /// copy the html into clipboard
        /// </summary>
        /// <param name="html"></param>
        /// <returns></returns>
        static public bool CopyHTMLToClipboard(string html)
        {
            uint CF_HTML = RegisterClipboardFormatA("HTML Format");
            bool bResult = false;
            if (OpenClipboard(IntPtr.Zero))
            {
                if (EmptyClipboard())
                {
                    byte[] bs = System.Text.Encoding.UTF8.GetBytes(html);

                    int size = Marshal.SizeOf(typeof(byte)) * bs.Length;

                    IntPtr ptr = Marshal.AllocHGlobal(size);
                    Marshal.Copy(bs, 0, ptr, bs.Length);

                    IntPtr hRes = SetClipboardData(CF_HTML, ptr);
                    CloseClipboard();
                }
            }
            return bResult;
        }

        //将HTML代码按照Windows剪贴板格进行格式化
        public static string HtmlClipboardData(string html)
        {
            StringBuilder sb = new StringBuilder();
            Encoding encoding = Encoding.UTF8; //Encoding.GetEncoding(936);
            string Header = @"Version: 1.0
                            StartHTML: {0:000000}
                            EndHTML: {1:000000}
                            StartFragment: {2:000000}
                            EndFragment: {3:000000}
                            ";
            string HtmlPrefix = @"<!DOCTYPE HTML PUBLIC ""-//W3C//DTD HTML 4.0 Transitional//EN"">
                            <html>
                            <head>
                            <meta http-equiv=Content-Type content=""text/html; charset={0}"">
                            </head>
                            <body>
                            <!--StartFragment-->
                            ";
            HtmlPrefix = string.Format(HtmlPrefix, "gb2312");

            string HtmlSuffix = @"<!--EndFragment--></body></html>";

            // Get lengths of chunks
            int HeaderLength = encoding.GetByteCount(Header);
            HeaderLength -= 16; // extra formatting characters {0:000000}
            int PrefixLength = encoding.GetByteCount(HtmlPrefix);
            int HtmlLength = encoding.GetByteCount(html);
            int SuffixLength = encoding.GetByteCount(HtmlSuffix);

            // Determine locations of chunks
            int StartHtml = HeaderLength;
            int StartFragment = StartHtml + PrefixLength;
            int EndFragment = StartFragment + HtmlLength;
            int EndHtml = EndFragment + SuffixLength;

            // Build the data
            sb.AppendFormat(Header, StartHtml, EndHtml, StartFragment, EndFragment);
            sb.Append(HtmlPrefix);
            sb.Append(html);
            sb.Append(HtmlSuffix);

            //Console.WriteLine(sb.ToString());
            return sb.ToString();
        }
    }
```

调用：

```C#
	string html = "<p><input type='text' name='test'>abc</p>";
	html = HTMLUtils.HtmlClipboardData(html);
	HTMLUtils.CopyHTMLToClipboard(html);
	//then you can paste into word using WdPasteDataType.wdPasteHTML parameter
    object dataType = Word.WdPasteDataType.wdPasteHTML;
    newapp.Selection.PasteSpecial(ref nothing, ref nothing, ref nothing, ref nothing, ref dataType, ref nothing, ref nothing); ;
```
具体的实现请看[https://github.com/scalad/MathML2MathTypeEquation](https://github.com/scalad/MathML2MathTypeEquation)

