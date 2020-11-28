# minecraft

+ https://github.com/itzg/minecraft-server-charts/tree/master/charts/minecraft
+ https://gist.github.com/gabrielsson/2d110bb3f43b46597831f4a0e4065265
+ https://www.jeffgeerling.com/blog/2020/raspberry-pi-cluster-episode-4-minecraft-pi-hole-grafana-and-more


values.yaml
```yaml
imageTag: multiarch

resources:
  requests:
    memory: 4096Mi

livenessProbe: 
  initialDelaySeconds: 120
  periodSeconds: 60

readinessProbe:
  initialDelaySeconds: 120
  periodSeconds: 60

minecraftServer:
  # This must be overridden, since we can't accept this for the user.
  eula: "TRUE"

  # One of: LATEST, SNAPSHOT, or a specific version (ie: "1.7.9").
  version: "1.15.2"
  # This can be one of "VANILLA", "FORGE", "SPIGOT", "BUKKIT", "PAPER", "FTB", "SPONGEVANILLA"
  type: "SPIGOT"

  # Max view distance (in chunks).
  viewDistance: 7

  # Message of the Day
  motd: "Welcome to Minecraft on Ada Pi!"

  # If you adjust this, you may need to adjust resources.requests above to match.
  # If you are running on raspberry pi 4 you can utilize a lot more
  memory: 2048M

persistence:
  ## minecraft data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  # storageClass: "-"
  dataDir:
    # Set this to false if you don't care to persist state between restarts.
    enabled: true
    Size: 10Gi

```