# a Prometheus-agent

This is a simple configuration I wrote for using **Prometheus** as an **agent**, in my specific case to read data from the **vscode-exporter** extension, and write to a remote **Prometheus** endpoint, used instead as a collector, aggregator of data.  

Maybe you can also find this example useful if you're a newby with prometheus (like me) and need to start using it somehow, even in a different use case.  

## Setup

Running:
1.  copy the `.env.example` to a `.env`, changing needed variables;
2.  (optional) you can edit also some 
3.  run `prometheus.sh`, from this folder.  

Notes:
*   You should start this container at each startup, but the already specified docker property `restart: unless-stopped` should run it automatically.  
*   I suggest to enable the vscode exporter exension only for the needed project(s) (workspaces).  

## Files

`./${WAL_DATA_DIR}/` : (generated from script) where the agent stores its data. It's **mounted** as a volume  
`Dockerfile` :  
`.env.example` : used to generate `.env`, the env file for the setup script  
`prometheus.sh` : starting script  
`prometheus.yaml.template` : used to generate `prometheus.yaml`, the configuration file **mounted** to the container  
*   generation is made with `envsubst`, using the `.env` 

## Infrastructure (example)

The infrastructure I used for this project is the following:
1.  the `vscode-exporter` extension exposes metrics on the default port `9931`, at subpath `/metrics`
2.  this agent collects data and sends them to a remote endpoint
3.  the remote endpoint, instantiated on any machine, collects such data (so you will see a volume there just like this, local `wal_data`)
4.  these metricsc were used by a `grafana` instance, which is able to show nice charts on the browser.  

You can check the server side configuration at https://github.com/papum20/unibo__projects__stoCAS.  

## Debugging

vscode-exporter: you can check the collected metrics running the `vscode exporter - show metrics` command from vscode, and then also performing a get to `localhost:9931/metrics`  
any prometheus instance: access from the browser, and you will be able to do queries  
*   e.g.: `vscode_seconds_active` will show all collected metrics with such name
*   e.g.: `sum by(project) (increase(vscode_seconds_active{ instance="my-instance", project="my-project",focused="false"}[2595600s]))` : will show the total collected time with the vscode window open, for a specific instance, on a specific project, for the specified amount of time.
    *   `focused=true` : only show time when the vscode window was your focused window
    *   `focused=false` : only show time when vscode window was open in background, but you were focused on another one
    *   `focused` omitted : will count both when is `true` of `false`
