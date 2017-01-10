---
layout: post
title:  "下载文件名乱码问题的解决示例"
desc: "下载文件名乱码问题的解决示例"
keywords: "下载文件名乱码问题的解决示例,下载,乱码,java,kinglyjn"
date: 2012-11-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

代码示例：

```java
import java.io.File;
import java.io.IOException;
import java.io.OutputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.commons.io.FileUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
@RequestMapping("/test")
public class Test {

	@RequestMapping(value = "getpdfcredit")
	public void getPDFCreditReport(@RequestParam("cookies") String cookies,
			@RequestParam("imageCode") String imageCode,
			HttpServletRequest request, HttpServletResponse response) throws IOException {
		//
		File file = new File("xxx.pdf");
		//
		String charset = request.getCharacterEncoding();
		response.setHeader("Content-Disposition", 
			"attachement;filename="
			+new String(file.getName().getBytes(charset), "ISO8859-1"));
		response.setContentType("application/octet-stream");
		response.setCharacterEncoding("UTF-8");
		OutputStream os = response.getOutputStream();
		FileUtils.copyFile(file, os);
		os.close();
	}
}
```
