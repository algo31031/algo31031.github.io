---
layout: post
title: "使用jquery-fileupload-rails在IE下进行AJAX文件上传失败的解决办法"
date: 2012-10-19 09:29:55 +0800
comments: true
categories: rails
---
症状：提示“illegal character”错误,上传失败

原因：Chrome, Firefox, Safari等浏览器本身支持XHR形式的文件上传，但IE不支持，于是Jquery-Fileupload自动在页面添加隐藏的iframe,于是在上传文件时造成 CSRF token 无法传递。

解决：

    <script>
          $('#fileupload').fileupload();
    </script>
    <!--[if IE]>
      <script>
        $('#fileupload').bind('fileuploadsend', function(event, data) {
          auth_token = $('meta[name="csrf-token"]').attr('content');
          data.url = data.url + '?authenticity_token=' + encodeURIComponent(auth_token); 
          $.blueimpUI.fileupload.prototype.options.send.call(this, event, data);
        });
      </script>
    <![endif]-->

 同时在controller里：
 
    format.html {                                         
      render :json => [@episode.to_jq_upload].to_json,
      :content_type => 'text/plain',
      :layout => false
    }