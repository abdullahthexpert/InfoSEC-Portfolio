# DAY 1

cough cough, so i finally started working on this pathway, got my ubuntu server in the VBox running, got win10 updated and installed, phew that is chonky.

so first what i did was run `sudo apt update` and `sudo apt upgrade`, as a habit of mine to check for updates and apply them!

next i started with the basic hardening procedures, i added a firewall to the server (UFW), enabling communication to go out, but disabling incoming requests, enabling OpenSSH, and last but not least enabling the firewall itself.

then i created another user named "flow" and added it to the sudo group, so it can login and do...yk sudo stuff. point of this is to stop using root for daily driving — root should only get invoked when actually needed, not be the default account.

then i edited the permissions for SSH, so not everyone can just log in and try guess the password, or brute force their way in. set `MaxAuthTries 3` so a connection gets kicked after 3 failed attempts instead of the default 6 — makes brute forcing slower and way less practical.

also disabled root login thru ssh (`PermitRootLogin no`), tho userlogin is possible but still minimal bruteforce protection.

## the detour: `/run/sshd` error

while testing the SSH config with `sudo sshd -t`, got hit with:

```
Missing privilege separation directory: /run/sshd
```

turns out this happens because `/run` is tmpfs — basically it lives in RAM and gets wiped every reboot. normally some startup script recreates `/run/sshd` automatically before sshd starts, but on this install that didn't happen. so sshd had nowhere to store its runtime files (privilege separation = sshd splits itself into a small privileged root process + a bigger unprivileged process, and needs this dir to coordinate between them).

fix:
```bash
sudo mkdir -p /run/sshd
sudo chmod 0755 /run/sshd
sudo chown root:root /run/sshd
```

**lesson learned:** typed `chmod 0775` instead of `0755` first try, and sshd refused to start with "must be owned by root and not group or world-writable." the `7` in the group slot gave the group write access, which sshd flat out rejects — if the group could write to that directory, it could theoretically be tampered with. `0755` (group = read+execute only, no write) is what it wants. one digit, big difference.

also: since `/run/sshd` resets on every reboot, if SSH refuses to start again after restarting the VM, this is why — same fix applies.

## habit i'm building

test before touching anything live. always run `sudo sshd -t` to validate the config *before* restarting the actual service — catches mistakes without ever risking breaking the SSH session i'm currently in. only restart once the test comes back clean:

```bash
sudo systemctl restart ssh
sudo systemctl status ssh
```

confirmed `active (running)` — day 1 hardening done.
## trimming services

after SSH was locked down, went through `sudo systemctl list-units --type=service --state=running` to see what was actually running. reasoning: less running = less stuff that can go wrong or get exploited, so anything not actually needed should get cut.

kept the obvious core stuff (dbus, polkit, systemd-journald/logind/networkd/resolved/timesyncd/udevd, ssh, getty@tty1, user session) since those are either core OS plumbing or things i actually need (like tty1 being my actual console login). kept unattended-upgrades too since free automatic security patching > convenience of manual control, at least for a lab box. kept snap/prometheus since that one's intentional.

disabled:
- `multipathd` — this manages multipath storage (multiple physical paths to the same disk, common in enterprise SAN setups). i'm running a single virtual disk in VBox, so this does literally nothing for me here. dead weight.
- `getty@tty6` — extra virtual console login prompt. don't need six potential login points on a server, only using tty1.

```bash
sudo systemctl stop multipathd
sudo systemctl disable multipathd
sudo systemctl stop getty@tty6
sudo systemctl disable getty@tty6
```

**lesson learned #2:** disabling `multipathd.service` wasn't actually enough — it still showed up because of `multipathd.socket`. this is "socket activation": systemd keeps a lightweight socket listening, and only spins the actual service back up the moment something tries to connect to it. so disabling the service alone doesn't kill it, the socket will just relaunch it. had to stop + disable the socket separately:

```bash
sudo systemctl stop multipathd.socket
sudo systemctl disable multipathd.socket
```

verified both service and socket lists were clean after.

## day 1 done ✅
- fixed `/run/sshd` error
- hardened ssh (root login off, maxauthtries 3)
- trimmed unnecessary services + learned about socket activation the hard way

next: ssh key-based auth (day 2)


### next up

- SSH key-based auth setup
- keep going on the analyst + cloud security track
