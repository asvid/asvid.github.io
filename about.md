---
layout: about
title: About me
highlighter: null
permalink: /about/
---

I'm Android developer at [Fibaro Home Inteligence][f], but I used to do web development too.
I'm clean code fan interested in design patterns and dev-ops culture.

In my spare time, I play with [arduino][ard], or make my own Android apps. I also like wasting time on computer games :)

I want to use this space to write about technical problems I've experienced and how I solved (or avoided) them, usefull tools and tricks and... well anything else I want to write here.

### Projects

Here are listed some of my projects, with links to Google Play or code on GitHub.

{% for item in site.projects %}
  {% include project.html %}
{% endfor %}


### Technologies

Random order list of tech I like to use

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
