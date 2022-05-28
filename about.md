---
layout: about
title: About me
highlighter: null
permalink: /about/
---

I'm software engineer working with Android, currently in fintech. I also have some experience with Frontend, occasionally I do backend work.

I'm clean code fan interested in software architecture and design patterns. I am pretty lazy, so I really like automating stuff (:snake: :heart:).

In my spare time, I try not to forget how to play guitar :guitar:, read books :books:, watch really bad movies :movie_camera:, or wasting time on computer games :video_game:. I like camping :tent: and hiking :sunrise_over_mountains:.

### Contact me

Find me on [Linekdin][linkedin] / [Github][github] or just say `Hello` with an
[email](mailto:adam.swiderski89@gmail.com).

You can also check my [resume]({{ "/assets/cv.pdf" | absolute_url }})

### Technologies

Here are listed some of my projects, with links to Google Play or code on GitHub.

{% for item in site.data.projects %}
  {% include project.html %}
{% endfor %}


### Technologies

Alphabetically ordered list of tech I like to use

<div class="chipsContainer">
    <div class="row">
      {% for tech in site.data.technologies %}
      <a href=" {{ tech.link }}">
          <div class="chips"><img src="{{ tech.icon }}">{{ tech.name }}</div>
      </a>
      {% endfor %}
  </div>
</div>

[p]: https://www.cybersource.com
[github]: https://github.com/asvid
[linkedin]: https://pl.linkedin.com/in/aswiderski
