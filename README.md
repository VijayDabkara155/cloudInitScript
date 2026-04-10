# cloudInitScript
yaml file for install docker engine docker compose k3d and kubectl and create a cluster with 1 cp and 2 agents

---------------------------------------------------------------------------------------------------------------

#cloud-config

package_update: true
package_upgrade: true

runcmd:
  # Install required packages
  - apt-get update -y
  - apt-get install -y ca-certificates curl gnupg lsb-release

  # Install Docker
  - mkdir -p /etc/apt/keyrings
  - curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  - echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

  - apt-get update -y
  - apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

  # Start & enable Docker
  - systemctl enable docker
  - systemctl start docker

  # Install kubectl
  - curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  - install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

  # Install k3d
  - curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

  # Wait for Docker to be ready
  - sleep 10

  # Create k3d cluster (1 control-plane + 2 agents)
  - k3d cluster create mycluster --agents 2

  # Export kubeconfig
  - mkdir -p /root/.kube
  - k3d kubeconfig get mycluster > /root/.kube/config

  # Test cluster
  - kubectl get nodes
