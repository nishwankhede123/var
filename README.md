#!/bin/bash

# Directories to clean
directories=("/var/log" "/var/tmp" "/var/cache/yum/")

# Number of days to keep files
days=30

# File containing server addresses
servers_file="/nishant/servers.text"

# Log file
log_file="/nishant/cleanup_$(date +%F).log"

# Function to clean directories on a remote server
clean_remote() {
  local server="$1"
  echo "Starting cleanup on $server..." | tee -a "$log_file"

  for dir in "${directories[@]}"; do
    echo "Cleaning $dir on $server..." | tee -a "$log_file"
    ssh -o BatchMode=yes "$server" "find $dir -type f -mtime +$days -exec rm -f {} \;" \
      && echo "Cleaned $dir on $server" | tee -a "$log_file" \
      || echo "Failed to clean $dir on $server" | tee -a "$log_file"
  done
}

# Read server addresses and clean each
while IFS= read -r server; do
  clean_remote "$server" &
done < "$servers_file"

wait
echo "Cleanup complete for all servers!" | tee -a "$log_file"
