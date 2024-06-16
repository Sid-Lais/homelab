Copy this file `mount-ssd.service` to /etc/systemd/system/


Then Run the following commands -
```zsh
mkdir ~/ssd
sudo systemctl daemon-reload
sudo systemctl enable mount-ssd.service
```


To ensure that your SSD is mounted before Docker services start, you can make the docker.service unit file depend on your mount-sdd.service.

Here's how you can do it:

- Open the Docker service unit file for editing:

```zsh
sudo systemctl edit docker.service
```

- Add a Requires directive to specify the dependency on your mount-ssd.service. You can add it under the [Unit] section:

```ini
[Unit]
Requires=mount-ssd.service

[Service]
ExecStartPre=/bin/sleep 30
```

**Make sure to edit the file above the `Lines below this comment will be discarded` comment in the file**

- Reload and reboot

```zsh
sudo systemctl daemon-reload
sudo reboot now
```
