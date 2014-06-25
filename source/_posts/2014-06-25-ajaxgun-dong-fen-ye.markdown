---
layout: post
title: "ajax滚动分页"
date: 2014-06-25 16:14:10 +0800
comments: true
categories: rails
---
还是记录一下吧, 不然时间一久又会像之前一样把细节忘掉

### 需求
聊天记录只显示最近50条, 若要查看历史记录, 可将记录向上滚动. 至顶部后自动加载上50条. 

### 技术要点
页面加载完之后, 设置容器scrollTop将其定位到最底部

监测div的scroll事件, 判断容器滚动条到顶部时(`$("container").scrollTop == 0`), 发ajax请求.  
进行中的风车gif图, 可在ajax onCreate和onSuccess里控制显示/隐藏.

controller按照常规分页查询处理即可.

response时, 要分别保存新纪录插入容器前后, 容器的scrollHeight值, 存入局部变量,  
待新记录在容器里render完后, 修改容器的scrollTop值为2次scrollHeight之差, 
从而使容器滚动位置在页面看来保持不变

### 代码片段
rails2.3项目, js框架使用的prototype...

``` erb view层js
document.observe("dom:loaded", function() {
  $("chat_list").scrollTop = $("chat_list").scrollHeight;    

  $("chat_list").observe("scroll", function(){
    if(this.scrollTop == 0) {
      var page = $("page").value;
      new Ajax.Request('/mypage/chat_team/load_history_logs/<%= @chat_team.id %>', 
        { method: 'get', 
          parameters: { page: page },
          onCreate: function(){
            $("wait").show();
          },
          onSuccess: function(){
            $("wait").hide();
          }
        }
      );      
    }
  });    
});
```
``` ruby controller
def load_history_logs
  @page = params[:page].to_i + 1
  @chat_logs = @chat_team.chat_logs.paginate(
                    :all,
                    :page => @page, 
                    :per_page => ChatLog::RECENT_LOGS_SIZE, 
                    :order => "created_at DESC").reverse
  
  render :update do |page|
    page << "$('page').value = #{@page};"
    page << "var oldHeight = $('chat_list').scrollHeight;"
    page.insert_html(:top, "chat_list", render(:partial => "/mypage/chat_team/chat_log", :collection => @chat_logs))
    page << "var newHeight = $('chat_list').scrollHeight;"
    page << "$('chat_list').scrollTop = newHeight - oldHeight;"
  end
end
```