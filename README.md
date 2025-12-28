# Jenkins Inbound Agent with Docker

Custom Docker image for Jenkins inbound agents with Docker-in-Docker capabilities.

## Features
- Based on `jenkins/inbound-agent:jdk21`
- Docker CLI, Buildx, and Compose pre-installed
- Multi-architecture support (AMD64 + ARM64)
- Ready for Docker-in-Docker (DinD) workflows

## Usage

### Using the Pre-built Image

Pull the image from Docker Hub:
```bash
docker pull ossrobo/jenkins-inbound-agent-dind:jdk21
```

### Using in Jenkins Kubernetes Cloud

Add this to your Jenkins Configuration as Code (JCasC):
```yaml
jenkins:
  clouds:
  - kubernetes:
      templates:
      - containers:
        - name: "jenkins-agent"
          image: "ossrobo/jenkins-inbound-agent-dind:jdk21"
          envVars:
          - envVar:
              key: "DOCKER_HOST"
              value: "tcp://localhost:2376"
          - envVar:
              key: "DOCKER_TLS_CERTDIR"
              value: "/certs"
          - envVar:
              key: "DOCKER_CERT_PATH"
              value: "/certs/client"
          - envVar:
              key: "DOCKER_TLS_VERIFY"
              value: "1"
        - name: "dind"
          image: "docker:dind"
          privileged: true
          envVars:
          - envVar:
              key: "DOCKER_TLS_CERTDIR"
              value: "/certs"
        yaml: |-
          spec:
            containers:
            - name: jenkins-agent
              volumeMounts:
              - name: docker-certs
                mountPath: /certs
                readOnly: true
            - name: dind
              volumeMounts:
              - name: docker-certs
                mountPath: /certs
              startupProbe:
                exec:
                  command:
                  - docker
                  - info
                initialDelaySeconds: 5
                periodSeconds: 2
                failureThreshold: 30
            volumes:
            - name: docker-certs
              emptyDir: {}
```

### Running Docker Commands in Jenkins Jobs

Once configured, your Jenkins Freestyle or Pipeline jobs can use Docker commands:
```bash
#!/bin/bash
docker build -t myapp:latest .
docker push myapp:latest
```

## Building

### Build for Single Architecture
```bash
docker build -t ossrobo/jenkins-inbound-agent-dind:jdk21 .
docker push ossrobo/jenkins-inbound-agent-dind:jdk21
```

### Build for Multiple Architectures (AMD64 + ARM64)
```bash
# Create a new builder instance
docker buildx create --name multiarch --use

# Build and push for both architectures
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t ossrobo/jenkins-inbound-agent-dind:jdk21 \
  --push \
  .
```

### Verify Multi-Architecture Build
```bash
docker buildx imagetools inspect ossrobo/jenkins-inbound-agent-dind:jdk21
```

You should see both `linux/amd64` and `linux/arm64` in the output.

## License
MIT License - see LICENSE file
