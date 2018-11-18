```studyk8s
data
├── lesson-10
│   ├── redis-headless.yaml
│   └── redis-svc.yaml
├── lesson-11
│   ├── ingress-myapp.yaml
│   ├── ingress-tomcat-ssl.yaml
│   ├── ingress-tomcat.yaml
│   ├── mandatory.yaml
│   ├── service-nodeport.yaml
│   ├── ssl
│   │   ├── tls.crt
│   │   └── tls.key
│   ├── test-myapp.yaml
│   └── test-tomcat.yaml
├── lesson-12
│   ├── perdistentvolume.yaml
│   ├── test-configmap2.yaml
│   ├── test-configmap3.yaml
│   ├── test-configmap.yaml
│   ├── test-pvc.yaml
│   ├── volume-pod-emptyDir.yaml
│   ├── volume-pod-hostPath.yaml
│   ├── volume-pod-nfs.yaml
│   └── www.conf
├── lesson-14
│   ├── perdistentvolume.yaml
│   └── statefulset-demo.yaml
├── lesson-15
│   └── pod-demo.yaml
├── lesson-16
│   ├── rolebind-demo.yaml
│   └── role-demo.yaml
├── lesson-17
│   └── kubernetes-dashboard.yaml
├── lesson-18
│   ├── kube-flannel.yml
│   └── myapp-ds.yaml
├── lesson-19
│   ├── canal.yaml
│   ├── kube-flannel.yml
│   ├── namespace_default-allow_allpv_somepod.yaml
│   ├── namespace_default-allow_nodepv_somepod.yaml
│   ├── namespace_default-deny_all_outsidepv.yaml
│   ├── rbac.yaml
│   └── test-pod.yaml
├── lesson-21
│   ├── nodeAffinity-preferred.yaml
│   ├── nodeAffinity-required.yaml
│   ├── nodeSelector.yaml
│   ├── podAffinity-required.yaml
│   └── podAntiAffinity-required.yaml
├── lesson-22
│   ├── grafana.yaml
│   ├── heapster-rbac.yaml
│   ├── heapster.yaml
│   ├── influxdb.yaml
│   └── pod-demo.yaml
├── lesson-23
│   ├── auto-scale
│   │   └── hpa.yaml
│   ├── k8s-prom
│   │   ├── k8s-prometheus-adapter
│   │   │   ├── custom-metrics-apiserver-auth-delegator-cluster-role-binding.yaml
│   │   │   ├── custom-metrics-apiserver-auth-reader-role-binding.yaml
│   │   │   ├── custom-metrics-apiserver-deployment.yaml
│   │   │   ├── custom-metrics-apiserver-resource-reader-cluster-role-binding.yaml
│   │   │   ├── custom-metrics-apiserver-service-account.yaml
│   │   │   ├── custom-metrics-apiserver-service.yaml
│   │   │   ├── custom-metrics-apiservice.yaml
│   │   │   ├── custom-metrics-cluster-role.yaml
│   │   │   ├── custom-metrics-config-map.yaml
│   │   │   ├── custom-metrics-resource-reader-cluster-role.yaml
│   │   │   └── hpa-custom-metrics-cluster-role-binding.yaml
│   │   ├── kube-state-metrics
│   │   │   ├── 0.tar
│   │   │   ├── kube-state-metrics-deploy.yaml
│   │   │   ├── kube-state-metrics-rbac.yaml
│   │   │   └── kube-state-metrics-svc.yaml
│   │   ├── namespace.yaml
│   │   ├── node_exporter
│   │   │   ├── node-exporter-ds.yaml
│   │   │   └── node-exporter-svc.yaml
│   │   ├── podinfo
│   │   │   ├── podinfo-deploy.yaml
│   │   │   └── podinfo-svc.yaml
│   │   ├── prometheus
│   │   │   ├── prometheus-cfg.yaml
│   │   │   ├── prometheus-deploy.yaml
│   │   │   ├── prometheus-rbac.yaml
│   │   │   └── prometheus-svc.yaml
│   │   └── README.md
│   └── metrics-server
│       ├── auth-delegator.yaml
│       ├── auth-reader.yaml
│       ├── metrics-apiservice.yaml
│       ├── metrics-server-deployment.yaml
│       ├── metrics-server-service.yaml
│       └── resource-reader.yaml
├── lesson-5
│   └── pod-demo.yaml
├── lesson-6
│   └── pod-demo.yaml
├── lesson-7
│   ├── liveness-exec-pod.yaml
│   ├── liveness-httpget-pod.yaml
│   └── readiness-httpget-pod.yaml
├── lesson-8
│   └── rs-demo.yaml
└── lesson-9
    ├── myapp-deploy.yaml
    └── myapp-ds.yaml
