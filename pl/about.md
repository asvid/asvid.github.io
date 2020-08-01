---
layout: about
title: O mnie
highlighter: null
permalink: /about/
---

Jestem programistą w [Cybersource/Visa][p], głównie zajmuję się rozwojem naszego SDK, 
ale mam też doświadczenie z Frontendem, zaglądam czasami na Backend lub do niskopoziomowych logów terminali płatniczych :)

Jestem fanem "czystego kodu" i interesuje mnie architektura oprogramowania i wzorce projektowe. Jestem dość leniwy i
 uwielbiam automatyzować sobie pracę (:snake: :heart:).

W wolnym czasie staram się nie zapomnieć, jak się gra na gitarze :guitar:, czytam książki :books:, 
oglądam bardzo słabe filmy :movie_camera:, albo tracę czas na gry komputerowe :video_game:. Lubię też biwakować :tent: i chodzić po górach :sunrise_over_mountains:.

### Contact me

Znajdź mnie na [Linekdin][linkedin] / [Github][github] / [Facebook][fb] albo poprostu się przywitaj na [adam.swiderski89@gmail.com](adam.swiderski89@gmail.com).

Jeśli interesuje Cię moje [CV]({{ "/assets/cv.pdf" | absolute_url }})

### Technologie z którymi pracuję

{% for item in site.data.technologies %}
  {% include tech.html %}
{% endfor %}

### Kilka moich projektów
  
{% for item in site.data.projects %}
  {% include project.html %}
{% endfor %}

[p]: https://www.cybersource.com
[github]: https://github.com/asvid
[linkedin]: https://pl.linkedin.com/in/aswiderski
[fb]: https://www.facebook.com/adam.swiderski.pmi
