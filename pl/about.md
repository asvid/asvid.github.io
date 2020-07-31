---
layout: about
title: About me
highlighter: null
permalink: /about/
---

Tu będzie tekst po polsku

I'm Android developer at [Payworks][p], but I used to do web development.
I'm clean code fan interested in software architecture, design patterns and devops culture, I really like automating stuff (I'm lazy in positive sense).

In my spare time, I try not to forget how to play guitar, read books, watch really bad movies, or make my own Android apps - usually as sandboxes to check new libraries or approaches. I also like wasting time on computer games :)

I want to use this space to write about technical problems I've experienced and how I solved (or avoided) them, useful tools and tricks and... well anything else I want to write here.

### Contact me

Find me on [Linekdin][linkedin] / [Github][github] / [Facebook][fb] or just say `Hello` at
[adam.swiderski89@gmail.com](adam.swiderski89@gmail.com).

You can also check my [resume]({{ "/assets/cv.pdf" | absolute_url }}){:target="_blank"}.

### Projects

Here are listed some of my projects, with links to Google Play or code on GitHub.

{% for item in site.projects %}
  {% include project.html %}
{% endfor %}


### Technologies

Alphabetically ordered list of tech I like to use

<div class="chipsContainer">
    <div class="row">
      {% for tech in site.technologies %}
      <a href=" {{ tech.link }}">
          <div class="chips"><img src="{{ tech.icon }}">{{ tech.name }}</div>
      </a>
      {% endfor %}
  </div>
</div>


[f]: http://www.fibaro.com
[p]: https://payworks.com/
[github]: https://github.com/gayanvirajith
[linkedin]: https://pl.linkedin.com/in/aswiderski
[fb]: https://www.facebook.com/adam.swiderski.pmi
