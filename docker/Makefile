all: build

IMAGE:=regard3d

help:
	@echo "Regard3D Makefile"

.PHONY: build
build:
	@docker build -t $(IMAGE) .

.PHONY: run
run:
	@xhost +local:root # for the lazy and reckless
	@docker run --rm -it \
		--env="DISPLAY" \
		--volume="/etc/group:/etc/group:ro" \
		--volume="/etc/shadow:/etc/shadow:ro" \
		--volume="/etc/sudoers.d:/etc/sudoers.d:ro" \
		--volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
		--volume="/etc/passwd:/etc/passwd:ro" \
		$(IMAGE) /bin/bash
	@xhost -local:root

.PHONY: bash
bash:
	$(eval NAME?=$(shell docker container ls | grep $(IMAGE) | awk '{print $$NF}'))
	@docker exec -it $(NAME) bash
