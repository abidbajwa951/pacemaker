# Cluster Troubleshooting & Understanding Commands

## Checking Cluster Status
```sh
crm status       # Show overall cluster status
crm_mon -1       # Show a one-time cluster status output
crm_simulate -Ls # Simulate and display the current cluster state
```

## Checking Node & Resource Status
```sh
crm node status        # Show status of cluster nodes
crm resource status    # Show status of all cluster resources
crm resource list      # List all configured cluster resources
crm node utilization   # Display node utilization statistics
```

## Viewing Logs & Debugging
```sh
journalctl -u pacemaker -f  # View real-time logs for Pacemaker
journalctl -u corosync -f   # View real-time logs for Corosync
crm history                 # Show historical cluster events
crm report --from=today     # Generate a report of cluster events from today
```

## Checking Corosync Communication
```sh
corosync-cfgtool -s  # Show current ring status
corosync-quorumtool  # Display quorum status
crm corosync status  # Verify Corosync communication
```

## Checking Fencing & Failover Mechanisms
```sh
crm stonith history     # Show STONITH (fencing) history
crm node fence <node>   # Manually fence a node
crm_failcount -G -r <resource> -n <node>  # Check fail count for a resource
crm resource cleanup <resource>  # Clear resource failure history
```

## Simulating & Testing Cluster Configuration
```sh
crm_simulate -Ls        # Simulate cluster state
crm_verify -L           # Validate the cluster configuration
pcs cluster simulate    # Simulate cluster behavior (if using pcs)
```

## Managing Cluster Resources
```sh
crm resource stop <resource>    # Stop a resource
crm resource start <resource>   # Start a resource
crm resource restart <resource> # Restart a resource
crm resource migrate <resource> <node> # Move a resource to another node
crm resource unmigrate <resource> # Allow resource to move freely
```

## Managing Cluster Nodes
```sh
crm node standby <node>  # Put a node into standby mode
crm node online <node>   # Bring a node back online
crm node remove <node>   # Remove a node from the cluster
```

## Checking Cluster Configuration
```sh
crm configure show      # Display full cluster configuration
crm configure edit      # Edit cluster configuration interactively
crm cibdump            # Dump the Cluster Information Base (CIB) as XML
```

## Performance & Network Analysis
```sh
crm_node -i       # Show local node ID
crm_node -l       # List all cluster nodes with IDs
crm_whoami        # Show current node name
```

---
