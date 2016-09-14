###
使用时加入jsoup的包	
#
	<!-- https://mvnrepository.com/artifact/org.jsoup/jsoup -->
	<dependency>
	    <groupId>org.jsoup</groupId>
	    <artifactId>jsoup</artifactId>
	    <version>1.8.3</version>
	</dependency>
#
#

	import java.io.BufferedReader;
	import java.io.File;
	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.InputStreamReader;
	import java.net.HttpURLConnection;
	import java.net.URL;
	import java.net.URLConnection;
	
	import org.jsoup.Jsoup;
	import org.jsoup.nodes.Document;
	import org.jsoup.nodes.Element;
	import org.jsoup.select.Elements;
	
	/**
	 * Java开发图片批量采集
	 * @author cx112
	 * @version v1.0
	 */
	
	public class GrabPicture {
		/**
		 * 根据网站的地址和页面的编码集来获取网页的源代码
		 * 
		 * @author cx112
		 * @param url
		 *            网址路径
		 * @param encoding
		 *            编码集
		 * @return String 网页的源代码
		 */
		public static String gethtmlResourceByURL(String url, String encoding) {
			// 用于存储网页源代码
			StringBuffer buf = new StringBuffer();
			URL urlObj = null;
			URLConnection uc = null;
			InputStreamReader isr = null;
			BufferedReader buffer = null;
			try {
				// 建立网络连接
				urlObj = new URL(url);
				// 打开网络连接
				uc = urlObj.openConnection();
				// 将连接网络的输入流转换
				isr = new InputStreamReader(uc.getInputStream(), encoding);
				// 建立缓冲写入流
				buffer = new BufferedReader(isr);
				String line = null;
				while ((line = buffer.readLine()) != null) {
					buf.append(line + "\n");// 一行一行的追加代码
				}
			} catch (Exception e) {
				System.out.println("test");
				e.printStackTrace();
			} finally {
				try {
					if (isr != null) {
						isr.close();
					}
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			return buf.toString();
		}
	
		/**
		 * 根据图片的网络地址，下载图片带本地服务器
		 * 
		 * @author cx112
		 * @param filePath
		 *            文件保存的路径
		 * @param imgURL
		 *            图片的网络地址
		 */
		public static void DownImages(String filePath, String imgURL) {
			String fileName = imgURL.substring(imgURL.lastIndexOf("/"));
	
			try {
				// 创建文件目录
				File files = new File(filePath);
				// 判断是否存在文件夹
				if (!files.exists()) {
					files.mkdirs();
				}
				// 获取下载地址
				URL url = new URL(imgURL);
				// 连接网络地址
				HttpURLConnection huc = (HttpURLConnection) url.openConnection();
				// 获取连接的输出流
				InputStream is = huc.getInputStream();
				// 创建文件
				File file = new File(filePath + fileName);
				// 创建输入流，写入文件
				FileOutputStream out = null;
				if (file.getName().endsWith("jpg") || file.getName().endsWith("png") 
						|| file.getName().endsWith("jpeg") || file.getName().endsWith("jpg") ){				
					 out = new FileOutputStream(file);				 
					 int i = 0;
					 while ((i = is.read()) != -1) {
						 out.write(i);
					 }
					 is.close();
					 out.close();
				}
	
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	
		public static void getImage(String url, String encoding,String path){
			String htmlResouce = gethtmlResourceByURL(url, encoding);		
			// 解析网页源代码
			Document document = Jsoup.parse(htmlResouce);
			// 获取所以图片的地址<img src="" alt= "" width= "" height=""/>
			Elements elements = document.getElementsByTag("img");
			for (Element element : elements) {
				String imgSrc = element.attr("src");
				if (!"".equals(imgSrc) && imgSrc.startsWith("http://")) {
					System.out.println("下载图片的地址===" + imgSrc);
					DownImages(path, imgSrc);
				}
			}
		}
		
		public static void main(String[] args) {
			// 根据网页地址和网页的编码集 获取网页的内容
			String url = "http://www.tripadvisor.cn";
			String encoding = "gb2312";
			getImage(url, encoding, "e:\\test");
		}
	
	}
#