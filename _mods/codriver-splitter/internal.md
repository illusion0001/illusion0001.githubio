Ever wanted to have Co-driver calls play through headphones while the other sounds play through speakers? Now you are able to without hacking around a convoluted speaker setup!

Co-Driver Splitter is a plugin splitting audio between multiple devices. Now Co-driver, Spotter and Radio Communication calls will be played through a Communications device, while everything else plays over speakers!

{% if include.series == 'dirt' %}
<div align="center" class="video-container">
<iframe src="https://www.youtube.com/embed/S4psNp2mhUs" frameborder="0" allowfullscreen></iframe>
</div>

<div align="center" class="video-container">
<iframe src="https://www.youtube.com/embed/X3FQYK2GUuk" frameborder="0" allowfullscreen></iframe>
</div>
{% else %}
**NOTE:** Beware that due to the way Co-Driver Splitter works, perfect audio isolation is not possible. Therefore, some distant environment sounds are still faintly audible through headphones.
{% endif %}

{% include setup-instructions.html %}

<a href="https://github.com/CookiePLMonster/CoDriver-Splitter/wiki" class="button docs" role="button">{{ site.theme_settings.docs_icon }} See setup instructions on GitHub Wiki</a> \\
{% if include.bitness == 32 %}<a href="https://github.com/CookiePLMonster/CoDriver-Splitter/releases/latest/download/Co-Driver-Splitter-32-bit.zip" class="button download" role="button">{{ site.theme_settings.download_icon }} Download (32-bit version)</a>{% elsif include.bitness == 64 %}<a href="https://github.com/CookiePLMonster/CoDriver-Splitter/releases/latest/download/Co-Driver-Splitter-64-bit.zip" class="button download" role="button">{{ site.theme_settings.download_icon }} Download (64-bit version)</a>{% endif %} \\
<a href="https://github.com/CookiePLMonster/CoDriver-Splitter" class="button github" role="button">{{ site.theme_settings.github_icon }} See source on GitHub</a> \\
<a href="https://forums.codemasters.com/topic/36088-co-driver-splitter-plugin-co-driver-calls-on-headphones/" class="button forums" role="button">{{ site.theme_settings.codemasters_icon }} Discuss on Codemasters Forums</a>