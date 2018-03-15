---
title: test
date: 2018-03-14 15:06:11
tags:
	- Test
	- Test comment with gitment
categories:
	- Guid	
---

<div id="container"></div>
<link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
<script>
var gitment = new Gitment({
  id: '页面 ID', // 可选。默认为 location.href
  owner: 'yorkLiu',
  repo: 'https://github.com/yorkLiu/yorkliu.github.io',
  oauth: {
    client_id: '5de7e66d11e669bdea4b',
    client_secret: 'e89611df6231762c6ceacf98c6b0378164569240',
  },
})
gitment.render('container')
</script>