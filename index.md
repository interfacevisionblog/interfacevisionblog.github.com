---
layout: page
title: Interface Vision
tagline: Blog and Such
group: navigation
---
{% include JB/setup %}

{% assign posts_collate = site.posts %}


<div class="row-fluid">
  <div class="span3">
    {% include themes/twitter/about %}    
  </div>
  <div class="span9">
    <div class="hero-unit">
      <h1>Blog Posts</h1>
      <ul>
        {% if site.JB.posts_collate.provider == "custom" %}
          {% include custom/posts_collate %}
        {% else %}
          {% for post in posts_collate  %}
            {% capture this_year %}{{ post.date | date: "%Y" }}{% endcapture %}
            {% capture this_month %}{{ post.date | date: "%B" }}{% endcapture %}
            {% capture next_year %}{{ post.previous.date | date: "%Y" }}{% endcapture %}
            {% capture next_month %}{{ post.previous.date | date: "%B" }}{% endcapture %}

            {% if forloop.first %}
              <h2>{{this_year}}</h2>
              <ul>
            {% endif %}

            <li><span>{{ post.date | date: "%B %e, %Y" }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a> by <a href="http://twitter.com/{{post.author_twitter}}" target="_blank">{{post.author}}</a></li>

            {% if forloop.last %}
              </ul>
            {% else %}
              {% if this_year != next_year %}
                </ul>
                <h2>{{next_year}}</h2>
                <ul>
              {% endif %}
            {% endif %}
          {% endfor %}
        {% endif %}
        {% assign posts_collate = nil %}
      </ul>
    </div>    
  </div>
</div>



<ul class="posts">

  {% if site.JB.posts_collate.provider == "custom" %}
    {% include custom/posts_collate %}
  {% else %}
    {% for post in posts_collate  %}
      {% capture this_year %}{{ post.date | date: "%Y" }}{% endcapture %}
      {% capture this_month %}{{ post.date | date: "%B" }}{% endcapture %}
      {% capture next_year %}{{ post.previous.date | date: "%Y" }}{% endcapture %}
      {% capture next_month %}{{ post.previous.date | date: "%B" }}{% endcapture %}

      {% if forloop.first %}
        <h2>{{this_year}}</h2>
        <ul>
      {% endif %}

      <li><span>{{ post.date | date: "%B %e, %Y" }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a> by <a href="http://twitter.com/{{post.author_twitter}}" target="_blank">{{post.author}}</a></li>

      {% if forloop.last %}
        </ul>
      {% else %}
        {% if this_year != next_year %}
          </ul>
          <h2>{{next_year}}</h2>
          <ul>
        {% endif %}
      {% endif %}
    {% endfor %}
  {% endif %}
  {% assign posts_collate = nil %}
</ul>



