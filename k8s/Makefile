OS   := $(shell go env GOOS)
ARCH := $(shell go env GOARCH)

KUBERNETES_VERSION         := 1.23.0
ISTIO_VERSION              := 1.12.1
KIND_VERSION               := 0.11.1
INGRESS_NGINX_VERSION      := 1.1.0

BIN_DIR := $(shell pwd)/bin

KUBECTL                 := $(abspath $(BIN_DIR)/kubectl)
KIND                    := $(abspath $(BIN_DIR)/kind)

KIND_CLUSTER_NAME := job-echo-example

KUBECTL_CMD := KUBECONFIG=./.kubeconfig $(KUBECTL)
KIND_CMD    := $(KIND) --name $(KIND_CLUSTER_NAME) --kubeconfig ./.kubeconfig

kubectl: $(KUBECTL)
$(KUBECTL):
	curl -Lso $(KUBECTL) https://storage.googleapis.com/kubernetes-release/release/v$(KUBERNETES_VERSION)/bin/$(OS)/$(ARCH)/kubectl
	chmod +x $(KUBECTL)

istioctl: $(ISTIOCTL)
$(ISTIOCTL):
ifeq ($(OS)-$(ARCH), darwin-amd64)
	curl -sSL "https://storage.googleapis.com/istio-release/releases/$(ISTIO_VERSION)/istioctl-$(ISTIO_VERSION)-osx.tar.gz" | tar -C $(BIN_DIR) -xzv istioctl
else ifeq ($(OS)-$(ARCH), darwin-arm64)
	curl -sSL "https://storage.googleapis.com/istio-release/releases/$(ISTIO_VERSION)/istioctl-$(ISTIO_VERSION)-osx-arm64.tar.gz" | tar -C $(BIN_DIR) -xzv istioctl
else
	curl -sSL "https://storage.googleapis.com/istio-release/releases/$(ISTIO_VERSION)/istioctl-$(ISTIO_VERSION)-$(OS)-$(ARCH).tar.gz" | tar -C $(BIN_DIR) -xzv istioctl
endif

kind: $(KIND)
$(KIND):
	curl -Lso $(KIND) https://github.com/kubernetes-sigs/kind/releases/download/v$(KIND_VERSION)/kind-$(OS)-$(ARCH)
	chmod +x $(KIND)

.PHONY: cluster
cluster: $(KIND) $(KUBECTL) $(ISTIOCTL)
	$(KIND_CMD) delete cluster
	$(KIND_CMD) create cluster --image kindest/node:v${KUBERNETES_VERSION} --config ./kind.yml
	make nats
	make echo-job

.PHONY: nats
nats:
	$(KUBECTL_CMD) delete deploy -n nats --ignore-not-found app
	$(KUBECTL_CMD) apply --filename ./nats.yml

.PHONY: echo-job
nats:
	$(KUBECTL_CMD) delete deploy -n echo-job --ignore-not-found app
	$(KUBECTL_CMD) apply --filename ./job.yml


