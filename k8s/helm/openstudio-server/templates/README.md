Instructions for launching on a cluster:

    # Create a traffic controller service:
    kubectl create -f nfs-service-service.yaml

    # Find the cluster address for the service:
    kubectl get svc

    # Modify the PV to point to the proper NFS address:
    vi nfs-pv.yaml

    # Launch the NFS server pod:
    kubectl create -f nfs-service-rc.yaml

    # Create a PV that connects to the running server:
    kubectl create -f nfs-pv.yaml

    # Define a claim that workloads can mount within:
    kubectl create -f nfs-pvc.yaml

Configuration spec to mount this NFS server:

    containers:
      - image: ubuntu
        volumeMounts:
          - name: local-volume-name
            mountPath: /mnt/path
    volumes:
      - name: local-volume-name
        persistentVolumeClaim:
          claimName: global-claim-name
