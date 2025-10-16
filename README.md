# los
This project demonstrates a lightweight approach to intrusion detection and prevention (IDPS) using Linux shell commands and scripting. It focuses on identifying suspicious activities and potentially malicious commands executed on a Linux system and provides a mechanism to alert administrators or automatically block such activities.
#!/bin/bash
LOG_PATTERN="/var/log/auth.log*"
THRESHOLD=3
BLOCK_LOG="./blocked_log.txt"

echo "Using log pattern: $LOG_PATTERN"
echo "Scanning for failed login attempts..."

# Step 1: Find all failed login attempts and count by IP
suspicious=$(grep "Failed password" $LOG_PATTERN | awk '{print $(NF-3)}' | sort | uniq -c | awk -v t=$THRESHOLD '$1>=t')

echo "Suspicious IP addresses (count ip):"
echo "$suspicious"

# Step 2: Block each suspicious IP
if [ ! -z "$suspicious" ]; then
  echo "$suspicious" | while read count ip; do
    echo "Blocking $ip with $count failed attempts..."
    sudo iptables -A INPUT -s $ip -j DROP
    echo "$(date '+%Y-%m-%d %H:%M:%S'): Blocked $ip (failed attempts=$count) from logs matching $LOG_PATTERN" >> $BLOCK_LOG
  done
else
  echo "No suspicious activity detected."
fi

echo "Intrusion detection complete!"
echo "Blocked log: $(realpath $BLOCK_LOG) (if any entries were added)"
