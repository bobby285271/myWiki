---
title: Header Editor Config
description: 
published: true
date: 2021-03-28T11:51:23.795Z
tags: firefox, google, header-editor
editor: markdown
dateCreated: 2021-03-28T11:51:06.867Z
---

## Header Editor 插件配置

主要是在不使用国际互联网时填写 Google 的验证码（替换到 `recaptcha.net`）。


```
{
	"request": [
		{
			"enable": 1,
			"name": "GoogleAPIs",
			"ruleType": "redirect",
			"matchType": "regexp",
			"pattern": "^http(s?)://ajax\\.googleapis\\.com/(.*)",
			"exclude": "",
			"isFunction": 0,
			"action": "redirect",
			"to": "https://gapis.geekzu.org/ajax/$2",
			"id": 1,
			"_reg": {}
		},
		{
			"enable": 1,
			"name": "reCAPTCHA",
			"ruleType": "redirect",
			"matchType": "regexp",
			"pattern": "^http(s?)://www\\.google\\.com/recaptcha/(.*)",
			"exclude": "",
			"isFunction": 0,
			"action": "redirect",
			"to": "https://recaptcha.net/recaptcha/$2",
			"id": 2,
			"_reg": {}
		}
	],
	"sendHeader": [],
	"receiveHeader": [
		{
			"enable": 1,
			"name": "CSP allows redirections",
			"ruleType": "modifyReceiveHeader",
			"matchType": "all",
			"pattern": "",
			"exclude": "",
			"isFunction": 1,
			"code": "let rt = detail.type;\nif (rt === 'script' || rt === 'stylesheet' || rt === 'main_frame' || rt === 'sub_frame') {\n  for (let i in val) {\n    if (val[i].name.toLowerCase() === 'content-security-policy') {\n      let s = val[i].value;\n      s = s.replace(/googleapis\\.com/g, '$& https://gapis.geekzu.org');\n      s = s.replace(/google\\.com/g, '$& https://recaptcha.net');\n      s = s.replace(/gstatic\\.com/g, '$& https://*.gstatic.cn');\n      val[i].value = s;\n      break;\n    }\n  }\n}",
			"id": 1
		}
	]
}
```