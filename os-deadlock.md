
Level 1:
![[lvl1.png]]

Level 3:
![[lvl3.png]]


Level 4:
Beta
![[lvl4-beta.png]]
Alpha
![[lvl4-alpha.png]]


## 🔹 Level 1: Virtual Vault Provisioning

We created two virtual disk files using `dd` and formatted them with `ext4`. Using `udisksctl`, we attached them as loop devices and mounted them without root access.

The command:

```
df -h | grep loop
```

confirms the drives are mounted, meaning the virtual disks are working like real storage.

---

## 🔹 Level 2: The Naive Sync

We created two scripts (`sync_up` and `sync_down`) that use `flock` to lock vaults before syncing data.

- `sync_up`: locks Alpha → then Beta
- `sync_down`: locks Beta → then Alpha

Because they lock resources in different orders, they can cause a deadlock.

---

## 🔹 Level 3: Local Deadlock

When both scripts run at the same time:

- `sync_up` locks Alpha and waits for Beta
- `sync_down` locks Beta and waits for Alpha

Both processes wait forever, causing a deadlock (circular wait). The system appears frozen.

---

## 🔹 Level 4: Distributed Deadlock

We extended the deadlock across two users by sharing lock files.

Each user locks their own vault first, then tries to lock the other user’s vault.

This creates a distributed deadlock where both users are stuck, simulating a denial-of-service scenario across the system.