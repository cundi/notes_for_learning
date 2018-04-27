# 附录

## select & poll's source code

select syscall:

```c
SYSCALL_DEFINE5(select, int, n, fd_set __user *, inp, fd_set __user *, outp,
        fd_set __user *, exp, struct timeval __user *, tvp)
{
    struct timespec64 end_time, *to = NULL;
    struct timeval tv;
    int ret;

    if (tvp) {
        if (copy_from_user(&tv, tvp, sizeof(tv)))
            return -EFAULT;

        to = &end_time;
        if (poll_select_set_timeout(to,
                tv.tv_sec + (tv.tv_usec / USEC_PER_SEC),
                (tv.tv_usec % USEC_PER_SEC) * NSEC_PER_USEC))
            return -EINVAL;
    }

    ret = core_sys_select(n, inp, outp, exp, to);
    ret = poll_select_copy_remaining(&end_time, tvp, 1, ret);

    return ret;
}
```

do_select:

```c
int do_select(int n, fd_set_bits *fds, struct timespec64 *end_time)
{
    ktime_t expire, *to = NULL;
    struct poll_wqueues table;
    poll_table *wait;
    int retval, i, timed_out = 0;
    u64 slack = 0;
    unsigned int busy_flag = net_busy_loop_on() ? POLL_BUSY_LOOP : 0;
    unsigned long busy_end = 0;

    rcu_read_lock();
    retval = max_select_fd(n, fds);
    rcu_read_unlock();

    if (retval < 0)
        return retval;
    n = retval;

    poll_initwait(&table);
    wait = &table.pt;
    if (end_time && !end_time->tv_sec && !end_time->tv_nsec) {
        wait->_qproc = NULL;
        timed_out = 1;
    }

    if (end_time && !timed_out)
        slack = select_estimate_accuracy(end_time);

    retval = 0;
    for (;;) {
        unsigned long *rinp, *routp, *rexp, *inp, *outp, *exp;
        bool can_busy_loop = false;

        inp = fds->in; outp = fds->out; exp = fds->ex;
        rinp = fds->res_in; routp = fds->res_out; rexp = fds->res_ex;

        for (i = 0; i < n; ++rinp, ++routp, ++rexp) {
            unsigned long in, out, ex, all_bits, bit = 1, mask, j;
            unsigned long res_in = 0, res_out = 0, res_ex = 0;

            in = *inp++; out = *outp++; ex = *exp++;
            all_bits = in | out | ex;
            if (all_bits == 0) {
                i += BITS_PER_LONG;
                continue;
            }

            for (j = 0; j < BITS_PER_LONG; ++j, ++i, bit <<= 1) {
                struct fd f;
                if (i >= n)
                    break;
                if (!(bit & all_bits))
                    continue;
                f = fdget(i);
                if (f.file) {
                    const struct file_operations *f_op;
                    f_op = f.file->f_op;
                    mask = DEFAULT_POLLMASK;
                    if (f_op->poll) {
                        wait_key_set(wait, in, out,
                                 bit, busy_flag);
                        mask = (*f_op->poll)(f.file, wait);
                    }
                    fdput(f);
                    if ((mask & POLLIN_SET) && (in & bit)) {
                        res_in |= bit;
                        retval++;
                        wait->_qproc = NULL;
                    }
                    if ((mask & POLLOUT_SET) && (out & bit)) {
                        res_out |= bit;
                        retval++;
                        wait->_qproc = NULL;
                    }
                    if ((mask & POLLEX_SET) && (ex & bit)) {
                        res_ex |= bit;
                        retval++;
                        wait->_qproc = NULL;
                    }
                    /* got something, stop busy polling */
                    if (retval) {
                        can_busy_loop = false;
                        busy_flag = 0;

                    /*
                     * only remember a returned
                     * POLL_BUSY_LOOP if we asked for it
                     */
                    } else if (busy_flag & mask)
                        can_busy_loop = true;

                }
            }
            if (res_in)
                *rinp = res_in;
            if (res_out)
                *routp = res_out;
            if (res_ex)
                *rexp = res_ex;
            cond_resched();
        }
        wait->_qproc = NULL;
        if (retval || timed_out || signal_pending(current))
            break;
        if (table.error) {
            retval = table.error;
            break;
        }

        /* only if found POLL_BUSY_LOOP sockets && not out of time */
        if (can_busy_loop && !need_resched()) {
            if (!busy_end) {
                busy_end = busy_loop_end_time();
                continue;
            }
            if (!busy_loop_timeout(busy_end))
                continue;
        }
        busy_flag = 0;

        /*
         * If this is the first loop and we have a timeout
         * given, then we convert to ktime_t and set the to
         * pointer to the expiry value.
         */
        if (end_time && !to) {
            expire = timespec64_to_ktime(*end_time);
            to = &expire;
        }

        if (!poll_schedule_timeout(&table, TASK_INTERRUPTIBLE,
                       to, slack))
            timed_out = 1;
    }

    poll_freewait(&table);

    return retval;
}
```

poll syscall:

```c
SYSCALL_DEFINE5(ppoll, struct pollfd __user *, ufds, unsigned int, nfds,
        struct timespec __user *, tsp, const sigset_t __user *, sigmask,
        size_t, sigsetsize)
{
    sigset_t ksigmask, sigsaved;
    struct timespec ts;
    struct timespec64 end_time, *to = NULL;
    int ret;

    if (tsp) {
        if (copy_from_user(&ts, tsp, sizeof(ts)))
            return -EFAULT;

        to = &end_time;
        if (poll_select_set_timeout(to, ts.tv_sec, ts.tv_nsec))
            return -EINVAL;
    }

    if (sigmask) {
        /* XXX: Don't preclude handling different sized sigset_t's.  */
        if (sigsetsize != sizeof(sigset_t))
            return -EINVAL;
        if (copy_from_user(&ksigmask, sigmask, sizeof(ksigmask)))
            return -EFAULT;

        sigdelsetmask(&ksigmask, sigmask(SIGKILL)|sigmask(SIGSTOP));
        sigprocmask(SIG_SETMASK, &ksigmask, &sigsaved);
    }

    ret = do_sys_poll(ufds, nfds, to);

    /* We can restart this syscall, usually */
    if (ret == -EINTR) {
        /*
         * Don't restore the signal mask yet. Let do_signal() deliver
         * the signal on the way back to userspace, before the signal
         * mask is restored.
         */
        if (sigmask) {
            memcpy(&current->saved_sigmask, &sigsaved,
                    sizeof(sigsaved));
            set_restore_sigmask();
        }
        ret = -ERESTARTNOHAND;
    } else if (sigmask)
        sigprocmask(SIG_SETMASK, &sigsaved, NULL);

    ret = poll_select_copy_remaining(&end_time, tsp, 0, ret);

    return ret;
}
```

do_poll:

```c
static int do_poll(struct poll_list *list, struct poll_wqueues *wait,
           struct timespec64 *end_time)
{
    poll_table* pt = &wait->pt;
    ktime_t expire, *to = NULL;
    int timed_out = 0, count = 0;
    u64 slack = 0;
    unsigned int busy_flag = net_busy_loop_on() ? POLL_BUSY_LOOP : 0;
    unsigned long busy_end = 0;

    /* Optimise the no-wait case */
    if (end_time && !end_time->tv_sec && !end_time->tv_nsec) {
        pt->_qproc = NULL;
        timed_out = 1;
    }

    if (end_time && !timed_out)
        slack = select_estimate_accuracy(end_time);

    for (;;) {
        struct poll_list *walk;
        bool can_busy_loop = false;

        for (walk = list; walk != NULL; walk = walk->next) {
            struct pollfd * pfd, * pfd_end;

            pfd = walk->entries;
            pfd_end = pfd + walk->len;
            for (; pfd != pfd_end; pfd++) {
                /*
                 * Fish for events. If we found one, record it
                 * and kill poll_table->_qproc, so we don't
                 * needlessly register any other waiters after
                 * this. They'll get immediately deregistered
                 * when we break out and return.
                 */
                if (do_pollfd(pfd, pt, &can_busy_loop,
                          busy_flag)) {
                    count++;
                    pt->_qproc = NULL;
                    /* found something, stop busy polling */
                    busy_flag = 0;
                    can_busy_loop = false;
                }
            }
        }
        /*
         * All waiters have already been registered, so don't provide
         * a poll_table->_qproc to them on the next loop iteration.
         */
        pt->_qproc = NULL;
        if (!count) {
            count = wait->error;
            if (signal_pending(current))
                count = -EINTR;
        }
        if (count || timed_out)
            break;

        /* only if found POLL_BUSY_LOOP sockets && not out of time */
        if (can_busy_loop && !need_resched()) {
            if (!busy_end) {
                busy_end = busy_loop_end_time();
                continue;
            }
            if (!busy_loop_timeout(busy_end))
                continue;
        }
        busy_flag = 0;

        /*
         * If this is the first loop and we have a timeout
         * given, then we convert to ktime_t and set the to
         * pointer to the expiry value.
         */
        if (end_time && !to) {
            expire = timespec64_to_ktime(*end_time);
            to = &expire;
        }

        if (!poll_schedule_timeout(wait, TASK_INTERRUPTIBLE, to, slack))
            timed_out = 1;
    }
    return count;
}
```