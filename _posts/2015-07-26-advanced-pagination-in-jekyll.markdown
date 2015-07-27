---
title:  "Advanced Pagination in Jekyll"
date:   2015-07-20 23:41:23
categories: Jekyll Pagination Liquid
---
[Jekyll][http://jekyllrb.com/] provides a powerful framework for creating a statically generated blog. Some of the features like templating and markdown processing are fairly straightforward, but creating a functioning paginator for your blog can be challenging.

The first steps to setup a paginator on your site can be found on the [pagination page][http://jekyllrb.com/docs/pagination/] on the Jekyll website. Lets review these steps briefly before moving onto a more advanced implementation.

The first step is to enable pagination within your _config.yml. This can be done by adding the paginate setting to your config file:

``` yaml
paginate: 2
```

This should be set to the number of posts that you would like to be displayed in each page of the paginator.

Once this is done you can start using the paginator in your site markup. First lets render the posts that will be in each page.

``` html
{% for post in paginator.posts %}
<article class="box post post-excerpt">
  <header>
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  </header>
  <div class="info">
    <span class="date">{{ post.date | date: "%B %-d, %Y" }}</span>
  </div>
  {% if post.image %}
    <a href="{{ post.url }}" class="image featured"><img src="{{ post.image }}" alt="" /></a>
  {% endif %}
  <p>
    {{ post.excerpt }}
  </p>
</article>
{% endfor %}
```

The above liquid markup will iterate through each of the posts in the current page and render some teaser information about each post. This is the easy part.

Next let's render the navigation to get to the next page of posts. To start lets use the simplest possible form of navigation: next and previous buttons.

``` html
{% if paginator.previous_page %}
  <a href="{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}" class="button previous">Previous Page</a>
{% endif %}

{% if paginator.next_page %}
  <a href="{{ paginator.next_page_path | prepend: site.baseurl | replace: '//', '/' }}" class="button next">Next Page</a>
{% endif %}
```

This markup first detects if there is a previous page using paginator.previous_page and then renders a link to that page if available.

Next lets render links for each of the pages available.

``` html
{% for page in (1..paginator.total_pages) %}
  {% if page == paginator.page %}
    <em>{{ page }}</em>
  {% elsif page == 1 %}
    <a href="{{ '/index.html' | prepend: site.baseurl | replace: '//', '/' }}">{{ page }}</a>
  {% else %}
    <a href="{{ site.paginate_path | prepend: site.baseurl | replace: '//', '/' | replace: ':num', page }}">{{ page }}</a>
  {% endif %}
{% endfor %}
```

This code will render a list of pages as links and does a couple of things well. It detects that you are on the current page and will render that page without a link. It also handles another edgecase that Jekyll does not take care of for you. By detecting page 1 and rendering the index page it prevents the paginator from rendering the wrong first page.

In order to do something more advanced we are going to have to start using some variables to keep track of state. Unfortunately, while Liquid does a lot of things well as a templating engine, it doesn't play well with dynamic ranges. Ideally we would like to be able to setup ranges that look something like this:

``` html
{% for page in (paginator.page - 1..paginator.page + 1) %}
{% endfor %}
```

Liquid will not parse paginator.page - 2 properly and give us the range we inspect. Instead we need to use something like the following:

``` html
{% assign start = paginator.page | minus:1 %}
{% assign end = paginator.page | plus:1 %}
{% for page in (start..end) %}
  {% if page == paginator.page %}
    <a href="#" class="active">{{ page }}</a>
  {% elsif page == 1 %}
    <a href="{{ '/index.html' | prepend: site.baseurl | replace: '//', '/' }}">{{ page }}</a>
  {% else %}
    <a href="{{ site.paginate_path | prepend: site.baseurl | replace: '//', '/' | replace: ':num', page }}">{{ page }}</a>
  {% endif %}
{% endfor %}
```

Using the above technique you can detect state (what the current page is) and then use the appropriate for loop to render the paginator. You can check out a working version of the code [here][].
