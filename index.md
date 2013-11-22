---
layout: page
title: Index
tagline:
---
{% include JB/setup %}

# Personal Info
{% highlight javascript %}
{
    'name':     'Qu Xiao',
    'gender':   'Male',
    'company':  'Baidu Inc.',
    'title':    'Software Engineer'
}
{% endhighlight %}


# 
# Post List
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


