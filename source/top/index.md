---
title: 阅读榜
comments: false
---
<blockquote class="blockquote-center">琅琊榜首，江左梅郎</blockquote>
<div id="top"></div>
<script src="https://cdn1.lncld.net/static/js/av-core-mini-0.6.4.js"></script>
<script>AV.initialize("XMCuiFsYYsbUlDih3XM5LI6F-9Nh9j0Va", "T6DpDouJtaUHt3WMcHqcCrxD");</script>
<script type="text/javascript">
  var time=0
  var title=""
  var url=""
  var query = new AV.Query('Counter');
  query.notEqualTo('id',0);
  query.descending('time');
  query.limit(1000);
  query.find().then(function (todo) {
    for (var i=0;i<1000;i++){
      var result=todo[i].attributes;
      time=result.time;
      title=result.title;
      url=result.url;
      // var content="<a href='"+"https://hoxis.github.io"+url+"'>"+title+"</a>"+"<br>"+"<font color='#fff'>"+"阅读次数："+time+"</font>"+"<br><br>";
      var content="<p>"+"<font color='#e20404'>"+"【热度"+time+"℃】"+"："+"<a href='"+"https://xiaoxiaoxiaoxiaolin.github.io"+url+"'>"+title+"</a>"+"</p>";
      document.getElementById("top").innerHTML+=content
    }
  }, function (error) {
    console.log("error");
  });
</script>