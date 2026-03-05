---
title: Missing Fullscreen Button on HTML5 Video Player
pubDate: 2019-01-08 15:51:47
tags:
- HTML5
- Troubleshooting
---
If you've ever worked with HTML5 video then you have probably wondered how you get a bunch of control buttons when you’ve only added a single `<video>` tag to your page. Recently, I get a problem that the fullscreen button on the video play is missing or greyed out if I put the video inside a IFRAME.
What happened?
<!-- more -->
__With fullscreen button:__
{% raw %}
<iframe src="https://mdn.mozillademos.org/en-US/docs/Web/HTML/Element/video$samples/Simple_video_example?revision=1444820" height="370" width="640" id="frame_Simple_video_example" frameborder="0" allowfullscreen="true" webkitallowfullscreen="true" mozallowfullscreen="true"></iframe>
{% endraw %}

__No fullscreen button:__
{% raw %}
<iframe src="https://mdn.mozillademos.org/en-US/docs/Web/HTML/Element/video$samples/Simple_video_example?revision=1444820" height="370" width="640" id="frame_Simple_video_example" frameborder="0"></iframe>
{% endraw %}

# Reason
Here is the code I used to embed the video in bad sample:
```
<iframe src="https://mdn.mozillademos.org/en-US/docs/Web/HTML/Element/video$samples/Simple_video_example?revision=1444820" height="370" width="640" id="frame_Simple_video_example" frameborder="0"></iframe>
```
The story is, by default, only **top-level documents** and their **same-origin** child frames can request and enter **fullscreen** mode. Therefore, if a IFRAME source origin is different from the parent page, the browser would prevent that IFRAME from using fullscreen mode.

# Workaround 
To workaround this, you can:
*   Enable [Feature-Policy: fullscreen](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Feature-Policy/fullscreen);
*   Add `allowfullscreen=true` attribute in IFRAME.

```
<iframe src="https://mdn.mozillademos.org/en-US/docs/Web/HTML/Element/video$samples/Simple_video_example?revision=1444820" height="370" width="640" id="frame_Simple_video_example" frameborder="0" allowfullscreen="true" webkitallowfullscreen="true" mozallowfullscreen="true"></iframe>
```

Reference:
* [Feature-Policy: fullscreen](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Feature-Policy/fullscreen)
* [The Video Embed element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video)
* [video.js inside a modal window: full screen not working](https://stackoverflow.com/a/14319859)