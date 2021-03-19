---
layout: page
title: O mnie
excerpt: Moje krótkie bio.
permalink: /about/pl/
hide: true
---

*Ta strona dostępna jest w:*
<a href="{% link pages/about.md %}"><span style="white-space: nowrap"><img style="height:1em;padding: 0 0.5em 0 1em"
    src="{% link assets/img/flags/gb.svg %}" alt="GB">*English*</span></a>
<a href="{% link pages/about-pl.md %}"><span style="white-space: nowrap"><img style="height:1em;padding: 0 0.5em 0 1em"
    src="{% link assets/img/flags/pl.svg %}" alt="PL">*polski*</span></a>

***

<div style="max-width:35%;float:right;text-align:center" >
<img style="padding:0 5%;float:right" src="https://i.imgur.com/nnXmF1k.jpg" alt="Photo">
<em><span style="display:inline-block">To może dziwić, ale</span>
<span style="display:inline-block">nie jestem niebieski ani futrzasty.</span></em>
</div>
Hej, jestem **Adrian** -- za dnia pracuję przy programowaniu gier, wieczorami moduję i&nbsp;zajmuję się inżynierią zwrotną.
Próbuję naprawiać różne gry. W&nbsp;sieci udzielam się jako **Silent** lub **CookiePLMonster**.

***

Odkąd pamiętam, zawsze w&nbsp;jakiś sposób grzebałem w&nbsp;plikach gier. Pierwsze "próby", do których sięgam pamięcią,
to m.in. podmienianie plików dźwiękowych w&nbsp;Driverze, modyfikowanie tekstur w&nbsp;różnych grach freeware,
próby rozbudowywania Javowych skryptów Street Legal Racing Redline (jednak z&nbsp;marnym skutkiem).
Pierwsze poważniejsze modyfikacje zacząłem tworzyć w&nbsp;2008 roku, kiedy po raz pierwszy zagłębiłem się
w&nbsp;Grand Theft Auto: San Andreas i&nbsp;używany w&nbsp;nim [język skryptowy](https://gtamods.com/wiki/SCM_language).
Po jakimś czasie zacząłem chcieć robić więcej niż pozwalały same skrypty, więc po około
dwóch latach zacząłem zagłębiać się w&nbsp;kod gry za pomocą inżynierii zwrotnej i&nbsp;eksperymentować z&nbsp;coraz to
bardziej złożonymi hackami w&nbsp;assemblerze (tak, zanim nauczyłem się jakiegokolwiek języka wysokiego poziomu,
programowałem w&nbsp;assemblerze). To w&nbsp;końcu spowodowało, że w&nbsp;połowie 2011 roku zacząłem uczyć się C++.

Wszystko toczyło się w&nbsp;swoim dość wolnym tempie aż do końca 2013 roku, kiedy wydałem SilentPatch do Grand Theft Auto III
i&nbsp;Vice City -- modyfikację naprawiającą błędy w&nbsp;grach. Okazała się ona dużo bardziej popularna niż pierwotnie zakładałem,
co spowodowało, że wydałem więcej SilentPatchy -- po kilku miesiącach do San Andreas, później do innych gier,
kompletnie niezwiązanych z&nbsp;GTA.

Przełomem dla mnie okazało się wydanie [Hotline Miami 2 XP Support Patch]({% link _games/hm2.md %}#hm2-xp)
w&nbsp;marcu 2015 roku, kilka dni po premierze gry. Patch został zauważony przez twórców i&nbsp;oficjalnie "zatwierdzony",
a&nbsp;także okazał się moim punktem wejścia do zdobycia kontaktów w&nbsp;branży gier -- co ostatecznie pozwoliło mi zdobyć
pierwszą pracę (w&nbsp;której jestem do dziś).

Czym zajmuję się obecnie? Utrzymuję się z&nbsp;portowania gier (sprawdź moje [Portfolio]({% link pages/portfolio.md %})),
dalej wydaję nowe modyfikacje i&nbsp;patche dosyć regularnie (sprawdź moje [Mody & Patche]({% link pages/mods.md %})),
a&nbsp;ostatnio też od czasu do czasu udzielam się w&nbsp;developmencie kilku emulatorów konsol
(sprawdź moją [aktywność na GitHubie](https://github.com/CookiePLMonster)).

{% assign lastdate = "2020-03-22" %}
{% assign m = lastdate | date: "%-m" %}
*Ostatnia aktualizacja:
{{ lastdate | date: "%-d" }} {% case m %}
  {% when '1' %}stycznia
  {% when '2' %}lutego
  {% when '3' %}marca
  {% when '4' %}kwietnia
  {% when '5' %}maja
  {% when '6' %}czerwca
  {% when '7' %}lipca
  {% when '8' %}sierpnia
  {% when '9' %}września
  {% when '10' %}października
  {% when '11' %}listopada
  {% when '12' %}grudnia
{% endcase %} {{ lastdate | date: "%Y" }}*