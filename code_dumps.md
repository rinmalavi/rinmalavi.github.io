---
layout: default
---

```html
<div id = "resolution"></div>
<script>
$('#resolution').text(function() {
	return "" + window.screen.availHeight + " " +window.screen.availWidth + " " + window.screen.height + " " +window.screen.width;
});
</script>
```

```scala  
private val index = detach() {
  getFromResourceDirectory("") ~
    path("") {
      getFromResource("web/index.html")
    }
}
```