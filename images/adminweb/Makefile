REPOSITORY_HOST=asia.gcr.io
OCTAV_CONFIG_NAME=octav-dev
APPDIR=admin.builderscon.io
APPNAME=adminweb
PROJECT_ID=$(shell gcloud config list 2>&1 | grep 'project = ' | sed -e 's/project = //')


tag:
	$(eval TAG := $(shell cd $(APPDIR) && git describe --tags >& /dev/null))
ifeq ($(TAG), )
	$(eval TAG := $(shell cd $(APPDIR) && git log -1 --pretty=format:%h))
endif
	$(eval TAG := $(TAG)-$(shell cd $(APPDIR) && (git diff | shasum | cut -c1-7))-$(shell shasum scripts/run.sh | cut -c1-7))
	$(eval TAG := $(TAG)-$(shell cat Dockerfile | shasum | cut -c1-7))
	$(eval IMAGE_NAME_BASE :=$(REPOSITORY_HOST)/$(PROJECT_ID)/$(APPNAME))
	$(eval IMAGE_NAME_LATEST :=$(IMAGE_NAME_BASE):latest)
	$(eval IMAGE_NAME :=$(IMAGE_NAME_BASE):$(TAG))
	@echo "Using tag '$(TAG)'"

$(APPDIR):
	@git clone git@github.com:builderscon/admin.builderscon.io.git
	$(eval FILES:=$(shell find $(APPDIR) -not -wholename '*/.git*' -type f))

gcloud-config:
	@echo "Activating configuration $(OCTAV_CONFIG_NAME)"
	@gcloud config configurations activate $(OCTAV_CONFIG_NAME)

latest:
	@make $(APPDIR)
	pushd $(APPDIR) && git pull && popd
	$(eval FILES:=$(shell find $(APPDIR) -not -wholename '*/.git*' -type f))

docker: latest $(APPDIR) $(FILES) Dockerfile
	@echo "Building image octav/$(APPNAME)"
	@docker build -t octav/$(APPNAME) .

publish: tag docker gcloud-config
	@echo "Publishing [ $(IMAGE_NAME) ]"
	@docker tag octav/$(APPNAME) $(IMAGE_NAME)
	@echo " --> pushing $(IMAGE_NAME)..."
	@docker tag $(IMAGE_NAME) $(IMAGE_NAME_LATEST)
	@gcloud docker -- push $(IMAGE_NAME)
	@gcloud docker -- push $(IMAGE_NAME_LATEST)

clean:
	# NO OP
