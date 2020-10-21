# NTP-OKD4

```bash
# Check Chrony Sources
oc debug node/worker01.openshift.instructor.io
ssh core@10.5X.5X.11

chrony sources

#Check MachineConfigPools
oc get machineconfigpools
oc get machineconfigs

#Create chrony configuration
mkdir ntp
vim chrony.conf
...
server 0.id.pool.ntp.org
server 1.id.pool.ntp.org
server 2.id.pool.ntp.org
server 3.id.pool.ntp.org
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
keyfile /etc/chrony.keys
leapsectz right/UTC
logdir /var/log/chrony
...

#change chrony config with python
yum -y install python3
cat chrony.conf | python3 -c "import sys, urllib.parse; print(urllib.parse.quote(''.join(sys.stdin.readlines())))"

#create machine config for master and worker

vim chrony-enable-worker.yaml
...
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 50-worker-chrony
spec:
  config:
    ignition:
      version: 2.2.0
    storage:
      files:
      - contents:
          source: data:,${OUTPUT_FROM_PYTHON}
        filesystem: root
        mode: 0644
        path: /etc/chrony.conf
...


vim chrony-enable-master.yaml
...
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: master
  name: 50-master-chrony
spec:
  config:
    ignition:
      version: 2.2.0
    storage:
      files:
      - contents:
          source: data:,${OUTPUT_FROM_PYTHON}
        filesystem: root
        mode: 0644
        path: /etc/chrony.conf
...

#verify new machineconfig
oc get machineconfigs

#wait until all pools UPDATED
oc get machineconfigpools

#change timezone all node
ssh core@10.5X.5X.11 
sudo timedatectl set-timezone Asia/Jakarta

```
