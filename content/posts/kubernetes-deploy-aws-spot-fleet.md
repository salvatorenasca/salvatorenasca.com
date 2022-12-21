---
title: "Kubernetes Deploy Aws Spot Fleet"
date: 2022-12-21T16:27:22-03:00
draft: false
---

**This is an example of a Kubernetes manifest that you can use to deploy a spot fleet with high availability in AWS.**


{{< highlight yaml >}}

apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 8
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - my-app
              topologyKey: "kubernetes.io/hostname"
      containers:
      - name: my-app
        image: my-app:latest
        ports:
        - containerPort: 80
---
apiVersion: spotinst.com/v1
kind: Fleet
metadata:
  name: my-spot-fleet
spec:
  type: kubernetes
  minimum: 1
  maximum: 8
  resource:
    instanceTypes:
      - type: c5.large
      - type: m5.large
      - type: r5.large
    launchSpecifications:
      iamInstanceProfile: arn:aws:iam::1234567890:instance-profile/my-instance-profile
      securityGroupIds:
        - sg-12345678
      subnetIds:
        - subnet-12345678
  availability:
    availabilityZones:
      - us-west-2a
      - us-west-2b
      - us-west-2c
  policies:
    gracefulTermination:
      enabled: true
      expirationPeriod: 60
    healthCheck:
      enabled: true
      gracePeriod: 60
      interval: 30
    scaleDown:
      evaluationPeriods: 1
      maxScaleDownPercentage: 50
      policies:
        - policyName: scale-down-policy
          type: cloudWatchAlarm
          threshold: 30
          period: 300
          evaluationPeriods: 3
          operator: less-than-or-equal
          statistic: average
          dimensions:
            - name: AutoScalingGroupName
              value: "{{.AutoscalingGroupName}}"
          namespaces:
            - AWS/EC2
          metricName: CPUUtilization
          unit: Percent
  integration:
    enabled: true
    clusterName: my-cluster
    namespace: default


***This manifest will create a Kubernetes service that exposes a load balancer, a deployment with 8 replicas of the my-app application, and a spot fleet with a minimum of 1 and a maximum of 8 instances. The spot fleet will use c5.large, m5.large, and r5.large instance types and will be launched in the us-west-2a, us-west-2b, and us-west-2***
