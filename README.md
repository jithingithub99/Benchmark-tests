# Benchmark-tests
This is the git for benchmark tests

#### Prerequisites for benchmark tests (CPU,MEMORY,IOPS,IPERF) ####

1) Connect vpn for Purestorage lab environment

2) Get kubeconfig for pure storage lab environment

3) Create it to local system under directory test & then export

`cd /test`

`nano purestorage-kubeconfig`

`export KUBECONFIG=purestorage-kubeconfig`

Create 3 pods

1 pod for performing FIO test -we need to have 300gb capacity PVC and get it mounted (/abc)inside pod (eg:fiopod) 2 pods for iperf tests (eg:iperfclientpod,iperfserverod)

We can choose any of these 3 pods for performing benchmark test for CPU & Memory

#### Run CPU benchmark test using sysbench ####

1) Login to pod

`kex sh`

Select pod name (for eg:fiopod)

2) Install sysbench tool

`apt install sysbench`

3) Run below command to get CPU becnhmark

`sysbench cpu --cpu-max-prime=20000 --threads=2 --time=60 run`

#### Run Memory benchmark test using sysbench ####

1) Login to pod

`kex sh`

Select pod name (for eg:fiopod)

2) Install sysbench tool (ignore this step if we have already installed sysbench)

`apt install sysbench`

`sysbench memory --memory-oper=write --memory-block-size=1K --memory-scope=global --memory-total-size=10G --threads=2 --time=30 run`

#### RUN iperf test between 2 pods ####

1) Find out 2 running pods & get the ip address

`kubectl get pods -o wide`

2) Create a namespace called iperf

`kubectl create ns iperf`

3) Select one pod as server and then other one as client & run below commands

4) Login to iperfserverpod and run below command to listen to iperf client requests

`iperf3 --server --version4 --interval 30`

5) Login to client pod (iperclientpod) and run below tcp & UDP benchmark commands

#### TCP test ####

`iperf3 --client --interval 30 --parallel <NUM_THREADS> --time 30 --format M –json`
#### UDP Test ####

`iperf3 --client -u --interval 30 --parallel <NUM_THREADS> --time 30 --format M –json`

where is IP address of iperfserverpod <NUM_THREADS> is no of CPU threads in the system

#### Run FIO tests ####
1) Login to pod where fiotests need to be performed

`kex sh`

(select fiopod)`

2) Check pvc volume(with size 300gb) is mounted (/abc) using df -h

3) Install fio utility

`apt install fio`

3) Run below fio commands to get the random Read/Write IOPS,Random Read/Write bandwidth,random read/Write latency,fdatasync (99th percentile value)

### Read IOPS ###
`fio --randrepeat=0 --verify=0 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=read_iops --directory=/abc --bs=4K --iodepth=16 --fdatasync=0 --size=50G --readwrite=randread --time_based --ramp_time=10s --runtime=30s`

### Write IOPS ###
`fio --randrepeat=0 --verify=0 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=write_iops --directory=/abc --bs=4K --iodepth=16 --fdatasync=0 --size=50G --readwrite=randwrite --time_based --ramp_time=10s --runtime=30s`

### Read Bandwidth ###
`fio --randrepeat=0 --verify=0 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=read_bw --directory=/abc --bs=128K --iodepth=16 --fdatasync=0 --size=50G --readwrite=randread --time_based --ramp_time=10s --runtime=30s`

### Write bandwidth ###

`fio --randrepeat=0 --verify=0 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=write_bw --directory=/abc --bs=128K --iodepth=16 --fdatasync=0 --size=50G --readwrite=randwrite --time_based --ramp_time=10s --runtime=30s`

### Read latency ###
`fio --randrepeat=0 --verify=0 --ioengine=libaio --direct=1 --name=read_latency --directory=/abc --bs=4K --iodepth=4 --size=50G --readwrite=randread --time_based --ramp_time=10s --runtime=30s`

### write latency ###
`fio --randrepeat=0 --verify=0 --ioengine=libaio --direct=1 --name=write_latency --directory=/abc --bs=4K --iodepth=4 --size=50G --readwrite=randwrite --time_based --ramp_time=10s --runtime=30s`


