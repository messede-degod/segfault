all: albuild fs-root/bin/docker-exec-sigproxy fs-root/bin/unix-socket-client fs-root/usr/sbin/sshd Dockerfile
	docker build --no-cache --network host -t sf-host .

albuild:
	bash -c "docker run --rm alpine-gcc true || \
		docker commit alpine-gcc alpine-gcc || { \
		docker run --network host --name alpine-gcc alpine sh -c 'apk update && apk add gcc patch libc-dev musl-dev zlib-dev openssl-dev make linux-headers libcap-dev bash' \
		&& docker commit alpine-gcc alpine-gcc; }"

# See mk_sshd.sh for manual debugging
fs-root/usr/sbin/sshd: sf-sshd.patch mk_sshd.sh
	docker run --rm -v$$(pwd):/src --net=host -w /tmp alpine-gcc /src/mk_sshd.sh

fs-root/bin/docker-exec-sigproxy: docker-exec-sigproxy.c
	docker run --rm -v$$(pwd):/src -w /src alpine-gcc gcc -Wall -O2 -o fs-root/bin/docker-exec-sigproxy docker-exec-sigproxy.c
	@echo SUCCESS

fs-root/bin/unix-socket-client: unix-socket-client.c
	docker run --rm -v$$(pwd):/src -w /src alpine-gcc gcc -Wall -O2 -o fs-root/bin/unix-socket-client unix-socket-client.c
	@echo SUCCESS

diff:
	cd dev && \
	diff -x '!*.[ch]' -u   openssh-9.2p1-orig/  openssh-9.2p1-sf/  | grep -Ev ^"(Only in|Common)" >../sf-sshd.patch

clean:
	rm -rf openssh-9.2p1-sf fs-root/usr/sbin/sshd
	docker image rm alpine-gcc

