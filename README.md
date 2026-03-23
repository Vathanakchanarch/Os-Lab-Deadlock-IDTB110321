# Distributed Synchronization & Deadlock Lab

## Overview

This project demonstrates how deadlocks occur in distributed systems during file synchronization and how they can be prevented and recovered using different operating system techniques.

The virtual vaults are implemented using loopback devices, allowing image files to behave like real storage devices. 

---

## Level 1: Virtual Vault Provisioning

### Observation Screenshot

<img width="975" height="360" alt="image" src="https://github.com/user-attachments/assets/61558668-ee1f-488b-bcb8-703b6dd9bbbd" />


### Analysis

The output shows that the loop devices are mounted and available in the filesystem, meaning the virtual disk images are working like real storage devices.

---

## Level 3: Local Deadlock

### Observation Screenshot

<img width="975" height="457" alt="image" src="https://github.com/user-attachments/assets/912cbe39-b513-4b9b-afb1-4d7d2dd47501" />
<img width="975" height="506" alt="image" src="https://github.com/user-attachments/assets/5c5e8524-895d-4ce9-8c20-6a97fe85f258" />
<img width="975" height="272" alt="image" src="https://github.com/user-attachments/assets/aa7bbf51-cb7b-4c3b-b431-5fdf5debfd13" />



### Analysis

The system entered a deadlock when both synchronization scripts were executed simultaneously.

* `sync_up` successfully locked Vault Alpha and then attempted to lock Vault Beta
* `sync_down` successfully locked Vault Beta and then attempted to lock Vault Alpha

At this point:

* `sync_up` is holding Vault Alpha and waiting for Vault Beta
* `sync_down` is holding Vault Beta and waiting for Vault Alpha

Since neither process releases its first lock while waiting for the second, both processes become stuck indefinitely due to a circular wait condition.

---

## Level 4: Site-to-Site Sync Deadlock

### Observation Screenshot
<img width="975" height="541" alt="image" src="https://github.com/user-attachments/assets/3f8b53fa-e583-475b-a3d9-67838ba2e804" />


### Analysis

This test simulated a distributed deadlock between two separate user accounts on the same server.

Player A locked its own local vault file first and then attempted to lock Player B’s vault file. At the same time, Player B locked its own local vault file first and then attempted to lock Player A’s vault file.

As a result:

* Player A held the Alpha lock and waited for the Beta lock
* Player B held the Beta lock and waited for the Alpha lock

Neither process could continue, so both became stuck indefinitely.

This simulates a distributed denial of service because two different systems become blocked while waiting on each other’s resources.

---

## Level 5: Global Resource Ordering Patch

### Observation Screenshot

<img width="975" height="535" alt="image" src="https://github.com/user-attachments/assets/e7b60626-709b-4ef7-8242-ecac6e177570" />


### Analysis

To prevent deadlock, both processes were forced to follow the same global lock order:

1. Lock Alpha first
2. Lock Beta second

Originally, Player A locked Alpha then Beta, while Player B locked Beta then Alpha. That opposite ordering created a circular wait condition.

After patching the scripts, both processes requested locks in the same order. This removed the circular wait condition because no process could hold Beta while waiting for Alpha.

Instead, one process waited safely until the first process completed and released its locks.

---

## Level 6: Deadlock Recovery with Timeout

### Observation Screenshot
<img width="975" height="574" alt="image" src="https://github.com/user-attachments/assets/b20f8643-0d85-4956-b8e7-fffb598db5a2" />


### Analysis

In this test, the `sync_timeout` script attempted to acquire a lock on Vault Alpha while it was already held by another process (`sync_up`).

Instead of waiting indefinitely, the script used `flock -w 5`, which limits the waiting time to 5 seconds. After failing to acquire the lock within this time, the script aborted and displayed an error message.

This demonstrates a timeout-based recovery mechanism. By preventing infinite waiting, the system avoids deadlock and allows processes to fail safely.

---

## Level 7: Safe Ejection (Teardown)

### Observation Screenshot
<img width="975" height="424" alt="image" src="https://github.com/user-attachments/assets/4128e0ef-aa4b-4d25-8b29-d043a7e9c0f5" />


### Analysis

After running the teardown script, the loop devices associated with the virtual vault images were successfully unmounted and removed.

Although some loop devices may still be visible, they belong to other users on the shared server. The absence of the user's own loop devices confirms that the teardown was successful.

Proper teardown is critical for system stability because mounted loopback devices consume kernel resources. If not properly unmounted, they can lead to resource leaks or filesystem corruption.

By unmounting the devices first and then deleting the loop mappings, all resources are released safely.

---

## Conclusion

This lab demonstrated how deadlocks occur due to improper resource locking and how they can be resolved using synchronization techniques.

Key takeaways:

* Deadlocks occur بسبب circular wait conditions
* Global resource ordering prevents deadlocks
* Timeouts provide a safe recovery mechanism
* Proper teardown ensures system stability

These techniques are essential for building reliable and scalable distributed systems.

---


