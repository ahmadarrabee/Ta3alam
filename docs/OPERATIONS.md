# Operations Manual

Owner: Ahmad Arrabee with authorized local facilitators. Applies to the Ubuntu/systemd installation described in [deployment](DEPLOYMENT.md).

## Open a session

1. A facilitator must be present before learners enter and throughout the session.
2. Power on the router with its WAN connection disconnected, then start the laptop server on mains power. Connect the server by Ethernet; confirm its reserved/static address is `192.168.1.100` and client isolation is disabled.
3. Start Kolibri on port 8080 and confirm its service state:

   ```bash
   sudo systemctl start kolibri
   systemctl is-active kolibri
   ```

4. Start the port 80 portal from the repository root, substituting the actual checkout path:

   ```bash
   cd /opt/offline-learning-hub
   sudo python3 -m http.server 80 --bind 192.168.1.100 --directory "$PWD"
   ```

   Keep this terminal open for a supervised session. This is the README's trusted-LAN demo server. An unattended deployment needs a dedicated static server with a restricted document root. Never place learner data or backups in the checkout.
5. Connect a test client to the local Wi-Fi. Open `http://192.168.1.100/custom_interface/index.html` and `http://192.168.1.100:8080`. Test a locally imported lesson and the coding lab. The portal heartbeat checks port 80 only; it does not prove Kolibri is healthy.

## Learner and staff accounts

In Kolibri's facility administration, create a learner account with a unique non-identifying handle and an individual password. Explain the [Arabic parent notice](PARENT_NOTICE_AR.md), avoid unnecessary profile fields, and assign the learner to the appropriate class. Test sign-in without granting staff rights. Keep any essential family lookup separate from learner records. Give facilitators individual coach/staff accounts; Ahmad approves administrator access. Never share administrator credentials with learners. Use the installed version's account recovery controls privately; do not announce passwords.

## Shared devices and closing

Sign each learner out of Kolibri before the next user. Clear the coding editor/output, downloaded work, saved credentials, browser history, and session/site data as appropriate; first preserve work the learner has asked to keep in authorized local storage. Disable browser password saving and cloud sync. Do not delete other learners' server records during device cleanup. Check the next learner sees a signed-out screen. Wipe physical surfaces using device-safe materials.

At closing, record attendance locally, sign out all clients, lock devices, and check free SSD/USB capacity. If the 02:00 backup is scheduled, leave the secured server powered on and the backup USB mounted. If shutting down, complete a manual consistent backup first, safely unmount the USB, then shut down Ubuntu and the router.

## Software and content updates

Ahmad or authorized technical support schedules updates outside sessions. Record current versions and take a verified stopped-service backup first. Prepare trusted Ubuntu/Kolibri packages and licensed Arabic content on a separate connected device and transfer them by USB; never move learner records to that device or enable cloud synchronization. Follow the installed Kolibri release's official upgrade/migration instructions and test compatibility before updating production. Record new versions, restart services, and test sign-in, lessons, progress, and backups. If rollback is necessary, use the matching pre-update software and snapshot; do not open an upgraded database with an older version.

## Troubleshooting

| Symptom | Checks and action |
| --- | --- |
| Wi-Fi connects but reports no internet | Expected offline. Keep the Wi-Fi connection selected; disable mobile-data switching if needed. |
| Cannot join Wi-Fi or reach server | Check router power, SSID/password, Ethernet, DHCP address on the same subnet, duplicate IPs, and disabled client isolation. Check server address with `ip -4 addr`. |
| Portal will not load | Use the full HTTP URL and correct path. Check the serving terminal and `sudo ss -ltnp` for port 80. A bind error may mean the address is absent, permissions are insufficient, or another process owns the port; identify it before stopping anything. |
| Portal works but Kolibri fails | Check `systemctl status kolibri` and `sudo journalctl -u kolibri -n 100 --no-pager`; confirm port 8080 is listening and the backup is not currently running. After resolving the cause, run `sudo systemctl restart kolibri`. |
| Local firewall blocks clients | Have technical support allow TCP 80 and 8080 from the learning LAN only. Do not open router WAN ports. |
| Lessons absent or progress cannot save | Confirm content is imported, inspect disk space with `df -h`, and check the actual data directory and service-user permissions. Do not delete or reset the database as a first response. |

Run `python3 maintenance/system_diagnostics.py` from the checkout for disk/service checks. Keep diagnostic logs local and restrict any learner information. Escalate unresolved issues to Ahmad Arrabee or local technical support.
