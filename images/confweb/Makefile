REPOSITORY_HOST=asia.gcr.io
OCTAV_CONFIG_NAME=octav-dev
APPDIR=conf.builderscon.io
APPNAME=confapp
PROJECT_ID=$(shell gcloud config list 2>&1 | grep 'project = ' | sed -e 's/project = //')

FILES=$(shell find $(APPDIR) -not -wholename '*/.git*' -type f)

tag:
	$(eval TAG := $(shell cd $(APPDIR) && git describe --tags >& /dev/null))
ifeq ($(TAG), )
	$(eval TAG := $(shell cd $(APPDIR) && git log -1 --pretty=format:%h))
endif
	$(eval TAG := $(TAG)-$(shell shasum config.json | cut -c1-7))
	$(eval IMAGE_NAME_BASE :=$(REPOSITORY_HOST)/$(PROJECT_ID)/$(APPNAME))
	$(eval IMAGE_NAME_LATEST :=$(IMAGE_NAME_BASE):latest)
	$(eval IMAGE_NAME :=$(IMAGE_NAME_BASE):$(TAG))
	@echo "Using tag '$(TAG)'"

gcloud-config:
	@echo "Activating configuration $(OCTAV_CONFIG_NAME)"
	@gcloud config configurations activate $(OCTAV_CONFIG_NAME)

docker-image: $(FILES) Dockerfile config.json
	@echo "Building image octav/$(APPNAME)"
	@cp config.json $(APPDIR)
	@./scripts/build-docker.sh

publish: tag docker-image gcloud-config
	@echo "Publishing [ $(IMAGE_NAME) ]"
	@docker tag octav/$(APPNAME) $(IMAGE_NAME)
	@echo " --> pushing $(IMAGE_NAME)..."
	@docker tag $(IMAGE_NAME) $(IMAGE_NAME_LATEST)
	@gcloud docker push $(IMAGE_NAME)
	@gcloud docker push $(IMAGE_NAME_LATEST)

deploy: tag deployment.yaml
	@kubectl apply -f deployment.yaml
	@kubectl apply -f service.yaml