---
layout: post
title:  keycloak 정리
date:   2022-09-06 16:08:00 +0800
categories: msa
tag: msa
sitemap :
  changefreq : daily
  priority : 1.0



---

# 용어 정리

## 1. 리소스서버

리소스 서버(보호된 리소스를 제공하는 응용 프로그램 또는 서비스)는 일반적으로 보호 리소스에 대한 액세스 권한을 부여해야 하는지 여부를 결정하기 위해 일종의 정보에 의존합니다.

RESTful 기반 리소스 서버의 경우 해당 정보는 일반적으로 서버에 대한 모든 요청에 대해 전달자 토큰으로 전송되는 보안 토큰에서 얻습니다.

종종 리소스 서버는 RBAC(역할 기반 액세스 제어)를 기반으로 권한 결정을 수행합니다. 여기서 보호된 리소스에 액세스하려는 사용자에게 부여된 역할은 이러한 동일한 리소스에 매핑된 역할에 대해 확인됩니다.

## 2. PDP

PDP는 정책 결정 지점을 제공한다. 정책 결정 지점에서는 인가 요청을 받고, 요청된 권한에 따라 정책이 평가된다. 그니까 PDP는 권한 요청을 받고 정책에 따라 그 권한 요청을 평가(권한을 주거나 말거나) 한다. 

## 3. Authorization Service

인가 서비스는 PDP의 일부이다. 

### (1) Token endpoint

Token endpoint 덕에 리소스 서버는 인가 서버에서 부여하고 액세스 토큰이 보유한 권한을 기반으로 보호된 리소스에 대한 액세스를 시행할 수 있습니다. 권한이 있는 액세스 토큰을 RPT라고 합니다.

### (2) Permission Ticket

rpt를 얻을 수 있는 토큰 정도로 생각하면 될듯. 권한 티켓은 권한 부여 서버에서 리소스 서버로 발행되며, 리소스 서버는 보호된 리소스에 액세스하려는 클라이언트에게 권한 티켓을 반환합니다. 이 부분이 잘 이해 안되는데, 권한 티켓은 바로 클라이언트에 발행되는거 아닌가? 왜 리소스 서버로 발행된다고 하는지 모르겠음. 그럼 id, pw 로그인을 리소스 서버로 한다는 뜻인가?

권한을 얻기 위해 인가 요청을 토큰 엔드포인트에 보내면 키클록이 요청된 자원 및 스코프에 관련된 모든 정책을 평가한 후, 권한이 포함된 rpt를 발급한다.

## 4. PEP

PEP는 리소스 서버 쪽에서 실제로 인가 결정을 실행한다.



# 인가 과정

인가 과정은 다음과 같이 나눌 수 있다. 리소스를 정의하고, 인가 로직을 정의하고(Policy), 리소스와 인가 로직을 결합시킨 후(Permission), 리소스 서버가 특정 리소스에 대한 권한(Permission)을 요청하는 것이다. 

1. Resource Management
2. Permission and Policy Management
3. Policy Enforcement



## 1. Reosource Management

1. Create Resource Server(ex: Life History)
2. Create Resource(ex: /times/*)
3. Create and Associate Scope(ex: view, delete / Life History - view)

Scope은 일반적으로 리소스에서 수행할 수 있는 작업을 나타내지만 이에 국한되지 않습니다. Scope을 사용하여 리소스 내에서 하나 이상의 속성을 나타낼 수도 있습니다.



## 2. Permission and policy management

1. Create Policy
2. Define Permission
3. Apply Policies to permission

이 절차는 resource를 제어하는 보안과 접근 요구사항을 정의한다.

Policy는 어떤 항목(Resourece 또는 Scope)에 액세스하거나 작업을 수행하기 위해 충족되어야 하는 조건을 정의하지만 보호 대상과 관련이 없습니다. 그것들은 여러 권한에 사용될 수 있고 더 복잡한 정책을 구축하는 데 재사용할 수 있습니다.

Permission은 보호하는 리소스와 연결됩니다. 여기에서 보호할 대상(리소스 또는 스코프)과 정책을 결합합니다.



## 3. Policy Enforcement

Policy Enforcer는 리소스 서버가 실제로 인가 결정을 내릴 수 있도록 하게하는 절차입니다. 이건 PEP에 의해 수행되는데, PEP는 authorrization server와 커뮤니케이션하여 authorization 서버에 인가 관련 정보를 얻고 그 정보(인가 결정 및 권한 정보)에 따라 보호 자원에 대한 접근을 제어합니다. 

### 코드레벨에서 살펴보는 Policy Enforcement

keycloak pep가 권한을 얻는 과정은 간단하게 리소스 서버에 있는 keycloakDeployment 정보와 인증된(인가X) 토큰을 토대로 PDP에 권한이 있는지 http 요청으로 물어보는 것이다. 권한이 있는 토큰을 rpt라고 부르는데, 리소스 서버가 가지고 있는 policy enforcer 정보와 인증된 토큰(이 인증된 토큰을 permission ticket이라고 해도 될 것 같은데... 확실하진 않다.)을 가지고 인증된 토큰에게 특정 권한을 포함한 rpt를 달라고 요청하고, rpt를 잘 가져오면 인가된것이다.

```java
KeycloakDeployment deployment = deploymentContext.resolveDeployment(facade); // 리소스 서버에 있는 키클록 서버 정보를 가져온다)
PolicyEnforcer policyEnforcer = this.deployment.getPolicyEnforcer();
AuthorizationContext authorizationContext = policyEnforcer.enforce(facade); // policy enforcer로 encforce하는게 아래에 인가요청을 보내는 것이다.
AuthorizationContext context = new KeycloakAdapterPolicyEnforcer(this).authorize(facade);
accessToken = requestAuthorizationToken(pathConfig, methodConfig, httpFacade, claims); // http 요청을 보낸다.
return super.isAuthorized(pathConfig, methodConfig, accessToken, httpFacade, claims);
```

requestAuthorizationToken에서는 결국 아래 http call을 한다.

```
curl -X POST \
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  -H "Authorization: Bearer ${access_token}" \
  --data "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket" \
  --data "audience={resource_server_client_id}" \
  --data "permission=Resource A#Scope A" \
  --data "permission=Resource B#Scope B"
```

그러니까 특정 인증된 주체(access_token)에게 특정 리소스(Resource A#ScopeA)에 대한 권한을 달라고 하는 것이다. 



리소스 서버 입장에선 policy enforcer 설정이 가장 중요하다. 그럼 그 설정에는 구체적으로 어떤 정보들이 들어갈까?

policy enforcer 설정이란 간단하게 "어떠어떠한 url들을 이용하려면 어떠어떠한 리소스에 대한 권한을 가져야 합니다"라고 정의하는 것이다. 밑에 보면 "/times/*" url의 GET method를 이용하려면 Times Resource의 view 스콥에 대한 권한을 가져야한다고 정의한 것이다.

```yaml
policy-enforcer-config:
  paths:
    - name: Times Resource
      path: /times/*
      methods:
        - method: GET
          scopes:
            - view
```

여기에 있는 url를 이용하려면 어떤 리소스에 대한 권한을 필요할까에 대한 정보는 아래 코드에서 가져온다.

```java
PathConfig pathConfig = getPathConfig(request);
```

만약 url을 public하게 이용 가능하게 하려면 리소스 정보를 안적으면 된다. spring security처럼 남은 url 전부 permitAll 하는 등의 설정이 있는진 모르겠다.

```yaml
policy-enforcer-config:
    paths:
      #      - name: Times Resource
      - path: /times/*
        methods:
          - method: GET
#            scopes:
#              - view
```



## 4. Security Contstraints 관련하여

poloicy enforcement를 공부하면서 Security Constraints 설정이 왜 필요한지 궁금했다.

공부한대로라면 Security Contstraint 설정은 필요 없었다. policy enforcement를 할 때 어떤 리소스에 대한 permission이 있는지 평가하기 때문이다. 미리 어떤 resource를 이용하려면 필요한 role을 정의하고, policy enforce를 할 때 어떤 role이 있는지 검사하면 Security Constraint가 하는것과 똑같은 검증(access token이 특정 role을 가지고 있는지 확인)을 할 수 있다.

keycloak 구현을 살펴봤을 때, 왜 이렇게 구현했는지는 파악을 못했는데 어찌됐든 현재 security constraint를 필요로 하는 이유를 "구현을 그렇게 했기 때문"이라고 규정하고 있다.

Policy Enforcer는 authorize를 할때 이미 security context에 인증된 주체가 있는 것을 전제한다. security context가 없으면 401 아니면 403이다. 

```java
if (securityContext == null) {
    if (!isDefaultAccessDeniedUri(request)) {
        if (pathConfig != null) {
            if (EnforcementMode.DISABLED.equals(pathConfig.getEnforcementMode())) {
                return createEmptyAuthorizationContext(true);
            } else {
                challenge(pathConfig, getRequiredScopes(pathConfig, request), httpFacade);
            }
        } else {
            handleAccessDenied(httpFacade);
        }
    }
    return createEmptyAuthorizationContext(false);
}
```

그러면 필수적으로 security context를 set 해줘야 한다는건데, security context set은 언제해주냐?

```java
if (authRequired) {
		...

    if (jaspicProvider == null && !doAuthenticate(request, response) ||
            jaspicProvider != null &&
                    !authenticateJaspic(request, response, jaspicState, false)) {
				...
        return;
    }

}
```

바로 authRequred일 때다.

그러면 언제 authRequired이냐?

그건 바로 securityConstraints가 있을 때이다(물론 jaspicProvider가 정의돼있을때도 authRequired하지만, 이 부분은 잘 모르기도 하고 일반적인 방법이 securityConstraints를 정의해주는 것 같아 넘어간다).

그니까 policy enforcer가 authorize(enforce)할 때 이미 인증이 돼있어야 하고, securityConstraints가 있을 때 인증을 하기 때문에 securityConstraints가 필요하다.

그러면 securityConstratins를 어떻게 정의할것인가?

결론적으로 "모든 url 패턴에 대하여 모든 role을 허용"하면 된다. 이렇게 하는 이유는

1. 어차피 policy enforce를 할 때 필요한 평가를 하기 때문에 인증할 때 중복 평가할 필요가 없다.
2. useRealmRoleMapping을 할 경우 구현상 principal은 realm의 모든 역할을 가지기 때문에, 그 사용자가 역할을 가지는지 안가지는지 평가하는게 무의미하다. 그 사용자가 모든 역할을 가지고 있게되고, 만약 역할이 없다면 사용자에 대한 검증이 아니라 "realm"이 특정 role을 가지고 있는지 검증하는게 된다. 할 필요 없다.
3. useResourceRoleMapping을 할 경우 2보다는 훨씬 의미있다. 사용자에게 특정 client role을 부여하고, 사용자가 리소스 서버에서 정의한 client role을 가지고 있는지 검증하는 것이기 때문에 사용자에 대한 검증은 맞다. 하지만 1번의 이유로 할 필요는 없고, 아직 복잡도가 낮기 때문에 현재 client role은 사용하지 않고 realm role만 사용할 생각이다. 만약 사용자가 리소스 서버에 대한 권한을 부여하고 싶다면 이건 필요할 듯 싶다. 하지만 지금은 아님!

특정 role을 가져야 한다고 constraint를 정의할 경우 "사용자가 특정 role을 가지는지 검증" 하지만, 모든 role이 된다고 하면 로직이 다음과 같다. role을 가지고 있다고 간주.

```java
public boolean hasRole(String role) {
    if ("*".equals(role)) { // Special 2.4 role meaning everyone
        return true;
    }
    if (role == null) {
        return false;
    }
    return Arrays.binarySearch(roles, role) >= 0;
}
```

