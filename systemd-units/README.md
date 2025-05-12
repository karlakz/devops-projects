# Systemd Hands-on Lab

This lab shows how to create and manage a custom service

## Create a script and make it executable

1. Create a script `timestamp.sh` that logs the current timestamp to a file every 10 seconds under the directory `/usr/local/bin`:

```
sudo mkdir -p /usr/local/bin;
sudo vi /usr/local/bin/timestamp.sh;
```

2. Add the following content to the file:

```
#!/bin/bash
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
