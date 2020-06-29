---
title : "@Bean & @Component Annotation"
date: 2020-03-03T21:16:53+09:00
draft: false
images:
tags: 
  - spring
  - annotation
---

@Bean 

- 개발자가 컨트롤이 <strong>불가능한 외부 라이브러리</strong>를 bean으로 등록
- bean으로 등록되는 객체를 리턴해야 한다. 
- <strong>Method level 어노테이션</strong>

@Component

- 개발자가 <strong>컨트롤이 가능한 Class</strong>를 bean으로 등록
- <strong>Class level 어노테이션</strong>
- classpath scanning

추가) scanning을 통한 bean 등록이 궁금하면 springframwork.context.annotation의 <a href="https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ClassPathScanningCandidateComponentProvider.html">classpathscanningcandidatecomponentprovider.scanCandidateComponents</a> 참고하기(springboot 2.2.2 기준)

