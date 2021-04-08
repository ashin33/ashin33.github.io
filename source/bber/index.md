---
title: 哔哔
date: 2021-04-08 11:15:45
aside: false
comments: false
---

<div id='speak'></speak>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/ispeak-bber/ispeak-bber.min.js" charset="utf-8" ></script>
<script>
ispeakBber
    .init({
      el: '#speak', // 容器选择器
      name: 'alon', // 显示的昵称
      envId: 'hexo-7ga4d1288683994e', // 环境id
      region: 'ap-shanghai', // 腾讯云地址，默认为上海
      limit: 10, // 每次加载的条数，默认为5
      avatar: '/img/namikaze_minato_4.jpeg',
      fromColor:'rgb(245, 150, 170)', // 下方标签背景颜色 默认 rgb(245, 150, 170)
      loadingImg: 'https://7.dusays.com/2021/03/04/d2d5e983e2961.gif', // 自定义loading的图片，示例值为默认值
      dbName:'talks' // 数据的名称，默认talks，避免有人的命名不是这个，所以加入此配置字段。
    })
    .then(function() {
      // 哔哔加载完成后的回调函数，你可以写你自己的功能
      console.log('哔哔 加载完成')
    })
</script>
