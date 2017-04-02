### JAVA URLConnection在线文件的下载

#
	/*
	 * 根据URL在线文件的下载
	 * 
	 * @param	文件的全路径地址
	 * @param   文件存放的本地路径的地址
	 * 	
	 */
	public static void downloadNet(String source, String target) {
		// 下载网络文件
		int bytesum = 0;
		int byteread = 0;

		try {
			
			URL url = new URL(source);
			URLConnection conn = url.openConnection();
			InputStream inStream = conn.getInputStream();
			target += "/" + source.substring(source.lastIndexOf("/") + 1);
			FileOutputStream fs = new FileOutputStream(target);

			byte[] buffer = new byte[1204];
			while ((byteread = inStream.read(buffer)) != -1) {
				bytesum += byteread;
				fs.write(buffer, 0, byteread);
			}
			System.out.println("下载 " + target + " 完成!");
			fs.close();
			inStream.close();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
#

测试

	public static void main(String[] args) {
		// 根据网页地址和网页的编码集 获取网页的内容
		String urls = "http://img3x5.ddimg.cn/32/11/60616445-1_b_17.jpg--http://img3x4.ddimg.cn/66/31/20689284-1_b_3.jpg--http://img3x4.ddimg.cn/39/36/60592494-1_b_532.jpg--http://img3x6.ddimg.cn/58/30/1001998066-1_b_0.jpg--http://img3x4.ddimg.cn/31/10/60616444-1_b_13.jpg--http://img3x5.ddimg.cn/34/33/1255478335-1_b_2.jpg--http://img3x8.ddimg.cn/36/33/60625458-1_b_16.jpg--http://img3x3.ddimg.cn/84/23/60611943-1_b_25.jpg--http://img3x5.ddimg.cn/59/22/60630035-1_b_10.jpg--http://img3x6.ddimg.cn/48/12/60612006-1_b_15.jpg--http://img3x0.ddimg.cn/38/35/60625460-1_b_12.jpg--http://img3x2.ddimg.cn/25/25/60069562-1_b_0.jpg--http://img3x5.ddimg.cn/40/0/60629125-1_b_4.jpg--http://img3x9.ddimg.cn/37/34/60625459-1_b_16.jpg--http://img3x8.ddimg.cn/49/20/1007843908-1_b_12.jpg--http://img3x7.ddimg.cn/49/1/1031039707-1_b_7.jpg--http://img3x3.ddimg.cn/0/11/1170375723-1_b_8.jpg--http://img3x5.ddimg.cn/32/34/1416676865-1_b_2.jpg--http://img3x3.ddimg.cn/80/29/1209617423-1_b_2.jpg--http://img3x1.ddimg.cn/66/25/20368821-1_b_2.jpg--http://img3x3.ddimg.cn/29/15/60266873-1_b_0.jpg--http://img3x1.ddimg.cn/53/17/60612011-1_b_28.jpg--http://img3x0.ddimg.cn/65/24/20368820-1_b_2.jpg--http://img3x8.ddimg.cn/47/13/1177618808-1_b_5.jpg--http://img3x0.ddimg.cn/30/21/1351538130-1_b_4.jpg--http://img3x9.ddimg.cn/58/26/20668189-1_b_1.jpg--http://img3x4.ddimg.cn/39/36/60629124-1_b_2.jpg--http://img3x5.ddimg.cn/6/35/60601965-1_b_1.jpg--http://img3x6.ddimg.cn/83/13/1125073406-1_b_0.jpg--http://img3x8.ddimg.cn/24/11/1079463948-1_b_2.jpg--http://img3x8.ddimg.cn/9/35/1162567008-1_b_6.jpg--http://img3x7.ddimg.cn/51/21/1320107037-1_b_2.jpg--http://img3x5.ddimg.cn/47/33/1252895735-1_b_3.jpg--http://img3x5.ddimg.cn/2/29/1199193635-1_b_3.jpg--http://img3x2.ddimg.cn/27/7/1340670402-1_b_5.jpg--http://img3x6.ddimg.cn/87/6/1244157936-1_b_3.jpg--http://img3x6.ddimg.cn/80/26/1269352736-1_b_2.jpg--http://img3x1.ddimg.cn/39/36/60625461-1_b_20.jpg--http://img3x8.ddimg.cn/10/30/1017023248-1_b_3.jpg--http://img3x0.ddimg.cn/8/35/1316786930-1_b_2.jpg--http://img3x2.ddimg.cn/84/9/1340676102-1_b_0.jpg--http://img3x8.ddimg.cn/29/36/1176191408-1_b_8.jpg--http://img3x5.ddimg.cn/2/28/1344684035-1_b_2.jpg--http://img3x7.ddimg.cn/15/17/1176358407-1_b_7.jpg--http://img3x5.ddimg.cn/58/6/60083455-1_b_3.jpg--http://img3x5.ddimg.cn/30/14/60577635-1_b_1.jpg--http://img3x8.ddimg.cn/33/17/60577638-1_b_1.jpg--http://img3x7.ddimg.cn/5/35/60572957-1_b_0.jpg--";
		String[] url = urls.split("--");
		for (int i = 0; i < url.length; i++) {
			downloadNet(url[i], "f:\\test");
		}
	}