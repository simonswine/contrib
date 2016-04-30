all: push

# 0.0 shouldn't clobber any release builds
TAG = 0.5
PREFIX = gcr.io/google_containers/nginx-ingress-controller

ifndef REPO_INFO
  REPO_INFO=$(shell git config --get remote.origin.url)
endif

ifndef VERSION
  VERSION := git-$(shell git rev-parse --short HEAD)
endif

APP_NAME=nginx-ingress-controller
PACKAGE_NAME=k8s.io/contrib/ingress/controllers/nginx
GO_VERSION=1.6
CONTAINER_DIR=/go/src/${PACKAGE_NAME}

controller: controller.go clean
	CGO_ENABLED=0 GOOS=linux godep go build -a -installsuffix cgo -ldflags \
		"-w -X main.version=${VERSION} -X main.gitRepo=${REPO_INFO}" \
		-o nginx-ingress-controller

container: controller
	docker build -t $(PREFIX):$(TAG) .

push: container
	gcloud docker push $(PREFIX):$(TAG)

clean:
	rm -f nginx-ingress-controller

prepare:
	which godep || go get github.com/tools/godep

docker_clean: 
	docker stop -t 0 $(APP_NAME) || true
	docker rm $(APP_NAME) || true

docker__controller: docker_clean
	docker create \
		-i \
		--name $(APP_NAME) \
		-w $(CONTAINER_DIR) \
		golang:${GO_VERSION} \
		/bin/sleep 3600
	
	docker cp . $(APP_NAME):$(CONTAINER_DIR)/
	docker cp ../../Godeps $(APP_NAME):$(CONTAINER_DIR)/Godeps
	
	docker start $(APP_NAME)
	docker exec $(APP_NAME) make prepare VERSION=$(VERSION) REPO_INFO=$(REPO_INFO)
	docker exec $(APP_NAME) make controller VERSION=$(VERSION) REPO_INFO=$(REPO_INFO)
	docker cp $(APP_NAME):$(CONTAINER_DIR)/nginx-ingress-controller .

docker_controller:
	${MAKE} docker__controller || ${MAKE} docker_clean
