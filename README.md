# helm-chart-template
- Deployment
- StatefulSet
- DaemonSet
- Job
- CronJob
- ConfigMap
- Secret
- PersistentVolumeClaim
- Ingress
- HorizontalPodAutoscaler
- Service

**Example values**
```
#######################
##__GLOBAL_SETTINGS__##
#######################

##__DEPLOYMENT TYPE: deployment, daemonset, statefulset or false if not needed
## https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
## https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
## https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
global_settings:
  deploy_type: deployment
  configmaps: true
  secrets:  true
  services: true
  ingress: true
  pvc: false
  job: false
  cronjob: false
  hpa: true

##__IMAGE PULL SECRET: default is the application name {{ template "app.fullname" . }}-pullsecret
## if the value is empty, then the secret is not created
## https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
imagePullSecret: ""
## imagePullSecret: "eyJhdXRocyI6eyJoYXJib3IubG9jYWwiOnsidXNlXRoIjoiWVdSdGFXNDZTR0Z5WW05eU1USXpORFU9In19fQ=="
## kubectl -n ${NAMESPASE} create secret docker-registry temp-secret --docker-server=${CI_HARBOR_URL} --docker-username=${CI_REGISTRY_USER} --docker-password=${CI_REGISTRY_PASSWORD}
## kubectl -n ${NAMESPASE} get secrets temp-secret -o jsonpath="{.data['\.dockerconfigjson']}"
## kubectl -n ${NAMESPASE} delete secrets temp-secret

##__VOLUMES
## https://kubernetes.io/docs/concepts/storage/
#supports multiple values
volumes:
  - name: config-vol
    configMap:
      name: nginx-external
      items:
        - key: nginx.conf
          path: nginx.conf
        - key: sidecar-nginx.conf
          path: sidecar-nginx.conf
#  - name: first-pv
#    persistentVolumeClaim:
#      claimName: first
#  - name: second-pv
#    persistentVolumeClaim:
#      claimName: second
#  - name: cache-volume
#    emptyDir:
#      sizeLimit: 500Mi

##__SECURITY CONTEXT FOR A PODS
## https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
#supports multiple values
podSecurityContext: {}
#  fsGroup: 1001

##__DEFAULT podAnnotations empty
## https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/
#supports multiple values
podAnnotations: {}
#  foo: boo
#  aff: baf

##__DON'T USE ROLLME IN ARGOCD
## https://helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments
rollme:
  enabled: false

##__NUMBER OF REPLICA POD default 1 if empty
replicaCount: 3

##__SETTINGS ONLY FOR STATEFULSET
##__POD MANAGEMENT POLICY only for StatefulSet, default OrderedReady if empty
## https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies
podManagementPolicy: {}
#OrderedReady or Parallel
serviceName: ""


##__SERVICE ACCOUNT FOR POD
## https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
serviceAccountName: ""

##__TOLERATIONS for pods assignment
## https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
#supports multiple values
tolerations: {}
#  - key: "key1"
#    operator: "Equal"
#    value: "value1"
#    effect: "NoSchedule"
#  - key: "key1"
#    operator: "Exists"
#    effect: "NoSchedule"

##__POD UPDATE STRATEGY default RollingUpdate 25%/25%
#supports multiple values
updateStrategy: {}
#  type: Recreate
#  type: RollingUpdate
#  rollingUpdate:
#    maxSurge: 0
#    maxUnavailable: 1

##__AFFINITY RULES 
## https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity
affinity: {}
#  podAntiAffinity:
#    requiredDuringSchedulingIgnoredDuringExecution:
#    - labelSelector:
#        matchExpressions:
#        - key: app.kubernetes.io/instance
#          operator: In
#          values:
#          - nginx
#      topologyKey: "kubernetes.io/hostname"
#  podAffinity:
#    requiredDuringSchedulingIgnoredDuringExecution:
#    - labelSelector:
#        matchExpressions:
#        - key: security
#          operator: In
#          values:
#          - S1
#      topologyKey: topology.kubernetes.io/zone
#  nodeAffinity: {}


##################
##__DEBUG_MODE__##
##################
debug_mode:
  enabled: false
  command: ["sleep"]
  args: ["infinity"]


##################
##__CONTAINERS__##
##################
containers:
#supports multiple values
  nginx:
    repository: nginx
    tag: latest
    imagePullPolicy: Always
    securityContext: {}
#    securityContext:
#      runAsUser: 1001
#      runAsNonRoot: true
#      readOnlyRootFilesystem: false
    livenessProbe: {}
#    livenessProbe:
#      httpGet:
#        path: /
#        port: 80
#      initialDelaySeconds: 3
#      periodSeconds: 3
    readinessProbe: {}
    startupProbe: {}
    command: []
    args: []
    lifecycle: {}
#    lifecycle:
#      postStart:
#         exec:
#           command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
#      preStop:
#         exec:
#           command: ["/bin/sh","-c","nginx -s quit; while killall -0 nginx; do sleep 1; done"]
    ports:
    #supports multiple values
      - name: http
        containerPort: 80
    requests:
      memory: "64Mi"
      cpu: "0.1"
    limits:
      memory: "128Mi"
      cpu: "0.2"    
    env_vars: {}
    #supports multiple values
      ## - name: FOO
      ##  value: "bar"
    env_vars_cm: ""
    env_vars_secret: "nginx"
    volumeMounts:
    #supports multiple values
      - name: config-vol
        mountPath: /etc/nginx/nginx.conf
        subPath: nginx.conf
  sidecar-nginx:
    repository: nginx
    tag: latest
    imagePullPolicy: Always
    securityContext: {}
#    securityContext:
#      runAsUser: 1001
#      runAsNonRoot: true
#      readOnlyRootFilesystem: false
    livenessProbe: {}
#    livenessProbe:
#      httpGet:
#        path: /
#        port: 80
#      initialDelaySeconds: 3
#      periodSeconds: 3
    readinessProbe: {}
    startupProbe: {}
    command: []
    args: []
    lifecycle: {}
#    lifecycle:
#      postStart:
#         exec:
#           command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
#      preStop:
#         exec:
#           command: ["/bin/sh","-c","nginx -s quit; while killall -0 nginx; do sleep 1; done"]
    ports:
    #supports multiple values
      - name: http
        containerPort: 8080
    requests:
      memory: "32Mi"
      cpu: "0.05"
    limits:
      memory: "64Mi"
      cpu: "0.1"    
    env_vars: {}
    #supports multiple values
      ## - name: FOO
      ##  value: "bar"
    env_vars_cm: ""
    env_vars_secret: ""  
    volumeMounts:
    #supports multiple values
      - name: config-vol
        mountPath: /etc/nginx/nginx.conf
        subPath: sidecar-nginx.conf


######################
##__INITCONTAINERS__##
######################
initcontainers:
#supports multiple values
#  init-nginx:
#    repository: busybox
#    tag: 1.28
#    imagePullPolicy: Always
#    securityContext: {}
#    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 30']
#    args: []
#    requests: {}
#      memory: "32Mi"
#      cpu: "0.05"
#    limits: {}
#      memory: "64Mi"
#      cpu: "0.1"    
#    env_vars: []
#    env_vars_cm: ""
#    env_vars_secret: ""  
#    volumeMounts: {}


################
##__SERVICES__##
################
services:
#supports multiple values
#  nginx:
#    type: NodePort
#    prometheus_metrics: {}
#    ports:
#      - port: 80
#        name: foo
#        targetPort: 80
#        nodePort: 30001
#      - port: 8080
#        name: boo
#        targetPort: 8080
#        nodePort: 30002
  nginx:
    type: ClusterIP
    prometheus_metrics:
      metrics_path: "/metrics"
      metrics_port: 5000
    ports:
      - name: foo
        port: 80
        targetPort: 80
      - name: boo
        port: 8080
        targetPort: 8080


###############
##__INGRESS__##
###############
ingress:
  #supports multiple values
  nginx:
    class: traefik
    annotations:
      traefik.ingress.kubernetes.io/router.tls: "false"
    host:
    #supports multiple values
      - name: "nginx.k3s.local"
        tls:
          enabled: false
        paths:
        #supports multiple values
          - path: "/"
            serviceName: nginx
            servicePort: 80
          - path: "/sidecar"
            pathType: ImplementationSpecific
            serviceName: nginx
            servicePort: 8080


##################
##__CONFIGMAPS__##
##################
configmaps:
#supports multiple values
  foo.conf: dGVzdHRlc3Q= #base64
  boo.conf: dGVzdHRlc3Q= #base64


###############
##__SECRETS__##
###############
secrets:
#supports multiple values, 
  foo: MeG@sEcReT
  boo: UbErSeCrEt


###############################
##__PERSISTENT_VOLUME_CLAIM__##
###############################
persistentVolumeClaim:
#supports multiple values
  first:
    accessModes: 
      - ReadWriteOnce
    storageCapasity: 100Mi
    storageClassName: local-path
  second:
    accessModes: 
      - ReadWriteOnce
    storageCapasity: 50Mi
    storageClassName: local-path


##################################
##__HORIZONTAL_POD_AUTOSCALING__##
##################################
autoscaling:
  minReplicas: 2
  maxReplicas: 4
  targetCPU: 60
  targetMemory: 80


############
##__JOBS__##
############
job:
#supports multiple values
  - name: test-job
    annotations: {}
#      "helm.sh/hook": post-install
#      "helm.sh/hook-weight": "-5"
#      "helm.sh/hook-delete-policy": hook-succeeded
    restartPolicy: "OnFailure"
    containers:
    #supports multiple values
      ubuntu:
        repository: ubuntu
        tag: latest
        imagePullPolicy: Always
        securityContext: {}
        command: ["bash"]
        args: ["-c",  "current=0; max=70; echo 1; echo 2; for((i=3;i<=max;)); do for((j=i-1;j>=2;)); do if [  `expr $i % $j` -ne 0 ] ; then current=1; else current=0; break; fi; j=`expr $j - 1`; done; if [ $current -eq 1 ] ; then echo $i; fi; i=`expr $i + 1`; done"]
        requests:
          memory: "32Mi"
          cpu: "0.1"
        limits:
          memory: "128Mi"
          cpu: "0.2"    
        env_vars: {}
        env_vars_cm: ""
        env_vars_secret: ""
        volumeMounts: {}


################
##__CRONJOBS__##
################
cronjob:
#supports multiple values
  - name: test-cronjob
    schedule: "*/10 * * * *"
    concurrencyPolicy: "Allow"
    successfulJobsHistoryLimit: 3
    failedJobsHistoryLimit: 1
    backoffLimit: 3
    activeDeadlineSeconds: 30
    restartPolicy: "OnFailure"
    containers:
    #supports multiple values
      first:
        repository: nginx
        tag: latest
        imagePullPolicy: Always
        securityContext: {}
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 10']
        args: []
        requests:
          memory: "32Mi"
          cpu: "0.05"
        limits:
          memory: "64Mi"
          cpu: "0.1"    
        env_vars: {}
        env_vars_cm: ""
        env_vars_secret: ""
        volumeMounts: {}
      second:
        repository: nginx
        tag: latest
        imagePullPolicy: Always
        securityContext: {}
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 10']
        args: []
        requests:
          memory: "32Mi"
          cpu: "0.05"
        limits:
          memory: "64Mi"
          cpu: "0.1"    
        env_vars: {}
        env_vars_cm: ""
        env_vars_secret: ""
        volumeMounts: {}
```