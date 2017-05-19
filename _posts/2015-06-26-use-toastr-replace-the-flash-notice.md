---
layout: post
title: '使用Toastr 取代 Flash notice'
date: 2015-06-26 23:15
tag: [web, rails]
---
application.js & applications.scss

加入 toastr.js & toastr.scss

-----

於application helper 中加入
```ruby
  def notice_message
    flash_messages = []
    flash.each do |type, message|
      type = 'success' if type == 'notice'
      type = 'error'   if type == 'alert'
      text = "<script>toastr.#{type}('#{message}');</script>"
      flash_messages << text.html_safe if message
    end
    flash_messages.join("\n").html_safe
  end
```

-----

於 appliacation layout中加入

<%= notice_message %>

!!注意!!需放置在Javascirpte_include_tag "application" 底下

