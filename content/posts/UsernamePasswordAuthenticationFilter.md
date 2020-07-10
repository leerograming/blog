---
title: "UsernamePasswordAuthenticationFilter"
date: 2020-03-09T15:31:53+09:00
draft: false
images:
tags: 
  - spring security
---

1. UsernamePasswordAuthenticationFilter의 attemptAuthentication 메소드에서 authenticationManager.authenticate(authenticationToken)를 호출.(authenticationManager 구현체인 ProviderManager의 authenticate()가 호출된다)
{{< gist leerograming 9b2caed87d1c64e5b4e704a03a244bc0 >}}

2. ProviderManager의 authenticate() 에서 하나, 혹은 그 이상의 AuthenticationProvider를 호출(특별한 설정이 없다면 DaoAuthenticationProvider 호출)
{{< gist leerograming 344bd98a46741082ac1eba2b1c578a62 >}}

3. DaoAuthenticationProvider retrieveUser() 메소드에서 UserDetailService의 loadUserByUsername() 호출

4. retrieveUser()에서 return 받은 UserDetails 객체와 authenticationToken 값을 additionalAuthenticationChecks()에서 비교(여기서 비밀번호 검증이 이루어진다.)
{{< gist leerograming 626785c59db9bd6fda96d0dd41f81f84 >}}
 
  ---------------------------------------------------
    
<script src="https://utteranc.es/client.js"
        repo="leerograming/leerograming-comment"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>