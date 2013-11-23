---
layout: page
title: Index
tagline:
---
{% include JB/setup %}

# Personal Info
{% highlight javascript %}
{
    'Name': {
                'Chinese': '屈啸',
                'English': 'Xavier'
            }
    'Gender':   'Male',
    'Company':  'Baidu Inc.',
    'Job':      'Software Engineer',
    'Hobbies':  ['Sports', 'Photography']
}
{% endhighlight %}


# 
# Post List
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


