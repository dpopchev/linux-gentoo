# Sudo

## Quickstart

```
emerge -v sudo
```

## Usage

Edit so to allow members of wheel group execute any command with `visudo`.

```
...
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL:ALL) ALL
...
```

Allow more consecutive authentication failures before `faillock` kick you out by editing `/etc/security/faillock.conf`

```
...
# Deny access if the number of consecutive authentication failures
# for this user during the recent interval exceeds n tries.
# The default is 3.
deny = 10
...
```

Grant access to trusted users by adding them into `wheel` group.
