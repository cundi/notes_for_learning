# 容器特性

引用：https://blog.lizzie.io/linux-containers-in-500-loc.html

- `namespaces` are used to group kernel objects into different sets that can be accessed by specific process trees. For example, pid namespaces limit the view of the process list to the processes within the namespace. There are a couple of different kind of namespaces. I'll go into this more later.
- `capabilities` are used here to set some coarse limits on what uid 0 can do.
- `cgroups` is a mechanism to limit usage of resources like memory, disk io, and cpu-time.
- `setrlimit` is another mechanism for limiting resource usage. It's older than cgroups, but can do some things cgroups can't.

## 编写和运行

运行：

```shell
[lizzie@empress l-c-i-500-l]$ sudo ./contained -m ~/misc/busybox-img/ -u 0 -c /bin/sh
=> validating Linux version...4.7.10.201610222037-1-grsec on x86_64.
=> setting cgroups...memory...cpu...pids...blkio...done.
=> setting rlimit...done.
=> remounting everything with MS_PRIVATE...remounted.
=> making a temp directory and a bind mount there...done.
=> pivoting root...done.
=> unmounting /oldroot.oQ5jOY...done.
=> trying a user namespace...writing /proc/32627/uid_map...writing /proc/32627/gid_map...done.
=> switching to uid 0 / gid 0...done.
=> dropping capabilities...bounding...inheritable...done.
=> filtering syscalls...done.
/ # whoami
root
/ # hostname
05fe5c-three-of-pentacles
/ # exit
=> cleaning cgroups...done.
```

编写`contained.c`，其代码结构如下:

```c
/* -*- compile-command: "gcc -Wall -Werror -lcap -lseccomp contained.c -o contained" -*- */
/* This code is licensed under the GPLv3. You can find its text here:
   https://www.gnu.org/licenses/gpl-3.0.en.html */


#define _GNU_SOURCE
#include <errno.h>
#include <fcntl.h>
#include <grp.h>
#include <pwd.h>
#include <sched.h>
#include <seccomp.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <unistd.h>
#include <sys/capability.h>
#include <sys/mount.h>
#include <sys/prctl.h>
#include <sys/resource.h>
#include <sys/socket.h>
#include <sys/stat.h>
#include <sys/syscall.h>
#include <sys/utsname.h>
#include <sys/wait.h>
#include <linux/capability.h>
#include <linux/limits.h>

struct child_config {
    int argc;
    uid_t uid;
    int fd;
    char *hostname;
    char **argv;
    char *mount_dir;
};

<<capabilities>>

<<mounts>>

<<syscalls>>

<<resources>>

<<child>>

<<choose-hostname>>

int main (int argc, char **argv)
{
	struct child_config config = {0};
	int err = 0;
	int option = 0;
	int sockets[2] = {0};
	pid_t child_pid = 0;
	int last_optind = 0;
	while ((option = getopt(argc, argv, "c:m:u:"))) {
		switch (option) {
		case 'c':
			config.argc = argc - last_optind - 1;
			config.argv = &argv[argc - config.argc];
			goto finish_options;
		case 'm':
			config.mount_dir = optarg;
			break;
		case 'u':
			if (sscanf(optarg, "%d", &config.uid) != 1) {
				fprintf(stderr, "badly-formatted uid: %s\n", optarg);
				goto usage;
			}
			break;
		default:
			goto usage;
		}
		last_optind = optind;
	}
finish_options:
	if (!config.argc) goto usage;
	if (!config.mount_dir) goto usage;

<<check-linux-version>>

	char hostname[256] = {0};
	if (choose_hostname(hostname, sizeof(hostname)))
		goto error;
	config.hostname = hostname;

<<namespaces>>

	goto cleanup;
usage:
	fprintf(stderr, "Usage: %s -u -1 -m . -c /bin/sh ~\n", argv[0]);
error:
	err = 1;
cleanup:
	if (sockets[0]) close(sockets[0]);
	if (sockets[1]) close(sockets[1]);
	return err;
}
```