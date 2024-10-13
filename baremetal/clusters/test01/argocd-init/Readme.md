### K8S 클러스터 구성 후 SRE 구성 요소 설치

1. argocd 를 설치 (cluster 구성요소를 설치하기 위한 cd tool)
* k8s 클러스터 정보를 추가 해줌 (configs > clusterCredentials > name, server)
* k8s 클러스터의 token 을 생성 해 줌 (https://shonm.tistory.com/779)
  - argocd 의 value.yaml 에서 configs > clusterCredentials > config > bearerToken 부분을 생성된 token 으로 변경 해 줌
* clusters/[클러스터명]/argocd-init 에서 아래의 명령어로 설치
* kustomize build . --enable-helm > temp.yaml
* kubectl apply -f temp.yaml 로 설치 해 줌
* kubectl get -n argocd-init secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
* 위의 명령어로 argocd 사이트의 로그인을 위한 admin 비밀 번호를 얻어옴

2. argocd-apps 설치 (SRE 구성요소 설치 및 관리)
* clusters/[클러스터명]/argocd-apps 에서 아래의 명령어로 설치
* kustomize build . --enable-helm > temp.yaml
* kubectl apply -f temp.yaml 로 설치 해 줌

## 특이 사항
** 처음 argocd 설치 시 https 가 처리가 안됨

때문에 아래와 같이 values.yaml 에서 처리를 해 주면 nginx 에서 ssl 처리가 안되고  argocd 로 인증서 처리가 미뤄짐

argocd 는 자체적으로 ssl 사설 인증서 처리를 하기 때문에 ingress 설정 시 해당 annotation 처리가 필요

* values.yaml 파일의 특이 설정

	configs:
	  params:
		server.insecure: true
    
	~~

    annotations: #{}
      nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
      nginx.ingress.kubernetes.io/ssl-passthrough: "true"
	