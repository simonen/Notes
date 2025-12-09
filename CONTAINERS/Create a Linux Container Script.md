
Claude 4.5

```bash
#!/bin/bash

# Container configuration
CONTAINER_ROOT="/tmp/my_container"
CONTAINER_NAME="basic-container"

# Create container root filesystem
create_rootfs() {
    echo "[*] Creating container root filesystem..."
    mkdir -p "$CONTAINER_ROOT"/{bin,lib,lib64,usr/bin,proc,sys,dev,etc,root,tmp}
    
    # Copy essential binaries
    BINARIES="/bin/bash /bin/ls /bin/ps /bin/cat /bin/echo /bin/mount /bin/umount"
    for binary in $BINARIES; do
        if [ -f "$binary" ]; then
            cp "$binary" "$CONTAINER_ROOT/bin/"
            
            # Copy dependencies
            for lib in $(ldd "$binary" | grep -o '/lib[^ ]*'); do
                mkdir -p "$CONTAINER_ROOT/$(dirname $lib)"
                cp "$lib" "$CONTAINER_ROOT/$lib" 2>/dev/null
            done
        fi
    done
    
    echo "[*] Root filesystem created at $CONTAINER_ROOT"
}

# Start container
start_container() {
    echo "[*] Starting container: $CONTAINER_NAME"
    
    sudo unshare --fork --pid --mount --uts --ipc --net \
        bash -c "
            hostname $CONTAINER_NAME
            chroot $CONTAINER_ROOT /bin/bash -c '
                mount -t proc proc /proc
                mount -t sysfs sys /sys
                echo \"Container $CONTAINER_NAME started!\"
                echo \"Hostname: \$(hostname)\"
                echo \"Processes:\"
                ps aux
                exec /bin/bash
            '
        "
}

# Cleanup
cleanup() {
    echo "[*] Cleaning up..."
    sudo umount "$CONTAINER_ROOT/proc" 2>/dev/null
    sudo umount "$CONTAINER_ROOT/sys" 2>/dev/null
    sudo rm -rf "$CONTAINER_ROOT"
}

# Main
case "$1" in
    create)
        create_rootfs
        ;;
    start)
        start_container
        ;;
    cleanup)
        cleanup
        ;;
    *)
        echo "Usage: $0 {create|start|cleanup}"
        exit 1
        ;;
esac
```

v2 Claude 4.5

```bash
#!/bin/bash
#
# Basic Container Environment using Linux Namespaces and Alpine Minirootfs
# This script demonstrates how containers work under the hood
#
# Requirements: Linux with namespace support, root privileges
#

set -e

# Configuration
CONTAINER_ROOT="/tmp/container-demo"
ALPINE_VERSION="3.19"
ALPINE_MINOR="3"
HOSTNAME="my-container"

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Check if running as root
if [ "$EUID" -ne 0 ]; then
    log_error "This script must be run as root"
    exit 1
fi

# Check if unshare is available
if ! command -v unshare &> /dev/null; then
    log_error "unshare command not found. Please install util-linux package."
    exit 1
fi

# Step 1: Setup directory structure
setup_directories() {
    log_info "Setting up directory structure at $CONTAINER_ROOT"
    
    # Clean up if exists
    if [ -d "$CONTAINER_ROOT" ]; then
        log_warn "Directory exists, cleaning up..."
        umount -l "$CONTAINER_ROOT/proc" 2>/dev/null || true
        umount -l "$CONTAINER_ROOT/sys" 2>/dev/null || true
        umount -l "$CONTAINER_ROOT/dev" 2>/dev/null || true
        rm -rf "$CONTAINER_ROOT"
    fi
    
    mkdir -p "$CONTAINER_ROOT"
    cd "$CONTAINER_ROOT"
}

# Step 2: Download Alpine minirootfs
download_alpine() {
    log_info "Downloading Alpine Linux minirootfs v${ALPINE_VERSION}.${ALPINE_MINOR}"
    
    local arch=$(uname -m)
    local url="https://dl-cdn.alpinelinux.org/alpine/v${ALPINE_VERSION}/releases/${arch}/alpine-minirootfs-${ALPINE_VERSION}.${ALPINE_MINOR}-${arch}.tar.gz"
    
    if [ -f "alpine-minirootfs.tar.gz" ]; then
        log_info "Alpine minirootfs already downloaded, skipping..."
    else
        wget -q --show-progress "$url" -O alpine-minirootfs.tar.gz
        log_info "Download complete"
    fi
}

# Step 3: Extract rootfs
extract_rootfs() {
    log_info "Extracting rootfs..."
    tar -xzf alpine-minirootfs.tar.gz -C "$CONTAINER_ROOT"
    rm alpine-minirootfs.tar.gz
    
    log_info "Rootfs extracted. Contents:"
    ls -la "$CONTAINER_ROOT" | head -n 10
}

# Step 4: Setup minimal required directories
setup_container_fs() {
    log_info "Setting up container filesystem..."
    
    # Create essential directories if they don't exist
    mkdir -p "$CONTAINER_ROOT"/{proc,sys,dev,tmp,old_root}
    
    # Setup basic DNS resolution
    echo "nameserver 8.8.8.8" > "$CONTAINER_ROOT/etc/resolv.conf"
    echo "nameserver 8.8.4.4" >> "$CONTAINER_ROOT/etc/resolv.conf"
    
    log_info "Filesystem setup complete"
}

# Step 5: Create container startup script
create_startup_script() {
    log_info "Creating container startup script..."
    
    cat > "$CONTAINER_ROOT/container-init.sh" << 'EOF'
#!/bin/sh
# Container initialization script

# Set proper PATH for Alpine
export PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

# Mount proc filesystem
mount -t proc proc /proc 2>/dev/null || true

# Mount sys filesystem
mount -t sysfs sysfs /sys 2>/dev/null || true

# Create basic device nodes if missing (requires mknod which Alpine has)
if command -v mknod >/dev/null 2>&1; then
    [ ! -e /dev/null ] && mknod -m 666 /dev/null c 1 3 2>/dev/null || true
    [ ! -e /dev/zero ] && mknod -m 666 /dev/zero c 1 5 2>/dev/null || true
    [ ! -e /dev/random ] && mknod -m 444 /dev/random c 1 8 2>/dev/null || true
    [ ! -e /dev/urandom ] && mknod -m 444 /dev/urandom c 1 9 2>/dev/null || true
fi

# Display welcome message
/bin/cat << 'WELCOME'
===============================================
  Welcome to Basic Container Environment
===============================================
You are now inside an isolated container using:
  - PID namespace (isolated processes)
  - Mount namespace (isolated filesystem)
  - UTS namespace (isolated hostname)
  - IPC namespace (isolated IPC)
  - Network namespace (isolated network)

Try these commands:
  ps aux          # See isolated processes
  hostname        # Check container hostname
  ip addr         # Check network isolation
  cat /etc/os-release  # See Alpine Linux info
  exit            # Leave container
===============================================
WELCOME

# Start shell
exec /bin/sh
EOF
    
    chmod +x "$CONTAINER_ROOT/container-init.sh"
}

# Step 6: Run container with namespaces
run_container() {
    log_info "Starting container with isolated namespaces..."
    log_warn "You are about to enter the container environment"
    sleep 2
    
    # Using unshare to create namespaces:
    # --pid: PID namespace (process isolation)
    # --mount: Mount namespace (filesystem isolation)
    # --uts: UTS namespace (hostname isolation)
    # --ipc: IPC namespace (inter-process communication isolation)
    # --net: Network namespace (network isolation)
    # --fork: Fork before executing command
    # --mount-proc: Mount /proc inside container
    
    unshare --pid --mount --uts --ipc --net --fork \
        bash -c "
            # Set hostname in new UTS namespace
            hostname $HOSTNAME
            
            # Mount proc filesystem
            mount -t proc proc $CONTAINER_ROOT/proc
            
            # Change root to container filesystem
            cd $CONTAINER_ROOT
            
            # Execute init script
            chroot . /container-init.sh
            
            # Cleanup on exit
            umount -l /proc 2>/dev/null || true
        "
}

# Alternative: Run container with pivot_root (more like real containers)
run_container_pivot() {
    log_info "Starting container with pivot_root (advanced method)..."
    log_warn "You are about to enter the container environment"
    sleep 2
    
    unshare --pid --mount --uts --ipc --net --fork \
        bash -c "
            # Set hostname
            hostname $HOSTNAME
            
            # Make container root a mount point
            mount --bind $CONTAINER_ROOT $CONTAINER_ROOT
            
            # Change to container root
            cd $CONTAINER_ROOT
            
            # Create old_root directory if not exists
            mkdir -p old_root
            
            # Pivot root
            pivot_root . old_root
            
            # Update PATH for Alpine
            export PATH=/bin:/sbin:/usr/bin:/usr/sbin
            
            # Mount proc, sys
            mount -t proc proc /proc
            mount -t sysfs sysfs /sys
            
            # Unmount old root
            umount -l /old_root
            rmdir /old_root
            
            # Show welcome message
            cat << 'WELCOME'
===============================================
  Container Started with pivot_root
===============================================
This is a more Docker-like container setup.
The old filesystem has been completely hidden.

Try: ps aux, ip addr, ls /, hostname
===============================================
WELCOME
            
            # Start shell
            exec /bin/sh
        "
}

# Main execution
main() {
    log_info "=== Building Container Environment ==="
    echo
    
    setup_directories
    download_alpine
    extract_rootfs
    setup_container_fs
    create_startup_script
    
    echo
    log_info "=== Setup Complete ==="
    echo
    log_info "Choose container launch method:"
    echo "  1) Simple chroot with namespaces (easier)"
    echo "  2) pivot_root method (more like Docker)"
    read -p "Enter choice [1-2]: " choice
    
    case $choice in
        1)
            run_container
            ;;
        2)
            run_container_pivot
            ;;
        *)
            log_error "Invalid choice"
            exit 1
            ;;
    esac
}

# Run main function
main
```

Deepseek

```bash
#!/bin/bash
# manual-container.sh

CONTAINER_ROOT="./container-root"

# Create namespaces manually
sudo unshare --mount --uts --ipc --net --pid --fork --user --map-root-user --mount-proc=$PWD/container-root/proc \
    chroot $CONTAINER_ROOT /bin/sh -c "
    mount -t proc proc /proc
    mount -t sysfs sysfs /sys
    mount -t tmpfs tmpfs /tmp
    hostname my-container
    export PS1='[container] \\w \\$ '
    exec /bin/sh
"
```