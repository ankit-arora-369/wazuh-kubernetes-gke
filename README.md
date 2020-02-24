# Wazuh Google Kubernetes

[![Documentation](https://img.shields.io/badge/docs-view-green.svg)](https://documentation.wazuh.com)
[![Documentation](https://img.shields.io/badge/web-view-green.svg)](https://wazuh.com)

Deploy a Wazuh cluster with a basic Elastic stack on Google Kubernetes Engine .

## Documentation

The *instructions.md* file describes how to deploy Wazuh on Kubernetes.

## Directory structure

    ├── wazuh-kubernetes
    │ ├── base
    │ │ ├── gcp-pd-storage-class.yaml
    │ │ ├── wazuh-ns.yaml
    │
    │ ├── elastic_stack
    │ │ ├── elasticsearch
    │ │ │ ├── cluster
    │ │ │ │ ├── elasticsearch-api-svc.yaml
    │ │ │ │ ├── elasticsearch-data-sts.yaml
    │ │ │ │ ├── elasticsearch-master-sts.yaml
    │ │ │
    │ │ │ ├── single-node
    │ │ │ │ ├── elasticsearch-api-svc.yaml
    │ │ │ │ ├── elasticsearch-sts.yaml
    │ │ │
    │ │ │ ├── elasticsearch-svc.yaml
    │ │
    │ │ ├── kibana
    │ │ │ ├── kibana-deploy.yaml
    │ │ │ ├── kibana-svc.yaml
    │ │ │ ├── nginx-deploy.yaml
    │ │ │ ├── nginx-svc.yaml
    | | | ├── nginx-ingress.yaml
    | |
    │ │ ├── logstash
    │ │ │ ├── logstash-deploy.yaml
    │ │ │ ├── logstash-svc.yaml
    | |
    │ ├── wazuh_managers
    │ │ ├── wazuh-cluster-svc.yaml
    │ │ ├── wazuh-master-conf.yaml
    │ │ ├── wazuh-master-sts.yaml
    │ │ ├── wazuh-master-svc.yaml
    │ │ ├── wazuh-worker-0-conf.yaml
    │ │ ├── wazuh-worker-0-sts.yaml
    │ │ ├── wazuh-worker-1-conf.yaml
    │ │ ├── wazuh-worker-1-sts.yaml
    │ │ ├── wazuh-workers-svc.yaml
    │ │
    │ ├── README.md
    │ ├── VERSION
    │ ├── LICENSE    
    │ ├── cleanup.md
    │ ├── instructions.md
    │ ├── upgrade.md

## Branches

* `master` branch contains the latest code, be aware of possible bugs on this branch.

## Contribute

If you want to contribute to our project please don't hesitate to send a pull request. You can also join our users [mailing list](https://groups.google.com/d/forum/wazuh), by sending an email to [wazuh+subscribe@googlegroups.com](mailto:wazuh+subscribe@googlegroups.com), to ask questions and participate in discussions.

## Credits and Thank you

Based on the previous work from wazuh-kubernetes [wazuh/wazuh-kubernetes](https://github.com/wazuh/wazuh-kubernetes) (2020/02/09).

## License and copyright

WAZUH
Copyright (C) 2016-2018 Wazuh Inc.  (License GPLv2)


## References

* [Wazuh website](http://wazuh.com)
