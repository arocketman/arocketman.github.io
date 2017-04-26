---
layout: post
title:  "Setting up OAuth2 to work with Spring Boot"
date:   2017-04-15 14:52:07 +0200
categories: main
icons: 
- fa-code
- icon-java

---
Recently I've been playing around with Spring Boot and coming from frameworks such as laravel or RoR one cannot deny that setting up users is way more tedious in Spring. Spring security is great, but still overly complicated and even though Spring boot attempts to remove unnecessary configurations, the whole frameworks still is not as intuitive as , for example , ruby on rails.

Anyhow, it's undeniable that spring is a mature framework and combined with the power of java it is truly a great tool.

Setting up users wasn't as easy as I expected, I came across the following guide: [spring docs oauth2][springdocs]

which explained quite a lot and along with some github examples provided by spring itself I was able to create a working project that authenticates users with oAuth2. You can find the working examples here: [github Repo][working-example]

Also, I've made a video-tutorial on youtube since I didn't seem to find any that resembled exactly what I wanted to do: [Youtube videotutorial][youtubevidt]

[springdocs]:https://projects.spring.io/spring-security-oauth/docs/oauth2.html
[working-example]: https://github.com/arocketman/Spring-oauth2-jpa-example
[youtubevidt]:https://www.youtube.com/watch?v=0pD7YeTAUkk