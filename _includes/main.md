<a name="top"></a>

# Table of contents

{% for section in site.sections %}* <a href="#{{ section[0] }}">{{ section[1] }}</a>
{% endfor %}

{% for section in site.sections %}
  <a name="{{ section[0] }}"></a>

  <h1>{{ section[1] }}</h1>

  [Back to top](#top) 
  
  {% assign path = '' %}
  {% capture path %}{{ section[0] }}.md{% endcapture %}
  {% include {{ path }} %}  
{% endfor %}
