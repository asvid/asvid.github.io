---
layout: about
title: About me
highlighter: null
permalink: /about/
---

I’m a software engineer with diverse experience, having worked on educational apps, smart home technology, and large- scale financial tech projects. I prioritize writing clean and understandable code and ensuring its stability through robust testing. While applying SOLID, DRY, and other principles, I try to avoid hasty abstractions, to keep things as simple
as possible, for as long as possible, and not over-engineer.

I have lots of experience with legacy code, but I also started some fresh green-field projects, and learned a lot by mistakes I made there.

Furthermore, I enjoy task automatization with CI/CD approach in mind. I used to design and implement release procedures for mobile apps.


In my spare time, I try not to forget how to play guitar :guitar:, read books :books:, watch really bad movies :movie_camera:, or wasting time on computer games :video_game:. I like camping :tent: and hiking :sunrise_over_mountains:.

### Contact me

Find me on [Linekdin][linkedin] / [Github][github] or just say `Hello` with an
[email](mailto:adam.swiderski89@gmail.com).

You can also check my [resume]({{ "/assets/cv.pdf" | absolute_url }})

### Project

Here are listed some of my projects:

{% for item in site.data.projects %}
  {% include project.html %}
{% endfor %}


### Technologies

Random ordered list of tech I like to use

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
