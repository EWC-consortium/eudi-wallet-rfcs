CONTAINER_SWAGGER ?= "openapi"

TERM_FLAGS ?= -ti

EXTRA_RUN_ARGS ?=

CONTAINER_DEFAULT_RUN_FLAGS := \
	--rm $(TERM_FLAGS) \
	$(EXTRA_RUN_ARGS) \
	-v $(CURDIR):/usr/share/nginx/html:ro

.DEFAULT_GOAL := help
.PHONY: help
help:
	@echo "------------------------------------------------------------------------"
	@echo "EWC RFC OpenAPI specification"
	@echo "------------------------------------------------------------------------"
	@grep -E '^[0-9a-zA-Z_/%\-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

run: ## Run dashboard locally for development purposes
	docker run \
		$(CONTAINER_DEFAULT_RUN_FLAGS) \
		-p 5000:80 \
		--name "${CONTAINER_SWAGGER}" \
		nginx:alpine

check-swagger-cli:
	@which swagger-cli > /dev/null || (echo "\033[31mPlease install swagger-cli before proceeding.\033[0m" && exit 1)

bundle: check-swagger-cli ## Bundles swagger files
	swagger-cli bundle index.yaml -o ./bundled.yaml -t yaml
