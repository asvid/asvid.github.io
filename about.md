---
layout: page
title: About me
highlighter: null
permalink: /about/
---

I'm Android developer at [Fibaro Home Inteligence][f], but I used to do web development too.
I'm clean code fan interested in design patterns and dev-ops.

In my spare time, I play with [arduino][ard], or make my own Android apps. I also like wasting time on computer games :)

I want to use this space to blog about technical problems I experienced and how did I solve (or avoid) them, so others can make use of my work.

### Projects

{% for item in site.projects %}
  <h4>{{ item.title }}</h4>
  <p>{{ item.description }}</p>
  {% if item.googlePlayLink %}  
      {% include googlePlayLink.html %}
  {% endif %}
  {% if item.github %}  
    <a href="{{ item.github }}"><img src="/assets/img/google-play.png"/></a>
  {% endif %}
{% endfor %}


### Technologies

<div class="chipsContainer">
    <div class="row">
      {% for tech in site.technologies %}
      <a href=" {{ tech.link }}">
          <div class="chips"><img src="{{ tech.icon }}">{{ tech.name }}</div>
      </a>
      {% endfor %}
  </div>
</div>

### Contact me

Find me on [Linekdin][linkedin] / [Github][github] / [Facebook][fb] or just say `Hello` at
[adam.swiderski89@gmail.com](adam.swiderski89@gmail.com).

You can also check my [resume]({{ "/assets/cv.pdf" | absolute_url }}){:target="_blank"}.

[f]: http://www.fibaro.com
[github]: https://github.com/gayanvirajith
[linkedin]: https://pl.linkedin.com/in/aswiderski
[ard]: https://www.arduino.cc/
[fb]: https://www.facebook.com/adam.swiderski.pmi
