---
title: 用CSS画一些图形
date: 2024-03-15 10:04:34
tags: CSS
categories: CSS
description: 用CSS画一些有趣的图形
---

## 半圆

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>draw</title>
	<style type="text/css">
		#semi-circle{
			width: 200px;
			height: 100px;
			background-color: red;
			border-radius:100px 100px 0 0;/* 左上、右上、右下、左下 */
		}
	</style>
</head>
<body>
	<div id="semi-circle"></div>
</body>
</html>
```

