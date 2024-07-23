# Use debian with i386 architecture (32-bit) as the base image
FROM --platform=i386 i386/debian:buster

# Set environment variables to avoid interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Update the package list, install required packages for most units
RUN apt-get update && \
    apt-get install -y curl dnsutils openssh-client sudo vim
# Install ssh server to enable ssh into the container
RUN apt-get install -y openssh-server
# Install git
RUN apt-get install -y git
# Clean up
RUN apt-get clean && \
        rm -rf /var/lib/apt-get/lists/*

# Create a new user named 'codepath' and set up sudo access
RUN useradd -m -s /bin/bash codepath && \
    echo 'codepath ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Set up SSH server
RUN mkdir /var/run/sshd

# Switch to the 'codepath' user
USER codepath
WORKDIR /home/codepath

# Create the .ssh directory and set the appropriate permissions
RUN mkdir -p /home/codepath/.ssh && \
    chmod 700 /home/codepath/.ssh && \
    chown -R codepath:codepath /home/codepath/.ssh

# Generate an SSH key pair named 'ubuntu_key'
RUN ssh-keygen -t rsa -b 4096 -f /home/codepath/.ssh/ubuntu_key -N ""

# add authorized_keys
RUN touch /home/codepath/.ssh/authorized_keys

# Copy the public key to the authorized_keys file
RUN cat /home/codepath/.ssh/ubuntu_key.pub >> /home/codepath/.ssh/authorized_keys

# Set permissions for the SSH directory and key files
RUN chmod 700 /home/codepath/.ssh && \
    chmod 600 /home/codepath/.ssh/ubuntu_key && \
    chmod 644 /home/codepath/.ssh/ubuntu_key.pub

# Create directory for unit
RUN mkdir -p /home/codepath/unit2

# Switch to root to make executable
USER root
RUN sudo chmod -R u+x /home/codepath/*

# Switch back to the root user to finalize configurations (optional)
USER root

# Update the SSH configuration to disable password authentication and enable public key authentication
RUN sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/#AuthorizedKeysFile .ssh\/authorized_keys .ssh\/authorized_keys2/AuthorizedKeysFile .ssh\/authorized_keys/' /etc/ssh/sshd_config && \
    echo "PasswordAuthentication no" >> /etc/ssh/sshd_config

# start ssh services
RUN service ssh start

# Expose SSH port
EXPOSE 22

# # Copy the assets into the container
COPY assets /home/codepath/

# Make the startup script executable
RUN chmod +x /home/codepath/startup.sh

# Set the default command to run the startup script
CMD ["/home/codepath/startup.sh"]
