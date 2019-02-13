{% for post in site.posts %}
<div class="w3-card-2 w3-margin w3-white">
	
	<div class="w3-container">
		<h2><b>{{ post.title }}</b></h2>
			<div class="w3-container">
				<span class="w3-tag w3-margin-right w3-round w3-red">教程</span>
				<span class="w3-tag  w3-round">系统</span>
				<span class="w3-opacity w3-right">{{ post.date|date_to_string}}
			</div>
	
	</div>
	
	<div class="w3-container">
		<p>{{ post.excerpt }}</p>
		<div class="w3-container w3-margin-bottom">
				<a href="{{ post.url }}" class="w3-button w3-padding-large w3-red w3-border w3-right">更&nbsp;多&nbsp;»</a>
		</div>
	</div>
</div>
{% endfor %}