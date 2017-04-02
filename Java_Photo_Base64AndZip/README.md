### 图片转Base64并压缩

首先需要Apache下的两个jar包

		<!-- https://mvnrepository.com/artifact/commons-codec/commons-codec -->
		<dependency>
		    <groupId>commons-codec</groupId>
		    <artifactId>commons-codec</artifactId>
		    <version>1.10</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/commons-io/commons-io -->
		<dependency>
		    <groupId>commons-io</groupId>
		    <artifactId>commons-io</artifactId>
		    <version>2.4</version>
		</dependency>
```Java
	package com.silene.base64;
	
	import java.io.ByteArrayInputStream;
	import java.io.ByteArrayOutputStream;
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.IOException;
	import java.util.zip.GZIPInputStream;
	import java.util.zip.GZIPOutputStream;
	
	import org.apache.commons.codec.binary.Base64;
	import org.apache.commons.io.FileUtils;
	
	public class TestBase64Zip {
	
		public static void main(String[] args) {
			
			String base64 = base64("e:\\question.jpg");
			System.out.println("经过base64转码和压缩" + base64);
			
			decode(base64, "test.jpg", "e:\\");
		}
	
		/**
		 * 把经过压缩过的base64串解码解压并写入打磁盘中
		 * @param base64 压缩过的base64串
		 * @param fileName 文件名
		 * @param path 路径地址
		 */
		public static void decode(String base64, String fileName, String path) {
			//解码
			byte[] data = Base64.decodeBase64(base64);
			data = unGZip(data);
			writeFile(data, fileName, path);
		}
		
		/**
		 * 二进制文件写入文件
		 * @param data 二进制数据
		 * @param fileName 文件名
		 * @param path 路径地址
		 */
		public static void writeFile(byte[] data, String fileName, String path) {
	        try
	        {
	        	String url = path + "//" + fileName;
	            FileUtils.writeByteArrayToFile(new File(url), data);
	        }
	        catch (IOException e)
	        {
	            System.out.println("写文件出错" + e);
	        }
		}
		
		/**
		 * 解壓Gzip
		 * @param data
		 * @return
		 */
	    public static byte[] unGZip(byte[] data){
	        byte[] b = null;
	        try{
	            ByteArrayInputStream bis = new ByteArrayInputStream(data);
	            GZIPInputStream gzip = new GZIPInputStream(bis);
	            byte[] buf = new byte[1024];
	            int num = -1;
	            ByteArrayOutputStream baos = new ByteArrayOutputStream();
	            while ((num = gzip.read(buf, 0, buf.length)) != -1)
	            {
	                baos.write(buf, 0, num);
	            }
	            b = baos.toByteArray();
	            baos.flush();
	            baos.close();
	            gzip.close();
	            bis.close();
	        }
	        catch (Exception ex){
	            System.out.println("解压数据流出错！！" + ex);
	        }
	        return b;
	    }
	    
		/**
		 * 读取文件并压缩数据然后转Base64编码
		 * @param pathName 图片的绝对路径地址
		 * @return
		 */
		public static String base64(String pathName) {
			byte[] data = getPicData(pathName);
			if (data == null) {
				return null;
			}
			byte[] zipData = gZip(data);
			return Base64.encodeBase64String(zipData);
		}
	
		/**
		 * @description 获取图片的二进制数据
		 * @param pathName 图片的绝对路径地址
		 * @return
		 */
		public static byte[] getPicData(String pathName) {
			byte[] data = null;
			try {
				FileInputStream fi = new FileInputStream(pathName);
				int length = fi.available();
				data = new byte[length];
				fi.read(data);
				fi.close();
			} catch (Exception e) {
				System.out.println(e);
			}
			return data;
		}
	
		/***
		 * @description 压缩GZip
		 * @param data 要压缩的二进制数据
		 * @return
		 */
		public static byte[] gZip(byte[] data) {
			byte[] b = null;
			try {
				ByteArrayOutputStream bos = new ByteArrayOutputStream();
				GZIPOutputStream gzip = new GZIPOutputStream(bos);
				gzip.write(data);
				gzip.finish();
				gzip.close();
				b = bos.toByteArray();
				bos.close();
			} catch (Exception ex) {
				ex.printStackTrace();
			}
			return b;
		}
	
	}
```