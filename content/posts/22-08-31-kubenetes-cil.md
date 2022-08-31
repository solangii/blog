---
title: "🎯 Kubenetes Snippets"
date: 2022-08-31T00:12:57+09:00
categories: snippet
tags: ['kubenetes', 'mlops']
draft: false
notoc: true
---



> 자주 사용하는 Kubenetes 명령어를 정리합니다.

---



### 1. 실행하기

```
kubectl apply -f <file_name>.yaml
```

　

　

### 2.  job/pod 확인

#### job 조회

```
kubectl get job # 이 경우, 전체 job이 나옴
kubectl get job <job_name>
```

　

#### job 정보 확인

*주로, pod가 안뜨거나, 만들어진 pod 정보 확인 할 때 사용함

```
kubectl describe job <job_name>
```

　

#### pod 조회

```
kubectl get pod # 이 경우, 전체 pod가 나옴
kubectl get pod <pod_name>
kubectl get pods -l job-name=<job_name> # job이 생성한 pod 조회
kubectl get pod -w # watch mode
```

　

#### pod 정보 확인

*컨테이너를 잘 띄우는지 확인 할 때 사용함.

```
kubectl describe pod <pod_name>
```

　　

　

### 3. 종료하기

```
kubectl delete job <job_name>
kubectl delete -f <file_name>
```

　

　

### 4. 로그보기

```
kubectl logs <pod_nema> -c main -f
```

　

　

### 5. 컨테이너에 접속

```
kubectl exec -it <pod_name> -c main -- /bin/bash
```

　

　

### 6. 자원 사용량 조회

```
kubectl top pod <pod_name> --sort-by=<cpu>
kubectl top pod <pod_name> --sort-by=<mem>
```

　

　

### 7. Quota 확인

```
kubectl get quota
kubectl get quota <name_space>
```

---

### Reference

- https://kubernetes.io/ko/docs/concepts/