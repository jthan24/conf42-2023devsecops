# Docker vul steps
## Create an image for Java App
- Create and usage a Java App
```bash
git clone https://github.com/dockersamples/spring-petclinic.git
cd spring-petclinic

## Create a docker image 
docker build -t petclinic-app . -f Dockerfile
docker run -p 8080:8080 -p 8000:8000 -it --rm --name petclinic-app petclinic-app

## Create a docker image with multi-stage
docker build -t petclinic-app-mini . -f Dockerfile.multi
docker run -p 8080:8080 -p 8000:8000 -it --rm --name petclinic-app-mini petclinic-app-mini
```

## First scan
- Validate docker config: Docker bench
```bash
docker run --rm --net host --pid host --userns host --cap-add audit_control \
-e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
-v /etc:/etc:ro \
-v /usr/bin/containerd:/usr/bin/containerd:ro \
-v /usr/bin/runc:/usr/bin/runc:ro \
-v /usr/lib/systemd:/usr/lib/systemd:ro \
-v /var/lib:/var/lib:ro \
-v /var/run/docker.sock:/var/run/docker.sock:ro \
--label docker_bench_security \
docker/docker-bench-security
```
## Misconfiguration Detection (trivy)
- Detect on images created
```bash
trivy image petclinic-app -f json > trivy-petclinic-app.json
trivy image petclinic-app-mini -f json > trivy-petclinic-app-mini.json
```
## Vulnerability Detection (snyk)
- snyk
```bash
mkdir snyk && cd snyk
curl https://static.snyk.io/cli/latest/snyk-linux -o snyk
chmod +x ./snyk
PATH=$(pwd):$PATH # Adding snyk to our path
mv ./snyk /usr/local/bin/ ## Optional 
snyk auth # Just one time

snyk container test petclinic-app:latest --org=61a695af-31ad-xxxx-xxxx-xxxxx
snyk container test petclinic-app-mini:latest --org=61a695af-31ad-xxxx-xxxx-xxxxx
```

- Clair (unstable)
```bash
git clone https://github.com/quay/clair.git
cd clair
export CLAIR_CONF=$(pwd)/config.yaml.sample 
export CLAIR_MODE=combo
docker-compose up
```

## Supply Chain Security 
- Verify image provenance: Use image signing and verification techniques to ensure that container images are obtained from trusted sources and have not been tampered with.
- Scan container images: Regularly scan container images for vulnerabilities and misconfigurations using automated tools to identify and remediate issues before deployment.
- Implement a strong CI/CD pipeline: Follow security best practices for the CI/CD pipeline, including access control, code reviews, and vulnerability scanning.
- Educate development teams: Provide training and awareness programs to developers on supply chain security risks and best practices for building secure container images.
- Monitor for suspicious activity: Continuously monitor containerized applications and the CI/CD pipeline for suspicious activity and implement automated responses to detect and mitigate potential attacks.
## IAM Enforcement
- Least Privilege Principle: Enforce the principle of least privilege, granting users only the permissions necessary to perform their tasks, minimizing the attack surface.
- Centralized IAM: Implement centralized IAM systems to manage user identities, access policies, and permissions across the entire containerized environment.
- Continuous Authorization: Continuously evaluate user access permissions based on changing conditions and security risks to ensure that access remains aligned with organizational policies.
- Audit and Monitoring: Regularly audit IAM logs and monitor user activity to detect anomalous behavior and potential unauthorized access attempts.
- Automate IAM Tasks: Automate repetitive IAM tasks, such as provisioning and deprovisioning user accounts and managing access permissions, to streamline operations and reduce human error.


# References
- https://github.com/dockersamples/spring-petclinic-docker
- https://github.com/quay/clair/blob/main/Documentation/howto/testing.md

