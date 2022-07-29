FROM public.ecr.aws/bitnami/python:3.6

# Create app directory
WORKDIR /


# Install app dependencies
RUN apt update
RUN apt install curl zip -y
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install
RUN curl https://helm-charts-tinystacks.s3.us-west-1.amazonaws.com/helm3.sh | bash -s -- --version v3.8.2
RUN curl -LO https://dl.k8s.io/release/v1.22.0/bin/linux/amd64/kubectl
RUN curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
RUN mv /tmp/eksctl /usr/local/bin
RUN chmod +x kubectl && mv kubectl /usr/local/bin/

ENTRYPOINT aws eks update-kubeconfig --name "$EKS_CLUSTER_NAME" --region "$STAGE_REGION" --role-arn "$EKS_ROLE_ARN" && \
    eksctl utils associate-iam-oidc-provider --cluster "$EKS_CLUSTER_NAME" --approve && \
    eksctl create iamserviceaccount --cluster="$EKS_CLUSTER_NAME"  --namespace=kube-system --name=lb-contrl --role-name "AmazonEKSLoadBalancerControllerRole" --attach-policy-arn="$LB_POLICY_ARN" --approve --override-existing-serviceaccounts && \
    helm repo add eks https://aws.github.io/eks-charts && \
    helm repo update && \
    helm upgrade --install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$EKS_CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=lb-contrl --set "serviceAccount.name=lb-contrl" && \
    helm upgrade --install $HELM_CHART $HELM_REPOSITORY$HELM_CHART-$HELM_VERSION.tgz
