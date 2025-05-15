# Systemd Hands-on Lab

This lab shows how to create and manage a `timestamp.service` custom service. It will include practical tasks like creating a script, writing unit files, enabling services, and checking logs.

## Create a Script

1. Create a script `timestamp.sh` under the directory `/usr/local/bin`:

```
sudo mkdir -p /usr/local/bin;
sudo vi /usr/local/bin/timestamp.sh;
```

2. Add the following content to the script `timestamp.sh` that logs the current timestamp to a file every 10 seconds:

```
#!/usr/bin/env bash
while true; do
    echo "Timestamp: $(date)" >> /var/log/timestamp.log
    sleep 10
done
```

3. Make the script executable:

```
sudo chmod +x /usr/local/bin/timestamp.sh
```

4. Test the script manually by running `sudo /usr/local/bin/timestamp.sh` and pressing `Ctrl+C` after a few seconds

5. Check the log:

```
cat /var/log/timestamp.log
```

This script runs indefinitely, writing timestamps to a log file.

## Create Systemd Unit File

1. Create a unit file :

```
sudo vi /etc/systemd/system/timestamp.service
```

A **unit** is just a file that tells `systemd` how to start, stop, or manage something — like a service, mount point, or device.

2. Add the following content to the unit file, save and exit it:

```
[Unit]
# This section describes the service and its dependencies

Description=Timestamp Logging Service
After=network.target  #Start this service only after the network is up and it controls order, not guarantee that network is fully ready

[Service]
#This section defines how to run the script (ExecStart), restart behavior, and user permissions.

ExecStart=/bin/bash /usr/local/bin/timestamp.sh
Restart=always   #auto-restarted on failure

[Install]
# This section specifies when the service should start (e.g., during boot)

WantedBy=multi-user.target #When the system reaches multi-user mode, also start this service.
```

The lab places `timestamp.sh` in `/usr/local/bin/` because it’s a standard directory for locally installed executables, accessible system-wide, and typically in the `PATH`. This ensures the script is available to systemd and other users without cluttering `/bin` or `/usr/bin`.

If your script cannot be executed properly by systemd directly, there might be an issue with systemd execution environment. Systemd runs services in a minimal environment with a limited `PATH` and no shell context. When `ExecStart=/usr/local/bin/timestamp.sh` is used, systemd relies on the script’s shebang (`#!/usr/bin/env bash`) to find the interpreter. However, systemd’s restricted environment might not have access to `/usr/bin/env` or the correct `PATH` to resolve bash. Therefore, specifying `ExecStart=/bin/bash  /usr/local/bin/timestamp.sh` explicitly invokes the Bash interpreter, bypassing the issue.

2. Reload systemd to recognize the new unit and start it:

```
sudo systemctl daemon-reload
sudo systemctl start timestamp.service
systemctl status timestamp.service
```

If it’s running you will see `Active: active (running)` status.

## Check Systemd Logs

1. Systemd logs can reveal if the service is running or failing. Run the command to view the logs:

```
journalctl -u timestamp.service -b
```

The `-b` flag limits logs to the current boot.  
If your user (vagrant in this case) lacks permissions to view systemd logs it is because not being in the `adm` or `systemd-journal` groups. This is standard on Ubuntu for non-privileged users.

2. Add vagrant to the `systemd-journal` Group

```
sudo usermod -aG systemd-journal vagrant
```

3. Log out and back in:

```
exit
vagrant ssh
```

Group changes require a new session. Exit your Vagrant SSH session and reconnect:
