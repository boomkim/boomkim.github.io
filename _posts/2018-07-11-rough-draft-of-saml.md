---
layout: post
title:  "SAML에 대한 매우 개략적인 정리"
date:   2018-07-11
categories: SAML AWS
---


## SAML 이란? 

**Security Asserting Markup Language**

즉, *보안 인증에 관한 정보를 기술하는 마크업 언어* 정도로 생각하면 될 것 같다. 혹은 *인증/인가 정보를 담은 XML* 정도로 간단하게 생각하면 될 것 같다. 

## 그렇다면 SAML은 어디에 쓰이는가?

**SSO(Single Sign-On)을 구현하기 위해 쓰인다.** SSO는 한 번의 로그인으로 여러개의 다른 도메인을 이용하려고 할때 필요한 기술이다. 예를 들어, 온프레미스에 이미 구축해 놓은 인증 시스템을 이용해서 AWS 리소스에 접근하려고 할때 AWS에 별도의 인증 시스템을 구축하지 않고 온프레미스의 인증을 그대로 쓰려고 할 때 사용한다. 

## SAML을 통한 인증 프로세스 

* 간략하게 설명한 인증 과정

**[과정1]**

![그림1-1](/images/saml_bcho1.png)

1. Browser에서 사이트 SpA로 접속한다.
2. 사이트 SpA에는 로그인이 되어 있지 않기 때문에 (세션이 없어서). SpA에서는 SAML request를 만들어서, Browser에게로 redirect URL을 보낸다.
3. Browser는 redirect URL에 따라 IdP에 접속하고, Idp에서 login form을 넣고 log in을 한다. 이때, IdP와 Brower 사이에 HttpSession 또는 Cookie로 Login에 대한 정보를 기록한다. 그리고 다시 사이트 SpA로의 SAML response를 포함한 redirect URL을 browser로 전송한다.
4. Browser는 SAML reponse를 가지고 SpA로 접속하면, SpA에는 인증된 정보를 가지고 로그인 처리를 한다. ※ 이 과정에서는 바로 사이트 SpA의 사용자 페이지(예를 들어 /home)등으로 가는 것이 아니라, SAML에 의해서 미리 정의한 SpA의 SAML response 처리 URL로 갔다가 SAML response를 처리가 끝나면 인증 처리를 한후, 사용자 페이지(/home)으로 다시 redirect한다.

**[과정2]**

![그림1-2](/images/saml_bcho2.png)

1. 사이트 SpA에서 로그인된 상태에서 SpB에 접속한다.
2. 사이트 SpB는 로그인이 되어 있지 않기 때문에, SAML 메시지를 만들어서 IdP의 login from으로 redirect URL을 보낸다.
3. 브라우져는 redirect URL을 따라서 IdP에 접속을 한다. IdP에 접속을 하면 앞의 과정에서 이미 Session 또는 Cookie가 만들어져 있기 때문에 별도의 로그인 폼을 띄위지 않고, SAML response message와 함께, SpB로의 redirect URL을 전송한다. 
4. Browser는 Sp B에 인증된 정보를 가지고 로그인한다.

출처: http://bcho.tistory.com/755?category=481504 [조대협의 블로그]
<br>

* 조금 더 디테일한 버전은 다음과 같다. 


![그림2](/images/saml_google.gif)

1. 유저는 서비스 제공자(Service Provider)에게 접근한다. (서비스 이용을 위하여)
2. 서비스 제공자는 SAMLRequest를 생성한다. 생성된 SAMLRequest은 XML format의 텍스트로 구성된다.
3. 유저의 브라우저를 이용하여 인증 제공자(Identity Provider)로 Redirect 한다.
4. 인증 제공자(Identity Provider)는 SAMLRequest를 파싱하고 유저인증을 진행한다.
5. 인증제공자(Identity Provider)는 SAML Response를 생성한다.
6. 인증제공자(Identity Provider)는 유저 브라우저를 이용하여 SAMLResponse data를 ACS로 전달한다.
7. ACS는 Service Provider가 운영하게 되는데 SAMLResponse를 확인하고 실제 서비스 사이트로 유저를 Forwarding한다.
8. 유저는 로그인에 성공하고 서비스를 제공받는다.

출처: http://civan.tistory.com/177 [행복만땅 개발자]


* AWS와 연동은 아래 그림과 같은 과정으로 구현된다.

![그림3](/images/saml-based-federation.diagram.png)

1. 조직 내 사용자가 클라이언트 앱을 사용해 조직의 IdP로부터 인증을 요청합니다.
2. IdP가 조직의 자격 증명 스토어를 이용하여 사용자를 인증합니다.
3. IdP가 사용자에 대한 정보로 SAML 어설션을 만들어 클라이언트 앱으로 보냅니다.
4. 클라이언트 앱이 AWS STS AssumeRoleWithSAML API를 호출하면서 SAML 공급자의 ARN, 수임할 역할의 ARN, IdP로부터 받은 SAML 어설션을 전달합니다.
5. 클라이언트 앱에 대한 API 응답에는 임시 보안 자격 증명이 포함되어 있습니다.
6. 클라이언트 앱은 임시 보안 자격 증명을 사용해 Amazon S3 API 작업을 호출합니다.