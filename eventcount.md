from https://locklessinc.com/articles/obscure_synch/

# Eventcounts

One rather useful synchronization primitive is an "eventcount". It is somewhat like the reverse of a seq-lock, where the waiter blocks if no signals have occurred between two points in time, rather than the converse. This primitive can be used to construct "almost lock free" data structures, that rarely call into the kernel. The trick is making sure the signal/broadcast functions are wait free in the common case where there are no waiters, and making the wait-check operation as cheap as possible.

Like other primitives, this one fits inside a 32bit Futex integer. Two billion or so possible signals between checks should be enough. (A similar amount is used to prevent the ABA bug in lock free linked lists.) We will use the bottom bit to denote whether there are waiters or not, just like the countdown event object.

```
typedef struct eventcount eventcount;
struct eventcount
{
	unsigned e;
};
```

The broadcast and signal operations are quite similar. The first needs to wake all waiters, whereas the second just one. However, we can quickly check if no waiters exist, and avoid making the relatively slow syscall into the kernel. The signal operation is quite useful in preventing the "thundering herd" issue from occurring.

```
void ec_broadcast(eventcount *e)
{
	/* Increment sequence counter */
	unsigned v = atomic_add(&e->e, 2);
	
	/* No waiters? */
	if (!v & 1) return;
	
	/* No more waiters, clear lsb */
	atomic_clear_bit(&e->e, 0);
	
	/* Wake everyone */
	sys_futex(e, FUTEX_WAKE_PRIVATE, INT_MAX, NULL, NULL, 0);
}

void ec_signal(eventcount *e)
{
	/* Increment sequence counter */
	unsigned v = atomic_add(&e->e, 2);
	
	/* No waiters? */
	if (!v & 1) return;
	
	/* Assume no more waiters, clear lsb */
	atomic_clear_bit(&e->e, 0);
	
	/* Wake someone */
	sys_futex(e, FUTEX_WAKE_PRIVATE, 1, NULL, NULL, 0);
}
```

The first part of the waiter code needs to determine the "key" used to represent the instance in time. To do this, we use the integer value of the object. As an optimization, we set the lower bit to make comparisons faster. (We don't care about the state of that bit in the waiter code.)

```
unsigned ec_getkey(eventcount *e)
{
	unsigned v;
	
	/* Prevent this from being reordered by the compiler */
	barrier();
	
	/* Set wait-bit in key, which makes comparison easier */
	v = e->e | 1;
	
	/* Prevent this from being reordered by the compiler */
	barrier();
	
	return v;
}
```

The most complex part of the code is in the wait function. It needs to determine whether or not there have been any signals (or broadcasts) since the thread has obtained its key. We can do that by checking the integer against our key. Again, we don't care about the least significant bit, so we make sure that is set to match what we have done in the ec_getkey() function. If there have been no relevant changes, then we drop into the wait loop.

```
void ec_wait(eventcount *e, unsigned key)
{
	unsigned v;
	
	/* Prevent out-of order reads */
	barrier();
	
	/* Compare with wait-bit set */
	v = e->e | 1;
		
	/* Fastpath - notice that the event has been signaled */
	if (v != key) return;
		
	while (1)
	{
		/* We want to wait, or make sure wait-bit is set for other waiters after ec_signal() */
		atomic_set_bit(&e->e, 0);
		
		/* Get new value */
		v = e->e | 1;
		
		/* Has the event has been signaled? */
		if (v != key) return;
		
		/* Wait for event to be triggered */
		sys_futex(e, FUTEX_WAIT_PRIVATE, v, NULL, NULL, 0);
	}
}
```
