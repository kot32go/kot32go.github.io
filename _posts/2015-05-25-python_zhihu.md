---
layout: post
title: 使用Python 登录知乎并抓取内容
description: "Python 抓取知乎"
tags: [python]
---


之前玩过一阵子 Python，现在复习一下，准备慢慢来做个小玩意儿。
注：要使用新注册的小号才能不被验证码。


	import requests
	from bs4 import BeautifulSoup

	login_url = "http://www.zhihu.com/login"
	headers = {'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64)\
	AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.118 Safari/537.36',}

	seesion=requests.session();

	my_post = dict(email='xxx@xxx.com',
	               _xsrf=BeautifulSoup(seesion.get('http://www.zhihu.com/').content).find(type='hidden')['value'],
	               password='xxx', login='登录', rememberme='y')

	r = seesion.post(login_url, data = my_post, headers = headers)
	html = r.text

	print html
	
