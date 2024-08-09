---
layout: page
title: Contributors
permalink: /contributors/
---

<h1 class="post-title center-text">Our Contributors</h1>

<div class="contributors">
	{%- for contributor in site.contributors -%}
	{%- if contributor.site -%}
	<a class="contributor" href="{{contributor.site}}" target="_blank">
	{%- else -%}
	<a class="contributor" href="https://github.com/{{contributor.github}}" target="_blank">
	{%- endif -%}
		<img class="contributor-avatar" src="https://avatars.githubusercontent.com/{{contributor.github}}">
		<div class="contributor-name">{{contributor.name}}</div>
	</a>
	{%- endfor -%}
</div>