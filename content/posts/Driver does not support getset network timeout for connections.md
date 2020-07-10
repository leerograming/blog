---
title: "Driver does not support get/set network timeout for connections"
date: 2020-07-10T15:20:47+09:00
draft: false
tags: 
  - HikariCP
  - SpringBoot
---
  
nexacro 연동 BMT 프로젝트를 개발하다가 <blockquote style="border-left:3px solid #3d3d3d; margin:0 0 24px; padding:10px 20px; font-size:15px; font-weight:bold;">HikariPool-1 - Driver does not support get/set network timeout for connections.(oracle.jdbc.driver.T4CConnection.getNetworkTimeout()I)</blockquote> 라는 log를 발견함.
Info level이라 별 문제는 없을 것 같아도 신경쓰여서 원인과 해결법을 찾아보게 되었다.
  
- <strong>프로젝트 환경</strong>
  - spring-boot 2.1.3
  - Oracle Database 10g 10.2.0.1
  - ojdbc6 11.2.0.3
    
<br>
<br>
  
- <strong>원인</strong>  
디버깅을 해서 해당 log가 찍히는 곳을 따라가보니 HikariCP의 PoolBase.class<sup><a href="#각주1">[출처]</a></sup> 내부 getAndSetNetworkTimeout()였다.
아래의 코드 5번 째 줄에 connection에서 getNetworkTimeout() 메소드를 호출하지 못해서 exception이 발생, log가 찍히는 것이었다.
``` java
private int getAndSetNetworkTimeout(final Connection connection, final long timeoutMs)
{
  if (isNetworkTimeoutSupported != FALSE) {
      try {
        final int originalTimeout = connection.getNetworkTimeout();
        connection.setNetworkTimeout(netTimeoutExecutor, (int) timeoutMs);
        isNetworkTimeoutSupported = TRUE;
        return originalTimeout;
      }
      catch (Throwable e) {
        if (isNetworkTimeoutSupported == UNINITIALIZED) {
            isNetworkTimeoutSupported = FALSE;

            LOGGER.info("{} - Driver does not support get/set network timeout for connections. ({})", poolName, e.getMessage());
            if (validationTimeout < SECONDS.toMillis(1)) {
              LOGGER.warn("{} - A validationTimeout of less than 1 second cannot be honored on drivers without setNetworkTimeout() support.", poolName);
            }
            else if (validationTimeout % SECONDS.toMillis(1) != 0) {
              LOGGER.warn("{} - A validationTimeout with fractional second granularity cannot be honored on drivers without setNetworkTimeout() support.", poolName);
            }
        }
      }
  }

  return 0;
}
```
  
- <strong>해결책</strong>  
<strong>ojdbc6은 getNetworkTimeout method를 지원하지 않아서 버전을 7로 변경</strong>했더니 간단하게 해결되었다.  
※ *ojdbc를 바꿀 수 없는 경우였다면  Hibernate의 dialect class인 Oracle10gDialect를 상속받아 구현했을 것.*
    
<br>
<br>
  
- <strong>추가 사항</strong>  
해당 프로젝트의 application.properies에 실수로 DiverClassName을 명시하지 않았는데 정상적으로 dataSource가 생성되었다.
당연히 에러가 생길 줄 알았는데 정상적으로 실행이 되어서 [Spring-boot Docs](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-sql)
를 찾아보았더니 아래의 내용이 있었다.
  
  <blockquote style="border-left:3px solid #3d3d3d; margin:0 0 24px; padding:10px 20px; font-size:15px; font-weight:bold;">
  	You often do not need to specify the driver-class-name, since Spring Boot can deduce it for most databases from the url.</blockquote>
    즉, 스프링부트가 URL에서 DriverClassName을 뽑아낼 수 있다는 의미이다.(그래도 웬만하면 명시를 해주는게 좋을 듯 하다)
    실제 구현은 Springboo autoconfigure 패키지의 DataSourceProperties<sup><a href="#각주2">[출처]</a></sup>의 determineDriverClassName() 에서 이루어진다.
    
``` java
public String determineDriverClassName() {
		if (StringUtils.hasText(this.driverClassName)) {
			Assert.state(driverClassIsLoadable(),
					() -> "Cannot load driver class: " + this.driverClassName);
			return this.driverClassName;
		}
		String driverClassName = null;
		if (StringUtils.hasText(this.url)) {
		 	driverClassName = DatabaseDriver.fromJdbcUrl(this.url).getDriverClassName();
		}
		if (!StringUtils.hasText(driverClassName)) {
			driverClassName = this.embeddedDatabaseConnection.getDriverClassName();
		}
		if (!StringUtils.hasText(driverClassName)) {
			throw new DataSourceBeanCreationException(
					"Failed to determine a suitable driver class", this,
					this.embeddedDatabaseConnection);
		}
		return driverClassName;
}
```    
<br>
<br>
  
-참고 및 출처
  - [stackoverflow](https://stackoverflow.com/questions/53940321/how-to-fix-driver-does-not-support-get-set-network-timeout-for-connections-whi/54906333#54906333)  
  - <a id="각주1">[poolBase](https://github.com/brettwooldridge/HikariCP/blob/dev/src/main/java/com/zaxxer/hikari/pool/PoolBase.java)</a>
  - <a id="각주2">[DataSourceProperties](https://github.com/spring-projects/spring-boot/blob/v2.3.1.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceProperties.java)</a>
 
  ---------------------------------------------------
    
<script src="https://utteranc.es/client.js"
        repo="leerograming/leerograming-comment"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>



